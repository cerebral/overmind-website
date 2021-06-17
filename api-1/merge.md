# merge

Allows you to merge configurations together.

{% tabs %}
{% tab title="overmind/index.js" %}
```typescript
import { merge } from 'overmind/config'
import * as moduleA from './moduleA'
import * as moduleB from './moduleB'

export const config = merge(moduleA, moduleB)
```
{% endtab %}
{% endtabs %}

Note that merge can be useful to combine a root configuration with **namespaced** or **lazy** configuration.

{% tabs %}
{% tab title="overmind/index.ts" %}
```typescript
import { merge, namespaced, lazy } from 'overmind/config'
import { state } from './state'
import * as moduleA from './moduleA'

export const config = merge(
  {
    state
  },
  namespaced({
    moduleA
  }),
  lazy({
    moduleB: async () => await import('./moduleB').config
  })
)
```
{% endtab %}
{% endtabs %}

