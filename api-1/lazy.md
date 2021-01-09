# lazy

You can lazy load configurations. You do this by giving each configuration a key with a function that returns the config when called. To actually load the configurations you can either call an effect or an action with the key of the configuration to load.

{% tabs %}
{% tab title="overmind/index.js" %}
```typescript
import { lazy } from 'overmind/config'

export const config = lazy({
  moduleA: async () => await import('./moduleA').config
})
```
{% endtab %}

{% tab title="overmind/moduleA/index.js" %}
```typescript
import { state } from './state'

export const config = {
  state
}
```
{% endtab %}

{% tab title="overmind/actions.js" %}
```typescript
export const loadModule = async ({ actions }) => {
  await actions.lazy.loadConfig('moduleA')
}
```
{% endtab %}
{% endtabs %}

