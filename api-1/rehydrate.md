# rehydrate

It is possible to update the complete state of Overmind using the **rehydrate** tool. It allows you to update the state either with a state object or an array of mutations, typically collected from server side rendering.

{% tabs %}
{% tab title="overmind/onInitialize.ts" %}
```typescript
import { rehydrate } from 'overmind'

export const onInitialize = ({ state, effects }) => {
  // Grab mutations from a server rendered version
  const mutations = window.__OVERMIND_MUTATIONS

  rehydrate(state, mutations)

  // Grab a previous copy of the state, for example stored in
  // localstorage
  rehydrate(state, effects.storage.get('previousState') || {})
}
```
{% endtab %}
{% endtabs %}

The function takes into acount the state structure of Overmind which is based on objects where functions are derived state. That means it will leave derived state alone, resulting in them rather updating if any of their dependecies where updated by rehydration.

