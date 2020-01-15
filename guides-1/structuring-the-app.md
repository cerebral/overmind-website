# Structuring the app

You will quickly see the need to give more structure to your application. The base structure of

`{ state, actions, effects }`

does not scale very well.

The abovementioned base structure is called **the configuration** of your application and tools are provided to create more complex configurations. But before we look at those tools, let’s talk about file structure.

## Domains

As your application grows you start to separate it into different domains. A domain might be closely related to a page in your application, or maybe it is strictly related to managing some piece of data. It does not matter. You define the domains of your application and they probably change over time as well. What matters in the context of Overmind though is that each of these domains will contain their own state, actions and effects. So imagine a file structure of:

```text
overmind/
  state.ts
  actions.ts
  effects.ts
  index.ts
```

In this structure we are splitting up the different components of the base structure. This is a good first step. The **index** file acts as the file that brings the state, actions and effects together.

But if we want to split up into actual domains it would look more like this:

```text
overmind/
  posts/
    state.ts
    actions.ts
    effects.ts
    index.ts
  admin/
    state.ts
    actions.ts
    effects.ts
    index.ts
  index.ts
```

In this case each domain **index** file bring its own state, actions and effects together and the **overmind/index** file is responsible for bringing the configuration together.

{% hint style="info" %}
Note that you do not define a **Config**, **Action** etc. type for each domain of your application. Only the main file that instantiates Overmind has this typing. This allows all the domains of the application to know about each other.
{% endhint %}

## The state file

You will typically define your **state** file by exporting a single constant named _state_.

{% tabs %}
{% tab title="overmind/posts/state.ts" %}
```typescript
export type Post = {
  id: number
  title: string
}

export type State = {
  posts: Post[]
}

export const state: State = {
  posts: []
}
```
{% endtab %}

{% tab title="overmind/admin/state.ts" %}
```typescript
export type User = {
  id: number
  name: string
}

export type State = {
  users: User[]
}

export const state: State = {
  users: []
}
```
{% endtab %}
{% endtabs %}

## The actions file

The actions are exported individually by giving them a name and a definition. Actions that are considered private are typically put into their own file named **internalActions**.

{% tabs %}
{% tab title="overmind/posts/internalActions.ts" %}
```typescript
import { Action } from 'overmind'

export const handleError: Action = ({ state }) => {
  // All actions can access all state
  state.posts
  state.admin
}
```
{% endtab %}

{% tab title="overmind/posts/actions.ts" %}
```typescript
import { AsyncAction } from 'overmind'
import * as internalActions from './internalActions'

export const internal = internalActions

export const getPosts: AsyncAction = async () => {}

export const addNewPost: AsyncAction = async () => {}
```
{% endtab %}

{% tab title="overmind/admin/actions.ts" %}
```typescript
import { AsyncAction } from 'overmind'

export const getUsers: AsyncAction = async () => {}

export const changeUserAccess: AsyncAction = async () => {}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Note that there are two action types, **Action** and **AsyncAction**
{% endhint %}

## The effects file

The effects are also exported individually where you would typically organize the methods in an object, but this could have been a class instance or just a plain function as well.

{% tabs %}
{% tab title="overmind/posts/effects.ts" %}
```typescript
export const postsApi = {
  getPostsFromServer(): Promise<Post[]> {}
}
```
{% endtab %}

{% tab title="overmind/admin/effects.ts" %}
```typescript
export const adminApi = {
  getUsersFromServer(): Promise<User[]> {}
}
```
{% endtab %}
{% endtabs %}

## Bring it together

Now let us export the state, actions and effects for each module and bring it all together into a **namespaced** configuration.

{% tabs %}
{% tab title="overmind/posts/index.ts" %}
```typescript
import { state } from './state'
import * as actions from './actions'
import * as effects from './effects'

export {
  state,
  actions,
  effects
}
```
{% endtab %}

{% tab title="overmind/admin/index.ts" %}
```typescript
import { state } from './state'
import * as actions from './actions'
import * as effects from './effects'

export {
  state,
  actions,
  effects
}
```
{% endtab %}

{% tab title="overmind/index.ts" %}
```typescript
import { IConfig } from 'overmind'
import { namespaced } from 'overmind/config'
import * as posts from './posts'
import * as admin from './admin'

export const config = namespaced({
  posts,
  admin
})

declare module 'overmind' {
  interface Config extends IConfig<typeof config> {}
}
```
{% endtab %}
{% endtabs %}

We used the **namespaced** function to put the state, actions and effects from each domain behind a key. In this case the key is the same as the name of the domain itself. This is an effective way to split up your app.

You can also combine this with the **merge** tool to have a top level domain.

{% tabs %}
{% tab title="overmind/index.ts" %}
```typescript
import { IConfig } from 'overmind'
import { merge, namespaced } from 'overmind/config'
import { state } from './state'
import * as posts from './posts'
import * as admin from './admin'

export const config = merge(
  {
    state
  },
  namespaced({
    posts,
    admin
  })
)

declare module 'overmind' {
  interface Config extends IConfig<typeof config> {}
}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Even though you split up into different domains each domain has access to the state of the whole application. This is an important feature of Overmind which allows you to scale up and explore the domains of the application without having to worry about isolation.
{% endhint %}

