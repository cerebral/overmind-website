# State

Typically we think of the user interface as the application itself. But the user interface is really just there to allow a user to interact with the application. This interface can be anything. A browser window, native, sensors etc. It does not matter what the interface is, the application is still the same.

The mechanism of communicating from the application to the user interface is called **state**. A user interface is created by **transforming** the current state. To communicate from the user interface to the application an API is exposed, called **actions** in Overmind. Any interaction can trigger an action which changes the state, causing the application to notify the user interface about any updated state.

![](../.gitbook/assets/state-ui.png)

## Core values

In JavaScript we can create all sorts of abstractions to describe values, but in Overmind we lean on the core serializable values. These are **objects**, **arrays**, **strings**, **numbers**, **booleans** and **null**. Serializable values means that we can easily convert the state into a string and back again. This is fundamental for creating great developer experiences, passing state between client and server and other features. You can describe any application state with these core values.

Let us talk a little bit about what each value helps us represent in our application.

### Objects

The root value of your state tree is an object, because objects are great for holding other values. An object has keys that point to values. Most of these keys point to values that are the actual state of the application, but these keys can also represent domains of the application. A typical state structure could be:

```javascript
{
  modes: ['issues', 'admin'],
  currentModeIndex: 0,
  admin: {
    currentUserId: null,
    users: {
      isLoading: false,
      data: {},
      error: null
    },
  },
  issues: {
    sortBy: 'name',
    isLoading: false,
    data: {},
    error: null
  }
}
```

### Arrays

Arrays are similar to objects in the sense that they hold other values, but instead of keys pointing to values you have indexes. That means it is ideal for iteration. But more often than not objects are actually better at managing lists of values. We can actually do fine without arrays in our state. It is when we produce the actual user interface that we usually want arrays. You can learn more about this in the [MANAGING LISTS](../guides-1/managing-lists.md) guide.

### Strings

Strings are of course used to represent text values. Names, descriptions and whatnot. But strings are also used for ids, types, etc. Strings can be used as values to reference other values. This is an important part in structuring state. For example in our **objects** example above we chose to use an array to represent the modes, using an index to point to the current mode, but we could also do:

```javascript
{
  modes: {
    issues: 0,
    admin: 1 
  },
  currentMode: 'issues'
  ...
}
```

Now we are referencing the current mode with a string. In this scenario you would probably stick with the array, but it is important to highlight that objects allow you to reference things by string, while arrays reference by number.

### Numbers

Numbers of course represent things like counts, age, etc. But just like strings, they can also represent a reference to something in a list. Like we saw in our **objects** example, to define what the current mode of our application is, we can use a number. You could say that referencing things by number works very well when the value behind the number does not change. Our modes will most likely not change and that is why an array and referencing the current mode by number, is perfectly fine.

### Booleans

Are things loading or not, is the user logged in or not? These are typical uses of boolean values. We use booleans to express that something is activated or not. We should not confuse this with **null**, which means “not existing”. We should not use **null** in place of a boolean value. We have to use either `true` or `false`.

### Null

All values, with the exception of booleans, can also be **null**. Non-existing. You can have a non-existing object, array, string or number. It means that if we haven’t selected a mode, both the string version and number version would have the value **null**.

## Undefined

You might wonder why **undefined** is not part of the core value types. Well, there are two reasons:

1. It is not a serializable value. That means if you explicitly set a value to _undefined_ it will not show up in the devtools
2. Undefined values can not be tracked. That means if you were to iterate an object and look at the keys of that object, any undefined values will not be tracked. This can cause unexpected behaviour

## Class values

Overmind also supports using class instances as state values. Depending on your preference this can be a powerful tool to organize your logic. What classes provide is a way to co locate state and logic for changing and deriving that state. In functional programming the state and the logic is separated and it can be difficult to find a good way to organize the logic operating on that state.

It can be a good idea to think about your classes as models. They model some state.

{% tabs %}
{% tab title="overmind/models.js" %}
```javascript
class LoginForm {
  constructor() {
    this.username = ''
    this.password = ''
  }
  isValid() {
    return Boolean(this.username && this.password)
  }
  reset() {
    this.username = ''
    this.password = ''
  }
}
```
{% endtab %}

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
It is import that you do **NOT** use arrow functions on your methods. The reason is that this binds the context of the method to the instance itself, meaning that Overmind is unable to proxy access and allow you to do tracked mutations
{% endhint %}

