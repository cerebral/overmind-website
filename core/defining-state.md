# State

Typically we think of the user interface as the application itself. But the user interface is really just there to allow a user to interact with the application. This interface can be anything. A browser window, native, sensors etc. It does not matter what the interface is, the application is still the same.

The mechanism of communicating from the application to the user interface is called **state**. A user interface is created by **transforming** the current state. To communicate from the user interface to the application an API is exposed, called **actions** in Overmind. Any interaction can trigger an action which changes the state, causing a new transform of the user interface.

![](../.gitbook/assets/state-ui.png)

## State tree

Overmind is structured as a single state tree. That means all of your state can be accessed through a single object, called the **state**. This state tree will hold values which describes different states of your application. The tree branches out using plain objects, which can be considered **branches** of your state tree.

```javascript
{ // branch
  modes: ['issues', 'admin'],
  currentModeIndex: 0,
  admin: { // branch
    currentUserId: null,
    users: { // branch
      isLoading: false,
      data: {},
      error: null
    },
  },
  issues: { // branch
    sortBy: 'name',
    isLoading: false,
    data: {},
    error: null
  }
}
```

## State tree values

The following are values to be used with the state tree.

### Objects

The plain objects are what **branches** out the tree. It is not really considered a value in itself, it is a state branch holding values.

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

Now we are referencing the current mode with a string. In this scenario you would probably stick with the array, but it is important to highlight that objects allow you to reference things by string, while arrays reference by index, a number.

### Numbers

Numbers of course represent things like counts, age, etc. But just like strings, they can also represent a reference to something. Like we saw in our **objects** example, to define what the current mode of our application is, we can use a number. You could say that referencing things by number works very well when the value behind the number does not change. Our modes will most likely not change and that is why an array and referencing the current mode by number, is perfectly fine.

### Booleans

Are things loading or not, is the user logged in or not? These are typical uses of boolean values. We use booleans to express that something is activated or not. We should not confuse this with **null**, which means “not existing”. We should not use **null** in place of a boolean value. We have to use either `true` or `false`.

### Null

All values, with the exception of booleans, can also be **null**. Non-existing. You can have a non-existing object, array, string or number. It means that if we haven’t selected a mode, both the string version and number version would have the value **null**.

### Derived

When you need to derive state you can add a derived function to your tree. Overmind treats these functions like a **getter**, but the returned value is cached and they can also access the root state of the application. A simple example of this would be:

{% tabs %}
{% tab title="overmind/state.js" %}
```typescript
import { derived } from 'overmind'

export const state = {
  title: 'My awesome title',
  upperTitle: derived(state => state.title.toUpperCase())
}
```
{% endtab %}
{% endtabs %}

The first argument of the function is the state the derived function is attached to. A second argument is also passed and that is the root state of the application, allowing you to access whatever you would need.

{% hint style="info" %}
Even though derived state is defined as functions you consume them as plain values. You do not have to call the derived function to get the value. Derived functions can also be dynamically added.
{% endhint %}

You can also return a function from the derived. For example:

{% tabs %}
{% tab title="overmind/state.js" %}
```typescript
import { derived } from 'overmind'

export const state = {
  users: {},
  getUserById: derived(state => id => state.users[id])
}
```
{% endtab %}
{% endtabs %}

The returned value here is indeed a function you call. The cool thing is that the function itself will never change, but whatever state you access when calling the function will be tracked by the caller of the function. So for example if a component uses **getUserById** during rendering it will track what is accessed in the function and continue tracking whatever you access on the returned value.

{% hint style="info" %}
You may use a derived for all sorts of calculations. But sometimes it's better to just use a plain action to manipulate some state than using a derived. Why? Imagine a table component having a lot of rows and columns. We assume the table component also takes care of sorting and filtering and is capable of adding new rows. Now if you solve the sorting and filtering using a derived the following could happen: User adds a new row but it is not displayed in the list because the derived immediately kicked in and filtered it out. Thats not a good user experience. Also in this case the filtering and sorting is clearly started by a simple user interaction \(setting a filter value, clicking on a column,...\) so why not just start an action which creates the new list of sorted and filtered keys? Also the heavy calculation is now very predictable and doesn't cause performance issues because the derived kicks in too often \(Because it could have many dependencies you did not think of\)
{% endhint %}

### Class instances

Overmind also supports using class instances as state values. Depending on your preference this can be a powerful tool to organize your logic. What classes provide is a way to co locate state and logic for changing and deriving that state. In functional programming the state and the logic is separated and it can be difficult to find a good way to organize the logic operating on that state. Read the [**Using classes**](../guides-1/using-classes.md) guide to learn moree.

### Statemachines

Very often you get into a situation where you define states as **isLoading**, **hasError** etc. Having these kinds of state can cause **impossible states**. For example:

```typescript
const state = {
  isAuthenticated: true,
  isAuthenticating: true
}
```

You can not be authenticating and be authenticated at the same time. This kind of logic very often causes bugs in applications. That is why Overmind allows you to define statemachines. It sounds complicated, but is actually very simple.

To properly understand state machines, please read the guide [**Using state machines**](../guides-1/using-state-machines.md). 

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
{% endtabs %}

{% tabs %}
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
{% endtabs %}

{% tabs %}
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
{% endtabs %}

{% tabs %}
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

