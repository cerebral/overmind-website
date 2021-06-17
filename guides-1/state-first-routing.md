# State first routing

With Overmind you can use whatever routing solution your selected view layer provides. This will most likely intertwine routing state with your component state, which is something Overmind would discourage, but you know… whatever you feel productive in, you should use :-\) In this guide we will look into how you can separate your router from your components and make it part of your application state instead. This is more in the spirit of Overmind and throughout this guide you will find benefits of doing it this way.

We are going to use [PAGE.JS](https://www.npmjs.com/package/page) as the router and we will look at a complex routing example where we open a page with a link to a list of users. When you click on a user in the list we will show that user in a modal with the URL updating to the id of the user. In addition we will present a query parameter that reflects the current tab inside the modal.

We will start with a simple naïve approach and then tweak our approach a little bit for the optimal solution.

## Set up the app

Before we go into the router we want to set up the application. We have some state helping us express the UI explained above. In addition we have three actions.

1. **showHomePage** tells our application to set the current page to _home_
2. **showUsersPage** tells our application to set the current page to _users_ and fetches the users as well
3. **showUserModal** tells our application to show the modal by setting an id of a user passed to the action. This action will also handle the switching of tabs later.

{% tabs %}
{% tab title="overmind/actions.js" %}
```typescript
export const showHomePage = ({ state }) => {
  state.currentPage = 'home'
}

export const showUsersPage = async ({ state, effects }) => {
  state.modalUser = null
  state.currentPage = 'users'
  state.isLoadingUsers = true
  state.users = await effects.api.getUsers()
  state.isLoadingUsers = false
}

export const showUserModal = async ({ state, effects }, params) => {
  state.isLoadingUserDetails = true
  state.modalUser = await effects.api.getUserWithDetails(params.id)
  state.isLoadingUserDetails = false
}
```
{% endtab %}
{% endtabs %}

## Initialize the router

**Page.js** is pretty straightforward. We basically want to map a URL to trigger an action. To get started, let us first add Page.js as an effect and take the opportunity to create a custom API. When a URL triggers we want to pass the params of the route to the action linked to the route:

{% tabs %}
{% tab title="overmind/effects.js" %}
```typescript
import page from 'page'

export const router = {
  initialize(routes) {
    Object.keys(routes).forEach(url => {
      page(url, ({ params }) => routes[url](params))
    })
    page.start()
  },
  open: (url) => page.show(url)
}
```
{% endtab %}
{% endtabs %}

Now we can use Overmind’s **onInitialize** to configure the router. That way the initial URL triggers before the UI renders and we get to set our initial state.

{% tabs %}
{% tab title="overmind/actions.js" %}
```typescript
// The other actions

export const onInitializeOvermind = ({ actions, effects }) => {
  effects.router.initialize({
    '/': actions.showHomePage,
    '/users': actions.showUsersPage,
    '/users/:id': actions.showUserModal
  })
}
```
{% endtab %}
{% endtabs %}

Take notice here that we are actually passing in the params from the router, meaning that the id of the user will be passed to the action.

## The list of users

When we now go to the list of users the list loads up and is displayed. When we click on a user the URL changes, our **showUser** action runs and indeed, we see a user modal.

{% tabs %}
{% tab title="React" %}
```typescript
// components/App.jsx
import * as React from 'react'
import { useAppState } from '../overmind'
import Users from './Users'

const App = () => {
  const state = useAppState()

  return (
    <div className="container">
      <nav>
        <a href="/">Home</a>
        <a href="/users">Users</a>
      </nav>
      {state.currentPage === 'home' ? <h1>Hello world!</h1> : null}
      {state.currentPage === 'users' ? <Users /> : null}
    </div>
  )
}

export default App

// components/Users.jsx
import * as React from 'react'
import { useAppState } from '../overmind'
import UserModal from './UserModal'

const Users = () => {
  const state = useAppState()

  return (
    <div className="content">
      {state.isLoadingUsers ? (
        <h4>Loading users...</h4>
      ) : (
        <ul>
          {state.users.map(user => (
            <li key={user.id}>
              <a href={"/users/" + user.id}>{user.name}</a>
            </li>
          ))}
        </ul>
      )}
      {state.isLoadingUserDetails || state.modalUser ? <UserModal /> : null}
    </div>
  )
}

export default Users

// components/UserModal.jsx
import * as React from 'react'
import { useAppState } from '../overmind'

const UserModal = () => {
  const state = useAppState()

  return (
    <a href="/users" className="backdrop">
      <div className="modal">
        {state.isLoadingUserDetails ? (
          <h4>Loading user details...</h4>
        ) : (
          <>
            <h4>{state.modalUser.name}</h4>
            <h6>{state.modalUser.details.email}</h6>
            <nav>
              <a href={"/users/" + state.modalUser.id + "?tab=0"}>bio</a>
              <a href={"/users/" + state.modalUser.id + "?tab=1"}>address</a>
            </nav>
            {state.currentUserModalTabIndex === 0 ? (
              <div className="tab-content">{state.modalUser.details.bio}</div>
            ) : null}
            {state.currentUserModalTabIndex === 1 ? (
              <div className="tab-content">{state.modalUser.details.address}</div>
            ) : null}
          </>
        )}
      </div>
    </a>
  )
}

export default UserModal
```
{% endtab %}

{% tab title="Angular" %}
```typescript
// components/app.component.ts
import { Component } from '@angular/core';
import { Store } from '../overmind'

@Component({
  selector: 'app-component',
  template: `
  <div class="container" *track>
    <nav>
      <a href="/">Home</a>
      <a href="/users">Users</a>
    </nav>
    <h1 *ngIf="state.currentPage === 'home'">Hello world!</h1>
    <users-list *ngIf="state.currentPage === 'users'"></users-list>
  </div>
  `
})
export class AppComponent {
  state = this.store.select()
  constructor(private store: Store) {}
}

// components/users-list.component.ts
import { Component } from '@angular/core';
import { Store } from '../overmind'

@Component({
  selector: 'users-list',
  template: `
  <div class="content" *track>
    <h4 *ngIf="state.isLoadingUsers">Loading users...</h4>
    <ul *ngIf="!state.isLoadingUsers">
      <li *ngFor="let user of state.users;trackby: trackById">
        <a href={"/users/" + user.id}>{{user.name}}</a>
      </li>
    </ul>
    <user-modal *ngIf="state.isLoadingUserDetails || state.userModal"></user-modal>
  </div>
  `
})
export class UsersList {
  state = this.store.select()
  constructor(private store: Store) {}
  trackById(index, user) {
    return user.id
  }
}

// components/user-modal.component.ts
import { Component } from '@angular/core';
import { Store } from '../overmind'

@Component({
  selector: 'user-modal',
  template: `
  <a href="/users" class="backdrop">
    <div class="modal">
      <h4 *ngIf="state.isLoadingUserDetails">Loading user details...</h4>
      <div *ngIf="!state.isLoadingUserDetails">
        <h4>{{state.modalUser.name}}</h4>
        <h6>{{state.modalUser.details.email}}</h6>
        <nav>
          <a [href]="'/users/' + state.modalUser.id + '?tab=0'">bio</a>
          <a [href]="'/users/' + state.modalUser.id + '?tab=1'">address</a>
        </nav>
        <div
          *ngIf="state.currentUserModalTabIndex === 0"
          class="tab-content"
        >
          {{modalUser.details.bio}}
        </div>
        <div
          *ngIf="state.currentUserModalTabIndex === 1"
          class="tab-content"
        >
          {{modalUser.details.address}}
        </div>
      </div>
    </div>
  </a>
  `
})
export class UserModal {
  state = this.store.select()
  constructor(private store: Store) {}
}
```
{% endtab %}

{% tab title="Vue" %}
```typescript
// components/App.vue
<template>
  <div class="container">
    <nav>
      <a href="/">Home</a>
      <a href="/users">Users</a>
    </nav>
    <h1 v-if="state.currentPage === 'home'">Hello world!</h1>
    <users-list v-if="state.currentPage === 'users'"></users-list>
  </div>
</template>

// components/UsersList.vue
<template>
  <div class="content">
    <h4 v-if="state.isLoadingUsers">Loading users...</h4>
    <ul v-else>
      <li v-for="user in state.users" :key="user.id">
        <a :href="'/users/' + user.id">{{ user.name }}</a>
      </li>
    </ul>
    <user-modal v-if="state.isLoadingUserDetails || state.userModal"></user-modal>
  </div>
</template>

// components/UserModal.vue
<template>
  <a href="/users" class="backdrop">
    <div class="modal">
      <h4 v-if="state.isLoadingUserDetails">Loading user details...</h4>
      <div v-else>
        <h4>{{ state.modalUser.name }}</h4>
        <h6>{{ state.modalUser.details.email }}</h6>
        <nav>
          <a :href="'/users/' + state.modalUser.id + '?tab=0'">bio</a>
          <a :href="'/users/' + state.modalUser.id + '?tab=1'">address</a>
        </nav>
        <div
          v-if="state.currentUserModalTabIndex === 0"
          class="tab-content"
        >
          {{ state.modalUser.details.bio }}
        </div>
        <div
          v-if="state.currentUserModalTabIndex === 1"
          class="tab-content"
        >
          {{ modalUser.details.address }}
        </div>
      </div>
    </div>
  </a>
</template>
```
{% endtab %}
{% endtabs %}

But what if we try to refresh now… we get an error. The router tries to run our user modal, but we are on the front page. The modal does not exist there. We want to make sure that when we open a link to a user modal we also go to the actual user list page.

## Composing actions

A straightforward way to solve this is to simply also change the page in the **showUserModal** action, though we would like the list of users to load in the background as well. The logic of **showUsers** might also be a lot more complex and we do not want to duplicate our code. When these scenarios occur where you want to start calling actions from actions, it indicates you have reached a level of complexity where a functional approach might be better. Let us look at how you would implement this both using a functional approach and a plain imperative one.

### Imperative approach

{% tabs %}
{% tab title="overmind/actions.js" %}
```typescript
import { Page } from './types'

export const showHomePage = ({ state }) => {
  state.currentPage = Page.HOME
}

export const showUsersPage = async ({ state, effects }) => {
  state.currentPage = Page.USERS
  state.modalUser = null

  if (!Object.keys(state.users).length) {
    state.isLoadingUsers = true
    state.users = await effects.api.getUsers()
    state.isLoadingUsers = false
  }
}

export const showUserModal = async ({ state, actions }, params) => {
  actions.showUsersPage()
  state.isLoadingUserDetails = true
  state.modalUser = await effects.api.getUserWithDetails(params.id)
  state.isLoadingUserDetails = false
}
```
{% endtab %}
{% endtabs %}

Going functional depends on complexity and even though the complexity has indeed increased, we can safely manage it using plain imperative code. Whenever we open a user modal we can simply just call the action that takes care of bringing us to the users page as well.

When running actions from within other actions like this it will be reflected in the devtool.

### Functional approach

{% tabs %}
{% tab title="overmind/operators.js" %}
```typescript
import { filter } from 'overmind'

export const closeUserModal = () => ({ state }) => {
  state.modalUser = null
}

export const setPage = (page) => ({ state }) => {
  state.currentPage = page
}

export const shouldLoadUsers = () => filter(({ state }) => {
  return !Boolean(state.users.length)
})

export const loadUsers = () => async ({ state, effects }) {
  state.isLoadingUsers = true
  state.users = await effects.api.getUsers()
  state.isLoadingUsers = false
}

export const loadUserWithDetails = () => async ({ state, effects }, params) => {
  state.isLoadingUserDetails = true
  state.modalUser = await effects.api.getUserWithDetails(params.id)
  state.isLoadingUserDetails = false
}
```
{% endtab %}

{% tab title="overmind/actions.js" %}
```typescript
import {pipe } from 'overmind'
import * as o from './operators'

export const showHomePage = o.setPage('HOME')

export const showUsersPage = pipe(
  o.setPage('USERS'),
  o.closeUserModal(),
  o.shouldLoadUsers(),
  o.loadUsers()
)

export const showUserModal = pipe(
  o.setPage('USERS'),
  o.loadUserWithDetails(),
  o.shouldLoadUsers(),
  o.loadUsers(),
)
```
{% endtab %}
{% endtabs %}

By splitting up all our logic into operators we were able to make our actions declarative and at the same time reuse logic across them. The _operators_ file gives us maintainable code and the _actions_ file gives us readable code.

We could actually make this better though. There is no reason to wait for the user of the modal to load before we load the users list in the background. We can fix this with the **parallel** operator. Now the list of users and the single user load at the same time.

{% tabs %}
{% tab title="overmind/actions.js" %}
```typescript
import {pipe, parallel } from 'overmind'
import * as o from './operators'

export const showHomePage = o.setPage('HOME')

export const showUsersPage = pipe(
  o.setPage('USERS'),
  o.closeUserModal(),
  o.shouldLoadUsers(),
  o.loadUsers()
)

export const showUserModal = pipe(
  o.setPage('USERS'),
  parallel(
    o.loadUserWithDetails(),
    pipe(
      o.shouldLoadUsers(),
      o.loadUsers()
    ),
  )
)
```
{% endtab %}
{% endtabs %}

Now you are starting to see how the operators can be quite useful to compose flow. This flow is also reflected in the development tool of Overmind.

## The tab query param

**Page.js** also allows us to manage query strings, the stuff after the **?** in the url. Page.js does not parse it though, so we introduce a library which does just that, [QUERY-STRING](https://www.npmjs.com/package/query-string). With this we can update our router to also pass in any query params.

{% tabs %}
{% tab title="overmind/effects.js" %}
```typescript
import page from 'page'
import queryString from 'query-string'

export const router = {
  initialize(routes) {
    Object.keys(routes).forEach(url => {
      page(url, ({ params, querystring }) => {
        const payload = Object.assign({}, params, queryString.parse(querystring))

        routes[url](payload)
      })
    })
    page.start()
  },
  open: (url) => page.show(url)
}
```
{% endtab %}
{% endtabs %}

### Imperative approach

We now also handle the received tab parameter and make sure that when we change tabs we do not load the user again. We only want to load the user when there is no existing user or if the user has changed.

{% tabs %}
{% tab title="overmind/actions.js" %}
```typescript
export const showHomePage = ({ state }) => {
  state.currentPage = 'HOME'
}

export const showUsersPage = async ({ state, effects }) => {
  state.currentPage = 'USERS'
  state.modalUser = null

  if (!Object.keys(state.users).length) {
    state.isLoadingUsers = true
    state.users = await effects.api.getUsers()
    state.isLoadingUsers = false
  }
}

export const showUserModal = async ({ state, actions }, params) => {
  actions.showUsersPage()
  state.currentUserModalTabIndex = Number(params.tab)
  state.isLoadingUserDetails = true
  state.modalUser = await effects.api.getUserWithDetails(params.id)
  state.isLoadingUserDetails = false
}
```
{% endtab %}
{% endtabs %}

### Functional approach

Now we can add an operator which uses this **tab** query to set the current tab and then compose it into the action. We also add an operator to verify if we really should load a new user.

{% tabs %}
{% tab title="overmind/operators.ts" %}
```typescript
import { filter } from 'overmind'

export const closeUserModal = () => ({ state }) => {
  state.modalUser = null
}

export const setPage = (page) => ({ state }) => {
  state.currentPage = page
}

export const shouldLoadUsers = () => filter(({ state }) => {
  return !Boolean(state.users.length)
})

export const loadUsers = () => async ({ state, effects }) => {
  state.isLoadingUsers = true
  state.users = await effects.api.getUsers()
  state.isLoadingUsers = false
}

export const loadUserWithDetails = () => async ({ state, effects }, params) {
  state.isLoadingUserDetails = true
  state.modalUser = await effects.api.getUserWithDetails(params.id)
  state.isLoadingUserDetails = false
}

export const shouldLoadUserWithDetails = () => filter(({ state }, params) => {
  return !state.modalUser || state.modalUser.id !== params.id
})

export const setCurrentUserModalTabIndex = () => ({ state }, params) => {
  state.currentUserModalTabIndex = Number(params.tab)
}
```
{% endtab %}

{% tab title="overmind/actions.ts" %}
```typescript
import { Operator, pipe, parallel } from 'overmind'
import { Page } from './types'
import * as o from './operators'

export const showHomePage: Operator = o.setPage(Page.HOME)

export const showUsersPage: Operator = pipe(
  o.setPage(Page.USERS),
  o.closeUserModal(),
  o.shouldLoadUsers(),
  o.loadUsers()
)

export const showUserModal: Operator<{ id: string, tab: string }> = pipe(
  o.setPage(Page.USERS),
  parallel(
    pipe(
      o.setCurrentUserModalTabIndex(),
      o.shouldLoadUserWithDetails(),
      o.loadUserWithDetails()
    ),
    pipe(
      o.shouldLoadUsers(),
      o.loadUsers()
    )
  )
)
```
{% endtab %}
{% endtabs %}

## Summary

With little effort we were able to build a custom “**application state first**“ router for our application. Like many common tools needed in an application, like talking to the server, localStorage etc., there are often differences in the requirements. And even more often you do not need the full implementation of the tool you are using. By using simple tools you can meet the actual requirements of the application more “head on” and this was an example of that.

We also showed how you can solve this issue with an imperative approach or go functional. In this example functional is probably a bit overkill as there is very little composition required. But if your application needed to use these operators many times in different configurations you would benefit more from it.

