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

## Create the Upload File Feature Store

To keep your **NgRx** store organized, I recommend creating a separate Upload File Feature Store. Let's bundle it all together in a module named `upload-file-store.module.ts` and keep it under a sub-directory named `upload-file-store`.

When we are finished building out the store, the folder will look as follows:

! TODO INSERT FOLDER STRUCTURE HERE

### Step 1 - Create Feature Store Module

Create a feature store module using the following command: 

```shell
$ ng g module upload-file-store --flat false
```

### Step 2 - Create State Interface

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

### Step 3 - Create Actions

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

### Step 4 - Create the Feature Reducer

Create a new file underneath the `upload-file-store` folder, named `reducer.ts`. This file will hold the reducer we create to manage state transitions to the store. 

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
        isLoading: false,
        completed: false,
        progress: action.payload.progress,
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
    default: {
      return state;
    }
  }
}

```