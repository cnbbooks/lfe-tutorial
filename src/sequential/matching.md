## Matching and Guards and Scope of Variables

In the previous section we wrote a little example program for converting temperatures. In creating programs like that with special data structures (in our case, a list of cities and their temperatures), it's often useful to create utility functions which make working with our data more convenient. We will explore that below and use these functions to introduce some new concepts.

### A Utility Function

In this case, it could be useful to find the maximum and minimum temperature in our data. We can add support for this by creating the necessary code little bit at a time. Let's start with creating functions for finding the maximum value of the elements of a property list:

```lisp
(defmodule tut9
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
lfe> (c "tut9.lfe")
#(module tut9)
lfe> (tut9:list-max '(1 2 3 4 5 6 7 4 3 2 1))
7
```

### Pattern Matching

Before we talk about pattern matching, let's clarify why we have two different functions with the same name. Note that the two ``list-max`` functions above each take a different number of arguments (parameters). Or, another way of saying it: they have different *arity*. In LFE, functions having the same name but differing in arity are actually *different functions*. Where we need to distinguish between these functions we write ``<function name>/<arity>``, where ``<arity>`` is an integer representing the number of arguments that function takes. For our example above, we would write ``list-max/1`` and ``list-max/2``.

The next thing we should explain is the arguments for the ``list-max/2`` function, since that probably looks pretty strange the first time you see it. If you look closely, you will see that there are three clauses in ``list-max/2`` and each clause stats with the function arguments for that clause; in order, they are:

* an empty list and ``results``
* a ``cons`` and ``results-so-far`` with something called a *guard* (more on that soon)
* a ``cons`` and ``results-so-far`` just by itself

What each of these are doing is what is called *pattern matching* in LFE: if the arguments passed to ``list-max/2`` match the first pattern, the first clause gets executed and the others don't. If the first one does not match, the second one is tried, and so on.

So what is being "matched" here? Well, the first clause will match if it's first argument is an empty list. The second and third will match if the first element is a list. The second has something more, though: let's take a closer look.

### Guards

In the second clause of ``list-max/2`` we see a new form: ``(when ...)`` which contains a comparison operation. The special form ``when`` is a something we can use with LFE patterns to limit a match. In this case we use it to say: "only use this function clause if ``head`` is greater than ``result-so-far``. We call tests of this type a *guard*. If the guard isn't true (usually referred to as "if the guard fails"), then we try the next part of the function.

### Stepping Through the Function

Now that we know how there can be two functions with the same name and how the arguments for ``list-max/2`` are working, let's step through the functions above. They represent an example of walking through a list and "carrying" a value as we do so, in this case ``result-so-far`` is the variable that carries a value as we walk. ``list-max/1`` simply assumes that the max value of the list is the head of the list and calls ``list-max/2`` with the rest of the list and the value of the head of the list, in the above this would be ``(list-max '(2 3 4 5 6 7 4 3 2 1) 1)``. If we tried to use ``list-max/1`` with an empty list or tried to use it with something which isn't a list at all, we would cause an error. The LFE philosophy is not to handle errors of this type in the function they occur, but to do so elsewhere. More about this later.

In ``list-max/2`` we walk down the list and use ``head`` instead of ``result-so-far`` when ``head`` is greater than ``result-so-far``. In this function, if ``head`` *isn't* greater than ``result-so-far`` then it must be smaller or equal to it, and the next clause is executed under this condition.

To change the above program to one which works out the minimum value of the element in a list, all we would need to do is to write ``<`` instead of ``>`` in the guard ... but it would be wise to change the name of the function to ``list-min`` :-).

### Scope and ``let``

In a function, the arguments that are passed to it are *in scope* or "accessible" for all of that function. In another function which has been passed its own arguments, only those are in scope; arguments from other functions are not available for use.

In the case of functions which are pattern matching on the arguments, like our ``list-max/2``, we have three clauses, each with their own arguments and each with their own scope. The parentheses at the beginning of each clause mark this scope. In that case, the ``results`` variable is only available in the first clause; it is not in scope for the second two clauses.

But passing arguments to functions and pattern matching function arguments are not the only ways in which you can bind a value to a variable. There is a special form in LFE (and other Lisps) called ``let``. Let's[^1] take a look, shall we? Here's another way we could have written ``list-max/2``:

```lisp
(defun list-max
  (('() results)
   results)
  (((cons head tail) result-so-far) (when (> head result-so-far))
   (let ((new-result-so-far head))
     (list-max tail new-result-so-far)))
  (((cons head tail) result-so-far)
   (list-max tail result-so-far)))
```

We only made a change to the second clause: we assigned the value of ``head`` to the variable ``new-result-so-far``. This assignment didn't involve any computation or new values, it was essentially just a "renaming" mostly for the sake of demonstration, and arguably to make it more clear the purpose of the value stored in ``head``.


----

[^1] We are not going to apologise for that pun.
