---
layout: post
title:  "Shipping Linux binaries that don't break with Rust"
date:   2020-06-18 17:11:00 +0700
categories: dev
---
Operating systems differences can cause your Rust binaries to break when run in a different environment than they were compiled in. Here are the most common things to watch out for.

## Avoid Glibc incompatibilities
Your binary will fail to run if it depends on Glibc and was compiled with a newer Glibc version than your target system (or if your target system does not use Glibc at all). Tactics to avoid Glibc incompatibilities include compiling with MUSL or a really old Glibc version.

### Statically compile MUSL into your binaries
MUSL is a lightweight replacement for Glibc used in Alpine Linux. MUSL can be statically compiled into your Rust program to create a self-contained executable that will run without dependencies on Glibc.

Compiling your program with MUSL may impact your program’s performance, binary size, and limit your use of certain libraries, such as the jemalloc memory allocator. 

Simple programs can be [cross-compiled with MUSL](https://doc.rust-lang.org/edition-guide/rust-2018/platform-and-target-support/musl-support-for-fully-static-binaries.html) by adding the MUSL target in Rustup and specifying the target for cargo build. However, popular libraries, like crypto crate [ring](https://crates.io/crates/ring), still do some C compilation and require more MUSL development libraries to be available in your environment. 

A more robust solution to compile your Rust for MUSL is to use a builder that runs in a Docker container with MUSL support, such as:
[https://github.com/emk/rust-musl-builder](https://github.com/emk/rust-musl-builder)

### Compile with a really old version of Glibc
Glibc is backwards compatible so your program will run if it is compiled with an older version of Glibc than installed on your target system. To use this tactic, just run your production builds in a Docker image with the oldest version of Linux you can find. 

This tactic allows broad Linux compatibility and lets you use allocators such as jemalloc, but makes your binary depend on Glibc and prevents running on Alpine Linux (at least without special compatibility layers).

## Avoid SSL-library incompatibilities
Another thing that can break your application is incompatible TLS library installations or non-standard certificate locations.

### Use Rustls
[Rustls](https://crates.io/crates/rustls) replaces operatings system Transport Layers Security implementations, such as OpenSSL, with a memory-safe implementation in Rust. By compiling your program with Rustls your program no-longer depends on OpenSSL or other libraries being installed on the target system.

Popular communication libraries in Rust often allow you to specify Rustls using feature flags or by dependending on specific crates. For instance the reqwest http client uses Rustls by specifying the flag rustls-tls in the dependency

```reqwest = { version = "0.10", features = ["json", "rustls-tls"] }```

Read the documentation of your communication libraries for how to use Rustls.

**Certificates**  
Rustls includes certificates for many popular certificate authorities, but also lets you specify custom certificate stores. If you need to support custom certificates, your binary can search common locations of certificates in your target Linux distributions or allow users to configure the locations.

## Handling other library dependencies
Every library your binary depends on being available in the operating system, such as compression or regular expressions, risks breaking your binary if the user has not installed it. 

### Use a Rust port of the library
Many popular libraries have a corresponding version written in Rust, such as deflate compressor crate flate2 or RE2-like regex crate regex. Search [crates.io](https://crates.io/) for the name of your library to see if there is a Rust replacement.

Libraries in Rust also often improve security by being implemented in a memory-safe language, though it is a tradeoff with maturity of the codebases.

### Embed C libraries in your Rust binary
Rust makes it easy to compile C libraries into your Rust code. Consider embedding libraries in your binary by [adding a build.rs file](https://doc.rust-lang.org/cargo/reference/build-scripts.html) that compiles the library into your binary during the build step. The build.rs file can use the [CC crate](https://crates.io/crates/cc ) to invoke the C compiler. 

You also need to specify a Rust interface to call the C library. This can be done manually for simple APIs (see examples in the CC crate), and with tools like [bindgen](https://github.com/rust-lang/rust-bindgen) to generate more advanced bindings.

