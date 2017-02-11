---
layout: post

title: WebdriverIO - Testing Pages With Multiple/Nested IFrames
subtitle: "WebdriverIO queries/commands are limited to the scope of their frame. I'll discuss options for managing a page with multiple/nested frames."
cover_image: blog-cover.jpg

excerpt: "WebdriverIO is an incredible tool that allows for Selenium Webdriver testing using Javascript. This makes writing tests feel much more familiar to many of the people that this tool is intended for - web developers. And unfortunately, WebdriverIO doesn't always play well with another common item in web developement - the IFrame. Similar to JavaScript, WebdriverIO queries/commands are limited in scope to the current frame, so here are some methods we've come up with for improving cooperation between WebdriverIO and IFrames."

author:
  name: Eric Kenney
  bio: Web Developer
  image: logo.png
---


#WebdriverIO - Testing Pages With Multiple/Nested IFrames 

WebdriverIO is an incredible tool that allows for Selenium Webdriver testing using Javascript. This makes writing tests feel much more familiar to many of the people that this tool is intended for - web developers. And unfortunately, WebdriverIO doesn't always play well with another common item in web developement - the IFrame. Similar to JavaScript, WebdriverIO queries/commands are limited in scope to the current frame, so here are some methods we've come up with for improving cooperation between WebdriverIO and IFrames.

##Hostile Frames

First off