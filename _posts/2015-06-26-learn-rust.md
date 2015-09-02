---
layout: post
title: Rust 笔记
---

# Book: Rust Essentials

## Chapter 2

#### Global constants

The lifetime of an object in Rust is very important because it says how long the object will live in the program's memory. The Rust compiler adds the code to remove an object when its lifetime is over, freeing the memory that it occupied. The 'static lifetime is the longest possible lifetime; such an object stays alive throughout the entire application, and so it is available to all of its code.