You can now use this instance as normal and of course create new ones.

{% hint style="info" %}
Even though you can use **getters** as normal, they do not cache like **derived**. **Derived** is a concept of the state tree itself. It is unlikely that you need heavy computation within a single class instance though, it is typically across class instances, where **derived** fits the bill
{% endhint %}

### Serializing class values

If you have an application that needs to serialize the state, for example to local storage or server side rendering, you can still use class instances with Overmind. By default you really do not have to do anything, but if you use **Typescript** or you choose to use **toJSON** on your classesOvermind exposes a symbol called **SERIALIZE** that you can attach to your class.

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

### Rehydrating classes

The [**rehydrate**](../api-1/rehydrate.md) ****utility of Overmind allows you to rehydrate state either by a list of mutations or a state object, like the following:

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
Note that **rehydrate** gives you full type safety when adding the **SERIALIZE**  symbol to your classes. This is a huge benefit as Typescript will yell at you when the state structure changes, related to the rehydration
{% endhint %}

## Deriving state

### Getter

A concept in Javascript called a [GETTER](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/get) allows you to intercept accessing a property in an object. A getter is just like a plain value, it can be added or removed at any point. Getters do **not** cache the result for that very reason, but whatever state they access is tracked.

{% tabs %}
{% tab title="overmind/state.js" %}
```javascript
export const state = {
  user: {
    id: 1,
    firstName: 'Bob',
    lastName: 'Jackson',
    jwt: '1234567'
  },
  get isLoggedIn() {
    return Boolean(this.user && this.user.jwt)
  }
}
```
{% endtab %}
{% endtabs %}

### Cached getter

When you need to do more heavy calculation or combine state from different parts of the tree you can use a plain function instead. Overmind treats these functions like a **getter**, but the returned value is cached and they can also access the root state of the application. A simple example of this would be:

{% tabs %}
{% tab title="overmind/state.js" %}
```typescript
export const state: State = {
  title: 'My awesome title',
  upperTitle: state => state.title.toUpperCase()
}
```
{% endtab %}
{% endtabs %}

The first argument of the function is the state the derived function is attached to. A second argument is also passed and that is the root state of the application, allowing you to access whatever you would need. Two important traits of the derived function is:

1. The state accessed is tracked
2. The value returned is cached

That means the function only runs when accessed and the depending state has changed since last access.

{% hint style="info" %}
Even though derived state is defined as functions you consume them as plain values. You do not have to call the derived function to get the value. Derived state can not be dynamically added. They have to be defined and live in the tree from start to end of your application lifecycle.
{% endhint %}

### Dynamic getter

Sometimes you want to derive state based on some value coming from the user interface. You can do this by creating a function that returns a function. This can be useful for helper functions:

{% tabs %}
{% tab title="overmind/state.js" %}
```javascript
export const state = {
  users: {},
  userById: ({ users }) => id => users[id]
}

// state.userById('123')
```
{% endtab %}
{% endtabs %}

## Statemachines

Very often you get into a situation where you define states as **isLoading**, **hasError** etc. Having these kinds of state can cause **impossible states**. For example:

```typescript
const state = {
  isAuthenticated: true,
  isAuthenticating: true
}
```

You can not be authenticating and be authenticated at the same time. This kind of logic very often causes bugs in applications. That is why Overmind allows you to define statemachines. It sounds complicated, but is actually very simple.

### Defining

{% tabs %}
{% tab title="overmind/state.js" %}
```typescript
import { statemachine } from 'overmind'

export const state = {
  mode: statemachine({
    initial: 'unauthenticated',
    states: {
      unauthenticated: ['authenticating'],
      authenticating: ['unauthenticated', 'authenticated'],
      authenticated: ['unauthenticating'],
      unauthenticating: ['unauthenticated', 'authenticated']
    }
  }),
  user: null,
  error: null
}
```
{% endtab %}
{% endtabs %}

