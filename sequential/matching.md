## Matching, Guards and Scope of Variables

In the previous section we wrote a little example program for converting temperatures. In creating programs like that with special data structures (in our case, a list of cities and their temperatures), it's often useful to create utility functions which make working with our data more convenient.

In this case, it could be useful to find the maximum and minimum temperature in our data. We can add support for this by creating the necessary code little bit at a time. Let's start with creating functions for finding the maximum value of the elements of a property list:

```lisp
(defmodule tut6
  (export (list-max 1)))

(defun list-max
  (((cons head tail))
   (list-max tail head)))

(defun list-max
  (('() results)
   results)
  (((cons head tail) result-so-far) (when (> head result-so-far))
   (list-max tail head))
  (((cons head tail) result-so-far)
   (list-max tail result-so-far)))
```

Then, in the LFE REPL:

```lisp
> (c "tut6.lfe")
#(module tut6)
> (tut6:list-max '(1 2 3 4 5 6 7 4 3 2 1))
7
```

First note that we have two functions here with the same name: ``list-max``. However, each of these takes a different number of arguments (parameters). Or, another way of saying it: they have different arity. In LFE, functions having the same name but differing in arity are actually *different functions*. Where we need to distinguish between these functions we write name/arity, where name is the name of the function and arity is the number of arguments, in this case ``list-max/1`` and ``list-max/2``.

