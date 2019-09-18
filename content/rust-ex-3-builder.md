+++
title = "Rust exercise 3: The builder pattern"
date = 2019-09-16T14:00:00Z
[taxonomies]
categories = ["rust-exercises"]
+++


## Background

While we were able to get a long way using an argument struct in the previous
exercise, there are limitations to that pattern: Basic usage still requires
users to instantiate all the fields, even if there are lots of unused options.
We got around that by implementing the `Default` trait, but that requires some
funny looking syntax to fill the remaining fields of the struct.  In this
exercise, we will implement the builder pattern to provide a more
natural-looking syntax for implementing optional arguments.

## Task

In this pattern, we still create a secondary struct to contain the arguments,
but instead of treating it as a raw struct, we make the fields private, and
provide methods to set each option.  The other change is that when we are ready
to create the app object, instead of calling `App::new(&builder)`, we call `let
app = builder.create()`.  In some variations, the user creates the builder
directly, and in others the user calls a `::build()` associated function on the
`App` type to get an empty builder.  We will prefer the latter option, as the
user then only needs to import `App`.  You will see both in the wild though.
[`OpenOptions`]() in `std` is a builder for the [`File`]() type that gets
instantiated directly.  The [rocket](https://rocket.rs) web framework uses the
`::build()` pattern, but with the more colorfully named `::ignite()` function
(and `.create()` is replaced with `.launch()`).

Create an AppBuilder struct that implements this pattern.  After this
iteration, your app instantiation should look like this: 

```rust 
use applib::App;

let app = App::build()
    .host("localhost")
    .port(8888)
    .use_tls(true)
    .workers(3)
    .logfile("/var/log/app.log")
    .create()?;
```

Note that we no longer need to provide an option for the logfile.  If we call the
method, it gets an argument.  If we don't call the method, it remains None. 
Of course you should also have methods defined for the other, unused optional 
arguments.  Any actions that could result in an error, such as opening a network
socket, should happen in the `.create()` method.  


## Hints

1.  In order to implement the fluent method calls that make the builder pattern
    look clean, you'll need each method to return an object that you can call
    the next method on.  This means your signatures will either look like:

    ```rust
    fn host(&mut self, host: &str) -> &mut AppBuilder {}
    ```

    where each method takes and returns a mutable reference, or it will take
    ownership of the object and return it, like:
     
    ```rust
    fn host(self, host: &str) -> AppBuilder {}
    ```
    
    I tend to prefer the latter, as it leads to fewer surprises when dealing 
    with half-created builders.  I'm not sure there's standard terminology for 
    this, but for now, we'll call the former "borrowed builders" and the latter
    "owned builders."

2.  The AppBuilder struct will need to hold Option values for any arguments
    that don't have a default value, even if the value is not optional in
    the app itself.  In this case the None value indicates that the argument
    is not set *yet*.


## Going further: Required arguments come first

Depending on the instantiation logic, the create() method may need to return a
Result.  This may be due to unavailable resources, or file permissions, or it
may be because the app was misconfigured (we failed to provide a port number,
and the app doesn't assume a default, for example).  We can actually lean on
the type system to ensure at compile time that all required arguments are
provided.  

Assume for a moment that the App requires a host string and port.  We could
pass those values directly to the `::build()` function that is associated with
our App, so the builder always has these values available (or to the
`.create()` method, for that matter). but then we lose some of the
self-documenting benefits of the builder pattern.  In many cases this will be
a sufficient compromise between readability and safety, but what if we want
to keep the builder pattern?  

The first option is to provide multiple builder phases.  This only works if you
have an owned builder.  If you went with a borrowed builder before, change it
now.  The App::build function will return an AppHostBuilder, which only has a
`.host()` method.  That method will return an `AppPortBuilder`, which only has
a `.port()` method.  Calling `.port()` will return our general `AppBuilder`,
which now has a host and a port available to it.  We can remove the host and
port methods from `AppBuilder` now, and since there's no way for a caller to
create an `AppBuilder` without going through the AppHostBuilder and the
AppPortBuilder, we know at compile time that those values have been provided.

Types that enforce ordering of operations like this are known as "session types", 
and are a wonderfully useful form of state machine, that can be useful in a
number of contexts.

## Going further: Required arguments, any order 

But what if we still want the user to be able to call the methods in any order,
while guaranteeing that host() and port() have been called In what follows,
we'll go deep into the type system pretty quickly.  If something doesn't click
right away, don't worry about it.  What follows is rarely necessary, but will
help your understanding of the type system.  We can use generic types to
enforce that our host and port methods have been called.  Along the way, we'll
meet some new concepts like uninhabitable types and phantom data.

...
