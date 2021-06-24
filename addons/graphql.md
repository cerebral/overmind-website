# GraphQL

Using Graphql with Overmind gives you the following benefits:

* **Query:** The query for data is run with the rest of your application logic, unrelated to mounting components
* **Cache:** You integrate the data from Graphql with your existing state, allowing you to control when new data is needed
* **Optimistic updates:** With the data integrated with your Overmind state you can also optimistically update that state before running a mutation query

## Get up and running

Install the separate package:

```text
npm install overmind-graphql
```

### Initial state

The Graphql package is an _effect_. Though since we are operating on state, let us prepare some:

{% tabs %}
{% tab title="overmind/index.js" %}
```typescript
import { state } from './state'

export const config = {
  state
}
```
{% endtab %}

{% tab title="overmind/state.js" %}
```typescript
export const state = {
  posts: []
}
```
{% endtab %}
{% endtabs %}

### The effect

Now let us introduce the effect:

{% tabs %}
{% tab title="overmind/index.js" %}
```typescript
import { state } from './state'
import { onInitialize } from './onInitialize'
import { gql } from './effects/gql'

export const config = {
  onInitialize,
  state,
  effects: {
    gql
  }
}
```
{% endtab %}

{% tab title="overmind/onInitialize.js" %}
```javascript
export const onInitialize = ({ effects }) => {
  effects.gql.initialize({
    // query and mutation options
    endpoint: 'http://some-endpoint.dev',
  }, {
    // subscription options
    endpoint: 'ws://some-endpoint.dev',  
  })
}
```
{% endtab %}

{% tab title="overmind/effects/gql/index.js" %}
```javascript
import { graphql } from 'overmind-graphql'
import * as queries from './queries'
import * as mutations from './mutations'
import * as subscriptions from './subscriptions'

export const gql = graphql({
  queries,
  mutations,
  subscriptions
})
```
{% endtab %}

{% tab title="overmind/effects/gql/queries.js" %}
```typescript
import { gql } from 'overmind-graphql'

export const posts = gql`
  query Posts {
    posts {
      id
      title
    }
  }
`;
```
{% endtab %}

{% tab title="overmind/effects/gql/mutations.js" %}
```typescript
import { gql } from 'overmind-graphql'

export const createPost = gql`
  mutation CreatePost($title: String!) {
    createPost(title: $title) {
      id
    }
  }
`
```
{% endtab %}

{% tab title="overmind/effects/gql/subscriptions.js" %}
```javascript
import { gql } from 'overmind-graphql'

export const onPostAdded = gql`
  subscription PostAdded() {
    postAdded() {
      id
      title
    }
  }
`
```
{% endtab %}
{% endtabs %}

You define **queries,** **mutations** and **subscriptions** with the effect. That means you can have multiple effects holding different queries and even endpoints. The endpoints are defined when you initialize the effect. This allows you to dynamically create the endpoints based on state, and also pass state related to requests to the endpoints. The queries, mutations and subscriptions are converted into Overmind effects that you can call from your actions.

## Query

To call a query you will typically use an action. Let us create an action that uses our **posts** query.

{% tabs %}
{% tab title="overmind/actions.js" %}
```typescript
export const getPosts = async ({ state, effects }) => {
  const { posts } = await effects.gql.queries.posts()

  state.posts = posts
}
```
{% endtab %}
{% endtabs %}

## Mutate

Mutation queries are basically the same as normal queries. You would typically also call these from an action.

{% tabs %}
{% tab title="overmind/actions.js" %}
```typescript
export const getPosts = async ({ state, effects }) => {
  const { posts } = await effects.gql.queries.posts()

  state.posts = posts
}

export const addPost = async ({ effects }, title) => {
  await effects.gql.mutations.createPost({ title })
}
```
{% endtab %}
{% endtabs %}

## Subscription

Subscriptions are also available via actions. You typically give them an action which triggers whenever the subscription triggers.

{% tabs %}
{% tab title="overmind/actions.js" %}
```typescript
export const getPosts = async ({ state, effects, actions }) => {
  const { posts } = await effects.gql.queries.posts()

  state.posts = posts

  effects.gql.subscriptions.onPostAdded(actions.onPostAdded)
}

export const addPost = async ({ effects }, title) => {
  await effects.gql.mutations.createPost({ title })
}

export const onPostAdded = ({ state }, post) => {
  state.posts.push(post)
}
```
{% endtab %}
{% endtabs %}

## Cache

Now that we have the data from our query in the state, we can decide ourselves when we want this data to update. It could be related to moving back to a certain page, maybe you want to update the data in the background or maybe it is enough to just grab it once. You do not really think about it any differently here than with any other data fetching solution.

## Optimistic updates

Again, since our data is just part of our state we are in complete control of optimistically adding new data. Let us create an optimistic post.

