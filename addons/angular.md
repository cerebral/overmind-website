# Angular

Let us have a look at how you configure your app:

{% tabs %}
{% tab title="overmind/index.ts" %}
```typescript
import { IConfig } from 'overmind'
import { Injectable } from '@angular/core'
import { OvermindService } from 'overmind-angular'
import { state } from './state'
import * as actions from './actions'

export const config = { state, actions }

declare module 'overmind' {
  interface Config extends IConfig<typeof config> {}
}

@Injectable({
  providedIn: 'root'
})
export class Store extends OvermindService<typeof config> {}
```
{% endtab %}

{% tab title="app.module.ts" %}
```typescript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { createOvermind } from 'overmind';
import { OvermindModule, OvermindService, OVERMIND_INSTANCE } from 'overmind-angular'

import { config, Store } from './overmind'
import { AppComponent } from './app.component';

@NgModule({
  imports: [ BrowserModule, OvermindModule ],
  declarations: [ AppComponent ],
  bootstrap: [ AppComponent ],
  providers: [
    { provide: OVERMIND_INSTANCE, useValue: createOvermind(config) },
    { provide: Store, useExisting: OvermindService }
]
})
export class AppModule { }

```
{% endtab %}

{% tab title="components/app.component.ts" %}
```typescript
import { Component, ChangeDetectionStrategy } from '@angular/core'
import { Store } from '../overmind'

@Component({
  selector: 'app-root',
  template: `
<div *track>
  <h1 (click)="actions.changeTitle()">{{ state.title }}</h1>
</div>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class AppComponent {
  state = this.store.select()
  actions = this.store.actions
  constructor(private store: Store) {}
}
```
{% endtab %}
{% endtabs %}

The **service** is responsible for exposing the configuration of your application. The **\*track** directive is what does the actual tracking. Just put it at the top of your template and whatever state you access will be optimally tracked. You can also select a namespace from your state to expose to the component:

{% tabs %}
{% tab title="components/app.component.ts" %}
```typescript
import { Component, ChangeDetectionStrategy } from '@angular/core'
import { Store } from '../overmind'

@Component({
  selector: 'app-root',
  template: `
<div *track>
  <h1 (click)="actions.changeAdminTitle()">{{ state.adminTitle }}</h1>
</div>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class AppComponent {
  state = this.store.select(state => state.admin)
  actions = this.store.actions.admin
  constructor(private store: Store) {}
},
```
{% endtab %}
{% endtabs %}

You can now access the **admin** state and actions directly with **state** and **actions**.

## NgZone

Since Overmind knows when your components should update you can safely turn **ngZone** to `"noop"`. Note that other 3rd party libraries may not support this.

## Rendering

When you connect Overmind to your component and expose state you do not have to think about how much state you expose. The exact state that is being accessed in the template is the state that will be tracked. That means you can expose all the state of the application to all your components without worrying about performance.

## Passing state as input

When you pass state objects or arrays as input to a child component that state will by default be tracked on the component passing it along, which you can also see in the devtools. By just adding the **\*tracker** directive to the child template, the tracking will be handed over:

{% tabs %}
{% tab title="components/todo.component.ts" %}
```typescript
import { Component, Input, ChangeDetectionStrategy } from '@angular/core'
import { Todo } from '../overmind/state'

@Component({
  selector: 'todos-todo',
  template: `
<li *track>{{ todo.title }}</li>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class TodoComponent {
  @Input() todo: Todo
}
```
{% endtab %}

{% tab title="components/todos.component.ts" %}
```typescript
import { Component, ChangeDetectionStrategy } from '@angular/core'
import { Store } from '../overmind'

@Component({
  selector: 'todos-list',
  template: `
<ul *track>
  <todos-todo *ngFor="let todo of state.todos;" [todo]="todo"></todos-todo>
</ul>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class ListComponent {
  state = this.store.select()
  constructor(private store: Store) {}
}
```
{% endtab %}
{% endtabs %}

What is important to understand here is that Overmind is **not** immutable. That means if you would change any property on any todo, only the component actually looking at the todo will render. The list is untouched.

## Reactions

To run effects in components based on changes to state you use the **reaction** function in the lifecycle hooks of Angular.

{% tabs %}
{% tab title="components/app.component.ts" %}
```typescript
import { Component } from '@angular/core'
import { Store } from '../overmind'

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html'
})
export class AppComponent {
  disposeReaction: () => void
  constructor (private store: Store) {}
  ngOnInit() {
    this.disposeReaction = this.store.reaction(
      ({ currentPage }) => currentPage,
      () => document.querySelector('#app').scrollTop = 0
    )
  }
  ngOnDestroy() {
    this.disposeReaction()
  }
}
```
{% endtab %}
{% endtabs %}

