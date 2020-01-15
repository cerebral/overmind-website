# namespaced

Allows you to namespace configurations by a key.

{% tabs %}
{% tab title="overmind/index.ts" %}
```typescript
import {IConfig } from 'overmind'
import { namespaced } from 'overmind/config'
import * as moduleA from './moduleA'
import * as moduleB from './moduleB'

export const config = namespaced({
  moduleA,
  moduleB
})

declare module 'overmind' {
  interface Config extends IConfig<typeof config> {}
}
```
{% endtab %}
{% endtabs %}

