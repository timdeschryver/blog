---
layout: post
title: Making Upgrades to Angular's NgOnDestroy
author: Wesley Grimes
subtitle: Overcoming limitations with how and when `ngOnDestroy` is called.
date: 2019-03-29 05:34:51 -0500
categories: angular
tags: [angular, javascript, enterprise, typescript, dom, lifecycle]
excerpt: We will discuss limitations with how and when `ngOnDestroy` is called. We will also discuss ways to overcome those limitations using async and HostListener.
header-img: 'assets/post_headers/ngondestroy.jpg'
---

![](/assets/post_headers/ngondestroy.jpg)

This article is a continuation of an Angular Hot Tip tweet that I sent out earlier this week. It became widely popular and generated quite a discussion. The concepts explored in this article reflect that discussion, so you should probably take some time and go check it out here:

{% twitter https://twitter.com/wesgrimes/status/1110603853089701888 %}

As an extension of the above mentioned tweet, we will discuss limitations with how and when `ngOnDestroy` is called. We will also discuss ways to overcome those limitations. If you are new to Angular, or new to lifecycle methods in Angular, then I suggest you check out the [official docs here](https://angular.io/guide/lifecycle-hooks). 

---

## NPM Package Versions

For context, this article assumes you are using the following `npm` `package.json` versions:

- `@angular/*`: 7.2.9

---

## A Brief Primer On NgOnDestroy

Before we dig too deep, let's take a few minutes and review `ngOnDestroy`. 

NgOnDestroy is a lifecycle method that can be added by implementing `OnDestroy` on the class and adding a new class method named `ngOnDestroy`. It's primary purpose according to the [Angular Docs](https://angular.io/guide/lifecycle-hooks#lifecycle-sequence) is to "Cleanup just before Angular destroys the directive/component. Unsubscribe Observables and detach event handlers to avoid memory leaks. Called just before Angular destroys the directive/component."

### A Leaky MyValueComponent

Let's imagine that we have a component named `MyValueComponent` that subscribes to a value from `MyService` in the `ngOnInit` method:

```typescript
import { Component, OnInit } from '@angular/core';
import { MyService } from './my.service';

@Component({
  selector: 'app-my-value',
  templateUrl: './my-value.component.html',
  styleUrls: [ './my-value.component.css' ]
})
export class MyValueComponent implements OnInit {
  myValue: string;

  constructor(private myService: MyService) {}

  ngOnInit() {
      this.myService.getValue().subscribe(value => this.myValue = value);
  }
}
```

If this component is created and destroyed multiple times in the lifecycle of an Angular application, each time it's created the `ngOnInit` would be called creating a brand new subscription. This could quickly get out of hand, with our value being updated exponentially. This is creating what is called a "memory leak". Memory leaks can wreak havoc on the performance of an application and in addition add unpredictable or unintended behaviors. Let's read on to learn how to plug this leak.

### Fixing the Leak on MyValueComponent

To fix the memory leak we need to augment the component class with an implementation of `OnDestroy` and `unsubscribe` from the subscription. Let's update our component adding the following:

* Add `OnDestroy` to the typescript `import`
* Add `OnDestroy` to the `implements` list
* Create a class field named `myValueSub: Subscription` to track our subscription
* Set `this.myValueSub` equal to the value of `this.myService.getValue().subscription`
* Create a new class method named `ngOnDestroy`
* Call `this.myValueSub.unsubscribe()` within `ngOnDestroy` if a subscription has been set.

The updated component will look something like this:

```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import { MyService } from './my.service';

@Component({
  selector: 'app-my-value',
  templateUrl: './my-value.component.html',
  styleUrls: [ './my-value.component.css' ]
})
export class MyValueComponent implements OnInit, OnDestroy {
  myValue: string;
  myValueSub: Subscription;

  constructor(private myService: MyService) {}

  ngOnInit() {
      this.myValueSub = this.myService.getValue().subscribe(value => this.myValue = value);
  }

  ngOnDestroy() {
    if (this.myValueSub) {
        this.myValueSub.unsubscribe();
    }
  }
}
```

### Moving Beyond Memory Leaks

Great! Now you have some background on `ngOnDestroy` and how cleaning up memory leaks is the primary use case for this lifecycle method. But what if you want to take it a step further and add additional cleanup logic? How about making server-side cleanup calls? Maybe preventing user navigation away?

As you read on we will discuss three methods to upgrade your `ngOnDestroy` for optimum use.

---

## Upgrade #1 - Making NgOnDestroy Async

As with other lifecycle methods in Angular, you can modify `ngOnDestroy` with `async`. This will allow you to make calls to methods returning a `Promise`. This can be a powerful way to manage cleanup activities in your application. As you read on we will explore an example of this.

### Adding logic to call AuthService.logout from ngOnDestroy

Let's pretend that you need to perform a server-side logout when `MyValueComponent` is destroyed. To do so we would update the method as follows:

* Add `AuthService` to your `imports`
* Add `AuthService` to your `constructor`
* Add `async` in front of the method name `ngOnDestroy`
* Make a call to an `AuthService` to `logout` using the `await` keyword.

Your updated `MyValueComponent` will look something like this:

```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import { MyService } from './my.service';
import { AuthService } from './auth.service';

@Component({
  selector: 'app-my-value',
  templateUrl: './my-value.component.html',
  styleUrls: [ './my-value.component.css' ]
})
export class MyValueComponent implements OnInit, OnDestroy {
  myValue: string;
  myValueSub: Subscription;

  constructor(private myService: MyService, private authService: AuthService) {}

  ngOnInit() {
      this.myValueSub = this.myService.getValue().subscribe(value => this.myValue = value);
  }

  async ngOnDestroy() {
    if (this.myValueSub) {
        this.myValueSub.unsubscribe();
    }

    await this.authService.logout();
  }
}
```

Tada! Now when the component is destroyed an `async` call will be made to log the user out and destroy their session on the server.

## Upgrade #2 - Ensure Execution During Browser Events

Many developers are surprised to learn that `ngOnDestroy` is only fired when the class which it has been implemented on is destroyed within the context of a running browser session.

In other words, `ngOnDestroy` is *not* reliably called in the following scenarios:

* Page Refresh
* Tab Close
* Browser Close
* Navigation Away From Page

This could be a deal-breaker when thinking about the prior example of logging the user out on destroy. Why? Well, most users would simply close the browser session or navigate to another site. So how do we make sure to capture or hook into that activity if `ngOnDestroy` doesn't work in those scenarios?

### Decorating ngOnDestroy with HostListener

> TypeScript decorators are used throughout Angular applications. More information can be found here in the [official TypeScript docs](https://www.typescriptlang.org/docs/handbook/decorators.html).

To ensure that our `ngOnDestroy` is executed in the above mentioned browser events, we can add one simple line of code to the top of `ngOnDestroy`. Let's continue with our previous example of `MyValueComponent` and decorate `ngOnDestroy`:

* Add `HostListener` to the `imports`
* Place `@HostListener('window:beforeunload')` on top of `ngOnDestroy`

Our updated `MyValueComponent` will look something like this:

```typescript
import { Component, OnInit, OnDestroy, HostListener } from '@angular/core';
import { MyService } from './my.service';
import { AuthService } from './auth.service';

@Component({
  selector: 'app-my-value',
  templateUrl: './my-value.component.html',
  styleUrls: [ './my-value.component.css' ]
})
export class MyValueComponent implements OnInit, OnDestroy {
  myValue: string;
  myValueSub: Subscription;

  constructor(private myService: MyService, private authService: AuthService) {}

  ngOnInit() {
      this.myValueSub = this.myService.getValue().subscribe(value => this.myValue = value);
  }

  @HostListener('window:beforeunload')
  async ngOnDestroy() {
    if (this.myValueSub) {
        this.myValueSub.unsubscribe();
    }

    await this.authService.logout();
  }
}
```

Now our `ngOnDestroy` method is called both when the component is destroyed by Angular AND when the browser event `window:beforeunload` is fired. This is a powerful combination!

### More about HostListener

`@HostListener()` is an Angular decorator that can be placed on top of any class method. This decorator takes two arguments: `eventName` and optionally `args`. In the above example, we are passing `window:beforeunload` as the DOM event. This means that Angular will automatically call our method when the DOM event `window:beforeunload` is fired. For more information on `@HostListener` check out the [official docs](https://angular.io/api/core/HostListener).

If you want to use this to prevent navigation away from a page or component then:

* Add `$event` to the `@HostListener` arguments
* Call `event.preventDefault()`
* Set `event.returnValue` to a string value of the message you would like the browser to display

An example would look something like this:

```typescript
@HostListener('window:beforeunload', ['$event'])
async ngOnDestroy($event) {
  if (this.myValueSub) {
    this.myValueSub.unsubscribe();
  }

  await this.authService.logout();

  $event.preventDefault();
  $event.returnValue = 'A message.';
}
```

> PLEASE NOTE: This is not officially supported by Angular! `OnDestroy` and `ngOnDestroy` suggest that there is no input argument on `ngOnDestroy` allowed. While unsupported, it does in fact still function as normal.

### More about window:beforeunload

`window:beforeunload` is an event fired right before the `window` is unloaded. More details can be found in the documentation here: https://developer.mozilla.org/en-US/docs/Web/API/Window/beforeunload_event. 

A couple points to be aware of:

* This event is currently supported in all major browsers EXCEPT iOS Safari.

* If you need this functionality in iOS Safari then consider reviewing this [Stack Overflow thread](https://stackoverflow.com/questions/14645011/window-onbeforeunload-and-window-onunload-is-not-working-in-firefox-safari-o/14647730#14647730).

* If you are using this event in an attempt to block navigation away you must set the `event.returnValue` to a string of the message you would like to display. More details in this [example](https://developer.mozilla.org/en-US/docs/Web/API/Window/beforeunload_event#Examples).


## Conclusion

I realize that some of the tips recommended in this article are not mainstream and may generate some concern. Please remember as always to try these out and see if they fit for what you are doing in your application. If they work great! If not, then it's ok to move on. 

If you have any comments or questions feel free to contact me on [Twitter](https://twitter.com/wesgrimes)

---

## Additional Resources

I would highly recommend enrolling in the Ultimate Angular courses. It is well worth the money and I have used it as a training tool for new and experienced Angular developers. Follow the link below to signup.

[Ultimate Courses: Expert online courses in JavaScript, Angular, NGRX and TypeScript](https://bit.ly/2WubqhW)
