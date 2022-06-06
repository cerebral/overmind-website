# operators

Overmind also provides a functional API to help you manage complex logic. This API is inspired by asynchronous flow libraries like RxJS, though it is designed to manage application state and effects. Operators are actually interoperable with plain actions, meaning you can define an operator just like an action:

```typescript
export const changeFoo= ({ state }) => {
  state.foo = 'bar'
}
```

{% hint style="info" %}
The reason operators by convention are defined as factories is because it makes them consistent in their declarative representation. 
{% endhint %}

Operators are small composable pieces of logic that can be combined in many ways. This allows you to express complexity in a declarative way. You typically use the **pipe** operator in combination with the other operators to do this. A typical convention is to define operators in their own **operators** file where the **actions** file imports and composes them together:

```typescript
import { pipe, debounce } from 'overmind'
import * as o from './operators'

export const search = pipe(
  o.setQuery,
  o.filterValidQuery,
  debounce(200),
  o.queryResult
)
```

Any of these operators can be used with other operators. You can even insert a pipe inside an other pipe. This kind of composition is what makes functional programming so powerful.

## branch

This operator works just like **pipe**, but branches out the execution and does not bring its input as output of the branch.

```typescript
import { pipe, branch } from 'overmind'

export const doSomething = pipe(
  () => 123,
  branch(
    (_, value) => String(value),
    (_, value) => `Hello ${value}`,
    (_, value) =>{
      value // "Hello 123"
    }
  ),
  (_, value) => {
    value // 123
  }
)
```

## catchError

**async**

This operator runs if any of the previous operators throws an error. It allows you to manage that error by changing your state, run effects or even return a new value to the next operators.

```typescript
import { pipe, catchError } from 'overmind'

export const doSomething = pipe(
  () => {
    throw new Error('foo')
  },
  () => {
    // This one is skipped
  },
  catchError(({ state }, error) => {
    state.error = error.message

    return 'value_to_be_passed_on'
  }),
  () => {
    // This one continues executing with replaced value
  }
)
```

## debounce

When action is called multiple times within the set time limit, only the last action will move beyond the point of the debounce.

```typescript
import { pipe, debounce } from 'overmind'

export const search = pipe(
  debounce(200),
  () => {
    // Executes last action call, when no new action call has
    // been made within 200ms
  }
)
```

## filter

Stop execution if it returns false.

```typescript
import { pipe, filter } from 'overmind'

export const doSomething = pipe(
  filter((_, value) => value.length <= 3)
)
```

## fork

Allows you to execute an operator/pipe based on the matching key.

{% tabs %}
{% tab title="overmind/operators.js" %}
```typescript
import { fork } from 'overmind'

export const forkUserType = (paths) =>
  fork((_, user) => user.type, paths)
```
{% endtab %}

{% tab title="overmind/actions.js" %}
```typescript
import { pipe, fork } from 'overmind'

export const getUser = pipe(
  ({ effects }) => effects.api.getUser(),
  forkUserType({
    'admin': o.doThis,
    'superuser': o.doThat
  })
)
```
{% endtab %}
{% endtabs %}

## noop

This operator does absolutely nothing. Is useful when paths of execution is not supposed to do anything.

```typescript
import { fork, noop } from 'overmind'

export const doSomething = fork((, user) => user.type, {
  superuser: () => {},
  admin: () => {},
  other: noop
})
```

## parallel

Will run every operator and wait for all of them to finish before moving on. Works like _Promise.all_.

```typescript
import { pipe, parallel } from 'overmind'

export const loadAllData = pipe(
  parallel(
    () => {
      return someData
    },
    () => {
      return someOtherData
    }
  ),
  (_, arrayOfResults) => {}
)
```

## pipe

The pipe is an operator in itself. Use it to compose other operators and pipes.

```typescript
import { pipe } from 'overmind'

export const openItem = pipe(
  () => {},
  () => {}
)
```

## throttle

This operator allows you to ensure that if an action is called, the next action will only continue past this point if a certain duration has passed. Typically used when an action is called many times in a short amount of time.

```typescript
import { pipe, throttle } from 'overmind'

export const onMouseDrag = pipe(
  throttle(200),
  () => {
    // Executes only when at least 200ms has passed since
    // last action call
  }
)
```

## tryCatch

This operator allows you to scope execution and manage errors. This operator does not return a new value to the execution.

```typescript
import { pipe, tryCatch } from 'overmind'

export const doSomething = tryCatch({
  try: () => {},
  catch: () => {}
})
```

## wait

Hold execution for set time.

```typescript
import { pipe, wait } from 'overmind'
import * as o from './operators'

export const search = pipe(
  wait(2000),
  () => {
    // Executes after 2 seconds
  }
)
```

## waitUntil

Wait until a state condition is true.

```typescript
import { pipe, waitUntil } from 'overmind'

export const search = pipe(
  waitUntil(state => state.count === 3),
  () => {
    // Executes when state.count is set with value 3
  }
)
```

## when

Go down the true or false path based on the returned value.

```typescript
import { when } from 'overmind'

export const whenUserIsAwesome = when(
  (_, user) => user.isAwesome,
  {
    true: () => {},
    false: () => {}
  }
)
```

