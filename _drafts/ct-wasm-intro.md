---
layout: post
title: "CT-WASM: Custom Crypto on the Web"
---
I've spent the past year of my PhD working on a series of APIs to enable web
developers to serve cryptographic implementations over the web. The result of this
work is a modest extension to WebAssembly that is provably safe from both direct
leakage (i.e. passing secret data somewhere you shouldn't) and indirect
leakage (i.e. timing attacks a la [Spectre][spectre]).

### The Problem
Javascript already has a cryptography API in the form of [Web Crypto][web-crypto],
but it has some serious drawbacks:

 - No guarantees of a common set of algorithms
 - Security flaws require browsers updates
 - Modern algorithms such as [Poly1305][poly1305] are missing

These problems are all avoidable if developers could ship crypto libraries to
the client side. Unfortunately, javascript is just not suited to this task.
The complex nature of javascript runtimes (especially JITs) cause
unpredictable runtime performance. While this is absolutely fine in the
common case, it can lead to timing attacks.

### Anatomy of a Timing Attack
The idea behind a timing attack is to measure the time it takes for certain
code to run and use that information to uncover secret data without needing
access to the data in memory. Typically, these attacks work by finding an
operation whose execution time varies depending on what data it's operating.

For example, consider the code that checks if a password has the right number
of uppercase and special characters. If it takes 0.1ms to check a single
character, and it takes .7ms to check your password, how long is your
password? Now that I know your password is 7 characters long, I've
dramatically reduced the number of guesses it takes to discover your password.

In the real world, the numbers are fuzzier and the attacks are much more
mathematically complex, but the takeaway here is simple:

> The value of secret data should not affect execution time.

### Enforcing "Constant Time"
When we use the term, "constant time" we mean constant with respect to secret
data. If the data is public, then the timing information would only leak things
we already know. For example, the load time of [xkcd.com][xkcd] might help us
figure out the size of today's comic, but we could access that more directly.
Conversely, the load time of your credit card statement might tell us how
frequently you use that card.

### 

[xkcd]: https://xkcd.com
[poly1305]: https://en.wikipedia.org/wiki/Poly1305
[spectre]: https://meltdownattack.com/
[web-crypto]: https://www.w3.org/TR/WebCryptoAPI/
