---
title: "Go SSR Wins over JavaScript"
date: 2021-01-31T20:00:00Z
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

To give some background, the previous SPA I helped create and worked with, that consumed my API's data, was created using Vue.js 2, SCSS with scoped styles (horror!), VueX, and TypeScript.

This combination has quickly lead to spaghetti code of terrible quality, and lots of it. To make matters worse, there was a transition of API v1 to v2, where some data models changed, and the developer decided to refuse that change, and write his own mappers, to convert those v2 data models to v1, and vice-versa, in order to comply with my API, which he was interfacing with.

As you can imagine, this has lead to an unmaintainable project, that no one dares to touch. 

So I decided to follow my beloved KISS principle, and to go ahead and get started on writing a SOLID coded Go application, that will act as the API consumer, and spit out processed HTML for the client's browser.

I also followed the principle of "graceful degradation", which in basic terms, means that the website should function perfectly without JavaScript, and it should only be used to enhance the user experience, like in heavy animations, sliders, etc. By no means should JavaScript be the center of the application, due to all the negative aspects it brings, and due to the superior language that Go is.

I have been met with much success with this application, and can really say confidently that my future endeavours will always go the same route, unless really necessary to code a more client-side application, in which case I will most likely turn to GopherJS or such, or perhaps some Go to WebAssembly, so I don't need to write any more JavaScript ever.

I want to list some of the advantages that I have found, and am sure anyone will find when doing things more in an "old but gold" old-school approach of building websites, server side rendered:

- In my case, one language for the whole platform 😀  Go !
- Native browser features, like remembering how far in the list of the previous page you were when you go back.
- SEO is greatly improved, and you don't even need to think about it as much. Search engines will easily crawl your site, and will read correctly formed HTML.
- Security improvements, due to not exposing application code, or API calls in the client's browser.
- Simpler front-end applications, no mapping, no state management, no problems. API data directly to HTML.
- Go is a well designed, compiled, statically typed and concurrent language. Why not use it for client-facing applications?
- The ecosystem around Go is much more stable and reliable than JavaScript's.
- The standard library of Go is great, and can power most of your needs without additional thrid-party dependencies.
- High-concurrency and asynchronous process capabilities in the client-facing applications.
- Accessibility is greatly improved due to the HTML being delivered as already rendered, and we can care more about writing semantically correct HTML, instead of caring so much about the newest JS framework.
- Supporting security conscious people and organizations: let's face it there's bad people doing heinous things with JavaScript, so the easiest solution is to get rid of JavaScript.
- Mobile devices, web-views, and embedded devices are better supported, without inconsistencies.
- Search engine spiders only follow real links - JavaScript confuses them.
- Cross site scripting attacks risk is greatly reduced.
- Hacking/Defacement through DOM manipulation is no longer possible.
- Severe speed improvements, both on first, and on subsequent loads.

I really have no intentions to be part of the spaghetti mess that JavaScript easily becomes, I want a consistent and predictible language, that compiles to binary code and can easily beat any out there.

As for the SPA fans out there, I know it has its use cases, but let's face it, JS is doing so much more than it should since React and buddies came along, this has gotten way out of hand and it is time to stop.

Thanks for your attention, and I look forward to seeing comments.
