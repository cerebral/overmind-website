# Configuration

Overmind is based on a core concept of:

`{ state, actions, effects }`

This data structure is called **the configuration** of your application. If it is a simple application you might have a single configuration, but typically you will create multiple of them and use tools to merge them together into one big configuration. But before we look at the scalability of Overmind, let’s talk about file structure.

## Namespaces

![](../.gitbook/assets/image%20%281%29.png)

As your application grows you start to separate it into different namespaces. A namespace might be closely related to a page in your application, or maybe it is strictly related to managing some piece of data. It does not matter. You define the namespaces of your application and they probably change over time as well. What matters in the context of Overmind though is that each of these namespaces will contain their own state, actions and effects. So imagine a file structure of:

```text
overmind/
  state.ts
  actions.ts
  effects.ts
  index.ts
```

In this structure we are splitting up the different components of the configuration. This is a good first step. The **index** file acts as the file that brings the **state**, **actions** and **effects** together.

But if we want to split up into actual namespaces it would look more like this:

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

In this case each namespace **index** file bring its own state, actions and effects together and the **overmind/index** file is responsible for bringing the whole configuration together.

## The state file

You will typically define your **state** file by exporting a single constant named _state_.

{% tabs %}
{% tab title="overmind/posts/state.js" %}
```typescript
export const state = {
  posts: []
}
```
{% endtab %}

{% tab title="overmind/admin/state.js" %}
```typescript
export const state: State = {
  users: []
}
```
{% endtab %}
{% endtabs %}

## The actions file

The actions are exported individually by giving them a name and a definition.

{% tabs %}
{% tab title="overmind/posts/actions.js" %}
```typescript
export const getPosts = async () => {}

export const addNewPost = async () => {}
```
{% endtab %}

{% tab title="overmind/admin/actions.js" %}
```typescript
export const getUsers = async () => {}

export const changeUserAccess = async () => {}
```
{% endtab %}
{% endtabs %}

## The effects file

The effects are also exported individually where you would typically organize the methods in an object, but this could have been a class instance or just a plain function as well.

{% tabs %}
{% tab title="overmind/posts/effects.js" %}
```typescript
export const postsApi = {
  getPostsFromServer() {}
}
```
{% endtab %}

{% tab title="overmind/admin/effects.js" %}
```typescript
export const adminApi = {
  getUsersFromServer() {}
}
```
{% endtab %}
{% endtabs %}

You might find it more useful to define a single effects file at the root of your application and rather create a file for each effect.

{% tabs %}
{% tab title="overmind/effects/index.js" %}
```typescript
import * as api from './api'

export {
  api
}
```
{% endtab %}

{% tab title="overmind/effects/api.js" %}
```typescript
export const getUser = async () => {}

export const getPosts = async () => {}
```
{% endtab %}
{% endtabs %}

## Bring it together

Now let us export the state, actions and effects for each module and bring it all together into a **namespaced** configuration.

{% tabs %}
{% tab title="overmind/posts/index.js" %}
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

{% tab title="overmind/admin/index.js" %}
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

{% tab title="overmind/index.js" %}
```typescript
import { namespaced } from 'overmind/config'
import * as posts from './posts'
import * as admin from './admin'

export const config = namespaced({
  posts,
  admin
})
```
{% endtab %}
{% endtabs %}

We used the **namespaced** function to put the state, actions and effects from each namespace behind a key. In this case the key is the same as the name of the namespace itself. This is an effective way to split up your app.

You can also combine this with the **merge** tool to have a top level namespace.

{% tabs %}
{% tab title="overmind/index.js" %}
```typescript
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
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Even though you split up into different namespaces each namespace has access to the state, actions and effects of the whole application. This is an important feature of Overmind which allows you to scale up and explore the domains of the application without having to worry about isolation.
{% endhint %}

