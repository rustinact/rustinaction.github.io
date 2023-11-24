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

Every program is made up of one or more threads. <mark>A thread is the smallest executable unit of a process</mark>. Threads enable us to split a program into multiple tasks, each executed in a separate thread. A program begins with exactly one thread, referred to as the "main thread"[^1]. This main thread will execute our main function, and can be used to spawn more threads if necessary. In Rust, new threads[^2] are spawned using the `thread::spawn` function from the standard library (`std` module[^3]). This `thread::spawn` function spawns a new **native operating system** thread, aka **kernel-level thread** (**KLT**).

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

In the above code, the `thread::spawn` function takes a function as an argument. When we call `thread::spawn` function and pass a function name as an argument, we are essentially creating a _new thread_ to execute the specified function concurrently. The function name is used as a way to define the code that will execute in the newly spawned thread. We spawn two threads in the above code that will both execute function `f` as their main function.

> **Thread ID**: The Rust standard library assigns every thread a unique identifier and this identifier is accessible through `Thread::id()`.

If we run our example above several times, we may observe that the output differs between executions. Unexpectedly, some of the output appears to be missing. Why does a portion of the output appear to be missing? Let's look at what's happening in this case. Here, the main thread completed the execution of the main function prior to the newly spawned threads completing the execution of their functions. When the main thread of a Rust program terminates, the entire program shuts down, even if other threads are still running. So how do we overcome this? It is straightforward: we need to set the main thread to wait until the completion of the execution of all spawned threads.

## Setting the main thread to wait until...

We can use the `join` method from the `std` library to wait for a thread to complete its execution. The `join` method is available on the `JoinHandle`[^4] returned by the `std::thread::spawn` function when a new thread is created. In other words, the `JoinHandle` type has a `join` method, and calling this method blocks the current thread until the associated thread completes. Having said that, `join` blocks the main thread until the spawned thread completes.

The "current thread" refers to the thread that is actively running the code at a given moment. Typically, the main thread is the thread that starts the program's execution. Therefore, the current thread typically refers to the main thread. The "associated thread," on the other hand refers to the thread that was spawned using `thread::spawn`. When we spawn a new thread using `thread::spawn`, it runs concurrently with the main thread or any other threads that may be executing.

Let's use `JoinHandle` returned by the spawn function:

```rust
fn main() {
    let handle_1 = std::thread::spawn(f);
    let handle_2 = std::thread::spawn(f);

    println!("Hello from the main thread.");

    handle_1.join().unwrap();
    handle_2.join().unwrap();
}
```

The `.join()` method waits until the thread has finished executing and returns a `std::thread::Result`. If the thread did not successfully finish its function because it panicked[^5], this will contain the panic message. The `join` method returns a `Result` that contains the result of the thread's execution or an error is generated if the thread panics. The `unwrap` used here is a method provided by the `Result`. It is used to extract the value from a `Result` when we are confident that the result does not encounter an error.

In the above code, we are storing each thread handle in its respective variable (`handle_1` and `handle_2`). Running the above updated version of our program will no longer result in truncated output.

Rather than passing the name of a function to `std::thread::spawn`, it’s more common to pass a **closure** containing the code to be executed in that thread, as shown below:

```rust
// Spawn a new thread
let handle = thread::spawn(|| {
    // Code to be executed in the new thread
    for i in 1..=5 {
        println!("Thread: {}", i);
    }
});
```

---

# Frequently asked questions (FAQ)

## How does concurrency differ from parallelism?

**Concurrency** refers to the ability of multiple tasks to be executed independently, making progress in an _overlapping time period_ but not necessarily simultaneously. It's important to be noted that concurrency is about managing multiple tasks and making progress on each task, but _not necessarily at the same time_, meaning, _tasks are not running simultaneously or in parallel_.  One of the tasks can begin before the preceding or previous one is completed; however, both of these tasks won’t be running at the same time. A single processor is used for running the tasks in the concurrency model.

Under the hood, a CPU can work on only one task at a time. If it is assigned multiple tasks, it simply switches between these tasks. Since this switching is so fast and seamless, it gives us a sense that these tasks are running in parallel. However, they are not parallel.

The main objective of concurrency is to maximize the CPU by minimizing its idle time

**Parallelism** is the ability to execute independent tasks of a program _simultaneously_ (at the same moment in time). Parallelism takes advantage of multi-core processors. Tasks can run "simultaneously" across multiple cores.

