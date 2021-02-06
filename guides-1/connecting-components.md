# Connecting components

Now that you have defined a state describing your application, you probably want to transform that state into a user interface. There are many ways to express this and Overmind supports the most popular libraries and frameworks for doing this transformation, typically called a view layer. You can also implement a custom view layer if you want to.

By installing the view layer of choice you will be able to connect it to your Overmind instance, exposing its state, actions and effects.

{% tabs %}
{% tab title="React" %}
{% code title="App.jsx" %}
```typescript
import * as React from 'react'
import { useAppState } from '../../overmind'

const App = () => {
  const state = useAppState()

  if (state.isLoading) {
    return <div>Loading app...</div>
  }

  return <h1>My awesome app</h1>
}

export default App
```
{% endcode %}
{% endtab %}

{% tab title="Angular" %}
{% code title="app.component.ts" %}
```typescript
import { Component } from '@angular/core';
import { Store } from '../overmind'

@Component({
  selector: 'app-root',
  template: `
<div *track>
  <div *ngIf="state.isLoading">
    Loading app...
  </div>
  <h1 *ngIf="!state.isLoading">
    My awesome app
  </h1>
</div>
  `
})
export class App {
  state = this.store.select()
  constructor(private store: Store) {}
}
```
{% endcode %}
{% endtab %}

{% tab title="Vue" %}
```typescript
<template>
  <div v-if="state.isLoading">
    Loading app...
  </div>
  <h1 v-else>My awesome app</h1>
</template>
```
{% endtab %}
{% endtabs %}

In this example we are accessing the **isLoading** state. When this component renders and this state is accessed, Overmind will automatically understand that this component is interested in this exact state. It means that whenever the value is changed, this component will render again.

## State

When Overmind detects that the **App** component is interested in our **isLoading** state, it is not looking at the value itself, it is looking at the path. The component is pointed to **state.isLoading** which means that when a mutation occurs on that path in the state, the component will render again. Since the value is a boolean value this can only happen when **isLoading** is replaced or removed. The same goes for strings and numbers as well. We do not say that we mutate a string, boolean or a number. We mutate the object or array that holds those values.

The story is a bit different if the state value is an object or an array. These values can not only be replaced and removed, they can also mutate themselves. An object can have keys added or removed. An array can have items added, removed and even change the order of items. Overmind knows this and will notify components respectively. Let us look at how Overmind treats the following scenarios to get a better understanding.

### Arrays

When we just access an array in a component it will re-render if the array itself is replaced, removed or we do a mutation to it. That would mean we push a new item to it, we splice it or sort it.

{% tabs %}
{% tab title="React" %}
```typescript
import * as React from 'react'
import { useAppState } from '../overmind'

const List = () => {
  const state = useAppState()

  return (
    <h1>{state.items}</h1>
  )
}

export default List
```
{% endtab %}

{% tab title="Angular" %}
```typescript
import { Component } from '@angular/core';
import { Store } from '../overmind'

@Component({
  selector: 'app-list',
  template: `
  <h1 *track>{{state.items}}</h1>
  `
})
export class List {
  state = this.store.select()
  constructor(private store: Store) {}
}
```
{% endtab %}

{% tab title="Vue" %}
```typescript
<template>
  <h1>{{ state.items }}</h1>
</template>
```
{% endtab %}
{% endtabs %}

But what happens if we iterate the array and access a property on each item?

{% tabs %}
{% tab title="React" %}
```typescript
import * as React from 'react'
import { useAppState } from '../overmind'

const List = () => {
  const state = useAppState()

  return (
    <ul>
      {state.items.map(item => 
        <li key={item.id}>{item.title}</li>
      )}
    </ul>
  )
}

export default App
```
{% endtab %}

{% tab title="Angular" %}
```typescript
import { Component } from '@angular/core';
import { Store } from '../overmind'

@Component({
  selector: 'app-list',
  template: `
  <ul>
    <li *ngFor="let item of state.items;trackby: trackById">
      {{item.title}}
    </li>
  </ul>
  `
})
export class List {
  state = this.store.select()
  constructor(private store: Store) {}
  trackById(index, item) {
    return item.id
  }
}
```
{% endtab %}

{% tab title="Vue" %}
```typescript
<template>
  <ul>
    <li v-for="item in state.items" :key="item.id">
      {{ item.title }}
    </li>
  </ul>
</template>
```
{% endtab %}
{% endtabs %}

Now the **List** component looks at the list itself and all the items. Meaning if any items are added/removed or any of the items change, the **List** component will render again. Read more about [**Managing lists**](managing-lists.md) to understand better how to optimize this.

### Objects

Objects are similar to arrays. If you access an object you track if that object is replaced or removed. As with arrays, you can mutate the object itself. When you add, replace or remove a key from the object, it is considered a mutation of the object. It means that if you just access the object, the component will render if any keys are added, replaced or removed.

