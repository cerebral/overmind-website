# React

## Install

```text
npm install overmind overmind-react
```

When you connect Overmind to a component you ensure that whenever any tracked state changes, only components interested in that state will re-render, and will do so “at their location in the component tree”. That means we remove a lot of unnecessary work from React. There is no reason for the whole React component tree to reconcile when only one component is interested in a change.

## Hook

{% tabs %}
{% tab title="Javascript" %}
```typescript
// overmind/index.js
import {
  createStateHook,
  createActionsHook,
  createEffectsHook,
  createReactionHook
} from 'overmind-react'
import { state } from './state'
import * as actions from './actions'

export const config = {
  state,
  actions
}

export const useAppState = createStateHook()
export const useActions = createActionsHook()
export const useEffects = createEffectsHook()
export const useReaction = createReactionHook()

// index.js
import * as React from 'react'
import { render } from 'react-dom'
import { createOvermind } from 'overmind'
import { Provider } from 'overmind-react'
import { config } from './overmind'
import App from './components/App'

const overmind = createOvermind(config)

render((
  <Provider value={overmind}>
    <App />
  </Provider>
), document.querySelector('#app'))

// components/App.jsx
import * as React from 'react'
import { useAppState, useActions, useEffects, useReaction } from '../overmind'

const App = () => {
  // General
  const state = useAppState()
  const actions = useActions()
  const effects = useEffects()
  const reaction = useReaction()
  // Or be specific
  const { isLoggedIn } = useState().auth
  const { login, logout } = useActions().auth

  return <div />
}

export default App
```
{% endtab %}

{% tab title="Typescript" %}
```typescript
// overmind/index.ts
import { IContext } from 'overmind'
import { 
  createStateHook,
  createActionsHook,
  createEffectsHook,
  createReactionHook
} from 'overmind-react'
import { state } from './state'
import * as actions from './actions'

export const config = {
  state,
  actions
}

export type Context = IContext<{
  state: typeof config.state
  actions: typeof config.actions
}>

export const useAppState = createStateHook<Context>()
export const useActions = createActionsHook<Context>()
export const useEffects = createEffectsHook<Context>()
export const useReaction = createReactionHook<Context>()

// index.tsx
import * as React from 'react'
import { render } from 'react-dom'
import { createOvermind } from 'overmind'
import { Provider } from 'overmind-react'
import { config } from './overmind'
import App from './components/App'

const overmind = createOvermind(config)

render((
  <Provider value={overmind}>
    <App />
  </Provider>
), document.querySelector('#app'))

// components/App.tsx
import * as React from 'react'
import { useAppState } from '../overmind'

const App = () => {
  const { isLoggedIn } = useAppState().auth
  const { login, logout } = useActions().auth
  
  return <div />
}

export default App
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
The benefit of using specific hooks is that if you only need actions in a component, you do not add tracking behaviour to the component by using **useActions**. Also it reduces the amount of destructuring needed, as you can point to a namespace on the hook.
{% endhint %}

### Rendering

When you use the Overmind hook it will ensure that the component will render when any tracked state changes. It will not do anything related to the props passed to the component. That means whenever the parent renders, this component renders as well. You will need to wrap your component with [**REACT.MEMO**](https://reactjs.org/docs/react-api.html#reactmemo) to optimize rendering caused by a parent.

### Scoped tracking

The state hook is able to scope the tracking to a speciic value. This is especially useful in lists. In the example below we are passing the `id` of a todo to the **Todo** child component. Inside that component we scope the tracking to the specific todo value in the state tree. That means the **Todo** components does not care about changes to added/removed todos, only changes related to what you access on the specific todo.

{% tabs %}
{% tab title="Javascript" %}
```typescript
// components/Todos.jsx
import * as React from 'react'
import { useAppState } from '../overmind'
import Todo from './Todo'

const Todos = () => {
  const state = useAppState()

  return (
    <ul>
      {Object.keys(tate.todos).map(id => <Todo key={id} id={id} />)}
    </ul<
  )
}

export default Todos

// components/Todo.jsx
import * as React from 'react'
import { useAppState } from '../overmind'

const Todo = React.memo(({ id }) => {
  const todo = useAppState(state => state.todos[id])

  return <li>{todo.title}</li>
})

export default Todo
```
{% endtab %}

{% tab title="Typescript" %}
```typescript
// components/Todos.tsx
import * as React from 'react'
import { useAppState } from '../overmind'
import Todo from './Todo'

const Todos = ({ id }: { id: string }) => {
  const state = useAppState()

  return (
    <ul>
      {Object.keys(state.todos).map(id => <Todo key={id} id={id} />)}
    </ul<
  )
}

export default Todos

// components/Todo.tsx
import * as React from 'react'
import { useAppState } from '../overmind'

const Todo = React.memo(({ id }: { id: string }) => {
  const todo = useAppState(state => state.todos[id])

  return <li>{todo.title}</li>
})

export default Todo
```
{% endtab %}
{% endtabs %}

### Reactions

The hook effect of React gives a natural point of running effects related to state changes. An example of this is is scrolling to the top of the page whenever the current page state changes.

{% tabs %}
{% tab title="Javascript" %}
```typescript
// components/App.jsx
import * as React from 'react'
import { useEffect } from 'react'
import { useAppState } from '../overmind'

const App = () => {
  const state = useAppState()

  useEffect(() => {
    document.querySelector('#app').scrollTop = 0
  }, [state.currentPage])

  return <div />
}

export default App
```
{% endtab %}

{% tab title="Typescript" %}
```javascript
// components/App.tsx
import * as React from 'react'
import { useEffect } from 'react'
import { useAppState } from '../overmind'

const App = () => {
  const state = useAppState()

  useEffect(() => {
    document.querySelector('#app').scrollTop = 0
  }, [state.currentPage])

  return <div />
}

export default App
```
{% endtab %}
{% endtabs %}

Here you can also use the traditional approach of subscribing to updates.

{% tabs %}
{% tab title="Javascript" %}
```typescript
// components/App.jsx
import * as React from 'react'
import { useEffect } from 'react'
import { useReaction } from '../overmind'

const Todos = () => {
  const reaction = useReaction()

  useEffect(() => reaction(
    ({ currentPage }) => currentPage,
    () => { 
      document.querySelector('#app').scrollTop = 0
    }
  ), [])

  return <div />
}

export default Todos
```
{% endtab %}

{% tab title="Typescript" %}
```javascript
// components/App.tsx
import * as React from 'react'
import { useEffect } from 'react'
import { useReaction } from '../overmind'

const Todos = () => {
  const reaction = useReaction()

  useEffect(() => reaction(
    ({ currentPage }) => currentPage,
    () => {
      document.querySelector('#app').scrollTop = 0
    }
  ), [])

  return <div />
}

export default Todos
```
{% endtab %}
{% endtabs %}

## React Native

Overmind supports React Native. What to take notice of though is that native environments sometimes abstracts away the main **render** function of React. That can be a bit confusing in terms of setting up the **Provider**. If your environment only exports an initial component, that component needs to be responsible for setting up the providers and rendering your main component:

```typescript
import { createOvermind } from 'overmind'
import { Provider } from 'overmind-react'
import { config } from './overmind'
import MyApp from './MyApp'

const overmind = createOvermind(config)

export function App() {
  return (
    <Provider value={overmind}>
      <MyApp />
    </Provider>
  )
}
```

