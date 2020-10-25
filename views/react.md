# React

There are two different ways to connect Overmind to React. You can either use a traditional **Higher Order Component** or you can use the new **hooks** api to expose state and actions.

When you connect Overmind to a component you ensure that whenever any tracked state changes, only components interested in that state will re-render, and will do so “at their location in the component tree”. That means we remove a lot of unnecessary work from React. There is no reason for the whole React component tree to re-render when only one component is interested in a change.

## Hook

{% tabs %}
{% tab title="Javascript" %}
```typescript
// overmind/index.js
import { createHook, createStateHook, createActionsHookcreateEffectsHook, createReactionHook } from 'overmind-react'
import { state } from './state'
import * as actions from './actions'

export const config = {
  state,
  actions
}

export const useOvermind = createHook()
export const useState = createStateHook()
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
import { useOvermind } from '../overmind'

const App = () => {
  // General
  const { state, actions, effects, reaction } = useOvermind()
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
import { IConfig } from 'overmind'
import { createHook } from 'overmind-react'
import { state } from './state'
import * as actions from './actions'

export const config = {
  state,
  actions
}

declare module 'overmind' {
  interface Config extends IConfig<typeof config> {}
}

export const useOvermind = createHook<typeof config>()
export const useState = createStateHook<typeof config>()
export const useActions = createActionsHook<typeof config>()
export const useEffects = createEffectsHook<typeof config>()
export const useReaction = createReactionHook<typeof config>()

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
import { useOvermind } from '../overmind'

const App: React.FunctionComponent = () => {
  // General
  const { state, actions, effects, reaction } = useOvermind()
  // Or be specific
  const { isLoggedIn } = useState().auth
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

### Passing state as props

If you pass a state object or array as a property to a child component you will also in the child component need to use the **useOvermind** hook to ensure that it is tracked within that component, even though you do not access any state or actions. The devtools will help you identify where any components are left “unconnected”.

{% tabs %}
{% tab title="Javascript" %}
```typescript
// components/Todos.jsx
import * as React from 'react'
import { useState } from '../overmind'
import Todo from './Todo'

const Todos = () => {
  const { todos } = useState()

  return (
    <ul>
      {todos.map(todo => <Todo key={todo.id} todo={todo} />)}
    </ul<
  )
}

export default Todos

// components/Todo.jsx
import * as React from 'react'
import { useState } from '../overmind'

const Todo = ({ todo }) => {
  useState()

  return <li>{todo.title}</li>
}

export default Todo
```
{% endtab %}

{% tab title="Typescript" %}
```typescript
// components/Todos.tsx
import * as React from 'react'
import { useState } from '../overmind'
import Todo from './Todo'

const Todos: React.FunctionComponent = () => {
  const { todos } = useState()

  return (
    <ul>
      {todos.map(todo => <Todo key={todo.id} todo={todo} />)}
    </ul<
  )
}

export default Todos

// components/Todo.tsx
import * as React from 'react'
import { useState } from '../overmind'

type Props = {
  todo: Todo
}

const Todo: React.FunctionComponent<Props> = ({ todo }) => {
  useState()

  return <li>{todo.title}</li>
}

export default Todo
```
{% endtab %}
{% endtabs %}

### Reactions

The hook effect of React gives a natural point of running effects related to state changes. An example of this is from the Overmind website, where we scroll to the top of the page whenever the current page state changes.

{% tabs %}
{% tab title="Javascript" %}
```typescript
// components/App.jsx
import * as React from 'react'
import { useEffect } from 'react'
import { useState } from '../overmind'

const App = () => {
  const { currentPage } = useState()

  useEffect(() => {
    document.querySelector('#app').scrollTop = 0
  }, [currentPage])

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
import { useState } from '../overmind'

const App: React.FunctionComponent = () => {
  const { currentPage } = useState()

  useEffect(() => {
    document.querySelector('#app').scrollTop = 0
  }, [currentPage])

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

  useEffect(() => {
    return reaction(
      ({ currentPage }) => currentPage,
      () => document.querySelector('#app').scrollTop = 0
    })
  }, [])

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

const Todos: React.FunctionComponent = () => {
  const reaction = useReaction()

  useEffect(() => {
    return reaction(
      ({ currentPage }) => currentPage,
      () => document.querySelector('#app').scrollTop = 0
    })
  }, [])

  return <div />
}

export default Todos
```
{% endtab %}
{% endtabs %}

## Higher Order Component

{% tabs %}
{% tab title="Javascript" %}
```typescript
// overmind/index.js
import { createConnect } from 'overmind-react'
import { state } from './state'
import * as actions from './actions'

export const config = {
  state,
  actions
}

export const connect = createConnect()

// index.jsx
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
import { connect, Connect } from '../overmind'

const App = ({ overmind }) => {
  const { state, actions, effects, addMutationListener } = overmind

  return <div />
}

export default connect(App)
```
{% endtab %}

{% tab title="Typescript" %}
```typescript
// overmind/index.ts
import { IConfig } from 'overmind'
import { createConnect, IConnect } from 'overmind-react'
import { state } from './state'
import * as actions from './actions'

export const config = {
  state,
  actions
}

declare module 'overmind' {
  interface Config extends IConfig<typeof config> {}
}