{% tabs %}
{% tab title="React" %}
```typescript
import * as React from 'react'
import { useAppState } from '../overmind'

const List = () => {
  const state = useAppState()

  return (
    <h1>{state.items}</h1>
  )
}

export default List
```
{% endtab %}

{% tab title="Angular" %}
```typescript
import { Component } from '@angular/core';
import { Store } from '../overmind'

@Component({
  selector: 'app-list',
  template: `
  <h1>{{state.items}}</h1>
  `
})
export class List {
  state = this.store.select()
  constructor(private store: Store) {}
}
```
{% endtab %}

{% tab title="Vue" %}
```typescript
<template>
  <h1>{{ state.items }}</h1>
</template>
```
{% endtab %}
{% endtabs %}

And just like an array you can iterate the values of en object, which results in the component rendering again when any object key value changes:

{% tabs %}
{% tab title="React" %}
{% code title="List.jsx" %}
```typescript
import * as React from 'react'
import { useAppState } from '../overmind'

const List = () => {
  const state = useAppState()

  return (
    <ul>
      {Object.keys(state.items).map(key => 
        <li key={key}>{state.items[key].title}</li>
      )}
    </ul>
  )
}

export default List
```
{% endcode %}
{% endtab %}

{% tab title="Angular" %}
{% code title="list.component.ts" %}
```typescript
import { Component Input } from '@angular/core';
import { Item } from '../overmind/state'

@Component({
  selector: 'app-list-item',
  template: `
  <li *track>
    {{item.title}}
  </li>
  `
})
export class List {
  @Input() item: Item;
}
```
{% endcode %}
{% endtab %}

{% tab title="Vue" %}
{% code title="List.vue" %}
```typescript
<template>
  <ul>
    <li is="Item" v-for="item in state.items" :item="item" :key="item.id" />
  </ul>
</template>
<script>
import Item from './Item'

export default {
  name: 'List',
  components: {
    Item,
  },
}
</script>
```
{% endcode %}
{% endtab %}
{% endtabs %}

Again look at [**Managing lists**](managing-lists.md) to understand better how to optimize lists.

## Actions

All the actions defined in the Overmind application are available to connected components.

{% tabs %}
{% tab title="overmind/actions.js" %}
```typescript
export const toggleAwesomeApp = ({ state }) => {
  state.isAwesome = !state.isAwesome
}
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="React" %}
```typescript
import * as React from 'react'
import { useActions } from '../overmind'

const App = () => {
  const actions = useActions()

  return (
    <button onClick={actions.toggleAwesomeApp}>
      Toggle awesome
    </button>
  )
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
  <button (click)="actions.toggleAwesomeApp()">
    Toggle awesome
  </button>
  `
})
export class App {
  actions = this.store.actions
  constructor(private store: Store) {}
}
```
{% endtab %}

{% tab title="Vue" %}
```typescript
<template>
  <button @click="actions.toggleAwesomeApp()">
    Toggle awesome
  </button>
</template>
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
If you need to pass multiple values to an action, you should rather use an **object** instead.
{% endhint %}

## Reactions

Sometimes you want to make something happen inside a component related to a state change. This is typically doing some manual work on the DOM. When you connect a component to Overmind it also gets access to **reaction**. This function allows you to subscribe to changes in state, mutations as we call them. 

This example shows how you can scroll to the top of the page every time you change the current article of the app.

{% tabs %}
{% tab title="React" %}
```typescript
import * as React from 'react'
import { useReaction } from '../../overmind'

const Article = () => {
  const reaction = useReaction()

  React.useEffect(() => reaction(
    (state) => state.currentArticle,
    () => {
      document.querySelector('#app').scrollTop = 0
    } 
  ), [])

  return <article />
}

export default Article
```
{% endtab %}

{% tab title="Angular" %}
```typescript
import { Component } from '@angular/core';
import { Store } from '../overmind'

@Component({
  selector: 'app-root',
  template: `
  <article></article>
  `
})
export class App {
  disposeEffect: () => void
  constructor(private store: Store) {}
  ngOnInit() {
    this.disposeReaction = this.store.reaction(
      (state) => state.currentArticle,
      () => document.querySelector('#app').scrollTop = 0   
    )
  }
  ngOnDestroy() {
    this.disposeReaction()
  }
}
```
{% endtab %}

{% tab title="Vue" %}
```typescript
<template>
  <article></article>
</template>
<script>
export default {
  name: 'Article',
  mounted() {
    this.disposeReaction = this.overmind.reaction(
      (state) => state.currentArticle,
      () => document.querySelector('#app').scrollTop = 0   
    })
  }
  destroyed() {
    this.disposeReaction()
  }
}
</script>
```
{% endtab %}
{% endtabs %}

## Effects

Any effects you define in your Overmind application are also exposed to the components. They can be found on the property **effects**. It is encouraged that you keep your logic inside actions, but you might be in a situation where you want some other relationship between components and Overmind. A shared effect is the way to go.

