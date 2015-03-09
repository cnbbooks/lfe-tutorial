# The LFE Tutorial

The LFE Tutorial is based on the Erlang tutorial [Getting Started with
Erlang](http://www.erlang.org/doc/getting_started/intro.html)


## Introduction

The members of the LFE community are
[working torward](https://github.com/lfe/docs/issues/40)
a more comprehensive LFE tutorial, but we wanted to
[get something out](https://github.com/lfe/docs/issues/42)
sooner rather than later, so we're starting with this. On the positive side,
it's material that has already been created and we have only had to make syntax
changes and other edits at our discretion. However, one downside of this as the first full LFE tutorial is that it's not 100% new-comer friendly:
it doesn't start from absolute basics (some knowledge is presumed) and the tutorial is not comprehensive (much is left out).

That being said, the this tutorial is an intentional "kick start", aimed at to getting you started with LFE. Everything here is true, but as mentioned, it only represents part of the truth. For example, we will typically only cover
simpler syntax and not explore all the esoteric forms. However, for the curious
reader we've annotated where one may get more detailed information on topics
covered lightly (or skipped entirely).


## Prerequisites

* Basic to intermediate skills in a programming language.
* Exposure to the concepts behind Erlang via other media such as the Erlang web
  site, Erlang conference presentations, and books such as the following:
  * [Concurrent Programming in ERLANG](http://www.erlang.org/download/erlang-book-part1.pdf)
  * [Learn You Some Erlang for great good!](http://learnyousomeerlang.com/)
  * [Programming Erlang](https://pragprog.com/book/jaerlang2/programming-erlang)
  * [Erlang and OTP in Action](http://www.manning.com/logan/)
  * [Erlang Programming](http://shop.oreilly.com/product/9780596518189.do)
* Exposure to a Lisp, whether by reading books or from a course in school (it
  will be most helpful if you've had experience with Common Lisp, and LFE has
  more in common with Lisp-2s than with Lisp-1s like Schemes and Clojure).


## What Is Covered

This tutorial covers the following topics:

* Sequential programming (installing LFE, the REPL, functions, modules, data
  types, the standard library, writing to the terminal, pattern matching,
  guards, conditionals, built-in functions, and higher-order functions)
* Concurrent programming (processes, message passing, registered processes,
  and distributed programming)
* Robustness (timeouts and error handling)
* Records and macros (header files, records, and macros)


## What Has Been Left Out

The following topics has been omitted from this tutorial:

* Erlang References
* Local error handling (catch/throw)
* Single direction links (monitor)
* Handling of binary data (binaries / bit syntax)
* List comprehensions
* How to communicate with the outside world and/or software written in other
  languages (ports).
* Very few of the Erlang libraries have been touched on (for example file
  handling)
* OTP has been totally skipped and in consequence the Mnesia database has been
  skipped.
* Hash tables for Erlang terms (ETS)
* Changing code in running systems

