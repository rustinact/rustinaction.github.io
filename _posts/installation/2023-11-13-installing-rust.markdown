---
layout: post
title:  "Installing Rust"
kicker: "Rust Installation"
summary: 'Rust is a great choice when performance matters and it solves pain points that bother many other languages. For the eighth year in a row, Rust has topped the chart as “the most desired programming language" in Stack Overflow’s annual developer survey, implying that many people who have had the chance to use it have fallen in love with it. The Rust community continues to grow.'
image: assets/images/posts-cover-images/rust-installation.png
author: senthil
date: 2023-11-13 00:00:00 +0530
tags: [ "installation" ]
categories: rust
featured: false
hidden: false
toc: true
---

# A brief introduction to Rust

Let's begin with a brief introduction to Rust. Rust is a great choice when performance matters and it solves pain points that bother many other languages. For the eighth year in a row, Rust has topped the chart as “the most desired programming language" in Stack Overflow’s annual developer survey, implying that many people who have had the chance to use it have fallen in love with it. The Rust community continues to grow.

---

# Rust Playground

The **Rust Playground** allows us to experiment with Rust without the need to install it locally. If you don't want to install any binaries in your current working environment, then Rust Playground would be the most practical way to begin writing Rust programs right away.

You can access Rust Playground [here <i class="fa-solid fa-arrow-up-right-from-square"></i>](https://play.rust-lang.org/){:target="_blank"}.

It has a number of features, including:

1. It has a minimalistic web IDE with syntax highlighting. 
2. It has the ability to compile in *debug* or *release* mode. 
3. The most popular and widely used crates and all of their dependencies are available for use.
4. It has the ability to quickly and easily share your code with others via permalink and Github Gist. 
5. **rustfmt** (Rust formatting tool) and **Clippy** (Rust lint) can be run against the source code.

To locally install Rust, proceed with the steps outlined below.

---

# Installing Rust

Rust runs on a wide variety of platforms and can be installed in a variety of methods. The most straightforward and recommended way to install Rust is through a tool called **Rustup**, which is a Rust installer and version management tool. In other words, Rust is installed and managed by the `rustup` tool. 

To develop in Rust, we need a set of tools that include, but are not limited to:

1. The **compiler**: `rustc`
2. The **build tool and package manager**: `cargo`
3. The **documentation generator**: `rustdoc`
4. A tool for **formatting Rust code** according to style guidelines: `rustfmt`
5. The **lint** to catch common mistakes and improve your Rust code: `clippy`

A set of these aforementioned tools is referred to as a **toolchain**. Rustup is the official tool used to install the Rust toolchain from the official release channels. Not only does Rustup install Rust toolchains and keep them updated, but it also allows us to seamlessly switch between the stable, beta, and nightly Rust compilers and keep them updated.

##  Installing Rust on macOS, Linux, or Unix-like OS

If you're running macOS, Linux, or any Unix-like OS, run the following in your terminal to download Rustup and install Rust:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

The above command will download and install Rustup itself, as well as the stable version of the Rust toolchain.

 - Rustup metadata and toolchains will be installed into the Rustup home directory, located at: `$HOME/.rustup`. 
 - The Cargo home directory is located at:  `$HOME/.cargo`. 
 - The Rust toolchain will be installed to Cargo's bin directory, located at: `$HOME/.cargo/bin`

### Verify installation

To verify that the Rust programming language is correctly installed on your system, you can follow these steps:

```bash
rustup --version
rustc --version
cargo --version
```

##  Installing Rust on Windows

If you are running Windows 64-bit, download and run [rustup‑init.exe](https://win.rustup.rs/x86_64) then follow the onscreen instructions.

## Update Rust

Rust undergoes frequent updates. It is likely that the version of Rust you are using is out of date if you installed Rustup some time ago. Get the latest version of Rust by running `rustup update` command.

### Updating Rustup alone (self update)

By executing the `rustup update` command, both Rustup and the Rust toolchain are examined for updates. Rustup and the toolchain are subsequently updated to the most recent versions. Execute `rustup self update` if you wish to install the most recent version of Rustup *without updating any installed toolchains*.

### Updating Rust toolchain alone (without updating Rustup)

Note that the `rustup` command will automatically update itself at the end of any toolchain installation as well. You can prevent this automatic behavior by passing the `--no-self-update`` argument when running `rustup update` or `rustup toolchain install`.


### Keeping Rust up to date

The Rust release process adopts "release train" model in which there are three different release channels through which the official Rust binaries are published: **nightly**, **beta**, and **stable**. Nightly gets promoted to beta and beta gets promoted to stable: **nightly → beta → stable**. The `rustup` is configured to use the stable channel by default and is released every six weeks. When a new version of Rust is released, we can run `rustup update` command to update to it.

## Uninstall Rust

If at any point you would like to uninstall Rust, you can execute `rustup self uninstall` command. You will not give thought to uninstalling Rust once you begin using it :-)

## Environment variables

- **RUSTUP_HOME** (default: `~/.rustup` or `%USERPROFILE%/.rustup` or `$HOME/.rustup`) - Sets the root `rustup` folder.
- **RUSTUP_TOOLCHAIN** (default: none) - If set, will override the toolchain used for all rust tool invocations.
- **RUSTUP_DIST_SERVER** (default: https://static.rust-lang.org) - Sets the root URL for downloading static resources related to Rust.
- **RUSTUP_UPDATE_ROOT** (default https://static.rust-lang.org/rustup) - Sets the root URL for downloading self-update.
- **RUSTUP_IO_THREADS** unstable (defaults to reported CPU count) - Sets the number of threads to perform close IO in. Set to 1 to force single-threaded IO for troubleshooting, or an arbitrary number to override automatic detection.
- **RUSTUP_TRACE_DIR** unstable (default: no tracing) - Enables tracing and determines the directory that traces will be written too.
- **RUSTUP_UNPACK_RAM** unstable (default 400M, min 100M) - Caps the amount of RAM rustup will use for IO tasks while unpacking.
- **RUSTUP_NO_BACKTRACE** - Disables backtraces on non-panic errors even when **RUST_BACKTRACE** is set.
- **RUST_BACKTRACE**
- **RUSTUP_PERMIT_COPY_RENAME**

---

# Cargo - Rust's build tool and package manager

Cargo is Rust's build tool and package manager. It comes installed with Rust if we use the official installation steps.

Cargo does lots of things, which include:

- Build our project with `cargo build` — takes care of downloading the libraries our code depends on.
- Run our project with `cargo run`
- Test our project with `cargo test`
- Build documentation of our project with `cargo doc`
- Publish a library to crates.io with `cargo publish`