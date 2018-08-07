---
layout: post
title: "Proposal: Rust Custom Test Frameworks"
date:   2018-08-06 01:00:00 -0700
categories: rust testing
---

The Rust community recently approved a [Custom Test Frameworks eRFC][eRFC]
which lays out a series of goals and possible directions of exploration for
implementing custom test frameworks. In this post, I present my own proposed
fulfillment of the RFC with rationale.

## Background

Today in Rust, anyone can write a test using the `#[test]` macro:

```rust
#[test]
fn my_test() {
    assert_eq!(2 + 2, 4);
}
```

This is incredibly ergonomic, but offers little control to people writing tests.
Every `#[test]` function must be a function of type `Fn() -> ()` and will be run using the
default `libtest` test runner. If a test author needs more than the `libtest` runner
can provide, then they can no longer use the `#[test]` macro. This proposal seeks
to offer the ergonomic power of `#[test]` while providing the flexibility required to
define and mix custom test formats and test runners.

## Summary
------------
Two small additions are enough to enable the creation of powerful test frameworks:

 - Framework-agnostic `#[test_case]` macro for test aggregation
 - Add a crate attribute to define a test runner function

This allows us to write code like so:

```rust
#![test_runner(tap::runner)]

use quickcheck::*;

#[quickcheck]
fn identity(a: i32) -> bool {
    a * 1 == a
}

#[test]
fn foo() {
    assert!(true);
}
```

This code contains two tests, written in two different formats, being executed by a third library.

### Framework-Agnostic `#[test_case]`
--------------------------------------
`#[test_case]` is a marker to the compiler to aggregate the item beneath it and pass it to the test runner.

**Semantics:**
 - The compiler will exclude annotated items will in non-test configurations
 - The compiler will pass all annotated items as a slice to the test runner
 - Annotated items must be a nameable `const`, `static`, or `fn`.

**Rationale:**
`#[test]` has special smarts for working with `libtest` that we want to continue to work, but also
be avoidable. If people want to provide syntactic sugar for declaring tests they can do so with their
own proc_macro attribute.

**Required Support Work:**
In order to avoid doing potentially expensive macro expansions in non-test builds, each third-party test macro needs to be two layers deep. The first step, would expand like so:

`#[quickcheck]` → `#[cfg(test)] #[quickcheck_inner]`

We can provide this in an external support library.

### `#![test_runner]` Crate Attribute
--------------------------------------
The goal of the `test_runner` attribute is to allow test frameworks to be written as simple functions.

**Semantics:**
 - If the attribute is not provided [`libtest::test_main_static`][libtest_main] is assumed.
 - The attribute requires exactly 1 parameter, which is the path to the runner function.
 - The type of the function must be `Fn(&[&mut Foo]) -> impl Termination` for some `Foo` which is the test type

**Rationale:**
As a crate attribute, declaration in-file and through command line is already
understood. The parameter is a function to make runner implementation simple.
Passing tests as an `&mut T` to allow for the use of trait objects. We don't
pass `Box` values so that testing is possible on systems without dynamic
allocation. We only allow one test runner because it will have to mediate
things like command line arguments.

**Required Support Work:**
Test runners will need to have a baseline trait that determines the minimal
interface of a test. This will serve as the compatiblity layer between
test-producing attributes and various test runners. Furthermore, we will need
to stabilize the [`TestDescAndFn`][testdaf] struct from `libtest` so that the trait can
be implemented for it, so custom test runners can run existing tests.

## Examples
-------------

### The Test Runner Author
--------------------------
Suppose a test author wants to be able to query and execute tests from within an IDE.
The editor has a standard API for test executables to adhere to, so they author
a test runner that adheres to that specification, starting with a new crate:

```bash
$ cargo new --lib editor_runner
```

They then add the community-defined `Testable` trait to their `Cargo.toml` like so:

```toml
[dev-dependencies]
testable = "0.4"
```

Now it's time to write the runner:

```rust
pub fn runner(tests: &[&dyn testable::Testable]) -> impl Termination {
    // parse args...
    // run tests
    // communicate through stdio
    // exit code
}
```

To use this test runner they add a Cargo
`dev-dependency` for the runner and add the following to their lib.rs:

