# createOvermindMock

The **createOvermindMock** factory creates an instance of Overmind which can be used to test actions. You can mock out effects and evaluate mutations performed during action execution.

{% tabs %}
{% tab title="overmind/actions.test.ts" %}
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

{% hint style="warning" %}
It is important that you separate your **config** from the instantiation of Overmind, meaning that **createOvermind** should not be used in the same file as the config you see imported here, it should rather be used where you render your application. This allows the config to be used for multiple purposes.
{% endhint %}

