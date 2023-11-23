---
layout: post
title:  "Rust Closure"
kicker: "Closure"
summary: "A closure is a programming construct that allows a function to capture and reference variables and parameters from its surrounding lexical scope. Closures offer many benefits and are a key part of the language's expressive and flexible features."
image: assets/images/posts-cover-images/rust_closure.jpg
image-credits: "AI-generated image by <a href='https://www.freepik.com/' target='_blank'>freepik</a>."
imageshadow: true
toc: true
author: senthil
date: 2023-11-21 00:00:00 +0530
# last_modified_at: 2023-11-18
tags: [ "closure" ]
categories: closure
featured: false
hidden: false
---

# Overview

In programming, a closure is a type of function that has the ability to access and manipulate (modify) variables and parameters that are defined outside its own scope. Closures are also known as **lexical closures** or **function closures**. In other words, closure allows a function to capture and reference variables from its surrounding lexical scope[^1], even after that scope has finished executing. Closures are also referred to as **anonymous functions** or **lambda functions** in some programming languages, including Rust. This closure, or anonymous function, is defined without a name.

---

# Rust Closure

A closure in Rust usually consists of an argument list, given between vertical bars, followed by an expression:

```rust
// Define a closure with a single argument. Rust infers the arguument and return types.
let is_even = |x| x % 2 == 0;

// Define a closure with two integer parameters. Types are explicitly annoted here.
let add_numbers = |x: i32, y: i32| -> i32 {
    x + y
};
```

Rust infers the argument types and return type (in `is_even` variable). We can also write them out explicitly as we did in `add_numbers`. Here, `|x: i32, y: i32|` declares a closure with two parameters `x` and `y`, both of type `i32` and `-> i32` specifies that the closure returns an `i32` type. Note that Rust's type inference is quite powerful, and in many cases, we don't need to _explicitly_ annotate the types as we did for `is_even` variable.

In order to maintain syntactic coherence[^2], the body of the closure must be a block if a return type is specified:

```rust
let is_even = |x: u64| -> bool x % 2 == 0; // error

let is_even = |x: u64| -> bool { x % 2 == 0 }; // ok
```

Calling a closure uses the same syntax as calling a function:

```rust
assert_eq!(is_even(14), true);
```

## Rust's closures are anonymous functions

Having said that befofe, Rust's closures are **anonymous functions**[^3] we can save in a variable or pass as arguments to other functions.

## Closures vs. functions

Both closures and functions are used to define blocks of code that can be called or executed. However, there are some key differences between them:

- **Environment capture**
  - **Function**: Functions do not capture variables from their surrounding environment or lexical scope. They rely on parameters passed to them explicitly.
  - **Closure**: Closures, on the other hand, can capture variables from their surrounding environment. This allows closures to "close over"[^4] or capture variables and use them even after the surrounding scope has finished executing. The ability to capture variables from the environment is a key feature that distinguishes closures from regular functions.

    ```rust
    // Function
    fn add(x: i32, y: i32) -> i32 {
        x + y
    }

    fn main() {
        let result = add(3, 5);

        // Closure capturing a variable, x from the environment
        let x = 10;
        let closure = |y| x + y;

        let closure_result = closure(5);
    }
    ```
- **Syntax**
  - **Function**: Functions are declared using the `fn` keyword, followed by the function name, parameters, return type, and the function body.
  - **Closure**: Closures, on the other hand, are anonymous function.

## Closures closely tied to its ownership and borrowing

Rust's closures are closely tied to its [**ownership and borrowing systems**](/ownership-borrowing/2023/ownership-and-borrowing){:target="_blank"}. Ownership and borrowing play a crucial role in enforcing memory safety and preventing data races. The way a closure captures variables from its surrounding environment determines how it interacts with ownership and borrowing:

### By Value (Move) aka move closure

If a closure captures a variable by value by means of `move` keyword, it takes ownership of that variable. Once a value is moved into the closure, the original variable is no longer accessible in the surrounding scope:

```rust
let s = String::from("hello");

let closure = move || {
    println!("{}", s)
};

// println!("{}", s1); Can't access s1 as it has moved into the clouser
closure();
```

### By Reference (Borrowing)

If a closure captures a variable by reference, it borrows the variable. This allows the closure to read or modify the variable without taking ownership. Borrowing is indicated by the absence of the `move` keyword.

```rust
let s = String::from("hello");

let closure = || {
    println!("{}", s)
};

println!("{}", s);  // 's' is accessible since it's not moved to clouse. Instead, closuer borrowed.
closure();
println!("{}", s);  // 's' is still accessible after closure call

```

## Rust's motivation for closure

- **Flexibility and expressiveness**
  - Closures provide a concise and expressive way to define and use functions inline.
  - Closures can be particularly useful when passing functions as arguments to other functions (aka higher-order functions).
  - Closures can be a good fit for defining short-lived, anonymous functions.
- **Encapsulation of state from its surrounding lexical scope**
  - Closures allow a function to capture and encapsulate variables from its surrounding lexical scope.
  - Closures allowing for encapsulation of state within the closure.
  - Closures enable the creation of functions that carry their environment with them, <mark>preserving state between invocations</mark>.

    ```rust
    fn main() {
        // Counter closure that captures and preserves state
        let mut counter = || {
            let mut count = 0;
            move || {
                count += 1;
                println!("Count: {}", count);
            }
        };

        // Creating two instances of the closure
        let mut counter1 = counter();
        let mut counter2 = counter();

        // Invoking the closures
        counter1(); // Outputs: Count: 1
        counter1(); // Outputs: Count: 2

        counter2(); // Outputs: Count: 1
        counter2(); // Outputs: Count: 2
    }
    ```

---

[^1]: **Lexical scope**, also known as _static scope_, is a property of programming languages where the scope of a variable is determined by its location or position in the source code. In other words, the visibility and accessibility of a variable are defined by the placement of the variable's declaration within the structure of the program. In a language with lexical scope, when we define a variable inside a block or a function, that variable is accessible only within that block or function and any nested blocks or functions.

[^2]: **Syntactic coherence** is a term to emphasize the importance of code being written in a way that is not only syntactically correct but also adheres to a consistent and understandable syntax throughout.

[^3]: An **anonymous function**, also known as a **lambda function**, is a function defined without a name. Anonymous functions are a concise way to express a function without the need to declare it using a formal function name. They are frequently used as arguments to _higher-order functions_.

[^4]: In the context of closures, **closing over** or **closing over variables** refers to the ability of a closure to capture and retain the values of variables from the surrounding lexical scope, even after that scope has finished executing.
