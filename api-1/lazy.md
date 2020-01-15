# lazy

You can lazy load configurations. You do this by giving each configuration a key with a function that returns the config when called. To actually load the configurations you can either call an effect or an action with the key of the configuration to load.

{% tabs %}
{% tab title="overmind/index.ts" %}
```typescript
import { IConfig } from 'overmind'
import { lazy } from 'overmind/config'
import { Config as ModuleAConfig } from './moduleA'

export const config = lazy({
  moduleA: async (): Promise<ModuleAConfig> => await import('./moduleA').config
})

declare module 'overmind' {
  interface Config extends IConfig<typeof config> {}
}
```
{% endtab %}

{% tab title="overmind/moduleA/index.ts" %}
```typescript
import { IConfig } from 'overmind'
import { state } from './state'

export const config = {
  state
}

export interface Config extends IConfig<typeof config> {}
```
{% endtab %}

{% tab title="overmind/actions.ts" %}
```typescript
import { AsyncAction } from 'overmind'

export const loadModule: AsyncAction = async ({ actions }) => {
  await actions.lazy.loadConfig('moduleA')
}
```
{% endtab %}
{% endtabs %}

