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

##### Big thanks to Cargo and crates.io teams for building this feature over the past three years.

<br/>
<Timeline/>

---
transition: slide-up
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
