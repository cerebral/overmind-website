# onInitialize

If you need to run logic as the application initializes you can use the **onInitialize** hook. This is defined as an action and it receives the application instance as the input value. You can do whatever you want here. Set initial state, run an action, configure a router etc.





{% tabs %}
{% tab title="overmind/onInitialize.ts" %}
```typescript
import { OnInitialize } from 'overmind'

export const onInitialize: OnInitialize = async ({
  state,
  actions,
  effects
}, overmind) => {
  const initialData = await effects.api.getInitialData()
  state.initialData = initialData
}
```
{% endtab %}

{% tab title="overmind/index.ts" %}
```typescript
import { IConfig } from 'overmind'
import { onInitialize } from './onInitialize'
import { state } from './state'
import * as actions from './actions'

export const config = {
  onInitialize,
  state,
  actions
}

// For explicit typing check the Typescript guide
declare module 'overmind' {
  interface Config extends IConfig<typeof config> {}
}
```
{% endtab %}
{% endtabs %}

