# Actions

Overmind has a concept of an **action**. An action is just a method where the first argument is injected. This first argument is called the **context** and it holds the state of the application, whatever effects you have defined and references to the other actions.

You define actions under the **actions** key of your application configuration.

{% tabs %}
{% tab title="overmind/actions.ts" %}
```typescript
import { Action } from 'overmind'

export const myAction: Action = (context) => {

}
```
{% endtab %}

{% tab title="overmind/index.ts" %}
```typescript
import { IConfig } from 'overmind'
import * as actions from './actions'

export const config = {
  actions
}

// For explicit typing check the Typescript guide
declare module 'overmind' {
  interface Config extends IConfig<typeof config> {}
}
```
{% endtab %}
{% endtabs %}

## Using the context

The context has three parts: **state**, **effects** and **actions**. Typically you destructure the context to access these pieces directly:

{% code title="overmind/actions.ts" %}
```typescript
import { Action } from 'overmind'

export const myAction: Action = ({ state, effects, actions }) => {

}
```
{% endcode %}

When you point to either of these you will always point to the “top of the application. That means if you use namespaces or other nested structures the context is always the root context of the application.

{% hint style="info" %}
The reason Overmind only has a root context is because having isolated contexts/domains creates more harm than good. Specifically when you develop your application it is very difficult to know exactly how the domains of your application will look like and what state, actions and effects belong together. By only having a root context you can always point to any domain from any other domain allowing you to easily manage cross-domain logic, not having to refactor every time your domain model breaks.
{% endhint %}

## Passing values

When you call actions you can pass a single value. This value appears as the second argument, after the context.

{% code title="overmind/actions.ts" %}
```typescript
import { Action } from 'overmind'

export const myAction: Action = ({ state, effects, actions }, value) => {

}
```
{% endcode %}

When you call an action from an action you do so by using the **actions** passed on the context, as this is the evaluated action that can be called.

{% code title="overmind/actions.ts" %}
```typescript
import { Action } from 'overmind'

export const myAction: Action = ({ state, effects, actions }) => {
  actions.myOtherAction('foo')
}

export const myOtherAction: Action<string> = ({ state, effects, actions }, value) {

}
```
{% endcode %}

## Organizing actions

Some of your actions will be called from the outside, publically, maybe from a component. Other actions are only used internally, either being passed to an effect or just holding some piece of logic you want to reuse. The convention to separate these two actions is to use a namespace of **internal**.

{% tabs %}
{% tab title="overmind/internalActions.ts" %}
```typescript
import { Action, AsyncAction } from 'overmind'

export const internalActionA: Action<string> = ({ state, effects, actions }, value) {}

export const internalActionB: AsyncAction = async ({ state, effects, actions }) {}
```
{% endtab %}

{% tab title="overmind/actions.ts" %}
```typescript
import { Action } from 'overmind'
import * as internalActions from './internalActions'

export const internal = internalActions

export const myAction: Action = ({ state, effects, actions }) => {
  actions.internal.internalActionA('foo')
  actions.internal.internalActionB()
}
```
{% endtab %}
{% endtabs %}

## Summary

The action in Overmind is a powerful concept. It allows you to define and organize logic that always has access to the core components of your application. State, effects and actions.

