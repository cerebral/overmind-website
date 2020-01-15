# createOvermindSSR

The **createOvermindSSR** factory creates an instance of Overmind which can be used on the server with server side rendering. It allows you to change state and extract the mutations performed, which can then be rehydrated on the client.

{% tabs %}
{% tab title="server/routePosts.ts" %}
```typescript
import { createOvermindSSR } from 'overmind'
import { config } from '../client/overmind'
import db from './db'

export default async (req, res) => {
  const overmind = createOvermindSSR(config)

  overmind.state.currentPage = 'posts'
  overmind.state.posts = await db.getPosts()

  const html = renderToString(
    // Whatever your view layer does to produce the HTML
  )

  res.send(`
<html>
  <body>
    <div id="app">${html}</div>
    <script>
      window.__OVERMIND_MUTATIONS = ${JSON.stringify(overmind.hydrate())}
    </script>
    <script src="/scripts/app.js"></script>
  </body>
</html>
`)
}
```
{% endtab %}
{% endtabs %}

## rehydrate

{% tabs %}
{% tab title="overmind/onInitialize.ts" %}
```typescript
import { OnInitialize, rehydrate } from 'overmind'

export const onInitialize: OnInitialize = ({ state }) => {
  const mutations = window.__OVERMIND_MUTATIONS

  rehydrate(state, mutations)
}
```
{% endtab %}
{% endtabs %}

