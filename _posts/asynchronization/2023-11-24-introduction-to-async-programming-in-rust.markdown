---
layout: post
title:  "Introduction to Asynchronous programming in Rust"
kicker: "Asynchronous Programming"
summary: "This post answers frequently asked questions about the Rust programming language. It neither serves as a comprehensive manual for the language nor as a tool for teaching the language."
image: assets/images/posts-cover-images/rust-asyn-programming-intro.jpg
image-credits: "AI-generated image by <a href='https://www.freepik.com/' target='_blank'>freepik</a>."
imageshadow: true
toc: true
author: senthil
date: 2023-11-24 00:00:00 +0530
# last_modified_at: 2023-11-18
tags: [ "asynchronous", "asynchronous-programming" ]
categories: Asynchronous-Programming
featured: false
hidden: false
draft: true
---

# What is asynchronous programming? 

Asynchronous programming is a programming paradigm[^1] that allows **multiple tasks to run independently without waiting for each other** (in a non-blocking manner). This promotes the efficient management of I/O operations that often involve waiting for external events, such as interacting with databases, etc.

**Synchronous programming**, aka **sequential programming**, is more widely adopted, mature, and "standardized" than concurrent programming. In traditional **synchronous programming**, aka **sequential programming**, when a program encounters an I/O operation (such as reading from a file or interacting with a database), it typically blocks and waits for the operation to complete before moving on to the next instruction. During this blocking period, the entire program may remain idle, potentially leading to insufficient utilization of resources.

Asynchronous programming, on the other hand, allows the program to continue executing other tasks _while running I/O operations in the background_. Upon completion of the I/O operation, the program is notified, and the appropriate callback or continuation is executed. Asynchronous programming enables our program to start a potentially long-running task and still be responsive to other events while that task runs, rather than having to wait until that task has finished.

The asynchronous programming model is typically more complicated for the developer but results in a faster runtime for I/O-heavy workloads.

<mark>Asynchronous programming pertains to situations in which the execution of control does not adhere to a predetermined order. Events outside the control of the program itself impact the sequence of what is executed. Those events are typically related to I/O, such as a device driver signaling that it is ready, etc.</mark>

## A glossary of terms relating to concurrency

### Various concurrency models

Let's understand some of the most popular concurrency models that can help us understand how asynchronous programming fits within the wider field of concurrent programming. The creation of a thread involves allocating resources such as a stack and a program counter, and the kernel manages the scheduling of threads on available CPU cores.

#### Operating System (OS) threads

- OS threads play a crucial role in implementing concurrency. 
- **Thread creation and management:** 
  - OS threads are managed by the operating system kernel. 
  - The creation of an OS thread involves allocating resources such as a stack and a program counter
- **Implementation**
  - Implementing concurrency with OS threads is a straightforward process, as it does not require any changes to the programming model.
- **Parallelism**
  - OS threads can enable parallelism by allowing multiple threads to execute in parallel on different CPU cores.
- **Synchronization**
  - Synchronization mechanisms, such as **locks**, **semaphores**[^3], and **condition variables**, are often used with OS threads to coordinate access to shared resources and avoid race conditions.
- **Resource overhead** 
  - Each thread has its own stack, program counter, and other resources, and managing a large number of threads can lead to increased memory usage and context-switching overhead.
  - However, **thread pools**[^2] can mitigate some of these costs, but not enough to support massive IO-bound workloads.

#### Event-driven programming

- Event-driven programming, in conjunction with callbacks, can be very performant but tends to result in complex and non-linear asynchronous control flow.
- Data flow and error propagation is often hard to follow.

#### Coroutines

- Coroutines, like threads, don't require changes to the programming model, which makes them easy to use. Like async, they can also support a large number of tasks. 
- However, they abstract away low-level details that are important for systems programming and custom runtime implementors.

#### Actor model

- The actor model divides all concurrent computation into units called actors, which communicate through message passing, much like in distributed systems. 
- The actor model can be efficiently implemented, but it leaves many practical issues unanswered, such as flow control and retry logic.

