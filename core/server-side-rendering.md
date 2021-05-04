# Server Side Rendering

Some projects require you to render your application on the server. There are different reasons to do this, like search engine optimizations, general optimizations and even browser support. What this means for state management is that you want to expose a version of your state on the server and render the components with that state. But that is not all, you also want to **hydrate** the changed state and pass it to the client with the HTML so that it can **rehydrate** and make sure that when the client renders initially, it renders the same UI.

## Preparing the project

When doing server-side rendering the configuration of your application will be shared by the client and the server. That means you need to structure your app to make that possible. There is really not much you need to do.

{% tabs %}
{% tab title="overmind/index.ts" %}
```typescript
import { IConfig } from 'overmind'
import { state } from './state'

export const config = {
  state
}

declare module 'overmind' {
  interface Config extends IConfig<typeof config> {}
}
```
{% endtab %}

{% tab title="index.ts" %}
```typescript
import { createOvermind } from 'overmind'
import { config } from './overmind'

const overmind = createOvermind(config)
```
{% endtab %}
{% endtabs %}

Here we only export the configuration from the main Overmind file. The instantiation rather happens where we prepare the application on the client side. That means we can now safely import the configuration also on the server.

## Preparing effects

The effects will also be shared with the server. Typically this is not an issue, but you should be careful about creating effects that run logic when they are defined. You might also consider lazy-loading effects so that you avoid loading them on the server at all. You can read more about them in [EFFECTS](running-side-effects.md).

## Rendering on the server

When you render your application on the server you will have to create an instance of Overmind designed for running on the server. On this instance you can change the state and provide it to your components for rendering. When the components have rendered you can **hydrate** the changes and pass them along to the client so that you can **rehydrate**.

{% hint style="info" %}
Overmind does not hydrate the state, but the mutations you performed. That means it minimizes the payload passed over the wire.
{% endhint %}

