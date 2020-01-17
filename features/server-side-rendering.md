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

