+++
title = "Rust exercise 1: Iterator extension trait"
date = 2019-09-16T12:00:00Z
[taxonomies]
categories = ["rust-exercises"]
+++

Here's a fun smallish, intermediate level rust exercise: Implement an extension
trait that allows you to call .skipping(n) on an iterator, and only yield every
nth item, starting with the first one.  (An extension trait is one that adds
functionality to anything that already implements a existing trait, usually one
in std).

It should pass the test below.


Here are some hints if you need help:

1. For examples of how to implement Iterator combinators, see the documentation
   (especially the src link) for [the std::iter::Take
   struct](https://doc.rust-lang.org/std/iter/struct.Take.html), and the
   [Iterator::take() method](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.take) 
   (or any of the other iterator combinator methods).
2. For examples of how to implement an extension trait, see the excellent
   [byteorder crate](https://github.com/BurntSushi/byteorder) (by Andrew
   Gallant, a.k.a. BurntSushi), and its `ReadBytesExt` and `WriteBytesExt` 
   traits, which let you read and write integers of different sizes to a file, 
   network stream, or other Reader or Writer.

Tests:

```rust
#[test]
fn skip_vec() {
    use skippable::Skippable; // This is your extension trait
    let mut it = (0..9).skipping(3);
    assert_eq!(it.next(), Some(0));
    assert_eq!(it.next(), Some(3));
    assert_eq!(it.next(), Some(6));
    assert_eq!(it.next(), None);
    let v = vec!["red", "orange", "yellow", "green", "blue", "purple"];  // Indigo isn't a real color
    assert_eq!(
        v.into_iter().skipping(2).take(2).collect::<Vec<_>>(),
        vec!["red", "yellow"]);
}                                              
```
