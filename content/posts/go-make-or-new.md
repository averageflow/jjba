---
title: "Go has make and new functions, what gives ?"
date: 2020-09-18T22:00:00Z
weight: 1
aliases: ["/go-make-or-new"]
tags: ["Go"]
author: "Josep Jesus Bigorra Algaba"
showToc: true
TocOpen: true
draft: false
hidemeta: false
disableShare: false
cover:
    image: "https://res.cloudinary.com/dehs6irlh/image/upload/v1605953501/jjba-site/home/go-daemon_xmsm1e.png"
    alt: "Go"
    caption: ""
    relative: false
    hidden: false
comments: true
description: "The Go language has both make and new functions, what gives ? If you are coming from another language, especially one that uses constructors, it may appear that new should be all you need, but Go is not those languages, nor does it have constructors..."
disableHLJS: false
---

This is a post about Go’s built in `make` and `new` functions.

As Rob Pike noted at Gophercon on 2014, Go has many ways of initialising variables. Among them is the ability to take the address of a struct literal which leads to several ways to do the same thing.

```go
s := &SomeStruct{}
v := SomeStruct{}
s := &v              // identical
s := new(SomeStruct) // also identical
```

It is fair that people point out this redundancy in the language and this sometimes leads them to search for other inconsistencies, most notably the redundancy between `make` and `new`. On the surface it appears that `make` and `new` do very similar things, so what is the rationale for having both ?

# Why can’t we use make for everything ?

Go does not have user defined generic types, but it does have several built in types that can operate as generic lists, maps, sets, and queues; slices, maps and channels.

Because `make` is designed to create these three built in generic types, it must be provided by the runtime as there is no way to express the function signature of `make` directly in Go.

Although `make` creates generic slice, map, and channel values, they are still just regular values; `make` does not return pointer values.

If `new` was removed in favour `make`, how would you construct a pointer to an initialised value ?

```go
var x1 *int
var x2 = new(int)
```

x1 and x2 have the same type, `*int`, x2 points to initialised memory and may be safely de-referenced, the same is not true for x1.

## Why can’t we use new for everything ?

Although the use of `new` is rare, its behaviour is well specified.

`new(T)` always returns a `*T` pointing to an initialised `T`. As Go doesn’t have constructors, the value will be initialised to T‘s zero value.

Using `new` to construct a pointer to a slice, map, or channel zero value works today and is consistent with the behaviour of `new`.

```go
s := new([]string)
fmt.Println(len(*s))  // 0
fmt.Println(*s == nil) // true

m := new(map[string]int)
fmt.Println(m == nil) // false
fmt.Println(*m == nil) // true

c := new(chan int)
fmt.Println(c == nil) // false
fmt.Println(*c == nil) // true
```

## Sure, but these are just rules, we can change them, right ?

For the confusion they may cause, `make` and `new` are consistent; `make` only makes slices, maps, and channels, `new` only returns pointers to initialised memory.

Yes, `new` could be extended to operate like `make` for slices, maps and channels, but that would introduce its own inconsistencies.

1. `new` would have special behaviour if the type passed to `new` was a slice, map or channel. This is a rule that every Go programmer would have to remember.
2. For slices and channels, `new` would have to become variadic, taking a possible length, buffer size, or capacity, as required. Again more special cases to have to remember, whereas before `new` took exactly one argument, the type.
3. `new` always returns a `*T` for the `T` passed to it. That would mean code like

    func Read(buf []byte) []byte
        // assume new takes an optional length
        buf := Read(new([]byte, length))

would no longer be possible, requiring more special cases in the grammar to permit `*new([]byte, length)`.

## In summary

`make` and `new` do different things.

If you are coming from another language, especially one that uses constructors, it may appear that `new` should be all you need, but Go is not those languages, nor does it have constructors.

My advice is to use `new` sparingly, there are almost always easier or cleaner ways to write your program without it.

As a code reviewer, the use of `new`, like the use of named return arguments, is a signal that the code is trying to do something clever and I need to pay special attention. It may be that code really is clever, but more than likely, it can be rewritten to be clearer and more idiomatic.