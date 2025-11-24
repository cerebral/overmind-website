# namespaced

Allows you to namespace configurations by a key.

The point of namespaces is to structure your code into domains, not isolate them. The reason being is that we more often than not design our namespaces wrong. We have no idea how the final app will look and getting into the issue of "cross domain logic and state" is a pain to refactor all the time due to wrong isolation.

So in Overmind isolation is a discipline, not a technical restriction.

{% tabs %}
{% tab title="overmind/index.ts" %}

```typescript
import { namespaced } from 'overmind/config'
import * as moduleA from './moduleA'
import * as moduleB from './moduleB'

export const config = namespaced({
  moduleA,
  moduleB
})
```

{% endtab %}
{% endtabs %}

## Automatic scoping (implemented in Overmind v28.2.1)

Actions within namespaced modules automatically receive **scoped context** where property access is resolved relative to their namespace first, then falls back to the root. This allows you to write self-contained, portable modules:

{% tabs %}
{% tab title="overmind/moduleA/actions.ts" %}

```typescript
export const updateData = async ({ state, effects, actions }) => {
  // Automatically scoped to moduleA namespace (mountpoint)
  state.loading = true              // References state.moduleA.loading
  const data = await effects.api()  // References effects.moduleA.api
  actions.processData(data)         // References actions.moduleA.processData
  
  // Can still access other namespaces explicitly
  state.moduleB.someValue = 'test'  // Cross-namespace access
}
```

{% endtab %}
{% endtabs %}

This makes modules truly portable - they can be moved, renamed, or reused without modifying internal code. The module doesn't need to know what namespace it's mounted under.
