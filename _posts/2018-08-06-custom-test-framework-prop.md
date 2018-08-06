---
layout: post
title: "Proposal: Rust Custom Test Frameworks"
date:   2018-08-01 01:00:00 -0700
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

### Framework Agnostic `#[test_case]`
--------------------------------------
`#[test_case]` is simply a marker to the compiler to aggregate the item beneath it and pass it to the test runner.

**Semantics:**
 - Annotated items will be excluded in non-test configurations
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

### `#![test_runner]` Crate Attribute
--------------------------------------
The goal of the `test_runner` attribute is allow test frameworks to be written as simple functions.

*Semantics:*
 - If the attribute is not provided [`libtest::test_static_main`][libtest_main] is assumed.
 - Exactly 1 parameter is required
 - Provided parameter must be a path to a function
 - Type of the function must be `Fn(&[&mut T]) -> impl Termination` for some `T` which is the test type

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
Suppose I want to be able to query and execute tests from within my IDE.
The editor has a standard API for test executables to adhere to, so I author
a test runner that adheres to that specification, starting with a new crate:

```bash
$ cargo new --lib editor_runner
```

I then add the community-defined `Testable` trait to my `Cargo.toml` like so:

```toml
[dependencies]
testable = "0.4"
```

Now it's time to write my runner:

```rust
pub fn runner(tests: &[Box<dyn test::Testable]) -> impl Termination {
    // parse args...
    // communicate through stdio
    // exit code
}
```

If anyone wants to use this test runner they simply add a Cargo
`dev-dependency` for the runner and add the following to their lib.rs:

```rust
#![test_runner(editor_runner::runner)]
```

### The Test Format Author
---------------------------
Many crates such as [`criterion`][criterion] and [`quickcheck`][quickcheck] offer
new ways to declare tests. Currently criterion relies on a custom test runner that the
user must insert. This would no longer be required. I'd simply have to pick a type that
captures the appropriate information:

```rust
struct CriterionTest<A> {
    name: String,
    fn: Fn(&mut Criterion) -> ()
}
```

and implement the `Testable` trait:

```rust
impl testable::Testable for CriterionTest {
    fn run(/*...*/){ /*...*/ }
    fn name(&self) -> String {
        self.name
     }
}
```

Lastly, to make things nice for my users, I'd want to create a macro that turns:

```rust
#[criterion]
fn foo(&mut Criterion) {
    /* ... */
}
```

into:

```rust
#[test_case]
const foo: CriterionTest = CriterionTest {
    name: "foo",
    fn: |&mut Criterion| { /*...*/ }
}
```

This allows people to use a given declaration format without necessarily being tied
to a given runner. Sometimes, however, we want specialized features in the runner
which are coupled to the declaration. This leads us to our third example:

### The Framework Author
-------------------------
Framework authors seek to extend the very idea of what it means to be a test. These
will require cooperation between the runner and the declaration format, but we can
still provide modularity and compatibility.

Imagine I want to write a test framework that supports nested test suites.
This model is actually compatible with existing simple tests that may already
exist in the project so we declare an extension of `Testable` for our
framework:

```rust
trait TestSuite: Testable {
    fn children(&self) -> Iterator<Item=TestSuite> {
        iter::empty()
    }

    fn run_suite(&self) -> impl Termination {
        self.run()
    }
}
```

Now, the test runner I write will accept `&[&mut dyn TestSuite]` instead of
`&[&mut dyn Testable]`. All that's left is to decide the form of the struct and macro
I wish to expose to my users. This structure would also allow people to write their own
`TestSuite` constructing macros and to produce alternate runners for `TestSuite`'s.

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