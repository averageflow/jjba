---
title: "Zen of Go"
date: 2020-10-04T22:00:00Z
weight: 1
aliases: ["/go-file-descriptors"]
tags: ["Go", "Clean Code"]
author: "Josep Jesus Bigorra Algaba"
showToc: true
TocOpen: true
draft: false
hidemeta: false
disableShare: false
cover:
    image: "https://res.cloudinary.com/dehs6irlh/image/upload/v1601732690/Gophers/fiveyears_zhewe0.jpg"
    alt: "Go"
    caption: ""
    relative: false
    hidden: false
comments: true
description: "Given nobody actively seeks to write bad code, this leads to the question; how do you know when you’ve written good Go code?"
disableHLJS: false
---

Something that I’ve been thinking about a lot recently, when reflecting on the body of my own work, is a common subtitle, how should I write good code? Given nobody actively seeks to write bad code, this leads to the question; how do you know when you’ve written good Go code? If there’s a continuum between good and bad, how to do we know what the good parts are? What are its properties, its attributes, its hallmarks, its patterns, and its idioms?

#### Maintainability counts

Clarity, readability, simplicity, are all aspects of maintainability. Can the thing you worked hard to build be maintained after you’re gone? What can you do today to make it easier for those that come after you?

#### Each package fulfils a single purpose

A well designed Go package provides a single idea, a set of related behaviours. A good Go package starts by choosing a good name. Package names should always be lowercase and not contain any underscores or dashes. Think of your package’s name as an elevator pitch to describe what it provides, using just one word. Common examples, `controllers`, `jobs`, `services`, `utils`.

#### Handle errors explicitly

Robust programs are composed from pieces that handle the failure cases before they pat themselves on the back. The verbosity of `if err != nil { return err }`is outweighed by the value of deliberately handling each failure condition at the point at which they occur. Panic and recover are not exceptions, they aren’t intended to be used that way. Ensure you always use `log.Printf(err.Error())` in order to log those errors to the application log for monitoring or debugging.

#### Return early rather than nesting deeply

Every time you indent you add another precondition to the programmer’s stack consuming one of the 7 ±2 slots in their short term memory. Avoid control flow that requires deep indentation. Rather than nesting deeply, keep the success path to the left using guard clauses.

#### Leave concurrency to the caller

Let the caller choose if they want to run your library or function asynchronously, don’t force it on them. If your library uses concurrency it should do so transparently.

#### Before you launch a goroutine, know when it will stop

Goroutines own resources; locks, variables, memory, etc. The sure fire way to free those resources is to stop the owning goroutine.

#### Avoid package level state

Seek to be explicit, reduce coupling, and spooky action at a distance by providing the dependencies a type needs as fields on that type rather than using package variables. Exceptions to this rule are a global singleton for the application config, and a database pool connection.

#### Simplicity matters

Simplicity is not a synonym for unsophisticated. Simple doesn’t mean crude, it means **_readable_** and **_maintainable_**. When it is possible to choose, defer to the simpler solution.

#### Write tests to lock in the behaviour of your package

Test first or test later, if you shoot for 100% test coverage or are happy with less, regardless, your package’s API is your contract with its users. Tests are the guarantees that those contracts are written in. Make sure you test for the behaviour that users can observe and rely on.

#### If you think it’s slow, first prove it with a benchmark

So many crimes against maintainability are committed in the name of performance. Optimisation tears down abstractions, exposes internals, and couples tightly. If you’re choosing to shoulder that cost, ensure it is done for good reason.

#### Moderation is a virtue

Use goroutines, channels, locks, interfaces, embedding, in moderation.

## Personal choices

#### Never use anything other than camelCase

My convention is using camelCase for all purposes, including JSON responses, and in Go, if you would like to have a member (function, variable, struct var) exported (public) you must do it with UpperCamelCase. I avoid snake_case at all costs, since I mostly use it for database fields.

#### Short variable names only on short functions

Only use one letter variables when the scope of them is so reduced and when it does not affect the readability and maintainability of the code they are lodged in.

#### Comments

Comments should only be written using //. The multi-line comment /* */ is left for disabling large pieces of code. Function comments should not include any type annotations or similar habits. GoDoc is not PHPDoc or JavaDoc. Every package should include a Doc.go file which serves as introduction to the package’s purpose, such as below. Document why, not how.

#### Slice allocation

If you happen to know the exact size that a slice will take, always allocate that. Only use append in the case where you may be filtering out items, and/or are not sure of the size the list will be. This is more clear and more efficient.