You set an **initial** state and then you create a relationship between the different states and what states they can transition into. So when **unauthenticated** is the state, only logic triggered with an **authenticating** transition will run, any other transition triggered will not run its logic.

### Transitioning

{% tabs %}
{% tab title="overmind/actions.js" %}
```typescript
export const login = async ({ state, effects }) => {
  return state.mode.authenticating(async () => {
    try {
      const user = await effects.api.getUser()
      return state.mode.authenticated(() => {
        state.user = user
      })
    } catch (error) {
      return state.mode.unauthenticated(() => {
        state.error = error
      })    
    }
  })
}

export const logout = async ({ state, effects }) => {
  return state.mode.unauthenticating(async () => {
    try {
      await effects.api.logout()
      return state.mode.unauthenticated()
    } catch (error) {
      return state.mode.authenticated(() => {
        state.error = error
      })    
    }
  })
}
```
{% endtab %}
{% endtabs %}

{% hint style="warning" %}
There are two important rules for predictable transitions:

1. The transition should be **returned** if the logic or logic runs asynchronously. This is the same as with actions in general
2. Only **synchronous** transitions can mutate the state, any async mutation will throw an error
{% endhint %}

What is important to realize here is that our logic is separated into **allowable** transitions. That means when we are waiting for the user on **line 4** and some other logic has changed the state to **unauthenticated** in the meantime, the user will not be set, as the **authenticated** transition is now not possible. This is what state machines do. They group logic into states that are allowed to run, preventing invalid logic to run.

### Current state

The current state is accessed, related to this example, by:

```typescript
state.mode.current
```

### Exit

It is also possible to run logic when a transition exits. An example of this is for example if a transition sets up a subscription. This subscription can be disposed when the transition is exited.

{% tabs %}
{% tab title="overmind/actions.js" %}
```typescript
export const login = async ({ state, effects }) => {
  return state.mode.authenticating(async () => {
    try {
      const user = await effects.api.getUser()
      let disposeSubscription
      state.mode.authenticated(
        () => {
          disposeSubscription = effects.api.subscribeNotifications()
          state.user = user
        },
        () => {
          disposeSubscription()
        }
      )
    } catch (error) {
      state.mode.unauthenticated(() => {
        state.error = error
      })    
    }
  })
}
```
{% endtab %}
{% endtabs %}

### Reset

You can reset the state of a statemachine, which also runs the exit of the current transition:

```typescript
state.mode.reset()
```

## References

When you add objects and arrays to your state tree, they are labeled with an “address” in the tree. That means if you try to add the same object or array in multiple spots in the tree you will get an error, as they can not have multiple addresses. Typically this indicates that you’d rather want to create a reference to an existing object or array.

So this is an example of how you would **not** want to do it:

{% tabs %}
{% tab title="overmind/state.js" %}
```typescript
export const state = {
  users: {},
  currentUser: null
}
```
{% endtab %}

{% tab title="overmind/actions.js" %}
```typescript
export const setUser = ({ state }, id) => {
  state.currentUser = state.users[id]
}
```
{% endtab %}
{% endtabs %}

You’d rather have a reference to the user id, and for example use a **getter** to grab the actual user:

{% tabs %}
{% tab title="overmind/state.js" %}
```typescript
export const state = {
  users: {},
  currentUserId: null,
  get currentUser(this) {
    return this.users[this.currentUserId]
  } 
}
```
{% endtab %}

{% tab title="overmind/actions.js" %}
```typescript
export const setUser = ({ state }, id) => {
  state.currentUserId = id
}
```
{% endtab %}
{% endtabs %}

## Exposing the state

We define the state of the application in **state** files. For example, the top level state could be defined as:

{% tabs %}
{% tab title="overmind/state.js" %}
```javascript
export const state = {
  isLoading: false,
  user: null
}
```
{% endtab %}
{% endtabs %}

To expose the state on the instance you can follow this recommended pattern:

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

{% hint style="info" %}
For scalability you can define **namespaces** for multiple configurations. Read more about that in [Structuring the app](structuring-the-app.md)
{% endhint %}

## Summary

This short guide gave you some insight into how we think about state and what state really is in an application. There is more to learn about these values and how to use them to describe the application. Please move on to other guides to learn more.

