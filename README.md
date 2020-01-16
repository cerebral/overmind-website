---
description: frictionless state management
---

# Overmind

> Web application development is about **defining**, **changing** and **consuming state** to produce a user experience. Overmind aims for a developer experience where that is all you focus on, reducing the orchestration of state management to a minimum. Making you a **happier** and more **productive** developer!

## APPLICATION INSIGHT

Develop the application state, effects and actions without leaving [VS Code](https://code.visualstudio.com/), or use the standalone development tool. Everything that happens in your app is tracked and you can seamlessly code and run logic to verify that everything works as expected without having to implement UI.

![](.gitbook/assets/amazing_devtools.png)

## A SINGLE STATE TREE

Building your application as a single state tree is the most straight forward mental model. You get a complete overview, but can still organize the state by namespacing it into domains. The devtools allows you to edit and mock out state.

```typescript
{
  isAuthenticating: false,
  dashboard: {
    issues: [],
    selectedIssueId: null,
  },
  user: null,
  form: new Form()
}
```

## SEPARATION OF LOGIC

Separate 3rd party APIs and logic not specific to your application by using **effects**. This will keep your application logic pure and without low level APIs cluttering your code.

```typescript
export const api = {
  async fetchItems(): Promise<Item[]> {
    const response = await fetch('/api/items')

    return response.json()
  }
}
```

## SAFE AND PREDICTABLE CHANGES

When you build applications that perform many state changes things can get out of hand. In Overmind you can only perform state changes from **actions** and all changes are tracked by the development tool.

```typescript
export const getItems: AsyncAction = async ({ state, effects }) => {
  state.isLoadingItems = true
  state.items = await effects.api.fetchItems()
  state.isLoadingItems = false
}
```

## FUNCTIONAL ACTIONS

When pieces of logic become complex it is beneficial to write functional code. Overmind provides an API named **operators** which gives you functional power. Ignore it, use it where it makes sense or make your whole codebase functional. It is up to you!

```typescript
export const search: Operator<string> = pipe(
  mutate(({ state }, query) => {
    state.query = query
  }),
  filter((_, query) => query.length > 2),
  debounce(200),
  mutate(async ({ state, effects }, query) => {
    state.isSearching = true
    state.searchResult = await effects.getSearchResult(query)
    state.isSearching = false
  })
)
```

## SNAPSHOT TESTING OF LOGIC

Bring in your application configuration of state, effects and actions. Create mocks for any effects. Take a snapshot of mutations performed in an action to ensure all intermediate states are met.

```typescript
import { createOvermindMock } from 'overmind'
import { config } from './'

test('should get items', async () => {
  const overmind = createOvermindMock(config, {
    api: {
      fetchItems: () => Promise.resolve([{ id: 0, title: "foo" }])
    }
  })

  await overmind.actions.getItems()

  expect(overmind.mutations).toMatchSnapshot()
})
```

## WE WROTE THE TYPING

Overmind has you covered on typing. If you choose to use Typescript the whole API is built for excellent typing support. You will not spend time telling Typescript how your app works, Typescript will tell you!

## RUNNING CODESANDBOX

![](.gitbook/assets/256x256.png)

Overmind is running the main application of [codesandbox.io](https://codesandbox.io). Codesandbox, with its state and effects complexity, benefits greatly combining Overmind and Typescript.

