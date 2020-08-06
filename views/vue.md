# Vue

## Install

```text
npm install overmind overmind-vue
```

There are two approaches to connecting Overmind to Vue.

## Plugin

Vue has a plugin system that allows us to expose Overmind to all components. This allows minimum configuration and you just use state etc. from any component.

{% tabs %}
{% tab title="overmind/index.js" %}
```typescript
import { createOvermind } from 'overmind'
import { createPlugin } from 'overmind-vue'

const overmind = createOvermind({
  state: {
    foo: 'bar'
  },
  actions: {
    onClick() {}
  }
})

export const OvermindPlugin = createPlugin(overmind)
```
{% endtab %}

{% tab title="index.js" %}
```typescript
import Vue from 'vue/dist/vue'
import { OvermindPlugin } from './overmind'

Vue.use(OvermindPlugin)

...
```
{% endtab %}

{% tab title="components/SomeComponent.vue" %}
```typescript
<template>
  <div @click="actions.onClick">
    {{ state.foo }}
  </div>
</template>
```
{% endtab %}
{% endtabs %}

If you rather want to expose state, actions and effects differently you can configure that.

{% tabs %}
{% tab title="index.js" %}
```typescript
import Vue from 'vue/dist/vue'
import { OvermindPlugin } from './overmind'

Vue.use(OvermindPlugin, ({ state, actions, effects }) => ({
  admin: state.admin,
  posts: state.posts,
  actions,
  effects
}))

...
```
{% endtab %}

{% tab title="components/SomeComponent.vue" %}
```typescript
<template>
  <div @click="actions.onClick">
    {{ admin.foo }} {{ posts.foo }}
  </div>
</template>
```
{% endtab %}
{% endtabs %}

### Rendering

Any state accessed in the component will cause the component to render when a mutation occurs on that state. Overmind actually uses the same approach to change detection as Vue itself. When using the plugin any component can access any state, though the only overhead that is added to the application is an instance of a “tracking tree” per component. This might sound scary, but it is a tiny little object that adds a callback function to Overmind as long as the component lives. These tracking trees are even reused as components unmount.

### Pass state as props

If you pass anything from the state to a child component it will just work out of the box. The child component will “rescope” the property to its own tracking tree. This ensures that the property you passed is tracked within that component.

{% tabs %}
{% tab title="components/Todo.vue" %}
```typescript
<template>
  <li>{{ todo.title }}</li>
</template>
<script>
export default {
  name: 'Todo',
  props: ["todo"]
}
</script>
```
{% endtab %}

{% tab title="components/Todos.vue" %}
```typescript
<template>
  <ul>
    <todo-component
      v-for="post in state.postsList"
      :todo="todo"
      :key="todo.id"
    ></todo-component>
  </ul>
</template>
<script>
import TodoComponent from './Todo'

export default {
  name: 'Todo',
  components: {
    TodoComponent
  }
}
</script>
```
{% endtab %}
{% endtabs %}

### Reactions

To run effects in components based on changes to state you use the **reaction** function in the lifecycle hooks of Vue.

{% tabs %}
{% tab title="components/SomeComponent.vue" %}
```typescript
<template>
  <div @click="overmind.actions.onClick">
    {{ overmind.state.foo }}
  </div>
</template>
<script>
import { connect } from '../overmind'

export default connect({
  mounted() {
    this.disposeReaction = this.overmind.reaction(
      ({ currentPage }) => currentPage,
      () => document.querySelector('#app').scrollTop = 0
    )
  },
  destroyed() {
    this.disposeReaction()
  }
})
</script>
```
{% endtab %}
{% endtabs %}

## Connect

If you want more manual control of what components connect to Overmind you can use the connector.

{% tabs %}
{% tab title="overmind/index.js" %}
```typescript
import { createOvermind } from 'overmind'
import { createConnect } from 'overmind-vue'

const overmind = createOvermind({
  state: {},
  actions: {}
})

export const connect = createConnect(overmind)
```
{% endtab %}

{% tab title="components/SomeComponent.vue" %}
```typescript
<template>
  <div @click="overmind.actions.onClick">
    {{ overmind.state.foo }}
  </div>
</template>
<script>
import { connect } from '../overmind'

const Component = {}

export default connect(Component)
</script>
```
{% endtab %}
{% endtabs %}

You can also expose parts of the configuration on custom properties of the component:

{% tabs %}
{% tab title="components/SomeComponent.vue" %}
```typescript
<template>
  <div @click="actions.someAdminAction">
    {{ state.someAdminState }}
  </div>
</template>
<script>
import { connect } from '../overmind'

const Component = {}

export default connect(({ state, actions, effects }) => ({
  state: state.admin,
  actions: actions.admin
}), Component)
</script>
```
{% endtab %}
{% endtabs %}

You can now access the **admin** state and actions directly with **state** and **actions**.

## Computed

Vue has its own observable concept that differs from Overmind. That means you can not use Overmind state inside a computed and expect the computed cache to be busted when the Overmind state changes. But computeds are really for caching expensive computation, which you will rather do inside Overmind using **derived** anyways.

## Using props

You can combine Overmind state with props to dynamically extract state.

{% tabs %}
{% tab title="components/SomeComponent.vue" %}
```typescript
<template>
  <div>
    {{ title }}
  </div>
</template>
<script>
export default {
  name: 'SomeComponent',
  props: ["id"],
  data: (self) => ({
    get title() {
      return self.state.titles[self.id]
    }
  })
}
</script>
```
{% endtab %}
{% endtabs %}

