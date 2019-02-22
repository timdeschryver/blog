---
layout: post
title: NgRx — Best Practices for Enterprise Angular Applications
author: Wesley Grimes
subtitle: A collection of best practices for organizing your NgRx Store
date: 2018-05-30 10:00:0 -0500
categories: angular
tags: [angular, ngrx, javascript, redux]
excerpt: The following represents a pattern that I’ve developed at my day job after building several enterprise Angular applications using the NgRx library. I have found that most online tutorials do a great job of helping you to get your store up and running, but often fall short of illustrating best practices for clean separation of concerns between your store feature slices, root store, and user interface.
header-img: "assets/post_headers/ngrx_best_practices_header.png"
---

![](/assets/post_headers/ngrx_best_practices_header.png)

## Before We Get Started

This article is not intended to be a tutorial on **NgRx**. There are several great resources that currently exist, written by experts much smarter than me. I highly suggest that you take time and learn **NgRx** and the **redux** pattern before attempting to implement these concepts.

- [Ultimate Angular — NgRx Store & Effects](https://platform.ultimatecourses.com/p/ngrx-store-effects?affcode=76683_ttll_neb)
- [Pluralsight — Play by Play Angular NgRx](https://www.pluralsight.com/courses/play-by-play-angular-ngrx)
- [NgRx Blog on Medium.com](https://medium.com/ngrx)
- [NgRx.io Docs](https://ngrx.io/docs)
- [NgRx.io Resources](https://ngrx.io/resources)

## Background

The following represents a pattern that I've developed at my day job after building several enterprise Angular applications using the **NgRx** library. I have found that most online tutorials do a great job of helping you to get your store up and running, but often fall short of illustrating best practices for clean separation of concerns between your store feature slices, root store, and user interface.

With the following pattern, your root application state, and each slice (property) of that root application state are separated into a `RootStoreModule` and per feature `MyFeatureStoreModule`.

## Prerequisites

This article assumes that you are building an [Angular v7 CLI](https://cli.angular.io/) generated application.

## Installing NgRx Dependencies

Before we get started with generating code, let's make sure to install the necessary **NgRx** node modules from a prompt:

```shell
$ npm install @ngrx/{store,store-devtools,entity,effects}
```

## Best Practice #1 — The Root Store Module

Create a Root Store Module as a proper Angular **NgModule's** that bundle together NgRx store logic. Feature store modules will be imported into the Root Store Module allowing for a single root store module to be imported into your application's main App Module.

### Suggested Implementation

1.  Generate `RootStoreModule` using the **Angular CLI:**

```shell
$ ng g module root-store --flat false --module app.module.ts
```

2. Generate `RootState` interface to represent the entire state of your application using the **Angular CLI:**

```shell
$ ng g interface root-store/root-state
```

This will create an interface named `RootState` but you will need to rename it to `State` inside the generated `.ts` file as we want to later on utilize this as `RootStoreState.State`

PLEASE NOTE: You will come back later on and add to this interface each feature module as a property.

## Best Practice #2 — Create Feature Store Module(s)

Create feature store modules as proper Angular NgModule's that bundle together feature slices of your store, including `state`, `actions`, `reducer`, `selectors`, and `effects`. Feature modules are then imported into your `RootStoreModule`. This will keep your code cleanly organizing into sub-directories for each feature store. In addition, as illustrated later on in the article, public `actions`, `selectors`, and `state` are name-spaced and exported with feature store prefixes.

### Naming Your Feature Store

In the example implementation below we will use the feature name `MyFeature`, however, this will be different for each feature you generate and should closely mirror the `RootState` property name. For example, if you are building a blog application, a feature name might be `Post`.

### Entity Feature Modules or Standard Feature Modules?

Depending on the type of feature you are creating you may or may not benefit from implementing [NgRx Entity](https://medium.com/ngrx/introducing-ngrx-entity-598176456e15). If your store feature slice will be dealing with an array of type then I suggest following the _Entity Feature Module_ implementation below. If building a store feature slice that does not consist of a standard array of type, then I suggest following the _Standard Feature Module_ implementation below.

### Suggested Implementation — Entity Feature Module

1.  Generate `MyFeatureStoreModule` feature module using the **Angular CLI:**

```shell
$ ng g module root-store/my-feature-store --flat false --module root-store/root-store.module.ts
```

2. Actions — Create an `actions.ts` file in the `app/root-store/my-feature-store` directory:

```typescript
import { Action } from @ngrx/store;
import { MyModel } from '../../models';

export enum ActionTypes {
  LOAD_REQUEST = '[My Feature] Load Request',
  LOAD_FAILURE = '[My Feature] Load Failure',
  LOAD_SUCCESS = '[My Feature] Load Success'
}

export class LoadRequestAction implements Action {
  readonly type = ActionTypes.LOAD_REQUEST;
}

export class LoadFailureAction implements Action {
  readonly type = ActionTypes.LOAD_FAILURE;
  constructor(public payload: { error: string }) {}
}

export class LoadSuccessAction implements Action {
  readonly type = ActionTypes.LOAD_SUCCESS;
  constructor(public payload: { items: MyModel[] }) {}
}

export type Actions = LoadRequestAction | LoadFailureAction | LoadSuccessAction;
```

3. State — Create a `state.ts` file in the `app/root-store/my-feature-store` directory:

```typescript
import { createEntityAdapter, EntityAdapter, EntityState } from '@ngrx/entity';
import { MyModel } from '../../models';

export const featureAdapter: EntityAdapter<MyModel> = createEntityAdapter<
  MyModel
>({
  selectId: model => model.id,
  sortComparer: (a: MyModel, b: MyModel): number =>
    b.someDate.toString().localeCompare(a.someDate.toString())
});

export interface State extends EntityState<MyModel> {
  isLoading?: boolean;
  error?: any;
}

export const initialState: State = featureAdapter.getInitialState({
  isLoading: false,
  error: null
});
```

4. Reducer — Create a `reducer.ts` file in the `app/root-store/my-feature-store` directory:

```typescript
import { Actions, ActionTypes } from './actions';
import { featureAdapter, initialState, State } from './state';

export function featureReducer(state = initialState, action: Actions): State {
  switch (action.type) {
    case ActionTypes.LOAD_REQUEST: {
      return {
        ...state,
        isLoading: true,
        error: null
      };
    }
    case ActionTypes.LOAD_SUCCESS: {
      return featureAdapter.addAll(action.payload.items, {
        ...state,
        isLoading: false,
        error: null
      });
    }
    case ActionTypes.LOAD_FAILURE: {
      return {
        ...state,
        isLoading: false,
        error: action.payload.error
      };
    }
    default: {
      return state;
    }
  }
}
```

5. Selectors — Create a `selectors.ts` file in the `app/root-store/my-feature-store` directory:

```typescript
import {
  createFeatureSelector,
  createSelector,
  MemoizedSelector
} from '@ngrx/store';

import { MyModel } from '../models';

import { featureAdapter, State } from './state';

export const getError = (state: State): any => state.error;

export const getIsLoading = (state: State): boolean => state.isLoading;

export const selectMyFeatureState: MemoizedSelector<
  object,
  State
> = createFeatureSelector<State>('myFeature');

export const selectAllMyFeatureItems: (
  state: object
) => MyModel[] = featureAdapter.getSelectors(selectMyFeatureState).selectAll;

export const selectMyFeatureById = (id: string) =>
  createSelector(
    this.selectAllMyFeatureItems,
    (allMyFeatures: MyModel[]) => {
      if (allMyFeatures) {
        return allMyFeatures.find(p => p.id === id);
      } else {
        return null;
      }
    }
  );

export const selectMyFeatureError: MemoizedSelector<
  object,
  any
> = createSelector(
  selectMyFeatureState,
  getError
);

export const selectMyFeatureIsLoading: MemoizedSelector<
  object,
  boolean
> = createSelector(
  selectMyFeatureState,
  getIsLoading
);
```

6. Effects — Create an `effects.ts` file in the `app/root-store/my-feature-store` directory with the following:

```typescript
import { Injectable } from '@angular/core';
import { Actions, Effect, ofType } from '@ngrx/effects';
import { Action } from '@ngrx/store';
import { Observable, of as observableOf } from 'rxjs';
import { catchError, map, startWith, switchMap } from 'rxjs/operators';
import { DataService } from '../../services/data.service';
import * as featureActions from './actions';

@Injectable()
export class MyFeatureStoreEffects {
  constructor(private dataService: DataService, private actions$: Actions) {}

  @Effect()
  loadRequestEffect$: Observable<Action> = this.actions$.pipe(
    ofType<featureActions.LoadRequestAction>(
      featureActions.ActionTypes.LOAD_REQUEST
    ),
    startWith(new featureActions.LoadRequestAction()),
    switchMap(action =>
      this.dataService.getItems().pipe(
        map(
          items =>
            new featureActions.LoadSuccessAction({
              items
            })
        ),
        catchError(error =>
          observableOf(new featureActions.LoadFailureAction({ error }))
        )
      )
    )
  );
}
```

#### Suggested Implementation — Standard Feature Module

1.  Generate `MyFeatureStoreModule` feature module using the **Angular CLI:**

```shell
$ ng g module root-store/my-feature-store --flat false --module root-store/root-store.module.ts
```

2. Actions — Create an `actions.ts` file in the `app/root-store/my-feature-store` directory:

```typescript
import { Action } from '@ngrx/store';
import { User } from '../../models';

export enum ActionTypes {
  LOGIN_REQUEST = '[My Feature] Login Request',
  LOGIN_FAILURE = '[My Feature] Login Failure',
  LOGIN_SUCCESS = '[My Feature] Login Success'
}

export class LoginRequestAction implements Action {
  readonly type = ActionTypes.LOGIN_REQUEST;
  constructor(public payload: { userName: string; password: string }) {}
}

export class LoginFailureAction implements Action {
  readonly type = ActionTypes.LOGIN_FAILURE;
  constructor(public payload: { error: string }) {}
}

export class LoginSuccessAction implements Action {
  readonly type = ActionTypes.LOGIN_SUCCESS;
  constructor(public payload: { user: User }) {}
}

export type Actions =
  | LoginRequestAction
  | LoginFailureAction
  | LoginSuccessAction;
```

3. State — Create a `state.ts` file in the `app/root-store/my-feature-store` directory:

```typescript
import { User } from '../../models';

export interface State {
  user: User | null;
  isLoading: boolean;
  error: string;
}

export const initialState: State = {
  user: null,
  isLoading: false,
  error: null
};
```

4. Reducer — Create a `reducer.ts` file in the `app/root-store/my-feature-store` directory:

```typescript
import { Actions, ActionTypes } from './actions';
import { initialState, State } from './state';

export function featureReducer(state = initialState, action: Actions): State {
  switch (action.type) {
    case ActionTypes.LOGIN_REQUEST:
      return {
        ...state,
        error: null,
        isLoading: true
      };
    case ActionTypes.LOGIN_SUCCESS:
      return {
        ...state,
        user: action.payload.user,
        error: null,
        isLoading: false
      };
    case ActionTypes.LOGIN_FAILURE:
      return {
        ...state,
        error: action.payload.error,
        isLoading: false
      };
    default: {
      return state;
    }
  }
}
```

5. Selectors — Create a `selectors.ts` file in the `app/root-store/my-feature-store` directory:

```typescript
import {
  createFeatureSelector,
  createSelector,
  MemoizedSelector
} from '@ngrx/store';

import { User } from '../../models';

import { State } from './state';

const getError = (state: State): any => state.error;

const getIsLoading = (state: State): boolean => state.isLoading;

const getUser = (state: State): any => state.user;

export const selectMyFeatureState: MemoizedSelector<
  object,
  State
> = createFeatureSelector<State>('myFeature');

export const selectMyFeatureError: MemoizedSelector<
  object,
  any
> = createSelector(
  selectMyFeatureState,
  getError
);

export const selectMyFeatureIsLoading: MemoizedSelector<
  object,
  boolean
> = createSelector(
  selectMyFeatureState,
  getIsLoading
);

export const selectMyFeatureUser: MemoizedSelector<
  object,
  User
> = createSelector(
  selectMyFeatureState,
  getUser
);
```

6. Effects — Create an `effects.ts` file in the `app/root-store/my-feature-store` directory with the following:

```typescript
import { Injectable } from '@angular/core';
import { Actions, Effect, ofType } from '@ngrx/effects';
import { Action } from '@ngrx/store';
import { Observable, of as observableOf } from 'rxjs';
import { catchError, map, startWith, switchMap } from 'rxjs/operators';
import { DataService } from '../../services/data.service';
import * as featureActions from './actions';

@Injectable()
export class MyFeatureStoreEffects {
  constructor(private dataService: DataService, private actions$: Actions) {}

  @Effect()
  loginRequestEffect$: Observable<Action> = this.actions$.pipe(
    ofType<featureActions.LoginRequestAction>(
      featureActions.ActionTypes.LOGIN_REQUEST
    ),
    switchMap(action =>
      this.dataService
        .login(action.payload.userName, action.payload.password)
        .pipe(
          map(
            user =>
              new featureActions.LoginSuccessAction({
                user
              })
          ),
          catchError(error =>
            observableOf(new featureActions.LoginFailureAction({ error }))
          )
        )
    )
  );
}
```

#### Suggested Implementation — Entity and Standard Feature Modules

Now that we have created our feature module, either Entity or Standard typed above, we need to import the parts (state, actions, reducer, effects, selectors) into the Angular NgModule for the feature. In addition, we will create a barrel export in order to make imports in our application components clean and orderly, with asserted name-spaces.

1.  Update the `app/root-store/my-feature-store/my-feature-store.module.ts` with the following:

```typescript
import { CommonModule } from '@angular/common';
import { NgModule } from '@angular/core';
import { EffectsModule } from '@ngrx/effects';
import { StoreModule } from '@ngrx/store';
import { MyFeatureStoreEffects } from './effects';
import { featureReducer } from './reducer';

@NgModule({
  imports: [
    CommonModule,
    StoreModule.forFeature('myFeature', featureReducer),
    EffectsModule.forFeature([MyFeatureStoreEffects])
  ],
  providers: [MyFeatureStoreEffects]
})
export class MyFeatureStoreModule {}
```

2. Create an `app/root-store/my-feature-store/index.ts` [barrel export](https://twitter.com/toddmotto/status/918818392680824832). You will notice that we import our store components and alias them before re-exporting them. This in essence is "name-spacing" our store components.

```typescript
import * as MyFeatureStoreActions from './actions';
import * as MyFeatureStoreSelectors from './selectors';
import * as MyFeatureStoreState from './state';

export { MyFeatureStoreModule } from './my-feature-store.module';

export { MyFeatureStoreActions, MyFeatureStoreSelectors, MyFeatureStoreState };
```

## Best Practice #1 — The Root Store Module (cont.)

Now that we have built our feature modules, let's pick up where we left off in best practice ##1 and finish building out our `RootStoreModule` and `RootState.`

### Suggested Implementation (cont.)

3. Update `app/root-store/root-state.ts` and add a property for each feature that we have created previously:

```typescript
import { MyFeatureStoreState } from './my-feature-store';
import { MyOtherFeatureStoreState } from './my-other-feature-store';

export interface State {
  myFeature: MyFeatureStoreState.State;
  myOtherFeature: MyOtherFeatureStoreState.State;
}
```

4. Update your `app/root-store/root-store.module.ts` by importing all feature modules, and importing the following **NgRx** modules: `StoreModule.forRoot({})` and `EffectsModule.forRoot([])`:

```typescript
import { CommonModule } from '@angular/common';
import { NgModule } from '@angular/core';
import { EffectsModule } from '@ngrx/effects';
import { StoreModule } from '@ngrx/store';
import { MyFeatureStoreModule } from './my-feature-store/';
import { MyOtherFeatureStoreModule } from './my-other-feature-store/';

@NgModule({
  imports: [
    CommonModule,
    MyFeatureStoreModule,
    MyOtherFeatureStoreModule,
    StoreModule.forRoot({}),
    EffectsModule.forRoot([])
  ],
  declarations: []
})
export class RootStoreModule {}
```

5. Create an `app/root-store/selectors.ts` file. This will hold any root state level selectors, such as a Loading property, or even an aggregate Error property:

```typescript
import { createSelector, MemoizedSelector } from '@ngrx/store';
import { MyFeatureStoreSelectors } from './my-feature-store';

import { MyOtherFeatureStoreSelectors } from './my-other-feature-store';

export const selectError: MemoizedSelector<object, string> = createSelector(
  MyFeatureStoreSelectors.selectMyFeatureError,
  MyOtherFeatureStoreSelectors.selectMyOtherFeatureError,
  (myFeatureError: string, myOtherFeatureError: string) => {
    return myFeature || myOtherFeature;
  }
);

export const selectIsLoading: MemoizedSelector<
  object,
  boolean
> = createSelector(
  MyFeatureStoreSelectors.selectMyFeatureIsLoading,
  MyOtherFeatureStoreSelectors.selectMyOtherFeatureIsLoading,
  (myFeature: boolean, myOtherFeature: boolean) => {
    return myFeature || myOtherFeature;
  }
);
```

6. Create an `app/root-store/index.ts` barrel export for your store with the following:

```typescript
import { RootStoreModule } from './root-store.module';
import * as RootStoreSelectors from './selectors';
import * as RootStoreState from './state';
export * from './my-feature-store';
export * from './my-other-feature-store';
export { RootStoreState, RootStoreSelectors, RootStoreModule };
```

## Wiring up the Root Store Module to your Application

Now that we have built our Root Store Module, composed of Feature Store Modules, let's add it to the main `app.module.ts` and show just how neat and clean the wiring up process is.

1.  Add `RootStoreModule` to your application's `NgModule.imports` array. Make sure that when you import the module to pull from the barrel export:

```typescript
import { RootStoreModule } from './root-store';
```

2. Here's an example `container` component that is using the store:

```typescript
import { Component, OnInit } from '@angular/core';
import { Store } from '@ngrx/store';
import { Observable } from 'rxjs';
import { MyModel } from '../../models';
import {
  RootStoreState,
  MyFeatureStoreActions,
  MyFeatureStoreSelectors
} from '../../root-store';

@Component({
  selector: 'app-my-feature',
  styleUrls: ['my-feature.component.css'],
  templateUrl: './my-feature.component.html'
})
export class MyFeatureComponent implements OnInit {
  myFeatureItems$: Observable<MyModel[]>;
  error$: Observable<string>;
  isLoading$: Observable<boolean>;

  constructor(private store$: Store<RootStoreState.State>) {}

  ngOnInit() {
    this.myFeatureItems$ = this.store$.select(
      MyFeatureStoreSelectors.selectAllMyFeatureItems
    );

    this.error$ = this.store$.select(
      MyFeatureStoreSelectors.selectUnProcessedDocumentError
    );

    this.isLoading$ = this.store$.select(
      MyFeatureStoreSelectors.selectUnProcessedDocumentIsLoading
    );

    this.store$.dispatch(new MyFeatureStoreActions.LoadRequestAction());
  }
}
```

## Finished Application Structure

Once we have completed implementation of the above best practices our Angular application structure should look very similar to something like this:

```shell
 ├── app
 │ ├── app-routing.module.ts
 │ ├── app.component.css
 │ ├── app.component.html
 │ ├── app.component.ts
 │ ├── app.module.ts
 │ ├── components
 │ ├── containers
 │ │    └── my-feature
 │ │         ├── my-feature.component.css
 │ │         ├── my-feature.component.html
 │ │         └── my-feature.component.ts
 │ ├── models
 │ │    ├── index.ts
 │ │    └── my-model.ts
 │ │    └── user.ts
 │ ├── root-store
 │ │    ├── index.ts
 │ │    ├── root-store.module.ts
 │ │    ├── selectors.ts
 │ │    ├── state.ts
 │ │    └── my-feature-store
 │ │    |    ├── actions.ts
 │ │    |    ├── effects.ts
 │ │    |    ├── index.ts
 │ │    |    ├── reducer.ts
 │ │    |    ├── selectors.ts
 │ │    |    ├── state.ts
 │ │    |    └── my-feature-store.module.ts
 │ │    └── my-other-feature-store
 │ │         ├── actions.ts
 │ │         ├── effects.ts
 │ │         ├── index.ts
 │ │         ├── reducer.ts
 │ │         ├── selectors.ts
 │ │         ├── state.ts
 │ │         └── my-other-feature-store.module.ts
 │ └── services
 │      └── data.service.ts
 ├── assets
 ├── browserslist
 ├── environments
 │ ├── environment.prod.ts
 │ └── environment.ts
 ├── index.html
 ├── main.ts
 ├── polyfills.ts
 ├── styles.css
 ├── test.ts
 ├── tsconfig.app.json
 ├── tsconfig.spec.json
 └── tslint.json
```

## Fully Working Example — Chuck Norris Joke Generator

I have put together a fully working example of the above best practices. It's a simple Chuck Norris Joke Generator that has uses `@angular/material` and the [http://www.icndb.com/](http://www.icndb.com/) api for data.

### Github

[https://github.com/wesleygrimes/angular-ngrx-chuck-norris](https://github.com/wesleygrimes/angular-ngrx-chuck-norris)

### Stackblitz

You can see the live demo at [https://angular-ngrx-chuck-norris.stackblitz.io](https://angular-ngrx-chuck-norris.stackblitz.io) and here is the [Stackblitz](https://stackblitz.com) editor:

[angular-ngrx-chuck-norris - StackBlitz](https://stackblitz.com/edit/angular-ngrx-chuck-norris)

## Conclusion

It's important to remember that I have implemented these best practices in several "real world" applications. While I have found these best practices helpful, and maintainable, I do not believe they are an end-all be-all solution to organizing NgRx projects; it's just what has worked for me. I am curious as to what you all think? Please feel free to offer any suggestions, tips, or best practices you've learned when building enterprise Angular applications with NgRx and I will update the article to reflect as such. Happy Coding!

---

## Additional Resources

I would highly recommend enrolling in the Ultimate Angular courses, especially the NgRx course. It is well worth the money and I have used it as a training tool for new Angular developers. Follow the link below to signup.

[Ultimate Courses: Expert online courses in JavaScript, Angular, NGRX and TypeScript](https://bit.ly/2WubqhW)