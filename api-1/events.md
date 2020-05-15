# events

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

