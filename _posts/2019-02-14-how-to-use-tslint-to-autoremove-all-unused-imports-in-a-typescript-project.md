---
layout: post
title: Automatically Remove All Unused Imports in a TypeScript Project
author: Wesley Grimes
subtitle: A way to automatically remove all unused TypeScript imports across all files in a project
date: 2019-02-14 14:34:51 -0500
categories: angular
tags: [javascript, enterprise, typescript, tslint]
excerpt: We will use the `tslint` command line tool, in conjuction with the `tslint-etc` rules, to automatically detect and remove all unused imports in the directory, recursively. If you have a large project, the process can take some time to run. It is important to double-check all files for correctness once the fix process is complete.
header-img: 'assets/post_headers/autoremove_unused_typescript_imports.jpg'
---
![](/assets/post_headers/autoremove_unused_typescript_imports.jpg)
# The Problem

Recently, I had a need arise to programatically and recursively traverse through all `*.ts` files in a given project and remove all unused TypeScript imports. At the time this article was written, there was no way to do this within Visual Studio Code without opening each individual `*.ts` file and hitting `CTRL + Shift + O` on Windows/Linux. After some research, and much appreciated help from Twitter colleagues, I found a solution that works. This should work for Angular, React, Vue.js, or any plain TypeScript project.

# The Solution

We will use the `tslint` command line tool, in conjuction with the `tslint-etc` rules, to automatically detect and remove all unused imports in the directory, recursively. If you have a large project, the process can take some time to run. It is important to double-check all files for correctness once the fix process is complete.

## Install Required Tools

Install the follow node packages required to for this process to work. 

```shell
$ npm install -g typescript tslint tslint-etc
```

## Create a temporary `tslint` config file

Create a new file named `tslint-imports.config` in the root of your project. This creates a hyper-focused tslint process that will only check for unused declarations. It is _important to note_ that this will throw `tslint` errors on unused imports, parameters and variables. We are only using this for the `--fix` process. As such, the `tslint-etc` rules only autofix unused imports. 

This file needs the following contents:

```json
{
  "extends": [
    "tslint-etc"
  ],
  "rules": {
    "no-unused-declaration": true
  }
}
```

## Run the autofix process
The following command will traverse, recursively through all `*.ts` files in the project and remove the unused imports. It save the files automatically in place. 

**BE CAREFUL!** This process is only reversible if you are using a source control solution like `git` or `svn`, where you can revert changes.

```shell
$ tslint --config tslint-imports.json --fix --project .
```

## Double-Check Your Files

At this point, I would highly recommend you double-check your files for correctness. This tool worked on all but 2 of my 195 `*.ts` files. Two of the components were incorrectly updated. I was able to spot this by running an `ng build --prod` as it was an `Angular` application. You could run a manual `tslint --project .` if your project doesn't use a build tool.

## Resources

* [Original Twitter Conversation](https://twitter.com/wesgrimes/status/1096134870726774787)
* [TSLint](https://palantir.github.io/tslint/usage/cli/)
* [tslint-etc](https://cartant.github.io/tslint-etc/)