### Context switching

Context switching is the process of switching the CPU (or virtual processor) from one thread to another. It is an essential component of multithreading, enabling the operating system to concurrently execute multiple tasks. When a thread is running, the operating system keeps track of its state, including its *program counter*, *stack*, and *registers*. When the operating system needs to switch to another thread, it saves the state of the current thread and subsequently restores the state of the new or other thread.

**Context switching can incur significant costs** as it forces the operating system to save and restore the thread's state. However, it is an indispensable evil needed for multitasking.

The frequency of context switches can be influenced by various factors, such as the number of active threads, the nature of the work performed by those threads, and the scheduling algorithm that the operating system is using. In general, the more threads that are running, the more often context switches will occur.

# Asynchronous programming in Rust

Developing a fast and reactive application requires the use of asynchronous programming. Rust's implementation of async differs from most languages in certain ways, which include:

- **Futures are inactive in Rust**
  - A future does not start executing its associated code until an executor or a runtime actively polls it. 
  - Dropping a future typically means that the associated computation is canceled or cleaned up, and it implies that the future will no longer make progress or complete its task.
- **Async is zero-cost in Rust**
  - It emphasizes that using asynchronous programming constructs should not impose any runtime overhead.
  - Specifically, we can use async without heap allocations and dynamic dispatch, which is great for performance and this also lets us use async in constrained environments, such as embedded systems.
