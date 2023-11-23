---
layout: post
title:  "Introduction to Actix Actor Model"
kicker: "Actor Model"
summary: "This post answers frequently asked questions about the Rust programming language. It neither serves as a comprehensive manual for the language nor as a tool for teaching the language."
image: assets/images/posts-cover-images/rust-actor-intro.jpg
image-credits: "AI-generated image by <a href='https://www.freepik.com/' target='_blank'>freepik</a>."
imageshadow: true
toc: true
author: senthil
date: 2023-11-23 00:00:00 +0530
# last_modified_at: 2023-11-18
tags: [ "actors", "actor-model", "actix" ]
categories: Actors
featured: false
hidden: true
---

# What is an Actor Model?

In computer science, the Actor Model is a mathematical model or theory that **deals with concurrent computation** in which an actor represents the basic building block of concurrent computation. It was introduced by **Carl Hewitt**, **Peter Bishop**, and **Richard Steiger** in a 1973 paper titled "A Universal Modular Actor Formalism for Artificial Intelligence." The Actor Model takes a different approach to solving the problem of concurrency by avoiding the issues caused by _threads_ and _locks_.

The idea is very similar to what we have in object-oriented languages: An object receives a message (a method call) and does something depending on which message it receives (which method we are calling). Unlike the object-oriented model where the objects are executed sequentially, the actors execute concurrently.

An actor mode has certain key principles that include:

- **Actors**
  - Actors are the fundamental building blocks in the Actor Model.
  - An actor is the foundation on which we build the structure of our application. Simply put, **an actor is a fundamental or primitive unit of computation or execution**.
- **Message-driven**
  - The Actor Model is message-driven i.e. actors communicate with each other exclusively by means of messages.
  - An actor can send a message to another actor.
  - Upon receiving a message, an actor can perform computations, update its state, and send messages to other actors.
- **Concurrency**
  - Actors operate concurrently, meaning that multiple actors can execute their computations independently and in "parallel".
  - This enables the implementation of parallelism in an efficient manner.
- **Isolation and encapsulation**
  - Like OOP, each actor has its own private state, and access to this state is controlled by the actor itself.
  - This state isolation and encapsulation prevent direct access by other actors, thereby encouraging modular and autonomous design.
- **Asynchronous processing**
  - Message exchange is asynchronous within the Actor Model.
  - When an actor sends a message, it can continue its computation without waiting for a response.
  - This asynchronous communication promotes loose coupling between actors.
- **Location transparency**
  - The Actor Model can be implemented in distributed systems, where actors may be located on different machines.
  - Despite potential distribution, the programming model remains the same, providing a degree of location transparency.
- **Actor lifecycle**
  - Actors can be created and terminated dynamically.
  - When an actor is created, it starts with an initial state and behavior. 
  - Actors can spawn (create) new actors during their execution.

The Actor Model provides a high-level abstraction for reasoning about concurrent and distributed systems, making it well-suited for designing scalable and fault-tolerant applications.

## One ant is no ant. One actor is no actor!

**Carl Hewitt** once said, “**One actor is no actor**," which is comparable in meaning to the phrase "**one ant is no ant**." The comparison between "one actor is no actor" and "one ant is no ant" establishes a correlation between the behavior of ants in a colony and the principles of the actor model in the context of concurrent and distributed systems.

The phrase "one ant is no ant" is often used to highlight the collective and collaborative nature of ants in a colony. Ants are social insects that work together for the benefit of the colony. While an individual ant might not seem particularly significant or influential, the colony as a whole is capable of accomplishing complex duties such as building nests, foraging for food, and defending against threats.

Similarly, the phrase "one actor is no actor" can be interpreted to highlight the idea that the power and effectiveness of the Actor Model arise from the interactions and collaboration among multiple actors. Although an individual actor can execute a particular computation or task, the model's true ability emerges when multiple actors work together, exchanging messages and coordinating their actions.

In summary, "one ant is no ant" emphasizes the collective strength of ants in a colony, and "one actor is no actor" highlights the collaborative nature of the Actor Model in concurrent and distributed computing, where the power resides in the interactions among multiple actors.

## Mailbox

A mailbox serves as an essential component within the Actor Model, enabling actors to exchange messages. It functions similarly to a conventional mailbox—a box for the deposit or delivery of mail. The mailbox is associated with each actor and serves as a message queue where incoming messages are stored until the actor processes them. The use of mailboxes is crucial to achieving asynchronous communication and decoupling actors in the actor model. 

When one actor sends a message to another actor, the message is placed in the recipient actor's mailbox. Typically, the mailbox is organized in a **first-in, first-out** (**FIFO**) order. Messages are processed in the order they arrive in the mailbox, ensuring that messages are handled in a predictable sequence.

## Execution context

In the context of the Actor Model, an "execution context" typically refers to the environment or context in which an actor operates. Each actor in the Actor Model has its own execution context, which includes its **state**, **behavior**, and **mailbox** for receiving messages.

---

# Introduction to Actix

Actix is a Rust library providing a framework for developing concurrent applications. It's built around the Actor Model, which enables the development of applications as a collection of actors that operate autonomously (independently) but collaborate through message-based communication. Actor runs within a specific execution context `Context<A>`. The context object is available only during execution. Each actor has a separate execution context. The lifecycle of an actor is also governed by the execution context. Actors are not referenced directly, but by means of addresses.


Any rust type can be an actor, it only needs to implement the `Actor` trait. In order to handle a specific message, the actor is required to provide a `Handler<M>` implementation for that message. Actor can spawn[^1] other actors or add futures[^2] or streams[^3] to execution context

## Actor lifecycle

- **Started**
  - An actor always starts in the `Started` state.
  - During this state the actor's `started()` method is called.
  - The `Actor` trait provides a default implementation for `started()` method.
  - The actor context is available during this state and the actor can start additional actors or register async streams or perform any other necessary configuration.
- **Running**
  - After an Actor's `started()` method is called, the actor enters the `Running` state. The Actor can stay in running state indefinitely.
- **Stopping**
  - The Actor's execution state changes to the stopping state in the following situations:
    - `Context::stop` is called by the actor itself.
    - All addresses associated with the actor get dropped. i.e. no other actor references it.
    - <mark>No event objects are registered in the context</mark>.
  - An actor can restore from the stopping state to the `running` state by creating a new address or adding an event object, and by returning `Running::Continue`.
  - If an actor changed state to `stopping` because `Context::stop()` is called then the context immediately stops processing incoming messages and calls `Actor::stopping()`.
  - If the actor does not restore back to the `running` state, all unprocessed messages are dropped.
  - By default <mark>this</mark> method returns Running::Stop which confirms the stop operation.
- **Stopped**
  - If the actor does not modify the execution context during the stopping state, the actor state changes to `Stopped`.
  - This state is considered final and at this point the actor is dropped.

---

[^1]: In the context of actors, **spawn** typically refers to the creation of new actor instances during runtime.

[^2]: In Rust, **futures** represent a value or error that will be available at some later time.

[^3]: In Rust, **streams** represent a sequence of values or errors over time.

