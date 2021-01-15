# statemachine

A statematchine allows you to wrap state with transitions. That means you can protect your logic from running in invalid states of the application.

Please read the [**statemachine guide**](../guides-1/using-state-machines.md) ****to learn more about statemachines and typing them.

## Create a statemachine

You define a whole namespace as a statemachine, you can have a nested statemachine or you can even put statemachines inside statemachines.

```javascript
import { statemachine } from 'overmind'

export const machine = statemachine({
  TOGGLE: (state, payload) => {
    return { current: state.current === 'FOO' ? 'BAR' : 'FOO' }
  }
})
```

You define a statemachine by setting up the events it should manage. Each even handler decides at what current state it should run its logic. It does this by checking the **current** state. The event handler can optionally return a new state, which transitions the machine.

## Instantiate machine

You instantiate a machine by calinng its **create** method. This takes the initial state and any optional base state the lives across all states of the machine.

```javascript
import { machine } from './myMachine'

export const state = {
  myMachine: machine.create({ current: 'FOO' }, { list: [] })
}
```

## Transition between states

You transition the state machine by sending events.

That means you can send **TOGGLE** as an event to the machine. Think of sending events as actually changing the state, but in a controlled way. You can optionally pass a payload with the event.

```javascript
export const toggle = ({ state, effects }) => {
  state.myMachine.send('TOGGLE', somePayload)
}
```

## Matching state

In actions you can check what state a machine is in by using the **matches** API.

```javascript
export const toggle = ({ state, effects }) => {
  if (state.myMachine.matches('FOO')) {
    state.myMachine.send('TOGGLE', 'Cool')
  } else {
    state.myMachine.send('TOGGLE', 'Not so cool')
  }
}
```

You can also directly use `state.myMachine.current` , but please read the guide to understand the different between these two ways of checking.

