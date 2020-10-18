# Svelte

There are two differente ways to connect Overmind to Svelte. You can use the **reactive declarations** or you can use the **reaction**.

When you connect Overmind to a component you ensure that whenever any tracked state changes, only components interested in that state will re-render.

## Reactive declarations

```javascript
import { createOvermind } from 'overmind'
import { createMixin } from 'overmind-svelte'

const overmind = {
  state: {
    count: 0
  },
  actions: {
    increase({ state }) {
      state.count++;
    },
    decrease({ state }) {
      state.count--;
    }
  }
}

const store = createMixin(createOvermind(overmind))

export const state = store.state
export const actions = store.actions
```

```javascript
<script>
  import { state, actions } from './overmind.js'

  $: count = $state.count
</script>

<p>Count: {count}</p>
<button id="increase" on:click={() => store.actions.increase()}>Increase</button>
<button id="decrease" on:click={() => store.actions.decrease()}>Increase</button>
```

## Reactions

```javascript
import { createOvermind } from 'overmind'
import { createMixin } from 'overmind-svelte'

const overmind = {
  state: {
    count: 0
  },
  actions: {
    increase({ state }) {
      state.count++;
    },
    decrease({ state }) {
      state.count--;
    }
  }
}

const store = createMixin(createOvermind(overmind))

export const state = store.state
export const actions = store.actions
export const reactions = store.reactions
```

```javascript
<script>
  import { state, actions, reactions } from './overmind.js'

  export let store

  $: count = $state.count

  let doubled = undefined

  store.reaction(
    (state) => state.count,
    (value) => {
        doubled = value * 2
    },
    {
        immediate: true
    }
  )
</script>

<p>Count: {count}</p>
<p>Doubled: {doubled}</p>
<button id="increase" on:click={() => store.actions.increase()}>Increase</button>
<button id="decrease" on:click={() => store.actions.decrease()}>Increase</button>
```

