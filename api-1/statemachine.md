# statemachine

A statematchine allows you to wrap state with transitions. That means you can protect your logic from running in invalid states of the application.

## Create a statemachine

You define a whole namespace as a statemachine, you can have a nested statemachine or you can even put statemachines inside statemachines.

```javascript
import { statemachine } from 'overmind'

export const state = statemachine({
  UNAUTHENTICATED: ['AUTHENTICATING'],
  AUTHENTICATING: ['UNAUTHENTICATED', 'AUTHENTICATED'],
  AUTHENTICATED: ['UNAUTHENTICATED']
}, {
  state: 'UNAUTHENTICATED'
})
```

Instead of only defining state, you first define a set of transitions. The key represents a transition state, here **UNAUTHENTICATED**, **AUTHENTICATING** and **AUTHENTICATED**. Then we define an array which shows the next transition state can occur in the given transition state. When **UNAUTHENTICATED** we can move into the **AUTHENTICATING** state for example. When in **AUTHENTICATING** state we can move either back to **UNAUTHENTICATED** due to an error or we might move to **AUTHENTICATED**. The point is... when you are **UNAUTHENTICATED**, you can not run logic related to being **AUTHENTICATED**. And when **AUTHENTICATING** you can not run that logic again until you are back in **UNAUTHENTICATED**. 

As actual state values we define the initial transition state of **UNAUTHENTICATED**.

If we wanted we could extend the state with other values, as normal.

```javascript
import { statemachine } from 'overmind'

export const state = statemachine({
  UNAUTHENTICATED: ['AUTHENTICATED'],
  AUTHENTICATING: ['UNAUTHENTICATED', 'AUTHENTICATED'],
  AUTHENTICATED: ['UNAUTHENTICATED']
}, {
  state: 'UNAUTHENTICATED',
  todos: {},
  filter: 'all'
})
```

## Transition between states

The transition states are also part of the resulting **state** object, in this case:

```javascript
// The resulting state object
export const state = {
  UNAUTHENTICATED: () => {...},
  AUTHENTICATING: () => {...},
  AUTHENTICATED: () => {...},
  state: 'UNAUTHENTICATED',
  todos: {},
  filter: 'all'
}
```

That means you can call **UNAUTHENTICATED**, **AUTHENTICATING** and **AUTHENTICATED** as functions to transition into the new states. And this is an example of how you would use them:

```javascript
export const login = ({ state, effects }) => {
  return state.AUTHENTICATING(() => {
    try {
      const user = await effects.api.login()
      return state.AUTHENTICATED(() => {
        state.user = user
      })
    } catch (error) {
      return state.UNAUTHENTICATED(() => {
        state.error = error
      })
    }
  })
}
```

When a component, or something else, calls the **login** action it will first try to move into the **AUTHENTICATING** state. If this is not possible, nothing else will happen. Then we go ahead an login, which returns a user. If we were to try to set the user immediately an error would be thrown, because it is being set "out of scope of the transition" \(asynchronously\). To actually set the user we first transition to **AUTHENTICATED** and given that is a valid transition the user will be set.

What we accomplish in practice here is to ensure that changes to state is guarded by these transitions, which results in more predictable and safer code.

