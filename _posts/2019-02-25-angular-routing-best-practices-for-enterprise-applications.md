---
layout: post
title: Angular Routing - Best Practices for Enterprise Applications
author: Wesley Grimes
subtitle: A collection of best practices for managing routes and lazy loading in large Angular applications
date: 2029-02-24 05:34:51 -0500
categories: angular
tags: [angular, javascript, enterprise, typescript, routing]
excerpt: The following represents a pattern that I've developed at my day job after building several enterpirse Angular applications. While most online tutorials do a great job laying out the fundamentals, I had a hard time locating articles that showed recommended conventions and patterns for large and scalable applications. With this pattern you should have a clean and concise organization for all routing related concerns in your applications.
---

![](/assets/post_headers/routing.jpg)

---

## Before We Get Started

This article is not intended to be a tutorial on routing in Angular. If you are new to Routing in Angular then I highly recommend you check out one of the the following resources: 

* [Ultimate Courses](https://bit.ly/2WubqhW)
* [Official Angular Docs](https://angular.io/guide/router)

---

## Background

The following represents a pattern that I've developed at my day job after building several enterpirse Angular applications. While most online tutorials do a great job laying out the fundamentals, I had a hard time locating articles that showed recommended conventions and patterns for large and scalable applications. 

With this pattern you should have a clean and concise organization for all routing related concerns in your applications.

---

## Prerequisites

For context, this article assumes you are using the following version of Angular:

* Angular v7.2.6

---

## Best Practice #1 - Create an `app.routes.ts` file

## Best Practice #2 - Create a `***.routes.ts` for each feature

## Best Practice #3 - Lazy-Loading Where It Makes Sense 

## Best Practice #4 - Create an `app.guards.ts` file

## Best Practice #5 - Create an `***.guards.ts` file for each feature

## Best Practice #6 - Create a 404 Route

---

## Finished Application Structure

---

## Example Repository

---

## Conclusion