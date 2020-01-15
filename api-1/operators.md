# operators

Overmind also provides a functional API to help you manage complex logic. This API is inspired by asynchronous flow libraries like RxJS, though it is designed to manage application state and effects. If you want to create and use a traditional action “operator style” you define it like this:

```typescript
import { Operator, mutate } from 'overmind'

export const changeFoo: <T>() => Operator<T> = () =>
  mutate(({ state }) => {
    state.foo = 'bar'
  })
```

The **mutate** function is one of many operators. Operators are small composable pieces of logic that can be combined in many ways. This allows you to express complexity in a declarative way. You typically use the **pipe** operator in combination with the other operators to do this:

```typescript
import { Operator, pipe, debounce } from 'overmind'
import { QueryResult } from './state'
import * as o from './operators'

export const search: Operator<string> = pipe(
  o.setQuery(),
  o.filterValidQuery(),
  debounce(200),
  o.queryResult()
)
```

Any of these operators can be used with other operators. You can even insert a pipe inside an other pipe. This kind of composition is what makes functional programming so powerful.

## catchError

**async**

This operator runs if any of the previous operators throws an error. It allows you to manage that error by changing your state, run effects or even return a new value to the next operators.

```typescript
import { Operator, pipe, mutate, catchError } from 'overmind'

export const doSomething: Operator<string> = pipe(
  mutate(() => {
    throw new Error('foo')
  }),
  mutate(() => {
    // This one is skipped
  })
  catchError(({ state }, error) => {
    state.error = error.message

    return 'value_to_be_passed_on'
  }),
  mutate(() => {
    // This one continues executing with replaced value
  })
)
```

## debounce

When action is called multiple times within the set time limit, only the last action will move beyond the point of the debounce.

```typescript
import { Operator, pipe, debounce } from 'overmind'
import * as o from './operators'

export const search: Operator<string> = pipe(
  debounce(200),
  o.performSearch()
)
```

## filter

Stop execution if it returns false.

```typescript
import { Operator, filter } from 'overmind'

export const lengthGreaterThan: (length: number) => Operator<string> = (length) =>
  filter(function lengthGreaterThan(_, value) {
    return value.length > length
  })
```

## forEach

Allows you to pass each item of a value that is an array to the operator/pipe on the second argument.

```typescript
import { Operator, pipe, forEach } from 'overmind'
import { Post } from './state'
import * as o from './operators'

export const openPosts: Operator<string, Post[]> = pipe(
  o.getPosts(),
  forEach(o.getAuthor())
)
```

## fork

Allows you to execute an operator/pipe based on the matching key.

{% tabs %}
{% tab title="overmind/operators.ts" %}
```typescript
import { fork, Operator } from 'overmind'
import { User } from './state'

export const forkUserType: (paths: { [key: string]: Operator<User> }) => Operator<User> = (paths) =>
  fork(function forkUserType(_, user) {
    return user.type
  }, paths)
```
{% endtab %}

{% tab title="overmind/actions.ts" %}
```typescript
import { Operator, pipe } from 'overmind'
import * as o from './operators'
import { UserType } from './state'

export const getUser: Operator<string, User> = pipe(
  o.getUser(),
  o.forkUserType({
    [UserType.ADMIN]: o.doThis(),
    [UserType.SUPERUSER]: o.doThat()
  })
)
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
You have to use **ENUM** for these keys
{% endhint %}

## map

Returns a new value to the pipe. If the value is a promise it will wait until promise is resolved.

```typescript
import { Operator, map } from 'overmind'
import { User } from './state'

export const getEventTargetValue: () => Operator<Event, string> = () =>
  map(function getEventTargetValue(_, event) {
    return event.currentTarget.value
  })
```

## mutate

**async**

You use this operator whenever you want to change the state of the app, but you can run effects as well. Any returned value is ignored.

```typescript
import { Operator, mutate } from 'overmind'

export const setUser: () => Operator<User> = () =>
  mutate(function setUser({ state }, user) {
    state.user = user
  })
```

## noop

This operator does absolutely nothing. Is useful when paths of execution is not supposed to do anything.

```typescript
import { Operator } from 'overmind'
import * as o from './operators'

export const doSomething: Operator = o.forkUserType({
  superuser: o.doThis(),
  admin: o.doThat(),
  other: o.noop()
})
```

## parallel

Will run every operator and wait for all of them to finish before moving on. Works like _Promise.all_.

```typescript
import { Operator, parallel } from 'overmind'
import * as o from './operators'

export const loadAllData: Operator = parallel(
  o.loadSomeData(),
  o.loadSomeMoreData()
)
```

## pipe

The pipe is an operator in itself. Use it to compose other operators and pipes.

```typescript
import { Operator, pipe } from 'overmind'
import { Item } from './state'
import * as o from './operators'

export const openItem: Operator<string, Item> = pipe(
  o.openItemsWhichIsAPipeOperator(),
  o.getItem()
)
```

## run

This operator allows you to run side effects. You can not change state and you can not return a value.

```typescript
import { Operator, run } from 'overmind'

export const doSomething: <T>() => Operator<T> = () =>
  run(function doSomething({ effects }) {
    effects.someApi.doSomething()
  })
```

## throttle

This operator allows you to ensure that if an action is called, the next action will only continue past this point if a certain duration has passed. Typically used when an action is called many times in a short amount of time.

```typescript
import { Operator, pipe, throttle } from 'overmind'
import * as o from './operators'

export const onMouseDrag: Operator<string> = pipe(
  throttle(200),
  o.handleMouseDrag()
)
```

## tryCatch

This operator allows you to scope execution and manage errors. This operator does not return a new value to the execution.

```typescript
import { Operator, pipe, tryCatch } from 'overmind'
import * as o from './operators'

export const doSomething: Operator<string> = tryCatch({
  try: o.somethingThatMightError(),
  catch: o.somethingToHandleTheError()
})
```

## wait

Hold execution for set time.

```typescript
import { Operator, pipe, wait } from 'overmind'
import * as o from './operators'

export const search: Operator<string> = pipe(
  wait(2000),
  o.executeSomething()
)
```

## waitUntil

Wait until a state condition is true.

```typescript
import { Operator, pipe, waitUntil } from 'overmind'
import * as o from './operators'

export const search: Operator<string> = pipe(
  waitUntil(state => state.count === 3),
  o.executeSomething()
)
```

## when

Go down the true or false path based on the returned value.

```typescript
import { Operator, when } from 'overmind'
import { User } from './state'

export const whenUserIsAwesome: (paths: { true: Operator<User>, false: Operator<User> }) => Operator<User> = (paths) => 
  when(function whenUserIsAwesome(_, user) {
    return user.isAwesome
  }, paths)
```

