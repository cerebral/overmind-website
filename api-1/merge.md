# merge

Allows you to merge configurations together.

{% tabs %}
{% tab title="overmind/index.ts" %}
```typescript
import {IConfig } from 'overmind'
import { merge } from 'overmind/config'
import * as moduleA from './moduleA'
import * as moduleB from './moduleB'

export const config = merge(moduleA, moduleB)

declare module 'overmind' {
  interface Config extends IConfig<typeof config> {}
}
```
{% endtab %}
{% endtabs %}

Note that merge can be useful to combine a root configuration with **namespaced** or **lazy** configuration.

{% tabs %}
{% tab title="overmind/index.ts" %}
```typescript
import {IConfig } from 'overmind'
import { merge, namespaced, lazy } from 'overmind/config'
import { state } from './state'
import * as moduleA from './moduleA'
import { Config as ModuleB } from './moduleB'

export const config = merge(
  {
    state
  },
  namespaced({
    moduleA
  }),
  lazy({
    moduleB: async (): Promise<ModuleB> => await import('./moduleB').config
  })
)

declare module 'overmind' {
  interface Config extends IConfig<typeof config> {}
}
```
{% endtab %}
{% endtabs %}

