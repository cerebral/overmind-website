# Operators

You get very far building your application with straightforward imperative actions. This is typically how we learn programming and is arguably close to how we think about the world. But this approach can cause you to overload your actions with imperative code, making them more difficult to read and especially reuse pieces of logic. As the complexity of your application increases you will find benefits to doing some of your logic, or maybe all your logic, in a functional style.

Let us look at a concrete example of how messy an imperative approach would be compared to a functional approach.

{% tabs %}
{% tab title="overmind/actions.ts" %}
```typescript
import { Action } from 'overmind'

let debounce
export const search: Action<Event> = ({ state, effects }, event) => {
  state.query = event.currentTarget.value

  if (query.length < 3) return

  if (debounce) clearTimeout(debounce)

  debounce = setTimeout(async () => {
    state.isSearching = true
    state.searchResult = await effects.api.search(state.query)
    state.isSearching = false

    debounce = null
  }, 200)
}
```
{% endtab %}
{% endtabs %}

What we see here is an action trying to express a search. We only want to search when the length of the query is more than 2 characters and we only want to trigger the search when the user has not changed the query for 200 milliseconds.

If we were to do this in a functional style it would look more like this:

{% tabs %}
{% tab title="overmind/actions.ts" %}
```typescript
import { Operator, pipe, debounce, mutate, filter } from 'overmind'

export const search: Operator<string> = pipe(
  mutate(({ state }, value) => {
    state.query = value
  }),
  filter(({ state }) => state.query.length > 2),
  debounce(200),
  mutate(({ state, effects }) => {
    state.isSearching = true
    state.searchResult = await effects.api.search(state.query)
    state.isSearching = false
  })
)
```
{% endtab %}
{% endtabs %}

As you can see our action is described more declaratively. We could have moved each individual piece of logic, each operator, into a different file. All these operators could now be reused in other action compositions.

## Structuring operators

You will typically rely on an **operators** file where all your composable pieces live. Inside your **actions** file you expose the operators and compose them together using **pipe** and other _composing_ operators. This approach ensures:

1. Each operator is defined separately and in isolation
2. The action composed of operators is defined with the other actions
3. The action composed of operators is declarative \(no inline operators with logic\)

Let us look at how the operators in the search example could have been implemented:

{% tabs %}
{% tab title="overmind/operators.ts" %}
```typescript
import { Operator, filter, mutate } from 'overmind'

export const setQuery: () => Operator<string> = () =>
  mutate(function setQuery({ state }, query) {
    state.query = query
  })

export const lengthGreaterThan: (length: number) => Operator<string> = (length) =>
  filter(function lengthGreaterThan(_, value) {
    return value.length > length
  })

export const getSearchResult: () => Operator<string> = () => 
  mutate(async function getSearchResult({ state, effects }, query) {
    state.isSearching = true
    state.searchResult = await effects.api.search(query)
    state.isSearching = false
  })
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Note that we give all the actual operator functions the same name as the exported variable that creates it. The reason is that this name is picked up by the devtools and gives you more insight into how your code runs.
{% endhint %}



You might wonder why we define the operators as functions that we call. We do that for the following reasons:

1. It ensures that each composition using the operator has a unique instance of that operator. For most operators this does not matter, but for others like **debounce** it actually matters.
2. Some operators require options, like the **lengthGreaterThan** operator we created above. Defining all operators as functions just makes things more consistent.
3. If you were to create an operator that is composed of other operators you can safely do so without thinking about the order of definition in the _operators_ file. The reason being that the operator is lazily created
4. With Typescript it opens up to partial and generic typed operators. Read more about this in the [Typescript](typescript.md) guide

Now, you might feel that we are just adding complexity here. An additional file with more syntax. But clean and maintainable code is not about less syntax. It is about structure, predictability and reusability. What we achieve with this functional approach is a super readable abstraction in our _actions_ file. There is no logic there, just references to logic. In our _operators_ file each piece of logic is defined in isolation with very little logic.

## Calling operators

You typically compose the different operators together with **pipe** and **parallel** in the _actions_ file, but any operator can actually be exposed as an action. With the search example:

{% tabs %}
{% tab title="overmind/actions.ts" %}
```typescript
import { Operator, pipe, debounce, mutate, filter } from 'overmind'

export const search: Operator<string> = pipe(
  mutate(({ state }, value) => {
    state.query = value
  }),
  filter(({ state }) => state.query.length > 2),
  debounce(200),
  mutate(({ state, effects }) => {
    state.isSearching = true
    state.searchResult = await effects.api.search(state.query)
    state.isSearching = false
  })
)
```
{% endtab %}
{% endtabs %}

You would call this action like any other:

```typescript
overmind.actions.search("something")
```

## Inputs and Outputs

To produce new values throughout your pipe you can use the **map** operator. It will put whatever value you return from it on the pipe for the next operator to consume.

{% tabs %}
{% tab title="overmind/operators.ts" %}
```typescript
import { Operator, map, mutate } from 'overmind'

export const toNumber: () => Operator<string, number> = () =>
  map(function toNumber(_, value) { 
    return Number(value)
  })

export const setValue: () => Operator<string> = () =>
  mutate(function setValue({ state}, value) {
    state.value = value
  })
