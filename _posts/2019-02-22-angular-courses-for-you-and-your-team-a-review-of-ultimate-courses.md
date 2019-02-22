---
layout: post
title: Angular courses for you and your team, a review of Ultimate Courses
author: Wesley Grimes
subtitle: In-depth review of Ultimate Courses
date: 2019-02-22 05:34:51 -0500
categories: angular
tags: [javascript, enterprise, typescript, tslint, training]
excerpt: While these are fantastic resources, it's often hard to find Angular training courses that teach on the latest and greatest versions of front-end libraries and frameworks. In this article, I will explore, Ultimate Courses, the offerings created and curated by Todd Motto (Google Developer Expert and Angular extraordinaire).
header-img: 'assets/post_headers/ultimate_courses.jpg'
hide_banner: true
---

![](/assets/post_headers/ultimate_courses.jpg)

As a senior developer in a small-to-medium sized software firm, I am often tasked with training new developers, or seasoned developers in new technologies. I am always on the lookout for ways to ease the burden and standardize the process for all parties involved. 

One-on-one training and instructor-led training sessions are great, but not everyone has the resources to do this, and often times our current workloads and "deliverables" prevent us from setting aside a week (or more) to devote to training on new topics. Most of you reading this are well aware of the mainstream online training offerings that exist. [Pluralsight](https://pluralsight.com/) and [Lynda](http://lynda.com/) come to mind.

While these are fantastic resources, it's often hard to find Angular training courses that teach on the latest and greatest versions of front-end libraries and frameworks. In this article, I will explore, [Ultimate Courses](https://bit.ly/2WubqhW), the offerings created and curated by [Todd Motto](https://toddmotto.com/) (Google Developer Expert and Angular extraordinaire).

---

## Let's Review The Packages

For Angular development, [Ultimate Courses](https://bit.ly/2WubqhW) offers two packages to choose from: [Angular Kickstart Package](https://bit.ly/2WubqhW) and [Angular Ultimate Package](https://bit.ly/2WubqhW). Let's quickly review the differences.

### [Angular Kickstart Package](https://bit.ly/2WubqhW)

If your team has previous TypeScript experience, then this is the package I would recommend. It includes: 

* *Angular Fundamentals*
* *Angular Pro*

### [Angular Ultimate Package](https://bit.ly/2WubqhW)

Learning Angular, for most developers, is not just as simple as learning the frameworks features, conventions and tooling. For most, it requires getting up to speed on [TypeScript](http://www.typescriptlang.org/), a powerful, typed superset of JavaScript. Teaching developers TypeScript is a must for any online solution that I recommend, and thankfully Ultimate Courses' [Angular Ultimate Package](https://bit.ly/2WubqhW) has you covered here. It includes: 

* *Angular Fundamentals*
* *Angular Pro*
* *TypeScript Basics*
* *TypeScript Masterclass*
* *NGRX Store + Effects*

## Individual Courses Available

Courses can be purchased in packages as stated above, however, they can also be purchased individually as needed which may make sense for some scenarios. 

## Team Licensing Available

If you are working with a team of developers, Ultimate Courses offers user licensing  with discounts as the user count grows. This is a great option for teams of developers learning Angular.

---

## Angular Fundamentals

This course starts out from the high-level and slowly dives deeper into the basic building blocks of an Angular single page application. The content is separated into the following sections:

* Architecture, setup, source files
* ES5 to ES6 and TypeScript refresher
* Getting started
* Template fundamentals
* Rendering flows
* Component Architecture and Feature Modules
* Services, Http and Observables
* Template-driven Forms, Inputs and Validation
* Component Routing

I won't dive too deep into each these sections, but I will say for an introductory course, this offering does a fantastic job of giving you just enough information to be dangerous (in a good way), while not overwhelming first-time Angular developers.

---

## Angular Pro

This course takes the concepts learned in Angular Fundamentals and goes deep, way deep. The topics covered in this course are vital to learn as any Angular app that grows in complexity will almost always need to handle these situations. I appreciate Todd's attention to detail. Topics covered include:

* Advanced Components — including dynamic component creation
* Directives
* Pipes
* Reactive Forms — This is a good one as the best practice for Angular forms now-a-days is considered Reactive Forms.
* Routing — this includes a nice deep drive into lazy loading modules, a method to speed up initial load times of large applications
* Unit Testing — A must have for distributed teams and complex applications. Todd walks through need-to-know topics around unit testing with built-in Angular
tooling.
* Dependency Injection and Zones
* Statement Management with Rx — although I recommend [NgRx](http://ngrx.io/)

---

## TypeScript Basics

This course is an introduction to [TypeScript](https://typescriptlang.org/). Developers coming from C# will appreciate this course in particular. In addition, this course can be purchased separately from the [package](https://bit.ly/2WubqhW) if you are building with TypeScript. Topics include:

* Overview, setup and source files
* ES6/7 and TypeScript
* Primitive Types
* Special Types
* Type Aliases and Assertions
* Diving into Interfaces
* Classes, Properties and Inheritance

---

## TypeScript Masterclass

Just as with any language, there are folks that use the basics and are off to the races. There are some cases, however, where you need to dig deep and really understand what's happening. If you are building Angular or NodeJS libraries, then this course is probably for you. Topics include:

* Understanding and Typing "this"
* Type Queries
* Mapped Types
* Exploring Type Guards
* Advanced Types and Practices
* Generics and Overloads
* Exploring Enums
* Declaration Files
* tsconfig and Compiler Options

---

## NGRX Store + Effects

> Personally, I have been using NgRx for a long time. I no longer build Angular applications without it! So for me, I was very curious to learn how Todd explains this topic. Redux is a complicated pattern for first-time developers to grasp, but it really is a must for creating quality applications.

In the Angular realm, the Redux pattern is implemented in several libraries, the most popular being NgRx and NGXS. For those of you new to Redux, redux is a pattern for managing global state in an application. It was originally developed at Facebook, and since has taken off and is widely used through most modern front-end frameworks. NgRx is, by far, the most widely used Angular redux library. As such, Ultimate Courses has chosen to focus it's offerings on NgRx. As we focus on this course, I must say upfront, I was pleasantly surprised and impressed with Todd's approach to teaching NgRx. The course has been so well received, in fact, that even Mike Ryan (NgRx Core Team/Google Developer Expert) recommends this course as the best way to get started!

#### Course Walkthrough

The course starts out by walking through what exactly state management is, how redux accomplishes that, and how JavaScript presents challenges with mutation. 

Once you have grasped the concept of state management using the Redux pattern, the course then has you build your very own vanilla Redux store using plain TypeScript. Realizing then, that NgRx is built on top of these concepts, it's an easy transfer into learning NgRx.

After having built a vanilla redux store, the course then walks through the process of setting up a store using the tools provided by NgRx. The course walks you through creating actions, reducers, selectors, effects. The course then walks through the process of structuring lists of entities using the Entity pattern.

Even folks with some NgRx experience will find this course helpful as it does a deep-dive into more advanced concepts like routing with the store, preloading state, and unit testing your NgRx store.

Below is a detailed list of the topics covered in this course:

* Redux Architecture
* Writing our own Redux Store
* Architecture: ngrx/store and components
* Core Essentials
* Effects and Entities
* Router State Composition
* Extending our State Tree
* Entity patterns, CRUD operations
* Routing via Dispatch
* State preload and protection via Guards
* Observables and Change Detection
* Unit Testing

---

## Conclusion

After taking these courses, and comparing other available options, I can safely recommend the [Angular Ultimate Package](https://bit.ly/2WubqhW) for teams looking to get into Angular Enterprise development. Todd's down-to-earth approach of explaining complex concepts, makes these courses both fun and educational. As an added bonus,  Todd does the voice-overs himself so you get to learn Angular with a British accent. Win-Win-Win.

<div style="text-align:center"><img src="/assets/post_headers/ultimate_courses_2.jpg" alt="10 out of 10 Recommend"></div>


### More Information on Ultimate Courses

Ultimate Courses: Expert online courses in JavaScript, Angular, NGRX and TypeScript
Expert online courses in JavaScript, Angular, NGRX and TypeScript. Join 50,000 others mastering new technologies with [Ultimate Courses](https://bit.ly/2WubqhW)