## What is asynchronous programming?

Asynchronous programming is a programming paradigm (approach) that allows tasks to proceed independently without waiting for each other. The program initiates asynchronous tasks and proceeds with the execution of other tasks in anticipation of their (aync tasks) completion. It's important to understand that _asynchronous tasks can run concurrently or sequentially_, but the program does not wait for the completion of each task before proceeding. It's often used for I/O-bound tasks to avoid blocking.

## What is synchronization in threading?

Symphonization in the context of threading refers to the coordination and control of the execution of multiple threads, with the objective of ensuring conflict-free interactions with shared resources (e.g. files, global variables, etc.) or critical sections[^6] and maintaining data consistency. Synchronization is crucial to prevent issues such as _race conditions_, _data corruption_, and _unexpected behavior_ that can arise when multiple threads access shared resources concurrently.

Here are some common synchronization mechanisms used in threading:

- **Locks (Mutexes)**: Locks, or mutual exclusions (mutexes), are mechanisms that control access to shared resources by allowing only one thread to acquire the lock at a time, preventing other threads from entering that section until the lock is released.
- **Semaphores**: Semaphores are synchronization primitives[^7] that maintain a count and allow a specified number of threads to access a critical section simultaneously. Semaphores can be used to control access to shared resources. Every semaphore has a current count, which is greater than or equal to 0. Any thread can decrement the count to lock the semaphore. When the count is attempted to be decremented beyond zero, the calling thread is required to wait until another thread releases the semaphore.
- **Conditions**: Conditions provide a way for threads to coordinate and communicate. They are often used in conjunction with locks. Threads can wait on a condition until a specific condition is met, and when that condition is met, other threads can signal or broadcast to wake up the waiting threads.
- **Atomic operations**: Atomic operations are operations that are executed in a single, uninterruptible step of execution. Some programming languages and threading libraries provide atomic operations to ensure that certain operations are performed atomically without the need for locks or interference from other threads.

---

# References

1. [Rust Atomics and Locks by Mara Bos](https://www.oreilly.com/library/view/rust-atomics-and/9781098119430/){:target="_blank"}
2. [Meet Safe and Unsafe](https://doc.rust-lang.org/nomicon/meet-safe-and-unsafe.html){:target="_blank"}
   
---

[^1]: In Rust, like in many other programming languages, the "**main thread**" refers to the initial thread of execution that is created when a Rust program starts running. The main thread is the thread in which the `main` function of a Rust program is executed. The `main` function is the entry point of a Rust program. The additional spawned threads can run concurrently with the main thread, allowing programmers to write concurrent and parallel code. When the main thread of a Rust program terminates, the entire program shuts down, even if other threads are still running.

[^2]: Rust only includes the native OS thread (also known as kernel thread) in its `std` module. An executing Rust program consists of a collection of native OS threads, each with their own stack and local state. Note that Rust doesn't have a green thread (also known as user-level or lightweight thread) implementation as part of its standard library (`std` module). Initially, Rust's standard library supported both native and green threading, but later the green thread was removed from the standard library due to the fact that it demanded a larger language runtime. For [here](https://github.com/rust-lang/rfcs/blob/master/text/0230-remove-runtime.md){:target="_blank"} for more information.

[^3]: A **module** in Rust is a collection of "related" items that includes functions, structs, and even other modules as well.

[^4]: The `thread::spawn` function returns a `JoinHandle`, which is a handle (reference or identifier) that represents a spawned thread. When we spawn a new thread using the `std::thread::spawn` function, it returns a `JoinHandle` that allows us to interact with the spawned thread and, importantly, wait for the thread to finish its execution and to obtain its result as well.

[^5]: A **panic** is the term used to describe the occurrence of a runtime error that causes the program to terminate abruptly. When a panic occurs, the runtime system begins to _unwind_ the stack of the thread where the panic happened. Unwinding involves the cleanup of stacks and resources, such as freeing memory and running destructors for local variables, in reverse order of their creation.

[^6]: A **critical section** refers to _any segment of code_ that deals with accessing shared resources. As multiple threads can access these shared resources concurrently, it becomes crucial to synchronize their access to prevent race conditions and data inconsistencies.

[^7]: A **synchronization primitive** in threading refers to a mechanism provided by a programming language or operating system to enable coordination and synchronization between multiple threads of execution.
