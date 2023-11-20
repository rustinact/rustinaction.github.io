---
layout: post
title:  "Basics of Rust Concurrency"
kicker: "Concurrency"
summary: "Rust's ownership and borrowing features prevent us from experiencing memory-related problems. Rust is a great choice when performance matters and it solves pain points that bother many other languages."
image: assets/images/posts-cover-images/rust-concurrency.jpg
image-credits: "AI-generated image by <a href='https://www.freepik.com/' target='_blank'>freepik</a>."
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

Every program is made up of one or more threads. <mark>A thread is the smallest executable unit of a process</mark>. Threads enable us to split a program into multiple tasks, each executed in a separate thread. A program begins with exactly one thread, referred to as the "main thread". This main thread will execute our main function, and can be used to spawn more threads if necessary. In Rust, new threads[^1] are spawned using the `thread::spawn` function from the standard library (`std` module[^2]). This `thread::spawn` function spawns a new **native operating system** thread, aka **kernel-level thread** (**KLT**).

Consider the following example:

```rust
use std::thread;

fn main() {
    thread::spawn(f);
    thread::spawn(f);

    println!("Hello from the main thread.");
}

fn f() {
    println!("Hello from another thread!");

    let id = thread::current().id();
    println!("This is my thread id: {id:?}");
}
```

In the above code, the `thread::spawn` function takes a function as an argument.



---

# Frequently asked questions (FAQ)

## How does concurrency differ from parallelism?

**Concurrency** refers to the ability of multiple tasks to be executed independently, making progress in an _overlapping time period_ but not necessarily simultaneously. It's important to be noted that concurrency is about managing multiple tasks and making progress on each task, but _not necessarily at the same time_. One of the tasks can begin before the preceding or previous one is completed; however, both of these tasks won’t be running at the same time. A single processor is used for running the tasks in the concurrency model.

Under the hood, a CPU can work on only one task at a time. If it is assigned multiple tasks, it simply switches between these tasks. Since this switching is so fast and seamless, it gives us a sense that these tasks are running in parallel. However, they are not parallel.

The main objective of concurrency is to maximize the CPU by minimizing its idle time

**Parallelism** is the ability to execute independent tasks of a program _simultaneously_ (at the same moment in time). Parallelism takes advantage of multi-core processors. Tasks can run "simultaneously" across multiple cores.

## What is synchronization in threading?

Symphonization in the context of threading refers to the coordination and control of the execution of multiple threads, with the objective of ensuring conflict-free interactions with shared resources (e.g. files, global variables, etc.) or critical sections[^3] and maintaining data consistency. Synchronization is crucial to prevent issues such as _race conditions_, _data corruption_, and _unexpected behavior_ that can arise when multiple threads access shared resources concurrently.

Here are some common synchronization mechanisms used in threading:

- **Locks (Mutexes)**: Locks, or mutual exclusions (mutexes), are mechanisms that control access to shared resources by allowing only one thread to acquire the lock at a time, preventing other threads from entering that section until the lock is released.
- **Semaphores**: Semaphores are synchronization primitives[^4] that maintain a count and allow a specified number of threads to access a critical section simultaneously. Semaphores can be used to control access to shared resources. Every semaphore has a current count, which is greater than or equal to 0. Any thread can decrement the count to lock the semaphore. When the count is attempted to be decremented beyond zero, the calling thread is required to wait until another thread releases the semaphore.
- **Conditions**: Conditions provide a way for threads to coordinate and communicate. They are often used in conjunction with locks. Threads can wait on a condition until a specific condition is met, and when that condition is met, other threads can signal or broadcast to wake up the waiting threads.
- **Atomic operations**: Atomic operations are operations that are executed in a single, uninterruptible step of execution. Some programming languages and threading libraries provide atomic operations to ensure that certain operations are performed atomically without the need for locks or interference from other threads.

---

# References

1. [Rust Atomics and Locks by Mara Bos <i class="fa-solid fa-arrow-up-right-from-square"></i>](https://www.oreilly.com/library/view/rust-atomics-and/9781098119430/){:target="_blank"}
2. [Meet Safe and Unsafe <i class="fa-solid fa-arrow-up-right-from-square"></i>](https://doc.rust-lang.org/nomicon/meet-safe-and-unsafe.html){:target="_blank"}
   
---

[^1]: Rust only includes the native OS (also known as kernel thread) in its `std` module. Rust doesn't have a green threading (also known as user-level or lightweight threading) implementation as part of its standard library (`std` module). Initially, Rust's standard library supported both native and green threading, but later the green thread was removed from the standard library due to the fact that it demanded a larger language runtime. For [here <i class="fa-solid fa-arrow-up-right-from-square"></i>](https://github.com/rust-lang/rfcs/blob/master/text/0230-remove-runtime.md){:target="_blank"} for more information.

[^2]: A **module** in Rust is a collection of "related" items that includes functions, structs, and even other modules as well.

[^3]: A **critical section** refers to _any segment of code_ that deals with accessing shared resources. As multiple threads can access these shared resources concurrently, it becomes crucial to synchronize their access to prevent race conditions and data inconsistencies.

[^4]: A **synchronization primitive** in threading refers to a mechanism provided by a programming language or operating system to enable coordination and synchronization between multiple threads of execution.