```
{% endtab %}

{% tab title="overmind/actions.ts" %}
```typescript
import { Operator, pipe } from 'overmind'
import * as o from './operators'

export const onValueChange: Operator<string, number> = pipe(
  o.toNumber(),
  o.setValue()
)
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Notice here that we are typing both the input and the output. Both for the **map** operator and the **pipe** operator. This is important to manage the composition of operators. Typically operators pass on the same value they receive, meaning that you do not need the second typing parameter for the **Operator** type.
{% endhint %}

## Custom operators

The operators concept of Overmind is based on the [OP-OP SPEC](https://github.com/christianalfoni/op-op-spec), which allows for endless possibilities in functional composition. But since Overmind does not only pass values through these operators, but also the context where you can change state, run effects etc., we want to simplify how you can create your own operators. The added benefit of this is that the operators you create are also tracked in the devtools.

### toUpperCase <a id="create-custom-operators-touppercase"></a>

Let us create an operator that simply uppercases the string value passed through. This could easily have been done using the **map** operator, but for educational purposes let us see how we can create our very own operator.

{% tabs %}
{% tab title="overmind/operators.ts" %}
```typescript
import { Operator, createOperator, mutate } from 'overmind'

export const toUpperCase: () => Operator<string> = () => {
  return createOperator('toUpperCase', '', (err, context, value, next) => {
    if (err) next(err, value)
    else next(null, value.toUpperCase())
  })
}

export const setTitle: Operator<string> = mutate(({ state }, title) => {
  state.title = title
})
```
{% endtab %}

{% tab title="overmind/actions.ts" %}
```typescript
import { Operator, pipe } from 'overmind'
import { toUpperCase, setTitle } from './operators'

export const setUpperCaseTitle: Operator<string> = pipe(
  toUpperCase(),
  setTitle
)
```
{% endtab %}
{% endtabs %}

We first create a function that returns an operator when we call it. We pass this operator a **name**, an optional **description** and the callback that is executed when the operator runs. This operator might receive an **error**, that you can handle if you want to. It also receives the **context**, the current **value** and a function called **next**.

In this example we did not use the **context** because we are not going to look at any state, run effects etc. We just wanted to change the value passed through. All operators need to handle the **error** in some way. In this case we just pass it along to the next operator by calling **next** with the error as the first argument and the current value as the second. When there is no error it means we can manage our value and we do so by calling **next** again, but passing **null** as the first argument, as there is no error. And the second argument is the new **value**.

### operations <a id="create-custom-operators-operations"></a>

You might want to run some logic related to your operator. Typically this is done by giving a callback. You can provide this callback whatever information you want, even handle its return value. So for example the **map** operator is implemented like this:

```typescript
import { Operator, Context, Config, createOperator } from 'overmind'

export function map<Input, Output>(
  operation: (context: Context, value: Input) => Output
): Operator<Input, Output extends Promise<infer U> ? U : Output> {
  return createOperator<Config>(
    'map',
    operation.name,
    (err, context, value, next) => {
      if (err) next(err, value)
      else next(null, operation(context, value))
    }
  )
}
```

### mutations <a id="create-custom-operators-mutations"></a>

You can also create operators that have the ability to mutate the state, it is just a different factory **createMutationFactory**. This is how the **mutate** operator is implemented:

```typescript
import { Operator, Context, Config, createMutationOperator } from 'overmind'

export function mutate<Input>(
  operation: (context: Context, value: Input) => void
): Operator<Input> {
  return createMutationOperator<Config>(
    'mutate',
    operation.name,
    (err, context, value, next) => {
      if (err) next(err, value)
      else {
        operation(context, value)
        next(null, value)
      }
    }
  )
}
```

### paths <a id="create-custom-operators-paths"></a>

You can even manage paths in your operator. This is how the **when** operator is implemented:

```typescript
import { Operator, Context, Config, createOperator } from 'overmind'

export function when<
  Input,
  OutputTrue,
  OutputFalse
>(
  operation: (context: Context, value: Input) => boolean,
  paths: {
    true: Operator<Input, OutputTrue>
    false: Operator<Input, OutputFalse>
  }
): Operator<Input, OutputTrue | OutputFalse> {
  return createOperator<Config>(
    'when',
    operation.name,
    (err, context, value, next) => {
      if (err) next(err, value)
      else if (operation(context, value))
        next(null, value, {
          name: 'true',
          operator: paths.true,
        })
      else
        next(null, value, {
          name: 'false',
          operator: paths.false,
        })
    }
  )
}
```

### aborting <a id="create-custom-operators-aborting"></a>

Some operators want to prevent further execution. That is also possible to implement, as seen here with the **filter** operator:

```typescript
import { Operator, Context, Config, createOperator } from 'overmind'

export function filter<Input>(
  operation: (context: Context, value: Input) => boolean
): Operator<Input> {
  return createOperator<Config>(
    'filter',
    operation.name,
    (err, context, value, next, final) => {
      if (err) next(err, value)
      else if (operation(context, value)) next(null, value)
      else final(null, value)
    }
  )
}
```

The **final** argument bypasses any other operators.

