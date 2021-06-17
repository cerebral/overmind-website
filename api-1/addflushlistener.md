# addFlushListener

The **addMutationListener** triggers whenever there is a mutation. The **addFlushListener** triggers whenever Overmind tells components to render again. It can have multiple mutations related to it.

{% tabs %}
{% tab title="overmind/actions.js" %}
```typescript
export const onInitializeOvermind = async ({ state, effects }, overmind) => {
  overmind.addFlushListener(effects.history.addMutations)
}
```
{% endtab %}
{% endtabs %}

