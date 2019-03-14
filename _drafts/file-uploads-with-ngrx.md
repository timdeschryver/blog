## Managing File Uploads With NgRx

In this article we will build a fully-functional file upload control, that is powered by **Angular** and is backed by an **NgRx** feature store. The control will provide the user with the following features: 

* The ability to upload files using the `<input #file type="file" />` html element.
* The ability to see an accurate upload progress via the `reportProgress` `HttpClient` option.
* The ability to cancel in-process uploads

As an added bonus, we will briefly dive into building the *server-side* ASP.NET Core WebAPI Controller that will handle the file uploads.

## Before We Get Started

In this article, I will show you how to manage file uploads using NgRx. If you are new to NgRx, then I highly recommend that you first read my article, [NgRx - Best Practices for Enterprise Angular Applications](https://wesleygrimes.com/angular/2018/05/30/ngrx-best-practices-for-enterprise-angular-applications.html). We will be using the techniques described in that article to build out the NgRx components for file uploads.

If you are new to Angular, then I recommend that you check out one of the following resources:

* [Ultimate Courses](https://bit.ly/2WubqhW)
* [Official Angular Docs](https://angular.io/guide/router)
* [NgRx Docs](https://ngrx.io/docs)


## NPM Package Versions

For context, this article assumes you are using the following `npm` `package.json` versions:

* `@angular/*`: 7.2.9
* `@ngrx/*`: 7.3.0

## Prerequisites

Before diving into building the file upload control, make sure that you have the following in place:

1. An Angular 7+ application generated
2. NgRx dependencies installed
3. NgRx Store wired up in your application. [e.g. Follow this guide](https://wesleygrimes.com/angular/2018/05/30/ngrx-best-practices-for-enterprise-angular-applications.html)

## Add the Upload File Control to your Component

!!TODO

## Create the Upload File Service

Let's create a brand new service in `Angular`. This service will be responsible for handling the file upload from the client to the server backend. We will use the amazing [`HttpClient`](https://angular.io/guide/http) provided with `Angular`. 


### Generate the service using the Angular CLI

> [Click here](https://angular.io/cli) for more details on using the powerful Angular CLI

```shell
$ ng g service file-upload
```

### Inject the HttpClient

Because we are using the `HttpClient` to make requests to the backend, we need to inject it into our service. Update `constructor` line of code so that it looks as follows:

```typescript
constructor(private httpClient: HttpClient) {}
```

### Add a private field for `_baseUrl`

> I typically store `API` base urls in the `src/environments` area. If you're interested in learning more about `environments` in `Angular` then check out this great article: [Becoming an Angular Environmentalist](https://blog.angularindepth.com/becoming-an-angular-environmentalist-45a48f7c20d8)

Let's create a new private field named `_baseUrl` so that we can use this in our calls to the backend `API`. 

One way to accomplish this would be to do the following:

```typescript
import { environment } from 'src/environments/environment';
...
private _baseUrl = environment.apiBaseUrl;
```

### Add a uploadFile public method

Let's create a new public method named `uploadFile` to the service. The method will take in a parameter `file: File` and return an `Observable<HttpEvent<{}>>`.

> Typically a `get` or `post` `Observable<>` is returned from a service like this. However, in this situation we are going to actually return the raw `request` which is an `Observable<HttpEvent<{}>>`. 

> By returning a raw `request` we have more control over the process, to pass options like `reportProgress` and allow cancellation of a `request`.

```typescript
public uploadFile(file: File): Observable<HttpEvent<{}>> {
  const formData = new FormData();
  formData.append('files', file, file.name);

  const options = {
    reportProgress: true
  };

  const req = new HttpRequest(
    'POST',
    `${this._baseUrl}api/file`,
    formData,
    options
  );
  return this.httpClient.request(req);
}
```

### Completed File Upload Service

The completed `file-upload.service.ts` will look as follows:

```typescript
import { HttpClient, HttpEvent, HttpRequest } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { Observable } from 'rxjs';
import { environment } from 'src/environments/environment';

@Injectable({
  providedIn: 'root'
})
export class FileUploadService {
  private _baseUrl = environment.apiBaseUrl;

  constructor(private httpClient: HttpClient) {}

  public uploadFile(file: File): Observable<HttpEvent<{}>> {
    const formData = new FormData();
    formData.append('files', file, file.name);

    const options = {
      reportProgress: true
    };

    const req = new HttpRequest(
      'POST',
      `${this._baseUrl}api/file`,
      formData,
      options
    );
    return this.httpClient.request(req);
  }
}
```

## Create the Upload File Feature Store

To keep your **NgRx** store organized, I recommend creating a separate Upload File Feature Store. Let's bundle it all together in a module named `upload-file-store.module.ts` and keep it under a sub-directory named `upload-file-store`.

When we are finished building out the store, the folder will look as follows:

! TODO INSERT FOLDER STRUCTURE HERE

### Create Feature Store Module

Create a feature store module using the following command: 

```shell
$ ng g module upload-file-store --flat false
```

### Create State Interface

Create a new file underneath the `upload-file-store` folder, named `state.ts`. The contents of the file will be as follows:

> These fields were specifically chosen to keep track of the upload process.

```typescript
export interface State {
  completed: boolean;
  isLoading: boolean;
  error: any | null;
  progress: number | null;
  cancel: boolean;
}

export const initialState: State = {
  completed: false,
  isLoading: false,
  error: null,
  progress: null,
  cancel: false
};
```

### Create Feature Actions

> If you would like to learn more about **NgRx Actions**, then check out the [official docs](https://ngrx.io/guide/store/actions).

Create a new file underneath the `upload-file-store` folder, named `actions.ts`. This file will hold the actions we want to make available on this store. 

We will create the following actions on our feature store:

* `UPLOAD_REQUEST` - This action is dispatched from the file upload form, it's payload will contain the actual `File` being uploaded.

* `UPLOAD_RESET` - This action is dispatched from the file upload form when the reset button is clicked. This will be used to reset the state of the store to defaults.

* `UPLOAD_CANCEL` - This action is dispatched from the file upload form when the cancel button is clicked. This will be used to cancel uploads in progress.

* `UPLOAD_FAILURE` - This action is dispatched from the file upload effect when the API returns an error. The payload will contain the specific error message returned from the API and place it into an `error` field on the store.

* `UPLOAD_SUCCESS` - This action is dispatched from the file upload effect when the API returns a success. There is no payload as the API just returns a `200 OK` repsonse.

* `UPLOAD_PROGRESS` - This action is dispatched from the file upload effect, `HttpClient` when the API reports progress. The payload will container the progress percentage as a whole number.

The final `actions.ts` file will look as follows:

```typescript
import { Action } from '@ngrx/store';

export enum ActionTypes {
  UPLOAD_REQUEST = '[File Upload Form] Request',
  UPLOAD_RESET = '[File Upload Form] Reset',
  UPLOAD_CANCEL = '[File Upload Form] Cancel',
  UPLOAD_FAILURE = '[File Upload API] Failure',
  UPLOAD_SUCCESS = '[File Upload API] Success',
  UPLOAD_PROGRESS = '[File Upload API] Progress',
}

export class UploadRequestAction implements Action {
  readonly type = ActionTypes.UPLOAD_REQUEST;
  constructor(public payload: { files: File }) {}
}

export class UploadFailureAction implements Action {
  readonly type = ActionTypes.UPLOAD_FAILURE;
  constructor(public payload: { error: any }) {}
}

export class UploadSuccessAction implements Action {
  readonly type = ActionTypes.UPLOAD_SUCCESS;
}

export class UploadProgressAction implements Action {
  readonly type = ActionTypes.UPLOAD_PROGRESS;
  constructor(public payload: { progress: number }) {}
}

export class UploadResetAction implements Action {
  readonly type = ActionTypes.UPLOAD_RESET;
}

export class UploadCancelAction implements Action {
  readonly type = ActionTypes.UPLOAD_CANCEL;
}

export type Actions =
  | UploadRequestAction
  | UploadFailureAction
  | UploadSuccessAction
  | UploadProgressAction
  | UploadResetAction
  | UploadCancelAction;
```

### Create the Feature Reducer

> If you would like to learn more about **NgRx Reducers**, then check out the [official docs](https://ngrx.io/guide/store/reducers).

Create a new file underneath the `upload-file-store` folder, named `reducer.ts`. This file will hold the reducer we create to manage state transitions to the store. 

We will handle state transitions as follows for the aforementioned actions:

* `UPLOAD_REQUEST` - Reset the state, with the exception of setting `state.isLoading` to `true`.

* `UPLOAD_RESET` - Reset the state tree on this action.

* `UPLOAD_CANCEL` - Reset the state tree, with the exception of setting `state.cancel` to `true` so that our `filter` pipe is triggered and short-circuits the `mergeMap` in the `uploadRequestEffect$` that will be defined later on in the article.

* `UPLOAD_FAILURE` - Reset the state tree, with the exception of setting `state.error` to the `error` that was throw in the `catchError` from the `API` in the `uploadRequestEffect` effect.

* `UPLOAD_PROGRESS` - Set `state.progress` to the current `action.payload.progress` provided from the action.

* `UPLOAD_SUCCESS` - Reset the state tree, with the exception of setting `state.completed` to `true` so that the UI can display a success message.


```typescript
import { Actions, ActionTypes } from './actions';
import { initialState, State } from './state';

export function featureReducer(state = initialState, action: Actions): State {
  switch (action.type) {
    case ActionTypes.UPLOAD_REQUEST: {
      return {
        ...state,
        isLoading: true,
        completed: false,
        progress: null,
        error: null,
        cancel: false
      };
    }
    case ActionTypes.UPLOAD_RESET: {
      return {
        ...state,
        isLoading: false,
        completed: false,
        progress: null,
        error: null,
        cancel: false
      };
    }
    case ActionTypes.UPLOAD_CANCEL: {
      return {
        ...state,
        isLoading: false,
        completed: false,
        progress: null,
        error: null,
        cancel: true
      };
    }
    case ActionTypes.UPLOAD_FAILURE: {
      return {
        ...state,
        isLoading: false,
        completed: false,
        error: action.payload.error,
        progress: null,
        cancel: false
      };
    }
    case ActionTypes.UPLOAD_PROGRESS: {
      return {
        ...state,
        progress: action.payload.progress
      };
    }
    case ActionTypes.UPLOAD_SUCCESS: {
      return {
        ...state,
        isLoading: false,
        completed: true,
        progress: null,
        error: null,
        cancel: false
      };
    }
    default: {
      return state;
    }
  }
}
```

### Create the Feature Effects

> If you would like to learn more about **NgRx Effects**, then check out the [official docs](https://ngrx.io/guide/effects).

Create a new file underneath the `upload-file-store` folder, named `effects.ts`. This file will hold the effects that we create to handle any side-effect calls to the backend `API` service.  This effect is where most of the magic happens in the application. 

#### Inject Dependencies

Let's add the necessary dependencies to our `constructor` as follows:

```typescript
constructor(
    private fileUploadService: FileUploadService,
    private actions$: Actions,
    private store$: Store<RootStoreState.State>
  ) {}
```

#### Add a new Upload Request Effect

> Effects make heavy-use of `rxjs` concepts and topics. If you are new to `rxjs` then I suggest you check out the [official docs](https://rxjs.dev)

Let's create a new effect in the file named `uploadRequestEffect$`. 

A couple comments about what this effect is going to do:

* Listen for the `UPLOAD_REQUEST` action and then make calls to the `fileUploadService.uploadFile` service method to initiate the upload process.

* Use the [`concatMap`](https://rxjs.dev/api/operators/concatMap) RxJS operator here so that multiple file upload requests are queued up and processed in the order they were dispatched.

* Use the [`takeUntil`](https://rxjs.dev/api/operators/takeUntil) RxJS operator connected to the `state.cancel` property, so that we can **short-circuit** any requests that are in-flight.

* Use the [`map`](https://rxjs.dev/api/operators/map) RxJS operator to route specific `HttpEvent` responses to dispatch specific `Actions` that we have defined in our `Store`.

* Use the [`catchError`](https://rxjs.dev/api/operators/catchError) RxJS operator to handle any errors that may be thrown from the `HttpClient`.

The effect will look something like this:

```typescript
@Effect()
uploadRequestEffect$: Observable<Action> = this.actions$.pipe(
  ofType<featureActions.UploadRequestAction>(
    featureActions.ActionTypes.UPLOAD_REQUEST
  ),
  concatMap(action =>
    this.fileUploadService.uploadFile(action.payload.file).pipe(
      takeUntil(
        this.store$
          .select(featureSelectors.selectUploadDocumentCancelRequest)
          .pipe(
            filter(
              cancel =>
                cancel !== null && cancel !== undefined && cancel === true
            )
          )
      ),
      map(event => this.handleProgress(event)),
      catchError(error => of(this.handleError(error)))
    )
  )
);
```

#### Add the handleProgress private method

> For more information on listening to progress events, check out the [official docs guide from here](https://angular.io/guide/http#listening-to-progress-events).

This method will be responsible for mapping specific `HttpEventType` to `Actions` that are dispatched.

* `HttpEventType.Sent` - This event occurs when the upload process has begun. We will dispatch an `UPLOAD_PROGRESS` action with a payload of `progress: 0` to denote that the process has begun.

* `HttpEventType.UploadProgress` - This event occurs when the upload process has made progress. We will dispatch an `UPLOAD_PROGRESS` action with a payload of `progress: Math.round((100 * event.loaded) / event.total)` to calculate the actual percentage complete of upload. This is because the `HttpClient` returns an `event.loaded` and `event.total` property in whole number format.

* `HttpEventType.Response` - This event occurs when the upload process has finished. It is important to note that this could be a success or failure so we need to interrogate the `event.status` to check for `200`. We will dispatch the `UPLOAD_SUCCESS` action if `event.status === 200` and `UPLOAD_FAILURE` if the `event.status !== 200` passing the `event.statusText` as the error payload.

```typescript
private handleProgress(event: HttpEvent<any>) {
  switch (event.type) {
    case HttpEventType.Sent: {
      return new featureActions.UploadProgressAction({ progress: 0 });
    }
    case HttpEventType.UploadProgress: {
      return new featureActions.UploadProgressAction({
        progress: Math.round((100 * event.loaded) / event.total)
      });
    }
    case HttpEventType.Response: {
      if (event.status === 200) {
        return new featureActions.UploadSuccessAction();
      } else {
        return new featureActions.UploadFailureAction({
          error: event.statusText
        });
      }
    }
    default: {
      return new featureActions.UploadProgressAction({ progress: 0 });
    }
  }
}
```

#### Add the handleError private method

> For more information on handling `HttpClient` errors, check out the [official docs guide from here](https://angular.io/guide/http#getting-error-details).

This method will be responsible for handling any errors that may be throw from the `HttpClient` during requests.

```typescript
private handleError(error: HttpErrorResponse) {
  if (error.error instanceof ErrorEvent) {
    // A client-side or network error occurred. Handle it accordingly.
    return new featureActions.UploadFailureAction({
      error: error.error.message
    });
  } else {
    // The backend returned an unsuccessful response code.
    // The response body may contain clues as to what went wrong,
    return new featureActions.UploadFailureAction({
      error: error.error
    });
  }
}
```

#### Completed Feature Effect

The completed effect will look something like this:

```typescript
import { HttpEvent, HttpEventType } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { Actions, Effect, ofType } from '@ngrx/effects';
import { Action, Store } from '@ngrx/store';
import { Observable } from 'rxjs';
import { concatMap, filter, map, takeUntil } from 'rxjs/operators';
import { RootStoreState } from '..';
import { FileUploadService } from '../../services';
import * as featureActions from './actions';
import * as featureSelectors from './selectors';

@Injectable()
export class UploadDocumentEffects {
  constructor(
    private fileUploadService: FileUploadService,
    private actions$: Actions,
    private store$: Store<RootStoreState.State>
  ) {}

  @Effect()
  uploadRequestEffect$: Observable<Action> = this.actions$.pipe(
    ofType<featureActions.UploadRequestAction>(
      featureActions.ActionTypes.UPLOAD_REQUEST
    ),
    concatMap(action =>
      this.fileUploadService.uploadFile(action.payload.file).pipe(
        takeUntil(
          this.store$
            .select(featureSelectors.selectUploadDocumentCancelRequest)
            .pipe(
              filter(
                cancel =>
                  cancel !== null && cancel !== undefined && cancel === true
              )
            )
        ),
        map(event => this.onUploadProgress(event))
      )
    )
  );

  private onUploadProgress(event: HttpEvent<any>) {
    switch (event.type) {
      case HttpEventType.Sent: {
        return new featureActions.UploadProgressAction({ progress: 0 });
      }
      case HttpEventType.UploadProgress: {
        return new featureActions.UploadProgressAction({
          progress: Math.round((100 * event.loaded) / event.total)
        });
      }
      case HttpEventType.Response: {
        if (event.status === 200) {
          return new featureActions.UploadSuccessAction();
        } else {
          return new featureActions.UploadFailureAction({
            error: event.statusText
          });
        }
      }
      default: {
        return new featureActions.UploadProgressAction({ progress: 0 });
      }
    }
  }
}
```
### Create the Feature Selectors

> If you would like to learn more about **NgRx Selectors**, then check out the [official docs](https://ngrx.io/guide/store/selectors).

Create a new file underneath the `upload-file-store` folder, named `selectors.ts`. This file will hold the selectors we will use to pull specific pieces of state out of the store. These are not necessary, but make for a cleaner interaction with the store from the UI components.

We will create a selector for each significant property of state. This includes the following properties: 

* `state.error`
* `state.isLoading`
* `state.completed`
* `state.progress`
* `state.cancel`

The completed selectors file will look something like the following: 

```typescript
import {
  createFeatureSelector,
  createSelector,
  MemoizedSelector
} from '@ngrx/store';
import { State } from './state';

export const getError = (state: State): any => state.error;

export const getIsLoading = (state: State): boolean => state.isLoading;

export const getCompleted = (state: State): boolean => state.completed;

export const getProgress = (state: State): number => state.progress;

export const getCancelRequest = (state: State): boolean => state.cancel;

export const selectUploadDocumentFeatureState: MemoizedSelector<
  object,
  State
> = createFeatureSelector<State>('uploadFile');

export const selectUploadDocumentError: MemoizedSelector<
  object,
  any
> = createSelector(selectUploadDocumentFeatureState, getError);

export const selectUploadDocumentIsLoading: MemoizedSelector<
  object,
  boolean
> = createSelector(selectUploadDocumentFeatureState, getIsLoading);

export const selectUploadDocumentCompleted: MemoizedSelector<
  object,
  boolean
> = createSelector(selectUploadDocumentFeatureState, getCompleted);

export const selectUploadDocumentProgress: MemoizedSelector<
  object,
  number
> = createSelector(selectUploadDocumentFeatureState, getProgress);

export const selectUploadDocumentCancelRequest: MemoizedSelector<
  object,
  boolean
> = createSelector(selectUploadDocumentFeatureState, getCancelRequest);
```