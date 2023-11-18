---
layout: post
title:  "Basics of Rust Concurrency"
kicker: "Concurrency"
summary: "Rust's ownership and borrowing features prevent us from experiencing memory-related problems. Rust is a great choice when performance matters and it solves pain points that bother many other languages."
image: assets/images/posts-cover-images/rust-concurrency.jpg
imageshadow: true
toc: true
author: senthil
date: 2023-11-17 00:00:00 +0530
# last_modified_at: 2023-11-18
tags: [ "concurrency" ]
categories: concurrency
featured: false
hidden: true
---

# Overview

Typically, in an operating system, multiple processes run concurrently. Operating systems make every effort to isolate these processes from one another. When processes are isolated from one another, it indicates that a process is typically not able to communicate with or access the memory of another process. However, threads within the same process are not isolated from each other. Threads within the process share memory and are able to communicate and collaborate using that shared memory.

> **Spawn:** In the context of threading, "spawn" typically refers to the creation of a new thread or process. The term is commonly used to describe the action of initiating a parallel or concurrent execution unit.

We will look at how threads are spawned in Rust and all the fundamental concepts surrounding them, such as how to safely share data between multiple threads.

---

# Threads in Rust

Every program is made up of one or more threads. <mark>A thread is the smallest executable unit of a process</mark>. Threads enable us to split a program into multiple tasks, each executed in a separate thread. A program begins with exactly one thread, referred to as the "main thread". This main thread will execute our main function, and can be used to spawn more threads if necessary. In Rust, new threads[^1] are spawned using the `thread::spawn` function from the standard library (`std` module[^2]). This `thread::spawn` function creates a **native operating system** thread, aka **kernel-level thread** (**KLT**).

---

# Frequently asked questions (FAQ)

## What is module in Rust?

When a project starts getting large, it’s considered good software engineering practice to split it up into a bunch of smaller files or namespaces and then fit them together. **Modules** in Rust help in splitting a program (typically a larger program) into logical units for better readability and organization. Having said that, a module is a collection of "related" items that includes _functions_, _structs_, and even other modules as well.

Modules allow us to _partition_ our code within the crate itself. This implies that a crate can have multiple modules within it. We can define modules to group related functionality, and these modules can be nested to create a hierarchy.

## What is crate vs. module?

todo

## What is module system?

In Rust, both crates and modules are used for _organizing code_, but they serve different purposes and operate at different levels of granularity within the code structure. A crate is the top-level organizational unit or entity in Rust. <mark>A crate can contain one or more modules</mark>.

A module is a way to organize code _within a crate_. It provides a means of grouping related items, such as _functions_, _structs_, _enums_, and other modules as well.

# References

1. [Rust Atomics and Locks by Mara Bos <i class="fa-solid fa-arrow-up-right-from-square"></i>](https://www.oreilly.com/library/view/rust-atomics-and/9781098119430/){:target="_blank"}
   
---

[^1]: Rust only includes the native OS (also known as kernel thread) in its `std` module. Rust doesn't have a green threading (also known as user-level or lightweight threading) implementation as part of its standard library (`std` module). Initially, Rust's standard library supported both native and green threading, but later the green thread was removed from the standard library due to the fact that it demanded a larger language runtime.

[^2]: A module in Rust is a collection of "related" items that includes functions, structs, and even other modules as well.