{% tabs %}
{% tab title="overmind/actions.js" %}
```typescript
export const getPosts = async ({ state, effects }) => {
  const { posts } = await effects.queries.posts()

  state.posts = posts
}

export const addPost = async ({ state, effects }, title) => {
  const optimisticId = String(Date.now())

  state.posts.push({
    id: optimisticId,
    title
  })

  const { id } = await effects.mutations.createPost({ title })
  const optimisticPost = state.posts.find(post => post.id === optimisticId)

  optimisticPost.id = id
}
```
{% endtab %}
{% endtabs %}

## Options

There are two points of options in the Graphql factory. The **headers** and the **options**.

The headers option is a function which receives the state of the application. That means you can produce request headers dynamically. This can be useful related to authentciation.

{% tabs %}
{% tab title="overmind/onInitialize.js" %}
```typescript
export const onInitialize = ({ state, effects }) => {
  effects.gql.initialize({
    endpoint: 'http://some-endpoint.dev',
    // This runs on every request
    headers: () => ({
      authorization: `Bearer ${state.auth.token}`
    }),
    // The options are the options passed to GRAPHQL-REQUEST
    options: {
      credentials: 'include',
      mode: 'cors',
    },
  }, {
    endpoint: 'ws://some-endpoint.dev',
    // This runs on every connect
    params: () => ({
      token: state.auth.token
    })
  })
}
```
{% endtab %}
{% endtabs %}

## Custom subscription socket

If you want to define your own socket for connecting to subscriptions, a function can be used instead:

{% tabs %}
{% tab title="overmind/onInitialize.js" %}
```javascript
export const onInitialize = ({ effects }) => {
  effects.gql.initialize(
    {
      endpoint: 'http://some-endpoint.dev',
    }, 
    () => new Websocket('ws://some-other-endpoint.dev')
  )
}
```
{% endtab %}
{% endtabs %}

## Disposing subscriptions

You can dispose any subscriptions in any action. There are two ways to dispose:

{% tabs %}
{% tab title="overmind/actions.js" %}
```typescript
export const disposeSubscriptions = async ({ state, effects }) => {
  // Disposes all subscriptions on "onPostAdded"
  effects.gql.subscriptions.onPostAdded.dispose()
  // If the subscription takes a payload, you can dispose specific
  // subscriptions
  effects.gql.subscriptions.onPostChange.disposeWhere(
    data => data.id === state.currentPostId
  )
}
```
{% endtab %}
{% endtabs %}

## Typescript

There is only a single type exposed by the library, **Query**. It is used for queries, mutations and subscriptions.

{% tabs %}
{% tab title="overmind/queries.ts" %}
```typescript
import { Query, gql } from 'overmind-graphql'
// You will understand this very soon
import { Posts } from './graphql-types'

export const posts: Query<Posts> = gql`
  query Posts {
    posts {
      id
      title
    }
  }
`;
```
{% endtab %}
{% endtabs %}

The first **Query** argument is the result of the query. There is also a second query argument which is the payload to the query, as seen here.

{% tabs %}
{% tab title="overmind/mutations.ts" %}
```typescript
import { Query, gql } from 'overmind-graphql'
// You will understand this very soon
import { CreatePost, CreatePostVariables } from './graphql-types'

export const createPost: Query<CreatePost, CreatePostVariables> = gql`
  mutation CreatePost($title: String!) {
    createPost(title: $title) {
      id
    }
  }
`
```
{% endtab %}
{% endtabs %}

### Generate typings

It is possible to generate all the typings for the queries and mutations. This is done by using the [APOLLO](https://www.apollographql.com/) project CLI. Install it with:

```text
npm install apollo --save-dev
```

Now you can create a script in your **package.json** file that looks something like:

```typescript
{
  "scripts": {
    "schema": "apollo schema:download --header='X-Hasura-Admin-Secret: password' --endpoint=http://some-endpoint.dev graphql-schema.json && apollo codegen:generate --localSchemaFile=graphql-schema.json --target=typescript --includes=src/overmind/**/*.ts --tagName=gql --no-addTypename --globalTypesFile=src/overmind/graphql-global-types.ts graphql-types"
  }
}
```

To update your types, simply run:

```text
npm run schema
```

Apollo will look for queries defined with the **gql** template tag and automatically produce the typings. That means whenever you add, remove or update a query in your code you should run this script to update the typings. It also produces what is called **graphql-global-types**. These are types related to fields on your queries, which can be used in your state definition and/or actions.

{% hint style="info" %}
Note that initially you have to define your queries without types and after running the script you can start typing them to get typing in your app and ensure that your app does not break when you change the queries either in the client or on the server
{% endhint %}

## Optimize query

It is possible to transpile the queries from strings into code. This reduces the size of your bundle, though only noticeably if you have a lot of queries. This can be done with the [BABEL-PLUGIN-GRAPHQL-TAG](https://github.com/gajus/babel-plugin-graphql-tag).