```rust
#![test_runner(editor_runner::runner)]
```

### The Test Format Author
---------------------------
Many crates such as [`criterion`][criterion] and [`quickcheck`][quickcheck] offer
new ways to declare tests. I call these *test formats*. Typically, these are proc_macro
attributes that allow for a different declaration syntax than `#[test]`. Some, like
`quickcheck` can just wrap `#[test]`, but this can get messy the more removed your
test format is from a simple function. Consider writing a test format for testing an
HTTP server:

```rust
#[http_test]
const TEST_INDEX: HttpTest = HttpTest {
    request: HttpRequest {
        url: "/",
        method: "GET"
    },
    response: HttpResponse {
        body: Some("Hello World")
    }
}
```

This test would perform the request and compare the response objects.
To enable this the format author first declares their struct type:

```rust
struct HttpTest {
    request: HttpRequest,
    response: HttpResponse,
    name: &'static str
}
```

then implements the `Testable` trait:

```rust
impl testable::Testable for HttpTest {
    fn run(&self) -> () {
        // Make request
        // Assert equality on response fields
    }
    fn name(&self) -> String {
        self.name
    }
}
```

Lastly, to make things nice for their users, they create a macro that automatically
records the test name by turning:

```rust
#[http_test]
const TEST_INDEX: HttpTest = HttpTest {
    //...
}
```

into:

```rust
#[test_case]
const TEST_INDEX: HttpTest = HttpTest {
    name: concat!(module_path!(), "TEST_INDEX")
    //...
}
```

Because `HttpTest` implements `Testable` it can be used with any test runner that
accepts `Testable`'s. Sometimes, however, we want specialized features in the runner
which are coupled to the declaration. This leads us to our third example:

### The Framework Author
-------------------------
Framework authors seek to extend the very idea of what it means to be a test. These
will require cooperation between the runner and the declaration format but can
still provide modularity and compatibility.

Imagine I want to write a test framework that supports nested test suites.
This model is actually compatible with existing simple tests that may already
exist in the project so we declare an extension of `Testable` for our
framework:

```rust
trait TestSuite: Testable {
    fn children(&self) -> impl Iterator<Item=TestSuite> {
        iter::empty() // A regular test has no children
    }
}

impl<T> TestSuite for T where T: Testable {}
```

Now, the test runner I write will accept `&[&dyn TestSuite]` instead of
`&[&dyn Testable]`, but all `Testable`'s will continue to work. All that's
left is to decide the form of the struct and macro I wish to expose to my
users. It could be something like this:

```rust
#[test_suite]
mod my_suite {
    #[suite_member]
    fn foo() {}

    #[suite_member]
    fn bar() {}
}
```

Because everything is still behind a trait, this approach would allow
people to write their own `TestSuite` constructing macros and to produce
alternate runners for `TestSuite`'s.

## Useful Properties
-----------------------
This proposal has some implicit properties that are worth calling out:
 - `cargo test` works out of the box
 - `#[test]` continues to work alongside new tests
 - Test frameworks are just regular libraries

## Open Questions
------------------
While the proposal seems strong to me, there are still questions that need answering:
 - Does this meet the needs of [`wasm-bindgen`][wasmb] and similar?
 - What needs to be in the `Testable` trait?
 - How will `cargo bench` work when test runners can change?
 - What have I missed?

If you're interested in playing around with the proposal, I've implemented it at
[djrenren/rust][github], and built some examples at [djrenren/rust-test-frameworks][frameworks].

[frameworks]: https://github.com/djrenren/rust-test-frameworks
[github]: https://github.com/djrenren/rust/tree/custom-test-frameworks
[criterion]: https://github.com/japaric/criterion.rs
[quickcheck]: https://github.com/BurntSushi/quickcheck
[wasmb]: https://github.com/rustwasm/wasm-bindgen
[testdaf]: https://doc.rust-lang.org/test/struct.TestDescAndFn.html
[libtest_main]: https://doc.rust-lang.org/test/fn.test_main_static.html
[eRFC]: https://github.com/rust-lang/rfcs/blob/master/text/2318-custom-test-frameworks.md