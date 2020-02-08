# Devtools

## VS Code

For the best experience you should install the [OVERMIND DEVTOOLS](https://marketplace.visualstudio.com/items?itemName=christianalfoni.overmind-devtools-vscode) extension. This will allow you to work on your application without leaving the IDE at all.

![](../.gitbook/assets/amazing_devtools.png)

{% hint style="info" %}
If you are using the **Insiders** version of VSCode the extension will not work. It seems to be some extra security setting.
{% endhint %}

## Standalone app

Alternatively you can install the standalone application of the devtools. You can start it with the NPM executor as:

```javascript
npx overmind-devtools@latest
```

{% hint style="info" %}
Adding **@latest** just ensures that you break any caching, meaning you will always run the latest version
{% endhint %}

You can also install the devtools with your project, allowing you to lock a specific version of the devtools to your project:

```javascript
npm install overmind-devtools
```

With the package [CONCURRENTLY](https://www.npmjs.com/package/concurrently) you can start the devtools as you start your build process:

```text
npm install overmind-devtools concurrently
```

{% code title="package.json" %}
```javascript
{
  ...
  "scripts": {
    "start": "concurrently \"overmind-devtools\" \"someBuildTool\""
  },
  ...
}
```
{% endcode %}

## Connecting from the application

When you create your application it will automatically connect through **localhost:3031**, meaning that everything should just work out of the box. If you need to change the port, connect the application over a network \(mobile development\) or similar, you can configure how the application connects:

```javascript
import { createOvermind } from 'overmind'
import { config } from './overmind'

const overmind = createOvermind(config, {
  devtools: '10.0.0.1:3031'
})
```

### Connecting on Chromebook

ChromeOS does not expose localhost as normal. That means you need to connect with **penguin.termina.linux.test:3031**, or you can use the following plugin to forward **localhost:**

{% embed url="https://chrome.google.com/webstore/detail/connection-forwarder/ahaijnonphgkgnkbklchdhclailflinn/related?hl=en-US" %}

## Hot Module Replacement

A popular concept introduced by Webpack is [HMR](https://webpack.js.org/concepts/hot-module-replacement/). It allows you to make changes to your code without having to refresh. Overmind automatically supports HMR. That means when **HMR** is activated Overmind will make sure it updates and manages its state, actions and effects. Even the devtools will be updated as you make changes.

