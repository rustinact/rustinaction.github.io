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

# Async vs threads in Rust

The primary alternative to async in Rust is using OS threads, either directly through std::thread or indirectly through a thread pool.

**OS threads** are suitable for a small number of tasks, since threads come with CPU and memory overhead. Spawning and switching between threads is quite expensive as even idle threads consume system resources. A thread pool library can help mitigate some of these costs, but not all. Threads let you reuse existing synchronous code without significant code changes. It is also possible to modify the priority of a thread in certain operating systems, which is beneficial for drivers and other applications that are sensitive to latency.

**Async** provides significantly reduced CPU and memory overhead, especially for workloads with a large amount of IO-bound tasks. Asynchronous programming is not superior than threads, but different. When performance does not require asynchronous operations, threads are frequently a more straightforward alternative.

---

# Frequently Asked Questions (FAQs)

todo

---

[^1]: A **programming paradigm** is a fundamental style or approach to programming comprising a set of principles, concepts, and patterns that guide the structuring, design, and organization of a software application. Here are some popular programming paradigms: **Object-oriented programming**, **functional programming**, **declarative programming**, **event-driven programming**, **concurrent programming**, etc.

[^2]: A **thread pool** is a collection of pre-created threads (ready-to-use threads) that can be reused to execute tasks. A thread pool reduces the resource consumption associated with creating and destroying threads repeatedly for short-lived tasks. Instead of creating a new thread for each task, threads from the pool are assigned to tasks as needed. This reuse helps reduce the overhead associated with thread creation and destruction.

[^3]: A **semaphore** is a _synchronization primitive_ to control access to a shared resource or a **critical section** of code. Semaphores help prevent race conditions. There are two types of semaphores: **binary semaphores** and **counting semaphores**. A counting semaphore is initialized with a non-negative integer value, which represents the number of available resources. Based on this number, resources will be allocated to threads.