# Using classes

Classes allows you to co locate state and logic. It can be a good idea to think about your classes as models. They model some state.

{% tabs %}
{% tab title="overmind/models.js" %}
```javascript
class LoginForm {
  constructor() {
    this.username = ''
    this.password = ''
  }
  get isValid() {
    return Boolean(this.username && this.password)
  }
  reset() {
    this.username = ''
    this.password = ''
  }
}
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="overmind/state.js" %}
```javascript
import { LoginForm } from './models'

export const state = {
  loginForm: new LoginForm()
}
```
{% endtab %}
{% endtabs %}

{% hint style="warning" %}
It is important that you do **NOT** use arrow functions on your methods. The reason is that this binds the context of the method to the instance itself, meaning that Overmind is unable to proxy access and track mutations
{% endhint %}

You can now use this instance as normal and of course create new ones.

#### Serializing class values

If you have an application that needs to serialize the state, for example to local storage or server side rendering, you can still use class instances with Overmind. By default you really do not have to do anything, but if you use **Typescript** or you choose to use **toJSON** on your classes Overmind exposes a symbol called **SERIALIZE** that you can attach to your class.

```typescript
import { SERIALIZE } from 'overmind'

class User {
  constructor() {
    this.username = ''
    this.jwt = ''
  }
  toJSON() {
    return {
      [SERIALIZE]: true,
      username: this.username
    }
  }
}
```

If you use **Typescript** you want to add **SERIALIZE** to the class itself as this will give you type safety when rehydrating the state.

```typescript
import { SERIALIZE } from 'overmind'

class User {
  [SERIALIZE] = true
  username = ''
  jwt = ''
}
```

{% hint style="info" %}
The **SERIALIZE** symbol will not be part of the actual serialization done with **JSON.stringify**
{% endhint %}

#### Rehydrating classes

The [**rehydrate**](../api-1/rehydrate.md) utility of Overmind allows you to rehydrate state either by a list of mutations or a state object, like the following:

{% tabs %}
{% tab title="overmind/actions.js" %}
```typescript
import { rehydrate } from 'overmind'

export const updateState = ({ state }) => {
  rehydrate(state, {
    user: {
      username: 'jenny',
      jwt: '123'
    }
  })    
}
```
{% endtab %}
{% endtabs %}

Since our user is a class instance we can tell rehydrate what to do, where it is typical to give the class a static **fromJSON** method:

{% tabs %}
{% tab title="overmind/models.js" %}
```typescript
import { SERIALIZE } from 'overmind'

class User {
  [SERIALIZE]
  constructor() {
    this.username = ''
    this.jwt = ''
  }
  static fromJSON(json) {
    return Object.assign(new User(), json)
  }
}
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="overmind/actions.js" %}
```typescript
import { rehydrate } from 'overmind'

export const updateState = ({ state }) => {
  rehydrate(
    state,
    {
      user: {
        username: 'jenny',
        jwt: '123'
      }
    },
    {
      user: User.fromJSON
    }
  )    
}
```
{% endtab %}
{% endtabs %}

It does not matter if the state value is a class instance, an array of class instances or a dictionary of class instances, rehydrate will understand it.

That means the following will behave as expected:

{% tabs %}
{% tab title="overmind/state.js" %}
```typescript
import { User } from './models'

export const state = {
  user: null, // Can be existing class instance or null
  usersList: [], // Expecting an array of values
  usersDictionary: {} // Expecting a dictionary of values
}
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="overmind/actions.js" %}
```typescript
import { rehydrate } from 'overmind'

export const updateState = ({ state }) => {
  rehydrate(
    state,
    {
      user: {
        username: 'jenny',
        jwt: '123'
      },
      usersList: [{...}, {...}],
      usersDictionary: {
        'jenny': {...},
        'bob': {...}
      }
    },
    {
      user: User.fromJSON,
      usersList: User.fromJSON,
      usersDictionary: User.fromJSON
    }
  )    
}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Note that **rehydrate** gives you full type safety when adding the **SERIALIZE** symbol to your classes. This is a huge benefit as Typescript will yell at you when the state structure changes, related to the rehydration
{% endhint %}

