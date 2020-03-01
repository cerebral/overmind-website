# createOvermind

The **createOvermind** factory is used to create the application instance. You need to create and export a mechanism to connect your instance to the components. Please look at the guides for each view layer for more information.

{% tabs %}
{% tab title="overmind/index.ts" %}
```typescript
import { IConfig } from 'overmind'
import { state } from './state'
import * as effects from './effects'
import * as actions from './actions'

export const config = {
  state,
  effects,
  actions
}

// For explicit typing check the Typescript guide
declare module 'overmind' {
  interface Config extends IConfig<typeof config> {}
}
```
{% endtab %}
{% endtabs %}

You can pass a second argument to the **createOvermind** factory. This is an options object with the following properties.

## options.devtools

If you develop your app on localhost the application connects to the devtools on **localhost:3031**. You can change this in case you need to use an IP address, the devtools is configured with a different port or you want to connect to localhost \(with default port\) even though the app is not on localhost.[EDIT ON GITHUB](https://github.com/cerebral/overmind/edit/next/packages/overmind-website/examples/api/app_options_devtools.ts)

```typescript
const overmind = createOvermind(config, {
  devtools: true // 'localhost:3031'
})
```

## options.logProxies

By default, in **development**, Overmind will make sure that any usage of **console.log** will log out the actual object/array, instead of any proxy wrapping it. This is most likely what you want. If you want to turn off this behaviour, set this option to **true**.[EDIT ON GITHUB](https://github.com/cerebral/overmind/edit/next/packages/overmind-website/examples/api/app_options_logproxies.ts)

```typescript
const overmind = createOvermind(config, {
  logProxies: false
})
```

## options.name

If you have multiple instances of Overmind on the same page you can name your app to differentiate them.[EDIT ON GITHUB](https://github.com/cerebral/overmind/edit/next/packages/overmind-website/examples/api/app_options_name.ts)

```typescript
const overmindA = createOvermind(configA, {
  name: 'appA'
})

const overmindB = createOvermind(configB, {
  name: 'appB'
})
```

## options.hotReloading

By default Overmind will do the necessary hot reloading mechanism to keep your state, actions and effects updated. You might not want to use this feature or Overmind does not correctly detect that hot reloading should be turned off. You can turn it off with an option.[EDIT ON GITHUB](https://github.com/cerebral/overmind/edit/next/packages/overmind-website/examples/api/app_options_hotreloading.ts)

```typescript
const overmind = createOvermind(config, {
  hotReloading: false
})
```

## options.delimiter

By default Overmind will create state paths using `.` as delimiter. This is used to give each state value an address and is used with the devtools. If any state keys uses `.` you will get weird behaviour in the devtools. You can now change this delimiter to a safe value, typically `' '` or `'|'` :

```typescript
const overmind = createOvermind(config, {
  delimiter: '.'
})
```

## events

Overmind emits events during execution of actions and similar. It can be beneficial to listen to these events for analytics or maybe you want to create a custom debugging experience. The following events can be listened to by adding a listener to the eventHub:

```typescript
overmind.eventHub.on('action:start', (execution) => {})
overmind.eventHub.on('action:end', (execution) => {})
overmind.eventHub.on('operator:start', (execution) => {})
overmind.eventHub.on('operator:end', (execution) => {})
overmind.eventHub.on('operator:async', (execution) => {})
overmind.eventHub.on('mutations', (executionAndMutations) => {})
overmind.eventHub.on('derived', (derived) => {})
overmind.eventHub.on('derived:dirty', (derivedPathAndFlush) => {})

// Only during development
overmind.eventHub.on('effect', (effectDetails) => {})
overmind.eventHub.on('getter', (getterDetails) => {})
overmind.eventHub.on('component:add', (componentDetails) => {})
overmind.eventHub.on('component:update', (componentDetails) => {})
overmind.eventHub.on('component:remove', (componentDetails) => {})
```

