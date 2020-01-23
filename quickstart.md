# Quickstart

From the command line install the Overmind package:

{% tabs %}
{% tab title="React" %}
```
npm install overmind overmind-react
```
{% endtab %}

{% tab title="Vue" %}
```
npm install overmind overmind-vue
```
{% endtab %}

{% tab title="Angular" %}
```text
npm install overmind overmind-angular
```
{% endtab %}
{% endtabs %}

### Setup

Now set up a simple application like this:

{% tabs %}
{% tab title="overmind/state.js" %}
```javascript
export const state = {
  title: 'My App'
}
```
{% endtab %}

{% tab title="overmind/index.js" %}
```typescript
import { state } from './state'

export const config = {
  state
}
```
{% endtab %}

{% tab title="index.js" %}
```typescript
import { createOvermind } from 'overmind'
import { config } from './overmind'

const overmind = createOvermind(config)
```
{% endtab %}
{% endtabs %}

And fire up your application in the browser or whatever environment your user interface is to be consumed in by the users.

Move on with the specific view layer of choice to connect your app:

\*\*\*\*[**REACT** ](views/react.md)**-** [**ANGULAR**](views/angular.md) **-** [**VUE**](views/vue.md)\*\*\*\*

