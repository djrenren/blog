---
layout: post
title: "Baby Steps: Fixing Test Scoping"
date:   2018-08-01 04:00:00 -0700
categories: rust testing
---

I recently set out to fix [a bug][bug] in [`#[test]` expansion in
Rust][test-in-2018]. What follows is a chronicle of that journey.

## The Bug
----------------
When `rustc` encounters a `#[test]` attribute it will do one of two things:

 - In `debug` or `release` mode, the annotated item is removed
 - In `test` mode, the item is made public so it can be called by the test runner

This is typically safe because direct references to `#[test]`
will cause errors in any mode where the function is removed.

Enter glob imports. Statements like `use my_mod::*` allow us to import an
entire namespace into another without naming the specific items. We now have
the requisite pieces to construct the bug:

```rust
mod A {
    pub fn foo() -> i32 { 0 }
}

mod B {
    #[test]
    fn foo() -> i32 { 6 }
}

use A::foo;
use B::*;

fn caller() {
    assert_eq!(foo(), 0);
}
```

When compiled normally, `use B::*` does nothing and the assert inside `caller` will
always pass. When compiled for test, however, `B::foo` will be marked as public,
which will cause `use B::*` to [shadow][shadow] the `foo` we got from `A::foo`.
Our assert now fails saying that 6 does not equal 0. This means `caller` behaves
differently in test builds which is undesirable. Worse still, it happens due to
an implementation detail in the compiler that authors should never worry about.

## The Fix
-----------
**The Constraint:**
Test functions must be made public but they must also not pollute other namespaces.

**The Solution:**
Give test functions a name that can never exist in user code.

Rust has an internal mechanism called `gensym` that allows the creation of
guaranteed unique names. If we rename test functions with a gensym'd name,
then it's okay for them to pollute namespaces because it's impossible to
reference them.

Just renaming isn't enough though, because rust currently allows one test
to call another. While I think this is a terrible thing to allow, we do and
so must continue to allow it. We then need to add the old name back into the
namespace. Just add a `use [gensymed] as foo` to add foo back to our namespace.

**The Nitty Gritty:**
While the theory behind this fix is sound, simply implementing it didn't work
the way I'd expected. I first attempted to implement the change inside
[`libsyntax`][libsyntax]'s `TestHarnessGenerator`, but this resulted in
strange errors about not being able to find imports despite the AST and HIR
being correct. While it seems obvious now, it took me a week to figure out
this was because harness generation runs midway through name resolution.

As a reminder, name resolution is the process by which the compiler realizes
that the variable `x` at one location, is talking about the same variable as
an `x` somewhere else in the code.

A good chunk of name resolution occurs during macro expansion in
[`libsyntax/ext/expand.rs`][expandrs]. In fact, this was already where tests
were being made `pub`. What I had originally discounted as a design mistake,
was actually crucial to correct operation. With a quick reimplementation
everything worked great. I realized I'd forgotten to account for the unused
variable lints, but that was a quick fix.

You can see the result in [this pull request][pr].

## Afterword: Why Write About This
----------------------------------
Honestly, very few people will get much out of this post, but being
lost in a complex open-source project can be disheartening at the very
least. By discussing these esoteric bits, I hope I can help someone who gets stuck
in similar bits of code in the future, or maybe just assure them that it's common
to feel lost. As always, feel free to reach out if you have questions.

[bug]: https://github.com/rust-lang/rust/issues/52557
[test-in-2018]: {% post_url 2018-07-19-test-in-2018 %}
[expandrs]: https://github.com/rust-lang/rust/blob/8eb4941e30d2a40bc03840dd0d99beb5aaf8159d/src/libsyntax/ext/expand.rs
[shadow]: https://doc.rust-lang.org/rust-by-example/variable_bindings/scope.html
[libsyntax]: https://github.com/rust-lang/rust/tree/master/src/libsyntax
[pr]: https://github.com/rust-lang/rust/pull/52890