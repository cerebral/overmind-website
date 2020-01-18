# GraphQL

Using Graphql with Overmind gives you the following benefits:

* **Query:** The query for data is run with the rest of your application logic, unrelated to mounting components
* **Cache:** You integrate the data from Graphql with your existing state, allowing you to control when new data is needed
* **Optimistic updates:** With the data integrated with your Overmind state you can also optimistically update that state before running a mutation query

{% hint style="info" %}
The Graphql package does not support **subscriptions** currently
{% endhint %}

## Get up and running

Install the separate package:

```text
npm install overmind-graphql
```

### Initial state

The Graphql package is a _configuration factory_. That means you need some existing configuration before going:

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

### The factory

Now let us introduce the factory:

{% tabs %}
{% tab title="overmind/index.js" %}
```typescript
import { graphql } from 'overmind-graphql'
import * as queries from './queries'
import * as mutations from './mutations'
import { state } from './state'

export const config = graphql({
  state
}, {
  endpoint: 'http://some-endpoint.dev',
  queries,
  mutations
})
```
{% endtab %}

{% tab title="overmind/queries.js" %}
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

{% tab title="overmind/mutations.js" %}
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
{% endtabs %}

You define **queries** and **mutations** as part of the second argument to the factory, with what **endpoint** you want to connect to. These queries and mutations are converted into Overmind effects that you can call from your actions.

## Query

To call a query you will typically use an action. Let us create an action that uses our **posts** query.

{% tabs %}
{% tab title="overmind/index.js" %}
```typescript
import { graphql } from 'overmind-graphql'
import * as actions from './actions'
import * as queries from './queries'
import * as mutations from './mutations'
import { state } from './state'

export const config = graphql({
  state,
  actions
}, {
  endpoint: 'http://some-endpoint.dev',
  queries,
  mutations
})
```
{% endtab %}

{% tab title="overmind/actions.js" %}
```typescript
export const getPosts = async ({ state, effects }) => {
  const { posts } = await effects.queries.posts()

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
  const { posts } = await effects.queries.posts()

  state.posts = posts
}

export const addPost = async ({ effects }, title) => {
  await effects.mutations.createPost({ title })
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
{% tab title="overmind/index.js" %}
```typescript
import { graphql } from 'overmind-graphql'
import * as queries from './queries'
import * as mutations from './mutations'
import { state } from './state'

export const config = graphql({
  state
}, {
  endpoint: 'http://some-endpoint.dev',
  headers: (state) => ({
    authorization: `Bearer ${state.auth.token}`
  }),
  queries,
  mutations
})
```
{% endtab %}
{% endtabs %}

The options are the options passed to [GRAPHQL-REQUEST](https://github.com/prisma-labs/graphql-request).

{% tabs %}
{% tab title="overmind/index.js" %}
```typescript
import { graphql } from 'overmind-graphql'
import * as queries from './queries'
import * as mutations from './mutations'
import { state } from './state'

export const config = graphql({
  state
}, {
  endpoint: 'http://some-endpoint.dev',
  headers: (state) => ({
    authorization: `Bearer ${state.auth.token}`
  }),
  options: {
    credentials: 'include',
    mode: 'cors',
  },
  queries,
  mutations
})
```
{% endtab %}
{% endtabs %}

## Typescript

There is only a single type exposed by the library, **Query**. It is used for both queries and mutations.

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
    "schema": "apollo schema:download --endpoint=http://some-endpoint.dev graphql-schema.json && apollo codegen:generate --localSchemaFile=graphql-schema.json --target=typescript --includes=src/overmind/**/*.ts --tagName=gql --no-addTypename --globalTypesFile=src/overmind/graphql-global-types.ts graphql-types"
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

