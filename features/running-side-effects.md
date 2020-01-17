# Effects

Developing applications is not only about managing state, but also managing side effects. A typical side effect would be an HTTP request or talking to localStorage. In Overmind we just call these **effects**. There are several reasons why you would want to use effects:

1. All the code in your actions will be domain specific, no low level generic APIs
2. Your actions will have less code and you avoid leaking out things like URLs, types etc.
3. You decouple the underlying tool from its usage, meaning that you can replace it at any time without changing your application logic
4. You can more easily expand the functionality of an effect. For example you want to introduce caching or a base URL to an HTTP effect
5. The devtool tracks its execution
6. If you write Overmind tests, you can easily mock them
7. You can lazy-load the effect, reducing the initial payload of the app

## Exposing an existing tool

Let us just expose the [AXIOS](https://github.com/axios/axios) library as an **http** effect.

{% tabs %}
{% tab title="overmind/effects.ts" %}
```typescript
export { default as http } from 'axios'
```
{% endtab %}

{% tab title="overmind/index.ts" %}
```typescript
import { IConfig } from 'overmind'
import { state } from './state'
import * as actions from './actions'
import * as effects from './effects'

export const config = {
  state,
  actions,
  effects
}

// For explicit typing check the Typescript guide
declare module 'overmind' {
  interface Config extends IConfig<typeof config> {}
}
```
{% endtab %}
{% endtabs %}

We are just exporting the existing library from our effects file and including it in the application config. Now Overmind is aware of an **http** effect. It can track it for debugging and all actions and operators will have it injected.

Let us put it to use in an action that grabs the current user of the application.

{% tabs %}
{% tab title="overmind/actions.ts" %}
```typescript
import { AsyncAction } from 'overmind'
import { User } from './state'

export const getCurrentUser: AsyncAction = async ({ effects, state }) => {
  state.currentUser = await effects.http.get<User>('/user')
}
```
{% endtab %}
{% endtabs %}

That was basically it. As you can see we are exposing some low level details like the http method used and the URL. Let us follow the encouraged way of doing things and create our own **api** effect.

## Specific API

It is highly encouraged that you avoid exposing tools with their generic APIs. Rather build your own APIs that are more closely related to the domain of your application. Maybe you have an endpoint for fetching the current user. Create that as an API for your app.

{% tabs %}
{% tab title="overmind/effects.ts" %}
```typescript
import * as axios from 'axios'
import { User } from './state'

export const api = {
  getCurrentUser() {
    return axios.get<User>('/user')
  }
}
```
{% endtab %}
{% endtabs %}

Now you can see how clean your application logic becomes:

{% tabs %}
{% tab title="overmind/actions.ts" %}
```typescript
import { AsyncAction } from 'overmind'

export const loadApp: AsyncAction = async ({ effects, state }) => {
  state.currentUser = await effects.api.getCurrentUser()
}
```
{% endtab %}
{% endtabs %}

## Initializing effects

It can be a good idea to not allow your side effects to initialize when they are defined. This makes sure that they do not leak into tests or server side rendering. For example if you want to use Firebase, instead of initializing the Firebase application immediately we rather do it behind an **initialize** method:

{% tabs %}
{% tab title="overmind/effects.ts" %}
```typescript
import * as firebase from 'firebase'
import { Post } from './state'

export const api = (() => {
  let app

  return {
    initialize() {
      app = firebase.initializeApp(...)      
    },
    async getPosts(): Promise<Post[]> {
      const snapshot = await app.database().ref('/posts').once('value')

      return snapshot.val()
    }
  }
})()
```
{% endtab %}
{% endtabs %}

We are doing two things here:

1. We use an [IIFE](https://developer.mozilla.org/en-US/docs/Glossary/IIFE) to create a scoped internal variable to be used for that specific effect
2. We have created an **initialize** method which we can call from the Overmind **onInitialize** action, which runs when the Overmind instance is created

Example of initializing the effect:

{% tabs %}
{% tab title="overmind/onInitialize.ts" %}
```typescript
import { OnInitialize } from 'overmind'

export const onInitialize: OnInitialize = async ({ effects }) => {
  effects.api.initialize()
  state.posts = await effects.api.getPosts()
}
```
{% endtab %}
{% endtabs %}

## Effects and state

Typically you explicitly communicate with effects from actions, by calling methods. But sometimes you need effects to know about the state of the application, or maybe some internal state in the effect should be exposed on your application state. Again we can take advantage of an **initialize** method on the effect:

{% tabs %}
{% tab title="overmind/onInitialize.ts" %}
```typescript
import { OnInitialize } from 'overmind'

export const onInitialize: OnInitialize = async ({ state, effects, actions }) => {
  effects.socket.initialize({
    onMessage: actions.onMessage,
    onStatusChange: actions.onSocketStatusChange,
    getToken() {
      return state.token
    }
  })
}
```
{% endtab %}
{% endtabs %}

Here we are passing in actions that can be triggered by the effect to expose internal state and/or other information that you want to manage.

## Lazy effects

You can also lazily load your effects in the **initialize** method. Let us say we wanted to load Firebase and its API on demand, or maybe just split out the code to make our app start faster.

{% tabs %}
{% tab title="overmind/effects.ts" %}
```typescript
import { Post } from './state'

export const api = (() => {
  let app

  return {
    async initialize() {
      const firebase = await import('firebase')

      app = firebase.initializeApp(...)
    },
    async getPosts(): Promise<Post[]> {
      const snapshot = await app.database().ref('/posts').once('value')

      return snapshot.val()
    }
  }
})()
```
{% endtab %}
{% endtabs %}

In our initialize\(\) we would just have to wait for the initialization to finish before using the API:

{% tabs %}
{% tab title="overmind/onInitialize.ts" %}
```typescript
import { OnInitialize } from 'overmind'

export const onInitialize: OnInitialize = async ({ effects }) => {
  await effects.api.initialize()
  state.posts = await effects.api.getPosts()
}
```
{% endtab %}
{% endtabs %}

We could have been even bolder here making our effect download its dependency related to using any of its methods. Imagine for example the **firebase** library downloading when you run a **login** method on the effect.

## Configurable effect

By defining a class we can improve testability, allow using environment variables and even change out the actual implementation.

{% tabs %}
{% tab title="overmind/effects.ts" %}
```typescript
import axios from 'axios'
import { User } from './state'

interface IRequest {
  get<T>(url: string): Promise<T>
}

export class Api {
  private baseUrl: string
  private request: IRequest
  constructor (baseUrl: string, request: IRequest) {
    this.baseUrl = baseUrl
    this.request = request
  }
  getCurrentUser(): Promise<User>  {
    return this.request.get(`${this.baseUrl}/user`)
  }
}

export const api = new Api(IS_PRODUCTION ? '/api/v1' : 'http://localhost:4321', axios)
```
{% endtab %}
{% endtabs %}

We export an instance of our **Api** to the application. This allows us to also create instances in isolation for testing purposes, making sure our Api class works as we expect.

## Summary

Importing side effects directly into your code should be considered bad practice. If you think about it from an application standpoint it is kinda weird that it runs HTTP verb methods with a URL string passed in. It is better to create an abstraction around it to make your code more consistent with the domain, and by doing so also improve maintainability.

