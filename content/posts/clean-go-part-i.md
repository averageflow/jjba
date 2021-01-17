---
title: "Clean Go - Part I"
date: 2020-10-02T22:00:00Z
weight: 1
aliases: ["/clean-go-part-i"]
tags: ["Go", "Clean Code"]
author: "Josep Jesus Bigorra Algaba"
showToc: true
TocOpen: true
draft: false
hidemeta: false
disableShare: false
cover:
    image: "https://res.cloudinary.com/dehs6irlh/image/upload/v1601732689/Gophers/bumper640x360_kjo0i8.png" 
    alt: "Go logo" 
    caption: "" 
    relative: false
    hidden: false
comments: false
description: "Whether you're working on a personal project or as part of a larger team, writing clean code is an important skill to have."
disableHLJS: true 
---

## Why Write Clean Code?

This document is a reference for the Go community that aims to help developers write cleaner code. Whether you're working on a personal project or as part of a larger team, writing clean code is an important skill to have. Establishing good paradigms and consistent, accessible standards for writing clean code can help prevent developers from wasting many meaningless hours on trying to understand their own (or others') work.

> We don’t read code, we decode it – Peter Seibel

As developers, we're sometimes tempted to write code in a way that's convenient for the time being without regard for best practices; this makes code reviews and testing more difficult. In a sense, we're encoding—and, in doing so, making it more difficult for others to decode our work. But we want our code to be usable, readable, and maintainable. And that requires coding the right way, not the easy way.

This document begins with a simple and short introduction to the fundamentals of writing clean code. Later, we'll discuss concrete refactoring examples specific to Go.

**About `gofmt`**

I'd like to take a few sentences to clarify my stance on `gofmt`because there are plenty of things I disagree with when it comes to this tool. This tool allows us to have a common standard for writing Go code, and that's a great thing. As a developer myself, I can certainly appreciate that Go programmers may feel somewhat restricted by gofmt, especially if they disagree with some of its rules. But in my opinion, homogeneous code is more important than having complete expressive freedom.

## Introduction to Clean Code

Clean code is the pragmatic concept of promoting readable and maintainable software. Clean code establishes trust in the codebase and helps minimize the chances of careless bugs being introduced. It also helps developers maintain their agility, which typically plummets as the codebase expands due to the increased risk of introducing bugs.

### Test-Driven Development

Test-driven development is the practice of testing your code frequently throughout short development cycles or sprints. It ultimately contributes to code cleanliness by inviting developers to question the functionality and purpose of their code. To make testing easier, developers are encouraged to write short functions that only do one thing. For example, it's arguably much easier to test (and understand) a function that's only 4 lines long than one that's 40.

Test-driven development consists of the following cycle:

1. Write (or execute) a test
2. If the test fails, make it pass
3. Refactor your code accordingly
4. Repeat

Testing and refactoring are intertwined in this process. As you refactor your code to make it more understandable or maintainable, you need to test your changes thoroughly to ensure that you haven't altered the behavior of your functions. This can be incredibly useful as the codebase grows.

### Naming Conventions

#### Comments

I'd like to first address the topic of commenting code, which is an essential practice but tends to be misapplied. Unnecessary comments can indicate problems with the underlying code, such as the use of poor naming conventions. However, whether or not a particular comment is "necessary" is somewhat subjective and depends on how legibly the code was written. For example, the logic of well-written code may still be so complex that it requires a comment to clarify what is going on. In that case, one might argue that the comment is helpful and therefore necessary.

In Go, according to gofmt, all public variables and functions should be annotated. I think this is absolutely fine, as it gives us consistent rules for documenting our code. However, I always want to distinguish between comments that enable auto-generated documentation and all other comments. Annotation comments, for documentation, should be written like documentation—they should be at a high level of abstraction and concern the logical implementation of the code as little as possible.

