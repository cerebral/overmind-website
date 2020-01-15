# reaction

Sometimes you need to react to changes to state. Typically you want to run some imperative logic related to something in the state changing.

```typescript
reaction(
  // Access and return some state to react to
  (state) => state.foo,

  // Do something with the returned value
  (foo) => {},

  {
    // If you return an object or array from the state you can set this to true.
    // The reaction will run when any nested changes occur as well
    nested: false,

    // Runs the reaction immediately
    immediate: false
  }
)
```

There are two points of setting up reactions in Overmind.

## onInitialize

The onInitialize hook is where you set up reactions that lives throughout your application lifetime. The reaction function returns a function to dispose it. That means you can give effects the possibility to create and dispose of reactions in any action.

{% tabs %}
{% tab title="overmind/onInitialize.ts" %}
```typescript
import { OnInitialize } from 'overmind'

export const onInitialize: OnInitialize = ({ effects }, instance) => {
  instance.reaction(
    ({ todos }) => todos,
    (todos) => effects.storage.saveTodos(todos),
    {
      nested: true
    }
  )
}
```
{% endtab %}
{% endtabs %}

## components

With components you typically use reactions to manipulate DOM elements or other UI related imperative libraries.

{% tabs %}
{% tab title="React" %}
```typescript
import * as React from 'react'
import { useOvermind } from '../overmind'

const App: React.FC = () => {
  const { reaction } = useOvermind()

  React.useEffect(() => reaction(
    ({ currentPage }) => currentPage,
    () => {
      document.querySelector('#page').scrollTop = 0
    }
  ))

  return <div id="page"></div>
}

export default App
```
{% endtab %}

{% tab title="Angular" %}
```typescript
import { Component } from '@angular/core';
import { Store } from '../overmind'

@Component({
  selector: 'app-root',
  template: `
  <div id="page"></div>
  `
})
export class App {
  constructor(private store: Store) {}
  ngOnInit() {
    this.store.reaction(
      ({ currentPage }) => currentPage,
      () => {
        document.querySelector('#page').scrollTop = 0
      }    
    )
  }
}
```
{% endtab %}

{% tab title="Vue" %}
```typescript
<template>
  <div id="page"></div>
</template>
<script>
export default {
  mounted() {
    this.overmind.reaction(
      ({ currentPage }) => currentPage,
      () => {
        document.querySelector('#page').scrollTop = 0
      }     
    )
  }
}
</script>
```
{% endtab %}
{% endtabs %}