export interface Connect extends IConnect<typeof config> {}

export const connect = createConnect<typeof config>()

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
import { connect, Connect } from '../overmind'

type Props = {} & Connect

const App: React.FunctionComponent<Props> = ({ overmind }) => {
  const { state, actions, effects, addMutationListener } = overmind

  return <div />
}

export default connect(App)
```
{% endtab %}
{% endtabs %}

### Rendering

When you connect a component with the **connect HOC** it will be responsible for tracking and trigger a render when the tracked state is updated. The **overmind** prop passed to the component you defined holds the state and actions. If you want to detect inside your component that it was indeed an Overmind state change causing the render you can compare the **overmind** prop itself.

{% tabs %}
{% tab title="Javascript" %}
```typescript
// components/App.jsx
import * as React from 'react'
import { connect } from '../overmind'

class App extends React.Component {
  shouldComponentUpdate(nextProps) {
    return this.props.overmind !== nextProps.overmind
  }
  render() {
    return <div />
  }
}

export default connect(App)
```
{% endtab %}

{% tab title="Typescript" %}
```typescript
// components/App.tsx
import * as React from 'react'
import { connect, Connect } from '../overmind'

type Props = {} & Connect

class App extends React.Component<Props> {
  shouldComponentUpdate(nextProps: Props) {
    return this.props.overmind !== nextProps.overmind
  }
  render() {
    return <div />
  }
}

export default connect(App)
```
{% endtab %}
{% endtabs %}

You will not be able to compare a previous state value in Overmind with the new. That is simply because Overmind is not immutable and it should not be. You will not use **shouldComponentUpdate** to compare state in Overmind, though you can of course still use it to compare props from a parent. This is a bit of a mindshift if you come from Redux, but it actually removes the mental burden of doing this stuff.

If you previously used **componentDidUpdate** to trigger an effect, that is no longer necessary either. You rather listen to state changes in Overmind using **addMutationListener** specified below in _effects_.

### Passing state as props

If you pass a state object or array as a property to a child component you will also in the child component need to **connect**. This ensures that the property you passed is tracked within that component, even though you do not access any state or actions from Overmind. The devtools will help you identify where any components are left “unconnected”.

{% tabs %}
{% tab title="Javascript" %}
```typescript
// components/Todos.jsx
import * as React from 'react'
import { connect } from '../overmind'
import Todo from './Todo'

const Todos = ({ overmind }) => {
  const { state } = overmind

  return (
    <ul>
      {state.todos.map(todo => <Todo key={todo.id} todo={todo} />)}
    </ul<
  )
}

export default connect(Todos)

// components/Todo.tsx
import * as React from 'react'
import { connect } from '../overmind'

const Todo = ({ todo }) => {
  return <li>{todo.title}</li>
}

export default connect(Todo)
```
{% endtab %}

{% tab title="Typescript" %}
```typescript
// components/Todos.tsx
import * as React from 'react'
import { connect, Connect } from '../overmind'
import Todo from './Todo'

type Props = {} & Connect

const Todos: React.FunctionComponent<Props> = ({ overmind }) => {
  const { state } = overmind

  return (
    <ul>
      {state.todos.map(todo => <Todo key={todo.id} todo={todo} />)}
    </ul<
  )
}

export default connect(Todos)

// components/Todo.tsx
import * as React from 'react'
import { connect, Connect } from '../overmind'

type Props = {
  todo: Todo
} & Connect

const Todo: React.FunctionComponent<Props> = ({ todo }) => {
  return <li>{todo.title}</li>
}

export default connect(Todo)
```
{% endtab %}
{% endtabs %}

### Reactions

To run reactions in components based on changes to state you use the **reaction** function in the lifecycle hooks of React.

{% tabs %}
{% tab title="Javascript" %}
```typescript
// components/App.jsx
import * as React from 'react'
import { connect } from '../overmind'

class App extends React.Component {
  private disposeReaction
  componentDidMount() {
    this.disposeReaction = this.props.overmind.reaction(
      (state) => state.currentPage,
      () => document.querySelector('#app').scrollTop = 0
    )
  }
  componentWillUnmount() {
    this.disposeReaction()
  }
  render() {
    const { state, actions } = this.props.overmind

    return <div />
  }
}

export default connect(App)
```
{% endtab %}

{% tab title="Typescript" %}
```typescript
// components/App.tsx
import * as React from 'react'
import { connect, Connect } from '../overmind'

type Props = {} & Connect

class App extends React.Component<Props> {
  disposeReaction: () => void
  componentDidMount() {
    this.disposeReaction = this.props.overmind.reaction(
      (state) => state.currentPage,
      () => document.querySelector('#app').scrollTop = 0
    )
  }
  componentWillUnmount() {
    this.disposeReaction()
  }
  render() {
    const { state, actions } = this.props.overmind

    return <div />
  }
}

export default connect(App)
```
{% endtab %}
{% endtabs %}

## React Native

Overmind supports React Native with **hook** and **Higher Order Component**. What to take notice of though is that native environments sometimes hides the **render** function of React. That can be a bit confusing in terms of setting up the **Provider**. If your environment only exports an initial component, that component needs to be responsible for settings up the providers and rendering your main component:

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

