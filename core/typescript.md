# Typescript

Overmind is written in Typescript and it is written with a focus on you dedicating as little time as possible to help Typescript understand what your app is all about. Typescript will spend a lot more time helping you. If you are not a Typescript developer Overmind is a really great project to start learning it as you will get the most out of the little typing you have to do.

## Configuration

First we need to define the typing of our configuration and there are two approaches to that.

### 1. Declare module

The most straightforward way to type your application is to use the **declare module** approach. This will work for most applications, but might make you feel uncomfortable as a hardcore Typescripter. The reason is that we are overriding an internal type, meaning that you can only have one instance of Overmind running inside your application.

{% tabs %}
{% tab title="overmind/index.ts" %}
```typescript
import { IConfig } from 'overmind'

const config = {}

declare module 'overmind' {
  // eslint-disable-next-line @typescript-eslint/no-empty-interface
  interface Config extends IConfig<{
    state: typeof config.state,
    actions: typeof config.actions,
    effects: typeof config.effects
  }> {}
  // Due to circular typing we have to define an
  // explicit typing of state, actions and effects since
  // TS 3.9
}
```
{% endtab %}
{% endtabs %}

Now you can import any type directly from Overmind and it will understand the configuration of your application. Even the operators are typed.

```typescript
import {
  Context,
  RootState,
  pipe,
  map,
  filter,
  ...
  // These are primitive types in Overmind you typically
  // do not need
  Action,
  AsyncAction,
  Operator
} from 'overmind'
```

### 2. Explicit typing

You can also explicitly type your application. This gives more flexibility.

{% tabs %}
{% tab title="overmind/index.ts" %}
```typescript
import {
  IConfig,
  IOnInitialize,
  IContext,
} from 'overmind'

export const config = {}

// Due to circular typing we have to define an
// explicit typing of state, actions and effects since
// TS 3.9
export interface Config extends IConfig<{
  state: typeof config.state,
  actions: typeof config.actions,
  effects: typeof config.effects
}> {}

export interface OnInitialize extends IOnInitialize<Config> {}

export interface Context extends IContext<Config> {}

export type RootState = Context['state']

/*
  NOTE! These types are typically not needed, but represents
  primitives in Overmind which you might want to pass as
  arguments etc.
*/
export interface Action<Input = void, Output = void> extends IAction<Config, Input, Output> {}

export interface AsyncAction<Input = void, Output = void> extends IAction<Config, Input, Promise<Output>> {}

export interface Operator<Input = void, Output = Input> extends IOperator<Config, Input, Output> {}

export interface Operator<Input = void, Output = Input> extends IOperator<Config, Input, Output> {}

```
{% endtab %}
{% endtabs %}

You only have to set up these types once, where you bring your configuration together. That means if you use multiple namespaced configuration you still only create one set of types, as shown above.

Now you only have to make sure that you import your types from this file, instead of directly from the Overmind package.

{% hint style="info" %}
The Overmind documentation is written for implicit typing. That means whenever you see a type import directly from the Overmind package, you should rather import from your own defined types.
{% endhint %}

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

{% hint style="info" %}
It is important that you use a **type** and not an **interface.** This has to do with the way Overmind resolves the state typing. ****
{% endhint %}

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

Note that with **Explicit Typing** you need to also pass the a third argument to the **derived** function, the **Config** type created in your main **index.ts** file.

{% tabs %}
{% tab title="overmind/state.ts" %}
```typescript
import { RootState } from 'overmind'

type State = {
  foo: string
  shoutedFoo: string
}

export const state: State = {
  foo: 'bar',
  shoutedFoo: derived(
    (state: State, rootState: RootState) => state.foo + '!!!'
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
import { Context } from 'overmind'

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

Operators is like the action: it can take an optional value, but it always produces an output. By default the output of an operator is the same as the input.

{% tabs %}
{% tab title="overmind/operators.ts" %}
```typescript
import { Context, mutate, filter, map } from 'overmind'

// Use the Context type for the first argument
export const changeSomeState = mutate(({ state }: Context) =>  {
  state.foo = 'bar'
})

// Type the value as the second argument
export const filterAwesomeUser = filter((_: Context, user: User) => {
  return user.isAwesome
})

// The output is inferred
export const toNumber = map((_: Context, value: number) => { 
  return Number(value)
})
```
{% endtab %}
{% endtabs %}

When you create a **pipe** that has an input when it is called you only need to type the first operator value.

```typescript
import { Context, pipe, map, mutate } from 'overmind'

export const doThis = pipe(
  map((context: Context, value: string) => {
    // actions.doThis("foo"), requires "string"
    return 123
  }),
  mutate((context: Context, value) => {
    // value is now "number"
  })
```