The following shows a very simple example using an [EXPRESS](https://expressjs.com/) middleware to return a server side rendered version of your app.

{% tabs %}
{% tab title="server/routePosts.ts" %}
```typescript
import { createOvermindSSR } from 'overmind'
import { config } from '../client/overmind'
import db from './db'

export default async (req, res) => {
  const overmind = createOvermindSSR(config)

  overmind.state.currentPage = 'posts'
  overmind.state.posts = await db.getPosts()

  const html = renderToString(
    // Whatever implementation your view layer provides
  )

  res.send(`
<html>
  <body>
    <div id="app">${html}</div>
    <script>
      window.__OVERMIND_MUTATIONS = ${JSON.stringify(overmind.hydrate())}
    </script>
    <script src="/scripts/app.js"></script>
  </body>
</html>
`)
}
```
{% endtab %}
{% endtabs %}

## Rehydrate on the client

On the client you just want to make sure that your Overmind instance rehydrates the mutations performed on the server so that when the client renders, it does so with the same state. The **onInitialize** hook of Overmind is the perfect spot to do this.

{% tabs %}
{% tab title="overmind/onInitialize.ts" %}
```typescript
import { OnInitialize, rehydrate } from 'overmind'

export const onInitialize: OnInitialize = ({ state }) => {
  const mutations = window.__OVERMIND_MUTATIONS

  rehydrate(state, mutations)
}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
If you are using state first routing, make sure you prevent the router from firing off the initial route, as this is not needed.
{% endhint %}

## OnInitialize

The `onInitialized` action does not run on the server. The reason is that it is considered a side effect you might not want to run, so we do not force it. If you do want to run an action as Overmind fires up both on the client and the server you can rather create a custom action for it.

{% tabs %}
{% tab title="overmind/actions.js" %}
```javascript
export const initialize = () => {
  // Whatever...
}
```
{% endtab %}

{% tab title="client/index.js" %}
```typescript
import { createOvermind } from 'overmind'
import { config } from './overmind'

const overmind = createOvermind(config)
overmind.actions.initialize()
```
{% endtab %}

{% tab title="server/index.js" %}
```javascript
import { createOvermindSSR } from 'overmind'
import { config } from '../client/overmind'

export default async (req, res) => {
  const overmind = createOvermindSSR(config)
  await overmind.actions.initialize()

  const html = renderToString(
    // Whatever implementation your view layer provides
  )

  res.send(`
<html>
  <body>
    <div id="app">${html}</div>
    <script>
      window.__OVERMIND_MUTATIONS = ${JSON.stringify(overmind.hydrate())}
    </script>
    <script src="/scripts/app.js"></script>
  </body>
</html>
`)
}
```
{% endtab %}
{% endtabs %}

## Next.js

The idea behind setting up overmind in `next.js` is the same as a standard express server but we have a lot of help from next to get us going.

Let's start by adding a `_app.js` and this is where we will initialize the SSR version of Overmind:

{% tabs %}
{% tab title="src/pages/\_app.js" %}
```javascript
import App from "next/app";
import { createOvermind, createOvermindSSR, rehydrate } from "overmind";
import { Provider } from "overmind-react";
import { config } from "../overmind";

export default class MyApp extends App {
  // CLIENT: On initial route
  // SERVER: On initial route
  constructor(props) {
    super(props);

    const mutations = props.pageProps.mutations || [];

    if (typeof window !== "undefined") {
      // On the client we just instantiate the Overmind instance and run
      // the "changePage" action
      this.overmind = createOvermind(config);
      this.overmind.actions.changePage(mutations);
    } else {
      // On the server we rehydrate the mutations to an SSR instance of Overmind,
      // as we do not want to run any additional logic here
      this.overmind = createOvermindSSR(config);
      rehydrate(this.overmind.state, mutations);
    }
  }
  // CLIENT: After initial route, on page change
  // SERVER: never
  componentDidUpdate() {
    // This runs whenever the client routes to a new page
    this.overmind.actions.changePage(this.props.pageProps.mutations || []);
  }
  render() {
    const { Component, pageProps } = this.props;
    return (
      <Provider value={this.overmind}>
        <Component {...pageProps} />
      </Provider>
    );
  }
}

```
{% endtab %}
{% endtabs %}

And then let's create a standard `Overmind` instance:

```javascript
import { rehydrate } from "overmind";
import { createHook } from "overmind-react";

export const config = {
  state: {},
  actions: {
    changePage({ state }, mutations) {
      rehydrate(state, mutations || []);
    },
  },
};

export const useOvermind = createHook();
```

And you are all set to get going with `overmind` and `next.js`. You can also take a look at [this example in the next.js examples directory](https://github.com/vercel/next.js/tree/canary/examples/with-overmind) if you need some help.

## Gatsby

When it comes to gatsby we need to prepare Overmind for static extraction and the idea is about the same.

We need first to wrap our whole app in the Overmind provider and we can do that in `gatsby-browser.js`:

```javascript
import React from "react"
import { createOvermind } from "overmind"
import { Provider } from "overmind-react"
import { config } from "./src/overmind"

const overmind = createOvermind(config);
  
export const wrapPageElement = ({ element }) => (
  <Provider value={createOvermind(config)}>
    {element}
  </Provider>
)

```

After this is done we can do the same thing for the server render and add that code in the `gatsby-ssr.js` file:

```javascript
import React from "react"
import { Provider } from "overmind-react"
import { createOvermindSSR } from "overmind"
import { ThemeProvider as ChakraProvider } from "@chakra-ui/core"
import { theme } from "@chakra-ui/core"
import { config } from "./src/overmind"

const overmind = createOvermindSSR(config)

export const wrapPageElement = ({ element }) => (
  <Provider value={createOvermind(config)}>
    {element}
  </Provider>
)

```

As you can see the only difference we have here is that we createOvermindSSR in the `gatsby-ssr.js`

