# effects

{% tabs %}
{% tab title="overmind/effects.ts" %}
```typescript
import { User, Item } from './state'

export const api = {
  async getUser(): Promise<User> {
    const response = await fetch('/user')

    return response.json()
  },
  async getItem(id: number): Promise<Item> {
    const response = await fetch(`/items/${id}`)

    return response.json()
  }
}
```
{% endtab %}
{% endtabs %}

Effects is really just about exposing existing libraries or create your own APIs for doing side effects. When these effects are attached to the application they will be tracked by the devtools giving you additional debugging information. By “injecting” the effects this way also open up for better testability of your logic.

