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

**Application state**

This contains the initial state of the application.

```javascript
// overmind/state.js
export const state = {
  title: "My App",
};
```

**Application config**

This contains the sate, effects and actions. For now, we just
set the state.

```javascript
// overmind/index.js
import { state } from "./state";

export const config = {
  state,
};
```

**Create application**

This is where you boot your application by linking all the pieces together.

```javascript
// index.js
import { createOvermind } from "overmind";
import { config } from "./overmind";

const overmind = createOvermind(config);
// ... see below on how to link the overmind instance to specific view layers.
```

And fire up your application in the browser or whatever environment your user
interface is to be consumed in by the users.

Move on with the specific view layer of choice to connect your app:

\*\*\*\*[**REACT** ](views/react.md)**-** [**ANGULAR**](views/angular.md) **-** [**VUE**](views/vue.md)\*\*\*\*
