# Lists

Whereas tuples group things together, we also want to be able to represent lists of things. Lists in LFE are surrounded by "(" and ")". For example a list of the temperatures of various cities in the world could be:

```lisp
(#(Moscow #(C -10))
 #(Cape-Town #(F 70)) #(Stockholm #(C -4))
 #(Paris #(F 28)) #(London #(F 36)))
```

Note that this list was so long that it didn't fit on one line. This doesn't matter, LFE allows line breaks at all "sensible places" but not, for example, in the middle of atoms, integers etc.

A very useful way of looking at parts of lists is by using the constructor ``cons``. It can also be used as a pattern to match against lists. This is best explained by an example using the shell.

```lisp
> (set (cons first rest) '(1 2 3 4 5))
(1 2 3 4 5)
> first
1
> rest
(2 3 4 5)
```

We see here that ``set`` also allows you to define variables using patterns. Using ``cons`` we could separate the first element of the list from the rest of list (``first`` got the value 1 and ``rest`` the value (2 3 4 5)). We also see here that when we want to give a literal list we need to *quote* it with ``'``. This stops LFE trying to evaluate the list as a function call in the same way as quoting an atom stops LFE trying to evaluate the atom as a variable.

Another example:

```lisp
> (set (cons e1 (cons e2 r)) '(1 2 3 4 5 6 7))
(1 2 3 4 5 6 7)
> e1
1
> e2
2
> r
(3 4 5 6 7)
```

We see here nesting ``cons`` to get the first two elements from the list. Of course if we try to get more elements from the list than there are elements in the list we will get an error. Note also the special case of the list with no elements ().

```lisp
> (set (cons a (cons b c)) '(1 2))
(1 2)
> a
1
> b
2
> c
()
```

In all examples above we have been using new variables names, not reusing old ones. While ``set`` does allow you to rebind variables normally a variable can only be given a value once in its context (scope).

The following example shows how we find the length of a list:

```lisp
(defmodule tut4
  (export (list-length 1)))

(defun list-length
  ((()) 0)
  (((cons first rest))
   (+ 1 (list-length rest))))
```

Compile (file ``tut4.lfe``) and test:

```lisp
> (c "tut4")
#(module tut4)
> (tut4:list-length '(1 2 3 4 5 6 7))
7
```

Explanation:

```lisp
(defun list-length
  ((()) 0)
```

The length of an empty list is obviously 0.

```lisp
  (((cons first rest))
   (+ 1 (list-length rest))))
```

The length of a list with the first element ``first`` and the remaining elements ``rest`` is 1 + the length of ``rest``.

(Advanced readers only: This is not tail recursive, there is a better way to write this function).

In general we can say we use tuples where we would use "records" or "structs" in other languages and we use lists when we want to represent things which have varying sizes, (i.e. where we would use linked lists in other languages).

LFE does not have a string data type, instead strings can be represented by lists of Unicode characters. So the list ``(97 98 99)`` is equivalent to "abc". Note that we don't have to quote strings as we do lists. The LFE repl is "clever" and guesses the what sort of list we mean and outputs it in what it thinks is the most appropriate form, for example:

```lisp
> '(97 98 99)
"abc"
> "abc"
"abc"
> '"abc"
"abc"
```

Lists can also be surrounded by "[" and "]" instead of parentheses. They are equivalent but must match, for example:

```lisp
> '(a b c)
(a b c)
> '[a b c]
(a b c)
> '(a b c]
1: illegal ']'
```

This can be used to make list structures easier to read. For example, it is often used in function definitions for the list of arguments when there are multiple clauses:

```lisp
(defun list-length
  ([()] 0)
  ([(cons first rest)]
   (+ 1 (list-length rest))))
```
