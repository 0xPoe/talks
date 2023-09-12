---
theme: seriph
background: https://user-images.githubusercontent.com/47556145/251160363-27298f5c-43a4-4068-9154-5d07f9e37c11.svg
class: text-center
highlighter: shiki
lineNumbers: false
info: |
  From Git Index to Sparse Index.
drawings:
  persist: false
transition: slide-left
title: From Git Index to Sparse Index
mdc: true
---

# From Git Index to Sparse Index

A Deep Dive

RUSTIN LIU

<div class="pt-12">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    Begin <carbon:arrow-right class="inline"/>
  </span>
</div>

---
transition: slide-up
---

# Rustin Liu

<div class="leading-8 opacity-80">
PingCAP Database Kernel Engineer.<br/>
Cargo Active Contributor.<br/>
Rustup Previous Maintainer.<br/>
Crates.io Previous Maintainer.<br/>
</div>

<div my-10 grid="~ cols-[40px_1fr] gap-y4" items-center justify-center>
  <div i-ri-github-line op50 ma text-xl/>
  <div><a href="https://github.com/hi-rustin" target="_blank">hi-rustin</a></div>
  <div i-ri-twitter-line op50 ma text-xl/>
  <div><a href="https://twitter.com/hi_rustin" target="_blank">hi_rustin</a></div>
  <div i-ri-firefox-line op50 ma text-xl/>
  <div><a href="https://hi-rustin.rs" target="_blank">hi-rustin.rs</a></div>
  <div i-ri-youtube-line op50 ma text-xl/>
  <div><a href="https://www.youtube.com/@hi-rustin" target="_blank">hi-rustin</a></div>
</div>

<img src="https://avatars.githubusercontent.com/u/29879298?v=4" rounded-full w-30 abs-tr mt-22 mr-22/>

<div flex="~ gap2">
</div>

---
transition: slide-up
---
# Sparse Index

##### Big thanks to the Cargo and crates.io teams for building this feature over the past three years.

<br/>
<Timeline/>

---
transition: slide-left
layout: center
---

<div text-6xl fw100>
  Agenda
</div>

<br>

<div class="grid grid-cols-[3fr_2fr] gap-4">
  <div class="border-l border-gray-400 border-opacity-25 !all:leading-12 !all:list-none my-auto">

  - From Cargo to crates.io and back again
  - What is the problem?
  - How to solve it?
  - What is the result?
  - Q&A

  </div>
</div>

---
layout: center
---

# Architecture

<iframe style="border: 1px solid rgba(0, 0, 0, 0.1);" width="800" height="450" src="https://www.figma.com/embed?embed_host=share&url=https%3A%2F%2Fwww.figma.com%2Ffile%2FaWmwb7IwJcYaCYpaMX4Yyv%2FCargo-to-crates.io%3Ftype%3Dwhiteboard%26node-id%3D0%253A1%26t%3DpfzFBCkrzmTqMDOI-1" allowfullscreen></iframe>

---
transition: slide-up
layout: center
---

# From Cargo to crates.io

---
transition: slide-left
layout: two-cols-header
---

# From Cargo to crates.io

::left::

<div class="mx-2">

## cargo new --lib

```console
$ cargo new --lib hello-gosim
     Created library `hello-gosim` package
```

```rust
$ tree
.
├── Cargo.toml
└── src
    └── lib.rs
```
</div>

::right::

<div class="mx-2">

## cargo package

```console
$ cargo package --allow-dirty
...
    Packaged 3 files, 945.0B (738.0B compressed)
```

```rust{all|8,9}
$ tree -L 3
.
├── Cargo.toml
├── src
│   └── lib.rs
└── target
    └── package
        ├── hello-gosim-0.1.0
        └── hello-gosim-0.1.0.crate
```

```rust{all|2,3}
$ tar tzf ./target/package/hello-gosim-0.1.0.crate
hello-gosim-0.1.0/Cargo.toml
hello-gosim-0.1.0/Cargo.toml.orig
hello-gosim-0.1.0/src/lib.rs
```
</div>

---
transition: slide-up
---

# From Cargo to crates.io

## cargo publish

```console{all|4|6|7|10}
$ cargo publish --allow-dirty
    Updating crates.io index
    ...
   Packaging hello-gosim v0.1.0 (/Users/hi-rustin/hello-gosim)
    ...
   Uploading hello-gosim v0.1.0 (/Users/hi-rustin/hello-gosim)
    Uploaded hello-gosim v0.1.0 to registry `crates-io`
note: Waiting for `hello-gosim v0.1.0` to be available at registry `crates-io`.
You may press ctrl-c to skip waiting; the crate should be available shortly.
   Published hello-gosim v0.1.0 at registry `crates-io
```
<v-click>

<img width="1604" alt="image" src="https://github.com/rust-lang/crates.io-index/assets/29879298/faf1894b-aa69-49c4-96ce-7aaf8e2c4ce1">

</v-click>

---
transition: slide-up
layout: center
---

# From crates.io to Cargo

---
transition: slide-left
---

# Create a new project and add a dependency

## cargo new --bin

```console
$ cargo new --bin gosim
     Created binary (application) `gosim` package
```

## cargo add

```console
$ cargo add hello-gosim
    Updating crates.io index
      Adding hello-gosim v0.1.0 to dependencies.
    Updating crates.io index
```

---
transition: slide-up
---

# From crates.io to Cargo with Git Index

## Change to Git Index

```toml
# ~/.cargo/config.toml

registries.crates-io.protocol = "git"
```

## cargo build

```console{all|2|3,4|7}
$ cargo build
    Updating crates.io index
  Downloaded hello-gosim v0.1.0
  Downloaded 1 crate (785 B) in 2.02s
   Compiling hello-gosim v0.1.0
   Compiling gosim v0.1.0 (/Users/hi-rustin/gosim)
    Finished dev [unoptimized + debuginfo] target(s) in 1m 23s
```

---
transition: slide-up
layout: center
---

# What is the problem?

---
transition: slide-left
---

# From crates.io to Cargo with Sparse Index

## cargo build

```console{all|2,3}
$ cargo build
  Downloaded hello-gosim v0.1.0
  Downloaded 1 crate (785 B) in 1.82s
   Compiling hello-gosim v0.1.0
   Compiling gosim v0.1.0 (/Users/hi-rustin/gosim)
    Finished dev [unoptimized + debuginfo] target(s) in 2.80s
```

---
transition: slide-up
---


# From crates.io to Cargo with Sparse Index

## Crate file

```console
$ pwd
/Users/hi-rustin/.cargo/registry/src/index.crates.io-6f17d22bba15001f/hello-gosim-0.1.0
$ tree
.
├── Cargo.toml
├── Cargo.toml.orig
└── src
    └── lib.rs
```


```console
$ pwd
/Users/hi-rustin/.cargo/registry/cache/index.crates.io-6f17d22bba15001f
$ find . -type f -name "hello*"
./hello-gosim-0.1.0.crate
```

## Index file

```console
$ curl https://index.crates.io/he/ll/hello-gosim
{"name":"hello-gosim","vers":"0.1.0","deps":[],"cksum":"5add821f9e323a4eb7bc869eda30883a76408aef5bc1a68a3ffc1ebb1c93cf9c","features":{},"yanked":false}
```
