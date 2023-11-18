---
layout: post
title:  "Organizing code through packages, crates, and modules in Rust"
kicker: "Rust Basics"
summary: "As we write larger programs with more code, it will become increasingly essential to organize our code. By grouping related functionality, we will make it easier to locate the code that implements a specific feature and to modify its functionality. As our project expands, it's high time to organize our code by splitting it into multiple packages, crates, and modules to have better readability and maintainability. We will look at what's in Rust that helps us organize the code."
image: assets/images/posts-cover-images/organizing-code-in-rust.jpg
imageshadow: true
toc: true
author: senthil
date: 2023-11-18 00:00:00 +0530
# last_modified_at: 2023-11-18
tags: [ "packge", "crates", "modules" ]
categories: rust-basics
featured: false
hidden: false
---

# What is crates in Rust?

A **crate** is a way to organize Rust code into reusable _libraries_ or _executable programs_ (binaries). In Rust, a "crate" is the term used to refer to a _compilation unit_ (binary) or a _package_ (library) of Rust code. A crate is equivalent to the terms "library" or "package" in other programming languages. A crate can come in one of two forms: a **binary crate** or a **library crate**. Thus, a crate can produce an executable or a library, depending on the project.

There are _four types_ of crates in Rust:

- **Library crates**
- **Binary crates**
- **Workspace crates**
- **Package crates**

## Library crates

Library crates are intended to be used by other programs. They don't have a `main` function, and they don’t compile into an executable. They are usually meant to provide resuable functionality that can be imported and used by other Rust code.

Example of a library crate stored as `lib.rs`:

```rust
// lib.rs (crate root for the library)
pub mod my_module {
    pub fn my_function() {
        println!("Hello from my_function!");
    }
}
```

## Binary crates

Binary crates produce an executable when compiled. They typically contain the `main` function, which is the entry point of the program.

Example of a binary crate stored as `main.rs`. This binary crate uses library crate (`lib.rs`).

```rust
// main.rs (crate root for the binary)
mod my_module;

fn main() {
    my_module::my_function();
}
```

## Workspace crates

A workspace is a way to manage multiple crates together in the same directory hierarchy. A workspace typically includes one or more library or binary crates, and it is defined by a `Cargo.toml` file that lists the member crates as shown below:

```toml
[workspace]
members = [
    "my_library",
    "my_binary",
]
```

## Package crates (aka package)

A package crate is the combination of a library or binary crate along with its associated metadata (about the project and its dependencies) and configuration defined in the `Cargo.toml` file. A package is a bundle of one or more crates that provides a set of functionality. When we create a new Rust project using Cargo (`cargo new my_project`), it sets up a package crate for us. Package crates are equivalent to Java's JAR (Java Archive) files. The JAR files package compiled Java classes and resources into a single file.

> **Note**: A package can contain as many binary crates as we like, but at most only one library crate. A package must contain at least one crate, whether that’s a library or binary crate.

Example of a package crate (`Cargo.toml`):

```rust
[package]
name = "my_project"
version = "0.1.0"
edition = "2021"
```

Here are some key points about crates in Rust:

- **Single-file crate**: A crate can be as simple as a single Rust source file with associated modules. The file has the `.rs` extension.
- **Multi-file crate**: A crate can also consist of multiple Rust source files, organized into modules.
- **Dependency management**: Crates can depend on other crates, and these dependencies are specified in the `Cargo.toml` file. The Rust package manager, **Cargo**, is used for building (compiling), testing, and managing dependencies for Rust projects.

> **Note**: A crate is the smallest amount of code that the Rust compiler considers at any given moment. Even if we run the Rust compiler, `rustc`, and pass a single source code file, the compiler considers that single file to be a crate. Crates can contain modules, and the modules may be defined in other files that get compiled with the crate.

---

# What is module in Rust?

When a project starts getting large, it’s considered good software engineering practice to split it up into a bunch of smaller files or namespaces and then fit them together. **Modules** in Rust help in splitting a program (typically a larger program) into logical units for better readability and organization. Having said that, a module is a collection of "related" items that includes _functions_, _structs_, and even other modules as well.

Modules allow us to _partition_ our code within the crate itself. This implies that a crate can have multiple modules within it. We can define modules to group related functionality, and these modules can be nested to create a hierarchy.

---

# Frequently asked questions (FAQ)

## What happens if a Rust package contains `src/main.rs` and `src/lib.rs`—two crates with the same name as the package?

A package can have multiple binary crates by placing files in the src/bin directory: each file will be a separate binary crate[^1].

## What is crate root?

Every crate has a "crate root," which is the main Rust source file that serves as the entry point for the compilation. By convention, this file is named `lib.rs` or `mainrs`. 

Cargo follows a convention that `src/main.rs` is the crate root of a binary crate with the same name as the package. In a typical Rust project, we would have a `Cargo.toml` file in the project root that contains metadata about the project, including its name as shown below:

```toml
[package]
name = "my_project"
version = "0.1.0"
edition = "2021"
```

In this case, Cargo expects that the resulting executable will have the same name as specified in the `name` field of the `[package]` section in `Cargo.toml`. Thus, the executable would be named "my_project". Likewise, Cargo follows a similar convention for library crates (`src/lib.rs`). Cargo passes the crate root files to `rustc` (Rust compiler) to build the library or binary.

## What is crate vs. module?

In Rust, both crates and modules are used for _organizing code_, but they serve different purposes and operate at different levels of granularity within the code structure. A crate is the top-level organizational unit or entity in Rust. <mark>A crate can contain one or more modules</mark>.

A module is a way to organize code _within a crate_. It provides a means of grouping related items, such as _functions_, _structs_, _enums_, and other modules as well.

---

[^1]: [Packages and Crates <i class="fa-solid fa-arrow-up-right-from-square"></i>](https://doc.rust-lang.org/book/ch07-01-packages-and-crates.html#packages-and-crates){:target="_blank"}