- **No built-in async runtime**
  - No built-in async runtime is provided by Rust.
  - Instead, runtimes ([Tokio](https://tokio.rs/){:target="_blank"}, etc.) are provided by community maintained crates.
- **Both single- and multithreaded runtimes** are available in Rust, which have different strengths and weaknesses.

## Async vs threads in Rust

The primary alternative to async in Rust is using OS threads, either directly through `std::thread` or indirectly through a thread pool.

**OS threads** are suitable for a small number of tasks, since threads come with CPU and memory overhead. Spawning and switching between threads is quite expensive as even idle threads consume system resources. A thread pool library can help mitigate some of these costs, but not all. Threads let you reuse existing synchronous code without significant code changes. It is also possible to modify the priority of a thread in certain operating systems, which is beneficial for drivers and other applications that are sensitive to latency.

**Async** provides significantly reduced CPU and memory overhead, especially for workloads with a large amount of IO-bound tasks. Asynchronous programming is not superior than threads, but different. When performance does not require asynchronous operations, threads are frequently a more straightforward alternative.

## Language and external library support for async programming in Rust

Although Rust itself provides support for asynchronous programming, the majority of async applications rely on the functionality offered by community crates. Therefore, a combination of language features and library support is required:

- The `Future` trait (`std::future::Future`) is a fundamental part of the asynchronous programming model offered by the standard library in Rust.

    ```rust
    pub trait Future {
        type Output;

        // Required method
        fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
    }
    ```
The `poll` method allows checking whether the future has completed, is still pending, or encountered an error.

- The `async/await` syntax is supported directly by the Rust compiler.
- Many utility types, macros and functions are provided by the `futures` crate and they can be used in any async Rust application.
- Execution of async code, IO and task spawning are provided by **async runtimes**, such as **Tokio** and **async-std**. Most async applications, and some async crates, depend on a specific runtime.

## Asynchronous code and synchronous code

It is not always possible to combine synchronous and asynchronous code freely. For example, we can't directly call an async function from a sync function. Even asynchronous code cannot always be freely combined. Certain packages require a particular async runtime in order to operate. If so, it is usually specified in the crate's dependency list. It is crucial to investigate potential crates and async runtimes in advance, as these compatibility issues may restrict your options. Once we have settled in with a runtime, we won't have to worry much about compatibility.

## async/.await keyword

`async/.await` is Rust's built-in tool or keywords for writing asynchronous functions that look like synchronous code.

### `async` Keyword

- The `async` keyword is used to define asynchronous functions.
- An asynchronous function returns a `Future`, which represents a computation that may not have been completed yet but will be available later in time.
- Asynchronous functions are defined using the `async` fn syntax.
  
    ```rust
    async fn async_function() { /* Asynchronous code here */ }
    ```

  The value returned by `async fn` is a `Future`. 

- Under the hood, `async` transforms a block of code into a **state machine**[^4] that implements a trait called `Future`.
    
  Dependencies to be added to the `Cargo.toml` file:

    ```toml
    [dependencies]
    futures = "0.3"
    ```

### `await` Keyword

- The `await` keyword is used within an asynchronous function to suspend its execution until the result of a `Future` is ready.
- The `await` keyword can only be used inside functions marked as `async`.

    ```rust
    async fn example() -> i32 {
        let result = example_async_function().await;
        // Further code that executes after the awaited Future completes
        result + 10
    }
    ```

For any action to take place, the `Future` needs to be run on an executor.

> **futures::executor::block_on vs .await()**: TODO: https://rust-lang.github.io/async-book/01_getting_started/04_async_await_primer.html#asyncawait-primer

## future crate

The `futures` crate in Rust is a foundational library for working with asynchronous programming. The `futures` crate defines the `Future` trait, which is a fundamental building block for asynchronous programming. The `futures` crate is often used in conjunction with the `async/.await` syntax. Asynchronous functions using the `async/await` syntax return types that implement the `Future` trait, and the `futures` crate provides utilities to work with these async functions. It provides a variety of *combinators* and *utility functions*, allowing developers to *chain*, *map*, and *compose* asynchronous operations. Also, this crate introduces abstractions for working with **streams** (sequences of values over time) and **sinks** (consumers of values). The `futures` crate is designed to be executor-agnostic. It doesn't prescribe a specific runtime or executor, allowing developers to use it with various asynchronous runtimes, such as **Tokio**, **async-std**, or others.

## Future trait

A `Future` is an *asynchronous computation* that produce a *value* or an *error* at *some point in the future*. The `Future` trait might look something like this:

```rust
pub trait Future {
    type Output;

    // Required method
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

The `poll` method is where the actual work of an asynchronous operation is done, and it determines the current state and progress of the future. In other words, the `poll` method allows checking whether the future has completed, is still pending, or encountered an error. The `poll` method is called repeatedly until the final value is available.

The `Self::Output` is the associated type of a `poll` method indicating the type of value that the future will produce upon completion. If the future completes, it returns `Poll::Ready(result)`. If the future is not able to complete yet, it returns `Poll::Pending` and arranges for the `wake()` function to be called when the `Future` is ready to make more progress. With `wake()`, the executor knows exactly which futures are ready to be polled.

Now the question arises: **who constantly calls the poll method repeatedly until the final value is available?** One possible approach is to manually invoke the `poll` method from our synchronous application multiple times to obtain the final value. I understand what you are thinking. Not to worry. We won't be doing that kind of manual polling. We will hand over this polling task to **runtime**. Note that asynchronous Rust code does not run on its own, so we must choose a runtime to execute it.

Having said that, before we can make use of the async syntax, a runtime must be present. Async runtimes are libraries used for executing async applications. Note that there is no asynchronous runtime in the Rust's standard library. The following is a list of widely known community-provided async runtime crates:

- [Tokio](https://tokio.rs/){:target="_blank"}: A popular async ecosystem with HTTP, gRPC, and tracing frameworks.
- [async-std](https://docs.rs/async-std/){:target="_blank"}: A crate that provides asynchronous counterparts to standard library components.

As of the writing of this post, the Tokio library is the most widely used runtime. Let's use Tokio runtime.

## Tokio's role in asynchronous programming

Tokio is an *asynchronous runtime* for the Rust programming language. The need for an asynchronous runtime in Rust arises from the desire to: 

- Scheduling and managing the execution of asynchronous tasks efficiently.
- Handling I/O operations concurrently
- Providing a multi-threaded runtime for the execution of asynchronous code

When writing asynchronous code, we cannot use the ordinary blocking APIs provided by the Rust standard library, and must instead use asynchronous versions of them. These alternate versions are provided by Tokio, mirroring the API of the Rust standard library where it makes sense.

---

# Frequently Asked Questions (FAQs)

## What is the difference between concurrency and parallelism, and how do the two differ from asynchronous programming?

**Concurrency** refers to the ability of multiple tasks to be executed independently, making progress in an _overlapping time period_ but not necessarily simultaneously. It's important to be noted that concurrency is about managing multiple tasks and making progress on each task, but _not necessarily at the same time_, meaning, _tasks are not running simultaneously or in parallel_.  One of the tasks can begin before the preceding or previous one is completed; however, both of these tasks won’t be running at the same time. A single processor is used for running the tasks in the concurrency model.

Under the hood, a CPU can work on only one task at a time. If it is assigned multiple tasks, it simply switches between these tasks. Since this switching is so fast and seamless, it gives us a sense that these tasks are running in parallel. However, they are not parallel. Concurrency pretends that we have multiple cores, or CPUs.

The main objective of concurrency is to maximize the CPU by minimizing its idle time. Put simply, **concurrency is about doing multiple things, not at once**.

**Parallelism** is the ability to execute independent tasks of a program _simultaneously_ (at the same moment in time). Parallelism takes advantage of multi-core processors. Tasks can run "simultaneously" across multiple cores. Note that parallism is only possible with multiple cores or CPUs. Put simply, **parallelism is about doing multiple things at once**.

**Asynchronous programming** is a language feature that enables concurrency and/or parallelism. As it turns out, asynchronous programming is entirely unrelated to concurrency and parallelism.

## What is cooperative multitasking?

Each task in cooperative multitasking decides when it can be handed over to another.

## What is preemptive multitasking?

In preemptive multitasking, as opposed to cooperative multitasking, the system decides when to switch to other tasks.

## How does a kernel-level thread differ from a user-level thread?

**Kernel-level threads**, aka native threads, are managed by the operating system kernel. The operating system is responsible for scheduling and context switching. It's heavier than user-level threads in terms of overhead. It's better suited for scenarios requiring parallelism and direct interaction with system services. 

**Disadvantages**

- Relatively limited number we can create!

**User-level threads**, also known as lightweight threads or green threads, are managed entirely by a user-level thread library or runtime. The operating system kernel is unaware of these threads. It's lightweight and have lower overhead compared to kernel-level threads. Scheduling and context switching are performed by the user-level thread library or runtime. It's efficient for scenarios where a large number of threads need to be managed. Since it's lighter weight, we can create many more green threads.

**Disadvantages**

- Stack growth can cause issues!

---

[^1]: A **programming paradigm** is a fundamental style or approach to programming comprising a set of principles, concepts, and patterns that guide the structuring, design, and organization of a software application. Here are some popular programming paradigms: **Object-oriented programming**, **functional programming**, **declarative programming**, **event-driven programming**, **concurrent programming**, etc.

[^2]: A **thread pool** is a collection of pre-created threads (ready-to-use threads) that can be reused to execute tasks. A thread pool reduces the resource consumption associated with creating and destroying threads repeatedly for short-lived tasks. Instead of creating a new thread for each task, threads from the pool are assigned to tasks as needed. This reuse helps reduce the overhead associated with thread creation and destruction.

[^3]: A **semaphore** is a _synchronization primitive_ to control access to a shared resource or a **critical section** of code. Semaphores help prevent race conditions. There are two types of semaphores: **binary semaphores** and **counting semaphores**. A counting semaphore is initialized with a non-negative integer value, which represents the number of available resources. Based on this number, resources will be allocated to threads.

[^4]: A **state machine** is a mechanism used to represent the various *states* and *transitions* of a future as it progresses through its lifecycle. The `Future` trait defines an asynchronous computation that produces a result at some point in the future. `Future`s in Rust are often implemented as state machines, which manage asynchronous computations by utilizing the state machine pattern.