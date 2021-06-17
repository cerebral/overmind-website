# Testing

Testing is a broad subject and everybody has an opinion on it. We can only show you how we think about testing in general and how to effectively write those tests for your Overmind app. 

The most important tests you can write are those who test how your application works when it is all put together, as close to the user experience as possible, so called E2E tests. Testing solutions like [CYPRESS.IO](https://www.cypress.io/) are a great way to do exactly that. You can read more about Cypress and integration testing with Overmind in [THIS ARTICLE](https://www.cypress.io/blog/2019/02/28/shrink-the-untestable-code-with-app-actions-and-effects/#).

You can also do **unit testing** of actions and effects. This will cover expected changes in state and that your side effects behave in a predictable manner. This is typically more important when you are not using Typescript.

## Structuring the app

When you write tests you will create many instances of a mocked version of Overmind with the configuration you have created. To ensure that this configuration can be used many times we have to separate our configuration from the instantiation of the actual app.

{% tabs %}
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
import { createOvermind } from 'overmind'
import { config } from './overmind'

const overmind = createOvermind(config)
```
{% endtab %}
{% endtabs %}

Now we are free to import our configuration without touching the application instance. Lets go!

## Setting initial state

By passing a function either as second or third argument you can change the initial state of Overmind.

{% tabs %}
{% tab title="overmind/actions.test.js" %}
```typescript
import { createOvermindMock } from 'overmind'
import { config } from './'

describe('State', () => {
  test('should derive authors of posts', async () => {
    const overmind = createOvermindMock(config, (state) => {
      state.posts = { 1: { id: 1, author: 'Janet' } }
    })

    expect(overmind.state.authors).toEqual(['Janet'])
  })
})
```
{% endtab %}
{% endtabs %}

## Testing actions

When testing an action you’ll want to verify that changes to state are performed as expected. To give you the best possible testing experience Overmind comes with a mocking tool called **createOvermindMock**. It takes your application configuration and allows you to run actions as if they were run from components.

{% tabs %}
{% tab title="overmind/actions.js" %}
```typescript
export const getPost = async ({ state, api }, id) {
  state.isLoadingPost = true
  try {
    state.currentPost = await api.getPost(id)
  } catch (error) {
    state.error = error
  }
  state.isLoadingPost = false
}
```
{% endtab %}
{% endtabs %}

You might want to test if a thrown error is handled correctly here. This is an example of how you could do that:

{% tabs %}
{% tab title="overmind/actions.test.js" %}
```typescript
import { createOvermindMock } from 'overmind'
import { config } from './'

describe('Actions', () => {
  describe('getPost', () => {
    test('should get post with passed id', async () => {
      const overmind = createOvermindMock(config, {
        api: {
          getPost(id) {
            return Promise.resolve({
              id
            })
          }
        }
      })

      await overmind.actions.getPost('1')

      expect(overmind.state).toEqual({
        isLoadingPost: false,
        currentPost: { id: '1' },
        error: null
      })
    })
    test('should handle errors', async () => {
      const overmind = createOvermindMock(config, {
        api = {
          getPost() {
            throw new Error('test')
          }
        }
      })

      await overmind.actions.getPost('1')

      expect(overmind.state.isLoadingPost).toBe(false)
      expect(overmind.state.error.message).toBe('test')
    })
  })
})
```
{% endtab %}
{% endtabs %}

If your actions can result in multiple scenarios a unit test is beneficial. But you will be surprised how straightforward the logic of your actions will become. Since effects are encouraged to be application specific you will most likely be testing those more than you will test any action.

You do not have to explicitly write the expected state. You can also use for example [JEST](https://www.overmindjs.org/guides/intermediate/05_writingtests?view=react&typescript=true) for snapshot testing. The mock instance has a list of mutations performed. This is perfect for snapshot testing.

{% tabs %}
{% tab title="overmind/actions.test.js" %}
```typescript
import { createOvermindMock } from 'overmind'
import { config } from './'

describe('Actions', () => {
  describe('getPost', () => {
    test('should get post with passed id', async () => {
      const overmind = createOvermindMock(config, {
        api: {
          getPost(id) {
            return Promise.resolve({
              id
            })
          }
        }
      })

      await overmind.actions.getPost('1')

      expect(overmind.mutations).toMatchSnapshot()
    })
    test('should handle errors', async () => {
      const overmind = createOvermindMock(config, {
        api: {
          getPost() {
            throw new Error('test')
          }
        }
      })

      await overmind.actions.getPost('1')

      expect(overmind.mutations).toMatchSnapshot()
    })
  })
})
```
{% endtab %}
{% endtabs %}

In this scenario we would also ensure that the **isLoadingPost** state indeed flipped to _true_ before moving to _false_ at the end.

## Testing onInitializeOvermind

The **onInitializeOvermind** action will not trigger during testing. To test this action you have to trigger it yourself.

{% tabs %}
{% tab title="overmind/onInitialize.test.js" %}
```typescript
import { createOvermindMock } from 'overmind'
import { config } from './'

describe('Actions', () => {
  describe('onInitialize', () => {
    test('should initialize with local storage theme', async () => {
      const overmind = createOvermindMock(config, {
        localStorage: {
          getTheme() {
            return 'awesome' 
          }
        }
      })

      await overmind.actions.onInitializeOvermind()

      expect(overmind.state.theme).toEqual('awesome')
    })
  })
})
```
{% endtab %}
{% endtabs %}

### Testing effects <a id="writing-tests-testing-effects"></a>

Where you want to put in your effort is with the effects. This is where you have your chance to build a domain specific API for your actual application logic. This is the bridge between some generic tool and what your application actually wants to use it for.

A simple example of this is doing requests. Maybe you want to use e.g. [AXIOS](https://github.com/axios/axios) to reach your API, but you do not really care about testing that library. What you want to test is that it is used correctly when you use your application specific API.

This is just an example showing you how you can structure your code for optimal testability. You might prefer a different approach or maybe rely on integration tests for this. No worries, you do what makes most sense for your application:

{% tabs %}
{% tab title="overmind/effects.js" %}
```typescript
import * as axios from 'axios'

// This is the class we can create new instances of when testing
export class Api {
  constructor(request, options) {
    this.request = request
    this.options = options
  }
  async getPost(id: string) {
    try {
      const response = await this.request.get(this.options.baseUrl + '/posts/' + id, {
        headers: {
          'Auth-Token': this.options.authToken
        }
      })

      return response.data
    } catch (error) {
      throw new Error('Could not grab post with id ' + id)
    }
  }
}

// We export the default instance that we actually use with our
// application
export const api = new Api(axios, {
  authToken: '134981091031hfh31',
  baseUrl: '/api'
})
```
{% endtab %}
{% endtabs %}

Let’s see how you could write a test for it:

{% tabs %}
{% tab title="overmind/effects.test.js" %}
```typescript
import { Api } from './effects'

describe('Effects', () => {
  describe('Api', () => {
    test('should get a post using baseUrl and authToken in header', async () => {
      expect.assertions(3)
      const api = new Api({
        get(url, config) {
          expect(url).toBe('/test/posts/1')
          expect(config).toEqual({
            headers: {
              'Auth-Token': '123'
            }
          })

          return Promise.resolve({
            response: {
              id
            }
          })
        }
      }, {
        authToken: '123',
        baseUrl: '/test'
      })

      const post = await api.getPost('1')

      expect(post.id).toBe('1')
    })
  })
})
```
{% endtab %}
{% endtabs %}

Again, effects are where you typically have your most brittle logic. With the approach just explained you will have a good separation between your application logic and the brittle and “impure”. In an Overmind app, especially written in Typescript, you get very far just testing your effects and then do integration tests for your application. But as mentioned in the introduction we all have very different opinions on this, at least the API allows testing to whatever degree you see fit.

