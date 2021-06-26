# Typescript

Overmind is written in Typescript and it is written with a focus on you dedicating as little time as possible to help Typescript understand what your app is all about. Typescript will spend a lot more time helping you. If you are not a Typescript developer Overmind is a really great project to start learning it as you will get the most out of the little typing you have to do.

## Configuration

The only typing you need is the **Context**. This holds information about your state, actions and effects.

{% tabs %}
{% tab title="overmind/index.ts" %}
```typescript
import { IContext } from 'overmind'

export const config = {}

export type Context = IContext<typeof config>
```
{% endtab %}
{% endtabs %}

You only have to set up these types once, where you bring your configuration together. That means if you use multiple namespaced configuration you still only create a single **Context** type.

## State

The state you define in Overmind is just an object where you type that object.

{% tabs %}
{% tab title="overmind/state.ts" %}
```typescript
type State = {
  foo: string
  bar: boolean
  baz: string[]
  user: User
}

export const state: State = {
  foo: 'bar',
  bar: true,
  baz: [],
  user: new User()
}
```
{% endtab %}
{% endtabs %}

When writing Typescript you should **not** use optional values for your state \(**?**\), or use **undefined** in a union type. In a serializable state store world **null** is the value indicating _“there is no value”._

```typescript
type State = {
  // Do not do this
  foo?: string

  // Do not do this
  foo: string | undefined

  // Do this
  foo: string | null

  // Or this, if there always will be a value there
  foo: string
}

export const state: State = {
  foo: null
}
```

### Derived

{% tabs %}
{% tab title="overmind/state.ts" %}
```typescript
import { derived } from 'overmind'

type State = {
  foo: string
  shoutedFoo: string
}

export const state: State = {
  foo: 'bar',
  shoutedFoo: derived((state: State) => state.foo + '!!!')
}
```
{% endtab %}
{% endtabs %}

Note that the type argument you pass is the object the derived is attached to, so with nested derived:

{% tabs %}
{% tab title="overmind/state.ts" %}
```typescript
import { derived } from 'overmind'

type State = {
  foo: string
  nested: {
    shoutedFoo: string
  }
}

export const state: State = {
  foo: 'bar',
  nested: {
    shoutedFoo: derived((state: State['nested']) => state.foo + '!!!')
  }
}
```
{% endtab %}
{% endtabs %}

To access the root state you can use your **Context** type:

{% tabs %}
{% tab title="overmind/state.ts" %}
```typescript
import { Context } from 'app/overmind'

type State = {
  foo: string
  shoutedFoo: string
}

export const state: State = {
  foo: 'bar',
  shoutedFoo: derived(
    (state: State, rootState: Context["state"]) => state.foo + '!!!'
  )
}
```
{% endtab %}
{% endtabs %}

### Statemachine

Read the guide on [**Using state machines**](../guides-1/using-state-machines.md) to understand how to type them.

## Actions

You type your actions with the **Context** and an optional value. Any return type will be inferred.

```typescript
import { Context } from 'app/overmind'

export const noArgAction = (context: Context) => {
  // actions.noArgAction()
}

export const argAction = (context: Context, value: string) => {
  // actions.argAction("foo"), requires "string"
}

export const noArgWithReturnTypeAction = (context: Context) => {
  // actions.noArgWithReturnTypeAction(), with return type "string"
  return 'foo'
}

export const argWithReturnTypeAction = (context: Context, value: string) => {
  // actions.argWithReturnTypeAction("foo"), requires "string" and returns "string"
  return value + '!!!'
}
```

Any of these actions could be defined as an **async** function or simply return a promise to be typed that way.

## Effects

There are no Overmind specific types related to effects, you just type them in general.

{% tabs %}
{% tab title="overmind/effects.ts" %}
```typescript
export const api = {
  getUser: async (): Promise<User> => {
    const response = await fetch('/user')
    
    return response.json()
  }
}
```
{% endtab %}
{% endtabs %}

## Operators

Operators is like the action: it can take an optional value, but it always produces a **promise** output. By default the promised value of an operator is the same as the input.

{% tabs %}
{% tab title="overmind/operators.ts" %}
```typescript
import { Context, filter } from 'overmind'

// Actions are interoperable with operators. So type them
// like an action
export const changeSomeState = ({ state }: Context) =>  {
  state.foo = 'bar'
}

// Most operators takes an action signature, just type it as that
export const filterAwesomeUser = filter((_: Context, user: User) => 
  user.isAwesome
})
```
{% endtab %}
{% endtabs %}

When you create a **pipe** and inline other operators/actions their payloads are inferred. Only the first operator needs to type its payload so that when calling **doThis** you will have the correct typing for the initial payload.

```typescript
import { Context, pipe } from 'overmind'

export const doThis = pipe(
  (context: Context, value: string) => {
    // actions.doThis("foo"), requires "string"
    return 123
  },
  (context: Context, value) => {
    // value is now "number"
  }
)

// call action
doThis('foo') // Typed to string, as first operator needs string
```

