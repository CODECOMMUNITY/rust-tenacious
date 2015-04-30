## rust-tenacious 

[![Build Status](https://travis-ci.org/Manishearth/rust-tenacious.svg?branch=master)](https://travis-ci.org/Manishearth/rust-tenacious)

This plugin warns when types marked `#[no_move]` are being moved.

This is quite useful for ensuring that things don't get moved around when data is shared via an FFI. Servo [uses this](https://github.com/servo/servo/pull/5855) for safely sharing rooted values with the spidermonkey GC.

Note that `#[no_move]` is transitive, any struct or enum containing a `#[no_move]` type
must be annotated as well. Similarly, any type with `#[no_move]` substitutions in its type parameters
(E.g. `Vec<Foo>` where `Foo` is `no_move`) will be treated as immovable.

Example:


```rust
#![plugin(tenacious)]
#![feature(custom_attribute, plugin)]

#[no_move]
#[derive(Debug)]
struct Foo;

fn main() {
    let x = Foo;
    let y = x; // warning
    bar(Some(y)); // warning   
}

fn bar(t: Option<Foo>) {
    match t {
        Some(foo) => { // warning
            println!("{:?}", foo)
        },
        _ => ()
    }

}

struct MoreFoo {
    foos: Vec<Foo> // warning
}

#[no_move]
struct MoreFoo2 {
    foos: Vec<Foo> // no warning
}
```


Note that this will lint on the moving of temporaries (though it's easy to tweak it to do so). For example, if `foo()` returns a move-protected value, `bar(foo())` will be linted.
While usually there's no in-memory move happening when passing to such a function, the function may move internally if it's generic.


It will not catch moves within generic unsafe functions like `mem::swap()` and `mem::replace()``
