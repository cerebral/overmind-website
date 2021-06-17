# action

```typescript
export const getPosts = async ({ state, actions, effects }) => {
  state.isLoadingPosts = true
  state.posts = await effects.api.getPosts()
  state.isLoadingPosts = false
}
```

An action is where you write the logic of the application. Every action receives at least one argument and that is the **context**. This is the signature of the context:

`{ state, actions, effects }`

This _injected_ context allows Overmind to understand from where you are changing state and running effects. You can also use other actions defined in your application. Additionally with _injection_ your actions become highly testable as it can easily be mocked.

State changes are restricted to these actions. That means if you try to change the state outside of an action you will get an error. The state changes are also scoped to the action. That means it does not matter if you perform the state change asynchronously, either by defining the action as an **async** function or for example use a **setTimeout**, returning a promise which is resolved when it times out.

## Payload

When an action is called you can optionally pass it a payload. This payload is received as the second argument to the action.

```typescript
export const setTitle = ({ state }, title) => {
  state.title = title
}
```

{% hint style="info" %}
There is only one argument, which means if you want to pass multiple values you have to do so with an object
{% endhint %}

