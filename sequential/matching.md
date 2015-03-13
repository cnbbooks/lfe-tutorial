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

Our functions above represent an example of walking through a list and "carrying" a value as we do so, in this case ``result-so-far`` is the variable that carries a value as we walk. ``list-max/1`` simply assumes that the max value of the list is the head of the list and calls ``list-max/2`` with the rest of the list and the value of the head of the list, in the above this would be ``(list-max '(2 3 4 5 6 7 4 3 2 1) 1)``. If we tried to use ``list-max/1`` with an empty list or tried to use it with something which isn't a list at all, we would cause an error. The LFE philosophy is not to handle errors of this type in the function they occur, but to do so elsewhere. More about this later.

### Guards

In ``list-max/2`` we walk down the list and use ``head`` instead of ``result-so-far`` when ``head`` is greater than ``result-so-far``. The spcecial form ``when`` is a something we can use with LFE patterns to limit a match. In this case we use it to say: "only use this part of the function if ``head`` is greater than ``result-so-far``. We call tests of this type a *guard*. If the guard isn't true (usually referred to as "if the guard fails"), we try the next part of the function. In this case if ``head`` *isn't* greater than ``result-so-far`` then it must be smaller or equal to it, so we don't need a guard on the this part of the function.

[more later]
