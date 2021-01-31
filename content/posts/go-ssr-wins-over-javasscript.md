---
title: "Go SSR Wins over JavaScript"
date: 2021-01-31T22:00:00Z
weight: 1
aliases: ["/go-ssr-wins-over-javascript"]
tags: ["Go", "Clean Code", "JavaScript", "SSR"]
author: "Josep Jesus Bigorra Algaba"
showToc: true
TocOpen: true
draft: false
hidemeta: false
disableShare: false
cover:
    image: "https://res.cloudinary.com/dehs6irlh/image/upload/v1612126015/jjba-site/blog/go-ssr/iu-2_gdigyh.jpg"
    alt: "NoScript"
    caption: ""
    relative: false
    hidden: false
comments: true
description: "In this day and age, server side rendering proves it is stable and more effective than the JavaScript bloat."
disableHLJS: false
---

In this day and age, server side rendering proves it is stable and more effective than the JavaScript bloat we are growing used to. Any simple page, with perhaps at most 3 KB of actual text content will download over 2 MB of JavaScript in order to simply function.

Besides that fact, SPA and API situations, where you have a decoupled frontend from the backend, still do not justify the fact of using JavaScript, or even worse, any of its frameworks. That paradigm can easily be, in most cases, fulfilled by server side rendering in the backend language of your choice, which will spit out a digested HTML for your clients. 

In specific, if you keep the same programming language, in my case Go, across your applications, you can share a lot of logic, data models, and you get a consistent developer, and easier to maintain system.

Not to mention the fact that the JavaScript ecosystem goes with the wind, is the most unstable place I have ever seen, and most of the times I need to look into some source code written in that language, of some external package dependency, I start questioning the author's sanity, and mine, and also my career choices.

Going further, with the bloated state management tools, and broken concurrency model of JavaScript, as well as the insane "features" of the language, and then seeing TypeScript, a "sub-set" of the language, which also brings its own horrors and pre-compilation, I have no doubt that the show must come to a stop, and for my professional needs, as a software engineer at an e-commerce platform, I have decided to throw away all the SPA code (3 attempts) that has previously been in production, and go in a totally oposite direction.

This direction may seem unorthodox, but it has many reasons that make it a success, and a joy to work with, and extend.

The previous SPA I helped create and worked with, that consumed my API's data, was created using Vue.js 2, SCSS with scoped styles (horror!), VueX, and TypeScript.

This combination has quickly lead to spaghetti code of terrible quality, and lots of it. To make matters worse, there was a transition of API v1 to v2, where some data models changed, and the developer decided to refuse that change, and write his own mappers, to convert those v2 data models to v1, and vice-versa, in order to comply with my API, which he was interfacing with.