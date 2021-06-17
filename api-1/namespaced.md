# namespaced

Allows you to namespace configurations by a key.

The point of namespaces is to structure your code into domains, not isolate them. The reason being is that we more often than not design our namespaces wrong. We have no idea how the final app will look and getting into the issue of "cross domain logic and state" is a pain to refactor all the time due to wrong isolation.

So in Overmind isolation is a discipline, not a technical restriction.

{% tabs %}
{% tab title="overmind/index.ts" %}
```typescript
import { namespaced } from 'overmind/config'
import * as moduleA from './moduleA'
import * as moduleB from './moduleB'

export const config = namespaced({
  moduleA,
  moduleB
})
```
{% endtab %}
{% endtabs %}

