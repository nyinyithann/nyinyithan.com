---
layout: post
title: Hello World!
categories: [Rust]
tags: [Rust]
---

This post is just to test Rust syntax highlighting on GitHub page.
Here is a valid syntax in Rust.

```rust
let lambda = || || || "Hello, World!";
```

Let's try with main() function.

```rust
fn main() {
    println!(
        "{}",
        (|| || || || || || ||  "Hello, World!")()()()()()()()
    );
}
```
