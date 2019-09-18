+++
title = "Rust exercise 2: Argument struct"
date = 2019-09-16T13:00:00Z
[taxonomies]
categories = ["rust-exercises"]
+++

## Background 

Often, when a constructor takes a large number of arguments, it becomes unclear what each argument means.

Take for example this sample code:

```
use applib::App;

let app = App::new(
    "localhost",
    8888,
    true,
    3,
    Some("/var/log/app.log"),
    None,
    None,
);
app.run();
```

One could readily assume that the first two arguments are a host name and port
number, but what does true mean?  How about 3?  The next one seems to be a path
to a log file, but is it input or output?  The two closing None values are even
less clear.  The App constructor is both hard to read and hard to write without
constant referral to the documentation.

## Exercise

The next few exercises will look at various ways of making this code clearer,
and of using the type system to make sure we construct the object correctly.

Your task is to replace the argument list with a single `AppOptions` struct.
The AppOptions struct will then present the field names at the call site to
help document the meaning of the call, while keeping a separation between the
call and the internals of the App object.  (The App object might store a
TcpHandler instead of a host and port, for example).

Here's the old function signature:

```rust
impl App {
    fn new(
        host: &str, port: u16, use_tls: bool, workers: usize, 
        logfile: Option<&str>, outfile: Option<&str>, rng: Option<rand::Rng>,
    ) -> App { unimplemented!() }
}
```

After implementing the AppOptions struct, the call should look like:

```rust
use applib::{App, AppOptions};

let app = App::new(AppOptions {
    host: "localhost",
    port: 8888,
    use_tls: true,
    workers: 3,
    logfile: Some("/var/log/app.log"),
    outfile: None,
    rng: None,
});
```
 
## Hints:

Make sure you can create an AppOptions struct from anywhere you might want to
instantiate an App object.  This may mean anywhere within your crate.  It may
mean from any crate whatsoever.  Look into visibility modifiers for structs and
struct attributes.  

## Going further: cargo doc

Run `cargo doc --open` on your crate.  Is it easy to figure out how to
instantiate your App?  Add [documentation
comments](https://facility9.com/2016/05/writing-documentation-in-rust/) to your
structs and new() method to help programmers who want to use your App.

## Going further: Default AppOptions

This can still be pretty verbose, if there are a large number of unused
options.  Implementing the
[`Default`](https://doc.rust-lang.org/std/default/trait.Default.html) trait can
clean that up nicely. 

```rust
let app = App::new(AppOptions {
    use_tls: true,
    workers: 3,
    logfile: Some("/var/log/app.log"),
    ..AppOptions::default()
});
```

Appropriate defaults for these values will often be application-dependent, so
deriving Default probably isn't enough: The default host shouldn't be `""`; the
default port shouldn't be `0`.

## Going further: More complex derives with Derivative

After you have implemented default manually, take a look at the
[derivative](https://mcarton.github.io/rust-derivative/) crate.  The standard
library uses very simple heuristics for deriving Default, which, as we've seen,
don't always match an application developers needs.  The derivative crate uses
a few common patterns from the rust ecosystem to allow defining more complex
default implementations using a custom derive, plus custom struct and attribute
annotations.  Similar annotations are used by crates that we'll be using a lot,
like [serde](https://docs.serde.rs) and [structopt](https://docs.rs/structopt).
