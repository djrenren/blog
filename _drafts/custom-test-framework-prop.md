---
layout: post
title: "Proposal: Rust Custom Test Frameworks"
categories: rust testing
---

The Rust community recently approved a [Custom Test Frameworks eRFC][eRFC]
which lays out a series of goals and possible directions of exploration for
implementing custom test frameworks. In this post, I present my own proposed
fulfillment of the RFC with rationale.

## Summary
------------
Two small additions are enough to allow for the creation of powerful test frameworks:

 - Agnostic `#[test_case]` macro for test aggregation
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

## Framework Agnostic `#[test_case]`
--------------------------------------
`#[test_case]` is simply a marker to the compiler to aggregate the item beneath it and pass it to the test runner.

**Semantics:** 
 - Annotated items will be excluded in test configurations
 - Annotated items will be passed as an array to the test runner
 - Annotated items must be a nameable `const`, `static`, or `fn`.

**Rationale:**
`#[test]` has special smarts for working with `libtest` that we want to continue to work, but also
be avoidable. If people want to provide syntactic sugar for declaring tests they can do so with their
own proc_macro attribute.

**Required Support Work:**
In order to avoid doing potentially expensive macro expansions in non-test builds, each third-party test macro needs to be two layers deep. The first step, would simply expand like so:

`#[quickcheck]` → `#[cfg(test)] #[quickcheck_inner]`

We can provide this in an external support library.

## `#![test_runner]` Crate Attribute
--------------------------------------
The goal of the `test_runner` attribute is allow test frameworks to be written as simple functions.

*Semantics:*
 - If the attribute is not provided [`libtest::test_static_main`][libtest_main] is assumed.
 - Exactly 1 parameter is required
 - Provided parameter must be a path to a function
 - Type of the function must be `Fn(&[T]) -> impl Termination` for some `T` which is the test type

**Rationale:**
As a crate attribute, declaration in-file and through command line is already understood. The parameter is a function to make runner implementation simple.
We only allow test runner because it will have mediate things like command line arguments.

**Required Support Work:**
Test runners will need to have a baseline trait that determines the minimal
interface of a test. This will serve as the compatiblity layer between
test-producing attributes and various test runners. Furthermore, we will need
to stabilize the [`TestDescAndFn`][testdaf] struct from `libtest` so that the trait can
be implemented for it, so custom test runners can run existing tests.

## Concerns
-----------
 - Does this design meet the needs of [`wasm_bindgen`][wasmb] and similar?
 - Is test trait alignment reasonable?

[wasmb]: https://github.com/rustwasm/wasm-bindgen
[testdaf]: https://doc.rust-lang.org/test/struct.TestDescAndFn.html
[libtest_main]: https://doc.rust-lang.org/test/fn.test_main_static.html
[eRFC]: https://github.com/rust-lang/rfcs/blob/master/text/2318-custom-test-frameworks.md