I say this because there are other ways to explain code and ensure that it's being written comprehensibly and expressively. If the code is neither of those, some people find it acceptable to introduce a comment explaining the convoluted logic. Unfortunately, that doesn't really help. For one, most people simply won't read comments, as they tend to be very intrusive to the experience of reviewing code. Additionally, as you can imagine, a developer won't be too happy if they're forced to review unclear code that's been slathered with comments. The less that people have to read to understand what your code is doing, the better off they'll be.

Let's take a step back and look at some concrete examples. Here's how you shouldn't comment your code:

```go
// iterate over the range 0 to 9
// and invoke the doSomething function
// for each iteration
for i := 0; i < 10; i++ {
  doSomething(i)
}
```

This is what I like to call a tutorial comment; it's fairly common in tutorials, which often explain the low-level functionality of a language (or programming in general). While these comments may be helpful for beginners, they're absolutely useless in production code. Hopefully, we aren't collaborating with programmers who don't understand something as simple as a looping construct by the time they've begun working on a development team. As programmers, we shouldn't have to read the comment to understand what's going on—we know that we're iterating over the range 0 to 9 because we can simply read the code. Hence the proverb:

> Document why, not how. – Venkat Subramaniam

Following this logic, we can now change our comment to explain why we are iterating from the range 0 to 9:

```go
// instatiate 10 threads to handle upcoming work load
for i := 0; i < 10; i++ {
  doSomething(i)
}
```

Now we understand why we have a loop and can tell what we're doing by simply reading the code... Sort of.

