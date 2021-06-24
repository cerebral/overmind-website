# Using state machines

The Overmind state machines is heavily inspired by [XState](https://xstate.js.org/) and [Davids](https://twitter.com/DavidKPiano) evangelism of bringing this old idea to life in the JavaScript ecosystem. Typically state machines are explained with very specific concepts like street lights, timers or similar "machine like" concepts. For Overmind it was important that this concept could be used to describe the overall state of the application. This was a huge challenge and required several iterations, but we found a concept that holds the idea true and makes it a practical and optional way to manage your state. Use it for your whole application or use it for specific scenarios.

{% hint style="info" %}
The state machine API is designed for use with **TypeScript**. The reason is that the complexity of transition state matching is best expressed using [optional chaining](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining), which is not yet available in plain JavaScript.
{% endhint %}

## Creating a state machine

To understand the benefit of a state machine we have to use a very specific example. One such example that is typical for any application is authentication. Typically in Overmind you would define this as:

```typescript
type State = {
  isAuthenticating: boolean
  user: { username: string } | null
  signedOutReason: string | null
}

export const state: State = {
  isAuthenticating: true,
  user: null,
  signedOutReason: null
}
```

You would use the existence of the **user** to determine if you are actually authenticated or not. This _works**,**_ but it does not describe the states of your application explicitly. If you rather describe this state as:

```typescript
type State = {
  current: 'AUTHENTICATING'
} | {
  current: 'AUTHENTICATED'
  user: { username: string }
} | {
  current: 'UNAUTHENTICATED'
  signedOutReason: string
}

export const state: State = {
  current: 'AUTHENTICATING'
}
```

Now we are describing what states our application can actually be in, and what other state is available at that time.

State machines does not only help us describe explicit states, they act as a translator between the effects of the outside world and the state of your application. It basically ensures that whatever happens "out there" the state machine will ensure that your state is valid.

You express this by mapping **events** to **state changes**.

```typescript
type User = { username: string }

type States =
  | {
    current: 'AUTHENTICATING'
  }
  | {
    current: 'AUTHENTICATED'
    user: User
  }
  | {
    current: 'UNAUTHENTICATED'
    signedOutReason: string
  }

type Events = 
  | {
    type: 'SIGNING_IN'
  }
  | {
    type: 'SIGNED_IN'
    data: User
  }
  | {
    type: 'SIGNED_OUT'
    data: string
  }

export const auth = statemachine<States, Events>({
  SIGNING_IN: (state) => {
    if (state.current === 'UNAUTHENTICATED') {
      return { current: 'AUTHENTICATING' }
    }
  },
  SIGNED_IN: (state, user) => {
    if (state.current === 'AUTHENTICATING') {
      return { current: 'AUTHENTICATED', user }
    }
  },
  SIGNED_OUT: (state, signedOutReason) => {
    if (state.current === 'AUTHENTICATED') {      
      return { current: 'UNAUTHENTICATED', signedOutReason }
    }
  }
})
```

In the example above we are are dealing with three events. For each event we check the current state of the machine to see if we want to deal with it at all. When we decide to deal with an event we can change any of the state, for example using **data** from the event. Then we can optionally return a new **current** transition state, with the required state for that transition state to be valid.

What we have effectively done now is ensure that when these events happens we always deal with them correctly. It is not the event that decides what should happen, it is the machine that decides it based on one of your explicitly set states.

## Instantiating a machine

To actually use the machine as part of your state you need to **create** it.

```javascript
import { auth } from './state'
import * as actions from './actions'

const config = {
  state: auth.create({
    current: 'AUTHENTICATING'
  })
}
```

By explicitly instantiating the machine you are allowed to start it in different transition states and also give preset state if necessary. You will see this becomes beneficial later when nesting machines.

## Sending events

Instead of explicitly changing the state, you send an **event**. The events is handled by the state machine and it will ensure that it is valid before moving on. That means when you change from **AUTHENTICATING** to **AUTHENTICATED** you would express it something like:

```javascript
export const authChanged = ({ state }, user) => {
  if (user) {
    state.send('SIGNED_IN', user)
  } else {
    state.send('SIGNED_OUT')
  }
}
```

When sending the **SIGNED\_IN** event we also provide the **user**. The current transition state of the machine is what decides if the user is set or not.

## Guarding effects

Now, your state machine is in charge of how it acts on events coming form the outside world, but you might also want the outside world to react to changes in your state machine. So imagine related to transitioning into a state you wanted to change the title of the page. To ensure this logic only runs when your application actually transitions into the **UNAUTHENTICATED** or **AUTHENTICATED** state we can check if the machine actually is in this state after sending it a message.

```javascript
export const authChanged = ({ state, effects }, user) => {
  if (user && state.send('SIGNED_IN', user).matches('AUTHENTICATED')) {
    effects.browser.setTitle('Logged in')
  } else if (state.send('SIGNED_OUT').matches('UNAUTHENTICATED')) {
    effects.browser.setTitle('Logged out')
  }
}
```

## Base state

Let us introduce a new machine, a **todos** machine.

```typescript
import { StateMachine } from 'overmind'

type Todo = { title: string, completed: boolean }

type States = 
 | {
   current: 'LOADING'
 }
 | {
   current: 'LIST'
 }

 type BaseState {
   list: Todo[]
 }

type Events =
  | {
    type: 'TODOS_LOADED',
    data: Todo[]
  }
  | {
    type: 'TODO_ADDED',
    data: Todo
  }

export type TodosMachine = StateMachine<States, Events BaseState>

export const todos = statemachine<States, Events, BaseState>({
  TODOS_LOADED: (state, todos) => {
    if (state.current === 'LOADING') {      
      return { current: 'LIST', todos }
    }
  },
  TODO_ADDED: (state, todo) => {
    if (state.current === 'LIST') {
      state.list.push(todo)
    }
  }
})
```

In this simple example we introduced a todos machine that starts in a **LOADING** state and will at some point transition into a **LIST** state when the initial todos has been loaded. The machine introduces the concept of **base state**. That means state that is available no matter what transition state the machine is in. The purpose of **base state** is that it simplifies typing and the machine will also automatically remove state related to the current transition state, when transitioning to a new state. In the example above the **user** and the **signedOutReason** is deleted when moving out of **AUTHENTICATED** state.

## Nesting state machines

One of the goals of the Overmind implementation of state machines is that the machines becomes a natural part of your state tree. You can define them wherever you would normally define a value. That means you can create nested machines.

```typescript
import { TodosMachine, todos } from './Todos'

type States =
  | {
    current: 'AUTHENTICATING'
  }
  | {
    current: 'AUTHENTICATED'
    user: User
    todos: TodosMachine
  }
  | {
    current: 'UNAUTHENTICATED'
    signedOutReason: string
  }

type Events = {...}

export const auth = statemachine<States, Events>({
  SIGNING_IN: (state) => {
    if (state.current === 'UNAUTHENTICATED') {
      return { current: 'AUTHENTICATING' }
    }
  },
  SIGNED_IN: (state, user) => {
    if (state.current === 'AUTHENTICATING') {      
      return {
        current: 'AUTHENTICATED',
        user,
        todos: todos.create({ current: 'LOADING' }, { todos: [] })
      }
    }
  },
  SIGNED_OUT: (state, signedOutReason) => {
    if (state.current === 'AUTHENTICATED') {      
      return { current: 'UNAUTHENTICATED', signedOutReason }
    }
  }
})
```

Note that the **base state** of the **todos** is passed as a second argument.

Now we can go back to our authentication logic and introduce the loading of our todos.

```javascript
export const authChanged = async ({ state, effects }, user) => {
  if (user && state.send('SIGNED_IN', user).matches('AUTHENTICATED')) {
    const todos = await effects.api.getTodos()
    state.matches('AUTHENTICATED')?.todos.send('TODOS_LOADED', todos)
  } else if (state.send('SIGNED_OUT').matches('UNAUTHENTICATED')) {
    effects.browser.setTitle('Logged out')
  }
}
```

You will notice that with nested machines you will be using **matches** and optional chaining quite a bit. The reason simply being that you will always have to ensure that your machines are in the correct transition state before interacting with any of its state and nested machines.

## Identifying current state in components

All state machines has a **current** property. This can be used to evaluate what should be rendered, here shown with React:

```javascript
export const App = () => {
  const { state } = useOvermind()

  if (state.current === 'AUTHENTICATING') {
    return <div>Loading...</div>
  }

  if (state.current === 'AUTHENTICATED') {
    return <div>You are not authenticated</div>
  }

  return <div>Hello there!</div>
}
```

When dealing with nested machines you will have to do nested checks. This might seem unnecessary, maybe you loaded the **Todos** component only when the parent is in **AUTHENTICATED** state, but components can be moved and loaded anywhere, so this ensures it behaves exactly like we want it to.

```typescript
export const Todos = () => {
  const { state } = useOvermind()

  if (!state.current === 'AUTHENTICATED') return null

  return (
    <div>
      {state.todos.current === 'LOADING' ? 'Loading...' : null}
      <ul>{state.todos.list.map(() => ...)}</ul>
    </div>
  )
}
```

## Strict mode

In strict mode you are not able to change state in actions, you explicitly have to use a state machine transitions through the **send** API to make state changes.

```javascript
const overmind = createOvermind(config, {
  strict: true
})
```

```javascript
export const authChanged = ({ state }, user) => {
  // This would throw an error 
  state.user = user
}
```

