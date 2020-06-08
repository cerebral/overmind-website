# derived

You can add derived state to your application. You access derived state like any other value, there is no need to call it as a function. The derived value is cached and will only update when any accessed state changes.

{% tabs %}
{% tab title="overmind/state.ts" %}
```typescript
import { derived } from 'overmind'

export const state: State = {
  items: [],
  completedItems: derived((state, rootState) => {
    return state.items.filter(item => item.completed)
  })
}
```
{% endtab %}
{% endtabs %}

The function defining your derived state receives two arguments. The first argument is the object the derived function is attached to. Ideally you use this argument to produce the derived state, though you can access the second argument which is the root state of the application. The root state allows you to access any state.

{% hint style="info" %}
Accessing **rootState** might cause unnecessary updates to the derived function as it will track more state, though typically not an issue
{% endhint %}

