# onInitializeOvermind

If you need to run logic as the application initializes you can use the **onInitializeOvermind** action. This action receives the application instance as the input value. You can do whatever you want here. Set initial state, run an action, configure a router etc.





{% tabs %}
{% tab title="overmind/actions.js" %}
```typescript
export const onInitializeOvermind = async ({
  state,
  actions,
  effects
}, overmind) => {
  const initialData = await effects.api.getInitialData()
  state.initialData = initialData
}
```
{% endtab %}

{% tab title="overmind/index.js" %}
```typescript
import { state } from './state'
import * as actions from './actions'

export const config = {
  state,
  actions
}
```
{% endtab %}
{% endtabs %}

