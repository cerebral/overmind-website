# addMutationListener

It is possible to listen to all mutations performed in Overmind. This allows you to create special effects based on mutations within a certain domain of your app, or whatever else you come up with. Note that this method triggers right after any mutation occurs, you might rather want to use **addFlushListener** to be notified about batched changes, like the components does.

{% tabs %}
{% tab title="overmind/actions.js" %}
```typescript
export const onInitializeOvermind = async ({ state, localStorage }, overmind) => {
  overmind.addMutationListener((mutation) => {
    if (mutation.path.indexOf('todos') === 0) {
      localStorage.set('todos', state.todos)
    }
  })
}
```
{% endtab %}
{% endtabs %}

