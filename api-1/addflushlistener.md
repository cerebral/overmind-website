# addFlushListener

The **addMutationListener** triggers whenever there is a mutation. The **addFlushListener** triggers whenever Overmind tells components to render again. It can have multiple mutations related to it.

{% tabs %}
{% tab title="overmind/onInitialize.ts" %}
```typescript
import { OnInitialize } from 'overmind'

const onInitialize: OnInitialize = async ({ state, effects }, overmind) => {
  overmind.addFlushListener(effects.history.addMutations)
}

export default onInitialize
```
{% endtab %}
{% endtabs %}