This still isn't what I'd consider clean code. The comment is worrying because it probably should not be necessary to express such an explanation in prose, assuming the code is well written (which it isn't). Technically, we're still saying what we're doing, not why we're doing it. We can easily express this "what" directly in our code by using more meaningful names:

```go
for workerID := 0; workerID < 10; workerID++ {
  instantiateThread(workerID)
}
```

With just a few changes to our variable and function names, we've managed to explain what we're doing directly in our code. This is much clearer for the reader because they won't have to read the comment and then map the prose to the code. Instead, they can simply read the code to understand what it's doing.

Of course, this was a relatively trivial example. Writing clear and expressive code is unfortunately not always so easy; it can become increasingly difficult as the codebase itself grows in complexity. The more you practice writing comments in this mindset and avoid explaining what you're doing, the cleaner your code will become.

#### Function Naming

Let's now move on to function naming conventions. The general rule here is really simple: the more specific the function, the more general its name. In other words, we want to start with a very broad and short function name, such as `Run` or `Parse`, that describes the general functionality. Let's imagine that we are creating a configuration parser. Following this naming convention, our top level of abstraction might look something like the following:

```go
func main() {
    configpath := flag.String("config-path", "", "configuration file path")
    flag.Parse()

    config, err := configuration.Parse(*configpath)
    ...
}
```      

We'll focus on the naming of the `Parse` function. Despite this function's very short and general name, it's actually quite clear what it attempts to achieve.

When we go one layer deeper, our function naming will become slightly more specific:

```go
func Parse(filepath string) (Config, error) {
    switch fileExtension(filepath) {
    case "json":
        return parseJSON(filepath)
    case "yaml":
        return parseYAML(filepath)
    case "toml":
        return parseTOML(filepath)
    default:
        return Config{}, ErrUnknownFileExtension
    }
}
```
          

Here, we've clearly distinguished the nested function calls from their parent without being overly specific. This allows each nested function call to make sense on its own as well as within the context of the parent. On the other hand, if we had named the `parseJSON` function `json` instead, it couldn't possibly stand on its own. The functionality would become lost in the name, and we would no longer be able to tell whether this function is parsing, creating, or marshalling JSON.

Notice that `fileExtension` is actually a little more specific. However, this is because its functionality is in fact quite specific in nature:

```go
func fileExtension(filepath string) string {
    segments := strings.Split(filepath, ".")
    return segments[len(segments)-1]
}
```
          

This kind of logical progression in our function names—from a high level of abstraction to a lower, more specific one—makes the code easier to follow and read. Consider the alternative: If our highest level of abstraction is too specific, then we'll end up with a name that attempts to cover all bases, like `DetermineFileExtensionAndParseConfigurationFile`. This is horrendously difficult to read; we are trying to be too specific too soon and end up confusing the reader, despite trying to be clear!

##### Variable Naming

Rather interestingly, the opposite is true for variables. Unlike functions, our variables should be named from more to less specific the deeper we go into nested scopes.

> You shouldn’t name your variables after their types for the same reason you wouldn’t name your pets 'dog' or 'cat'. – Dave Cheney

Why should our variable names become less specific as we travel deeper into a function's scope? Simply put, as a variable's scope becomes smaller, it becomes increasingly clear for the reader what that variable represents, thereby eliminating the need for specific naming. In the example of the previous function `fileExtension`, we could even shorten the name of the variable `segments` to `s` if we wanted to. The context of the variable is so clear that it's unnecessary to explain it any further with longer variable names. Another good example of this is in nested for loops:

```go 
func PrintBrandsInList(brands []BeerBrand) {
    for _, b := range brands {
      fmt.Println(b)
    }
}
```
          

In the above example, the scope of the variable b is so small that we don't need to spend any additional brain power on remembering what exactly it represents. However, because the scope of brands is slightly larger, it helps for it to be more specific. When expanding the variable scope in the function below, this distinction becomes even more apparent:

```go 
func BeerBrandListToBeerList(beerBrands []BeerBrand) []Beer {
    var beerList []Beer
    for _, brand := range beerBrands {
        for _, beer := range brand {
            beerList = append(beerList, beer)
        }
    }
    return beerList
}
```
          

Great! This function is easy to read. Now, let's apply the opposite (i.e., wrong) logic when naming our variables:

```go    
func BeerBrandListToBeerList(b []BeerBrand) []Beer {
    var bl []Beer
    for _, beerBrand := range b {
        for _, beerBrandBeerName := range beerBrand {
            bl = append(bl, beerBrandBeerName)
        }
    }
    return bl
}
```
          

Even though it's possible to figure out what this function is doing, the excessive brevity of the variable names makes it difficult to follow the logic as we travel deeper. This could very well spiral into full-blown confusion because we're mixing short and long variable names inconsistently.

### Cleaning Functions

Now that we know some best practices for naming our variables and functions, as well as clarifying our code with comments, let's dive into some specifics of how we can refactor functions to make them cleaner.

##### Function Length

> How small should a function be? Smaller than that! – Robert C. Martin

When writing clean code, our primary goal is to make our code easily digestible. The most effective way to do this is to make our functions as short as possible. It's important to understand that we don't necessarily do this to avoid code duplication. The more important reason is to improve _code comprehension_.

It can help to look at a function's description at a very high level to understand this better:

```sh 
fn GetItem:
  - parse json input for order id
  - get user from context
  - check user has appropriate role
  - get order from database
```
          

By writing short functions (which are typically 5–8 lines in Go), we can create code that reads almost as naturally as our description above:

```go
var (
    NullItem = Item{}
    ErrInsufficientPrivileges = errors.New("user does not have sufficient privileges")
)

func GetItem(ctx context.Context, json []bytes) (Item, error) {
    order, err := NewItemFromJSON(json)
    if err != nil {
        return NullItem, err
    }
    if !GetUserFromContext(ctx).IsAdmin() {
          return NullItem, ErrInsufficientPrivileges
    }
    return db.GetItem(order.ItemID)
}
```
          

Using smaller functions also eliminates another horrible habit of writing code: **indentation hell**. This typically occurs when a chain of `if` statements are carelessly nested in a function. This makes it very difficult for human beings to parse the code and should be eliminated whenever spotted. Indentation hell is particularly common when working with `interface{}` and using type casting:

```go   
func GetItem(extension string) (Item, error) {
    if refIface, ok := db.ReferenceCache.Get(extension); ok {
        if ref, ok := refIface.(string); ok {
            if itemIface, ok := db.ItemCache.Get(ref); ok {
                if item, ok := itemIface.(Item); ok {
                    if item.Active {
                        return Item, nil
                    } else {
                      return EmptyItem, errors.New("no active item found in cache")
                    }
                } else {
                  return EmptyItem, errors.New("could not cast cache interface to Item")
                }
            } else {
              return EmptyItem, errors.New("extension was not found in cache reference")
            }
        } else {
          return EmptyItem, errors.New("could not cast cache reference interface to Item")
        }
    }
    return EmptyItem, errors.New("reference not found in cache")
}
```
          

First, indentation hell makes it difficult for other developers to understand the flow of your code. Second, if the logic in our if statements expands, it'll become exponentially more difficult to figure out which statement returns what (and to ensure that all paths return some value).

Yet another problem is that this deep nesting of conditional statements forces the reader to frequently scroll and keep track of many logical states in their head. It also makes it more difficult to test the code and catch bugs because there are so many different nested possibilities that you have to account for.

Indentation hell can result in reader fatigue if a developer has to constantly parse unwieldy code like the sample above. Naturally, this is something we want to avoid at all costs.

So, how do we clean this function? Fortunately, it's actually quite simple. On our first iteration, we will try to ensure that we are returning an error as soon as possible. Instead of nesting the `if` and `else` statements, we want to "push our code to the left," so to speak. Take a look:

```go    
func GetItem(extension string) (Item, error) {
    refIface, ok := db.ReferenceCache.Get(extension)
    if !ok {
        return EmptyItem, errors.New("reference not found in cache")
    }

    ref, ok := refIface.(string)
    if !ok {
        // return cast error on reference
    }

    itemIface, ok := db.ItemCache.Get(ref)
    if !ok {
        // return no item found in cache by reference
    }

    item, ok := itemIface.(Item)
    if !ok {
        // return cast error on item interface
    }

    if !item.Active {
        // return no item active
    }

    return Item, nil
}
```
          

Once we're done with our first attempt at refactoring the function, we can proceed to split up the function into smaller functions. Here's a good rule of thumb: If the `value, err :=` pattern is repeated more than once in a function, this is an indication that we can split the logic of our code into smaller pieces:

```go
func GetItem(extension string) (Item, error) {
    ref, ok := getReference(extension)
    if !ok {
        return EmptyItem, ErrReferenceNotFound
    }
    return getItemByReference(ref)
}

func getReference(extension string) (string, bool) {
    refIface, ok := db.ReferenceCache.Get(extension)
    if !ok {
        return EmptyItem, false
    }
    return refIface.(string)
}

func getItemByReference(reference string) (Item, error) {
    item, ok := getItemFromCache(reference)
    if !item.Active || !ok {
        return EmptyItem, ErrItemNotFound
    }
    return Item, nil
}

func getItemFromCache(reference string) (Item, bool) {
    if itemIface, ok := db.ItemCache.Get(ref); ok {
        return EmptyItem, false
    }
    return itemIface.(Item), true
}
```
          

As mentioned previously, indentation hell can make it difficult to test our code. When we split up our `GetItem` function into several helpers, we make it easier to track down bugs when testing our code. Unlike the original version, which consisted of several `if` statements in the same scope, the refactored version of `GetItem` has just two branching paths that we must consider. The helper functions are also short and digestible, making them easier to read.

> Note: For production code, one should elaborate on the code even further by returning errors instead of bool values. This makes it much easier to understand where the error is originating from. However, as these are just example functions, returning bool values will suffice for now. Examples of returning errors more explicitly will be explained in more detail later.

Notice that cleaning the `GetItem` function resulted in more lines of code overall. However, the code itself is now much easier to read. It's layered in an onion-style fashion, where we can ignore "layers" that we aren't interested in and simply peel back the ones that we do want to examine. This makes it easier to understand low-level functionality because we only have to read maybe 3–5 lines at a time.

This example illustrates that we cannot measure the cleanliness of our code by the number of lines it uses. The first version of the code was certainly much shorter. However, it was _artificially_ short and very difficult to read. In most cases, cleaning code will initially expand the existing codebase in terms of the number of lines. But this is highly preferable to the alternative of having messy, convoluted logic. If you're ever in doubt about this, just consider how you feel about the following function, which does exactly the same thing as our code but only uses two lines:

```go  
func GetItemIfActive(extension string) (Item, error) {
    if refIface,ok := db.ReferenceCache.Get(extension); ok {if ref,ok := refIface.(string); ok { if itemIface,ok := db.ItemCache.Get(ref); ok { if item,ok := itemIface.(Item); ok { if item.Active { return Item,nil }}}}} return EmptyItem, errors.New("reference not found in cache")
}
```
          

##### Function Signatures

Creating a good function naming structure makes it easier to read and understand the intent of the code. As we saw above, making our functions shorter helps us understand the function's logic. The last part of cleaning our functions involves understanding the context of the function input. With this comes another easy-to-follow rule: **Function signatures should only contain one or two input parameters**. In certain exceptional cases, three can be acceptable, but this is where we should start considering a refactor. Much like the rule that our functions should only be 5–8 lines long, this can seem quite extreme at first. However, I feel that this rule is much easier to justify.

Take the following function from [RabbitMQ's introduction tutorial to its Go library](https://www.rabbitmq.com/tutorials/tutorial-one-go.html):

```go
q, err := ch.QueueDeclare(
  "hello", // name
  false,   // durable
  false,   // delete when unused
  false,   // exclusive
  false,   // no-wait
  nil,     // arguments
)
```
          

The function `QueueDeclare` takes six input parameters, which is quite a lot. With some effort, it's possible to understand what this code does thanks to the comments. However, the comments are actually part of the problem—as mentioned earlier, they should be substituted with descriptive code whenever possible. After all, there's nothing preventing us from invoking the QueueDeclare function without comments:

```go
q, err := ch.QueueDeclare("hello", false, false, false, false, nil)
```

Now, without looking at the commented version, try to remember what the fourth and fifth `false` arguments represent. It's impossible, right? You will inevitably forget at some point. This can lead to costly mistakes and bugs that are difficult to correct. The mistakes might even occur through incorrect comments—imagine labeling the wrong input parameter. Correcting this mistake will be unbearably difficult to correct, especially when familiarity with the code has deteriorated over time or was low to begin with. Therefore, it is recommended to replace these input parameters with an 'Options' `struct` instead:

```go
type QueueOptions struct {
    Name string
    Durable bool
    DeleteOnExit bool
    Exclusive bool
    NoWait bool
    Arguments []interface{}
}

q, err := ch.QueueDeclare(QueueOptions{
    Name: "hello",
    Durable: false,
    DeleteOnExit: false,
    Exclusive: false,
    NoWait: false,
    Arguments: nil,
})
```
          

This solves two problems: misusing comments, and accidentally labeling the variables incorrectly. Of course, we can still confuse properties with the wrong value, but in these cases, it will be much easier to determine where our mistake lies within the code. The ordering of the properties also doesn't matter anymore, so incorrectly ordering the input values is no longer a concern.

The last added bonus of this technique is that we can use our QueueOptions struct to infer the default values of our function's input parameters. When structures in Go are declared, all properties are initialised to their default value. This means that our `QueueDeclare` option can actually be invoked in the following way:

```go 
q, err := ch.QueueDeclare(QueueOptions{
    Name: "hello",
})
```
          

The rest of the values are initialised to their default value of `false` (except for `Arguments`, which as an interface has a default value of `nil`). Not only are we much safer with this approach, but we are also much clearer with our intentions. In this case, we could actually write less code. This is an all-around win for everyone on the project.

One final note on this: It's not always possible to change a function's signature. In this case, for example, we don't actually have control over our `QueueDeclare`function signature because it's from the RabbitMQ library. It's not our code, so we can't change it. However, we can wrap these functions to suit our purposes:

```go  
type RMQChannel struct {
    channel *amqp.Channel
}

func (rmqch *RMQChannel) QueueDeclare(opts QueueOptions) (Queue, error) {
    return rmqch.channel.QueueDeclare(
        opts.Name,
        opts.Durable,
        opts.DeleteOnExit,
        opts.Exclusive,
        opts.NoWait,
        opts.Arguments,
    )
}
```
          

Basically, we create a new structure named `RMQChannel` that contains the `amqp.Channel` type, which has the `QueueDeclare` method. We then create our own version of this method, which essentially just calls the old version of the RabbitMQ library function. Our new method has all the advantages described before, and we achieved this without actually having to change any of the code in the RabbitMQ library.

We'll use this idea of wrapping functions to introduce more clean and safe code later when discussing `interface{}`.

##### Variable Scope

Now, let's take a step back and revisit the idea of writing smaller functions. This has another nice side effect that we didn't cover in the previous chapter: Writing smaller functions can typically eliminate reliance on mutable variables that leak into the global scope.

Global variables are problematic and don't belong in clean code; they make it very difficult for programmers to understand the current state of a variable. If a variable is global and mutable, then by definition, its value can be changed by any part of the codebase. At no point can you guarantee that this variable is going to be a specific value... And that's a headache for everyone. This is yet another example of a trivial problem that's exacerbated when the codebase expands.

Let's look at a short example of how non-global variables with a large scope can cause problems. These variables also introduce the issue of variable shadowing.

```go
func doComplex() (string, error) {
    return "Success", nil
}

func main() {
    var val string
    num := 32

    switch num {
    case 16:
    // do nothing
    case 32:
        val, err := doComplex()
        if err != nil {
            panic(err)
        }
        if val == "" {
            // do something else
        }
    case 64:
        // do nothing
    }

    fmt.Println(val)
}
```
          

What's the problem with this code? From a quick skim, it seems the `var val string` value should be printed out as `Success` by the end of the main function. Unfortunately, this is not the case. The reason for this lies in the following line:

```go    
val, err := doComplex()
```

This declares a new variable `val` in the switch's `case 32` scope and has nothing to do with the variable declared in the first line of `main`. Of course, it can be argued that Go syntax is a little tricky, which I don't necessarily disagree with, but there is a much worse issue at hand. The declaration of `var val string` as a mutable, largely scoped variable is completely unnecessary. If we do a very simple refactor, we will no longer have this issue:

```go 
func getStringResult(num int) (string, error) {
    switch num {
    case 16:
    // do nothing
    case 32:
       return doComplex()
    case 64:
        // do nothing
    }
    return ""
}

func main() {
    val, err := getStringResult(32)
    if err != nil {
        panic(err)
    }
    if val == "" {
        // do something else
    }
    fmt.Println(val)
}
```
          

After our refactor, val is no longer modified, and the scope has been reduced. Again, keep in mind that these functions are very simple. Once this kind of code style becomes a part of larger, more complex systems, it can be impossible to figure out why errors are occurring. We don't want this to happen—not only because we generally dislike software errors but also because it's disrespectful to our colleagues, and ourselves; we are potentially wasting each other's time having to debug this type of code. Developers need to take responsibility for their own code rather than blaming these issues on the variable declaration syntax of a particular language like Go.

On a side note, if the `// do something else` part is another attempt to mutate the val variable, we should extract that logic out as its own self-contained function, as well as the previous part of it. This way, instead of expanding the mutable scope of our variables, we can just return a new value:

```go 
func getVal(num int) (string, error) {
    val, err := getStringResult(32)
    if err != nil {
        return "", err
    }
    if val == "" {
        return NewValue() // pretend function
    }
}

func main() {
    val, err := getVal(32)
    if err != nil {
        panic(err)
    }
    fmt.Println(val)
}
```
          

##### Variable Declaration

Other than avoiding issues with variable scope and mutability, we can also improve readability by declaring variables as close to their usage as possible. In C programming, it's common to see the following approach to declaring variables:

```go   
func main() {
  var err error
  var items []Item
  var sender, receiver chan Item

  items = store.GetItems()
  sender = make(chan Item)
  receiver = make(chan Item)

  for _, item := range items {
    // ...
  }
}
```
          

This suffers from the same symptom as described in our discussion of variable scope. Even though these variables might not actually be reassigned at any point, this kind of coding style keeps the readers on their toes, in all the wrong ways. Much like computer memory, our brain's short-term memory has a limited capacity. Having to keep track of which variables are mutable and whether or not a particular fragment of code will mutate them makes it more difficult to understand what the code is doing. Figuring out the eventually returned value can be a nightmare. Therefore, to makes this easier for our readers (and our future selves), it's recommended that you declare variables as close to their usage as possible:

```go  
func main() {
    var sender chan Item
    sender = make(chan Item)

    go func() {
        for {
            select {
            case item := <-sender:
                // do something
            }
        }
    }()
}
```

However, we can do even better by invoking the function directly after its declaration. This makes it much clearer that the function logic is associated with the declared variable:

```go    
func main() {
  sender := func() chan Item {
    channel := make(chan Item)
    go func() {
      for {
        select { ... }
      }
    }()
    return channel
  }
}
```

And coming full circle, we can move the anonymous function to make it a named function instead:

```go
func main() {
  sender := NewSenderChannel()
}

func NewSenderChannel() chan Item {
  channel := make(chan Item)
  go func() {
    for {
      select { ... }
    }
  }()
  return channel
}
```
          

It is still clear that we are declaring a variable, and the logic associated with the returned channel is simple, unlike in the first example. This makes it easier to traverse the code and understand the role of each variable.

Of course, this doesn't actually prevent us from mutating our `sender` variable. There is nothing that we can do about this, as there is no way of declaring a `const struct` or static variables in Go. This means that we'll have to restrain ourselves from modifying this variable at a later point in the code.

> NOTE: The keyword const does exist but is limited in use to primitive types only.

One way of getting around this can at least limit the mutability of a variable to the package level. The trick involves creating a structure with the variable as a private property. This private property is thenceforth only accessible through other methods provided by this wrapping structure. Expanding on our channel example, this would look something like the following:

```go
type Sender struct {
  sender chan Item
}

func NewSender() *Sender {
  return &Sender{
    sender: NewSenderChannel(),
  }
}

func (s *Sender) Send(item Item) {
  s.sender <- item
}
```
          

We have now ensured that the `sender` property of our `Sender` struct is never mutated—at least not from outside of the package. As of writing this document, this is the only way of creating publicly immutable non-primitive variables. It's a little verbose, but it's truly worth the effort to ensure that we don't end up with strange bugs resulting from accidental variable modification.

```go
func main() {
  sender := NewSender()
  sender.Send(&Item{})
}
```
          

Looking at the example above, it's clear how this also simplifies the usage of our package. This way of hiding the implementation is beneficial not only for the maintainers of the package but also for the users. Now, when initialising and using the `Sender` structure, there is no concern over its implementation. This opens up for a much looser architecture.

Because our users aren't concerned with the implementation, we are free to change it at any point, since we have reduced the point of contact that users have with the package. If we no longer wish to use a channel implementation in our package, we can easily change this without breaking the usage of the `Send` method (as long as we adhere to its current function signature).

> NOTE: There is a fantastic explanation of how to handle the abstraction in client libraries, taken from the talk [AWS re:Invent 2017: Embracing Change without Breaking the World (DEV319)](https://www.youtube.com/watch?v=kJq81Y7OEx4).

That is it for Part I of this document. Hope you have learned something useful and that you are itching to get back to your codebase to apply these principles.