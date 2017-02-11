---
layout: post

title: WebdriverIO - Modularizing Test Specs for Reusability
subtitle: "Methods for Increasing the Reusability of Specs and Page Objects"
cover_image: blog-cover.jpg

excerpt: ""

author:
  name: Eric Kenney
  bio: Web Developer
  image: logo.png
---


# WebdriverIO - Modularizing Test Specs for Reusability

At work, we have a whole lot of test cases. If you were to try to test every combination of options across every device and browser, you'd be looking at tens of thousands of individual tests. In an attempt to make this less painful, we've been moving towards automation using WebdriverIO and BrowserStack. When designing our test framework, we had a few basic requirements:
- Specs should be agnostic to the page they are testing
- Specs should be reusable across test cases
- The suite should accept an array of test inputs over which it can iterate. This allows for one test suite to be reusable for all of our test input combinations.
- The [page object](http://martinfowler.com/bliki/PageObject.html) should be the only thing that needs changing when testing a new product.

In order to meet these goals, we ended up sort of crafting a new framework with a clear file structure and easy re-use of specs and assertions across test cases. So let's take a look...

> Note: Most of our testing is based around video ad playback. This includes playing, pausing, click-thru, etc. This framework is especially conducive to that sort of feature testing, but could easily be expanded to any WebdriverIO use case.

## Basics

First I'll explain the directory structure used and then go into what each portion does. The basic structure looks like this:

```
Project
│   README.md
│   file001.txt    
│
└───assertions
│   │   file011.txt
│   │   file012.txt
│   │
│   └───subfolder1
│       │   file111.txt
│       │   file112.txt
│       │   ...
│   
└───bundles
    │   file021.txt
    │   file022.txt
```
