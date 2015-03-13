## More About Lists

Remember how we used ``cons`` to "extract" head and tail values of a list when matching them? In the REPL, we would do it like so:

```lisp
> (set (cons head tail) (list 'Paris 'London 'Rome))
(Paris London Rome)
> head
Paris
> tail
(London Rome)
```
Well, that's not how ``cons`` started life[^1]; it's original use was in "cons"tructing lists, not taking them apart. Here is some classic usage:

```lisp
> (cons 'Madric tail)
(Madric London Rome)
```

Let's look at a more involved example where we use ``cons``es to reverse the order of a list:

```lisp
(defmodule tut8
  (export all))
  
(defun reverse (list)
  (reverse list '()))
  
(defun reverse
  (((cons head tail) reversed-list)
   (reverse tail (cons head reversed-list)))
  (('() reversed-list)
   reversed-list))
```

Then, in the REPL:

```lisp
> (c "tut8.lfe")
#(module tut8)
> (tut8:reverse (list 1 2 3))
(3 2 1)
```

Consider how ``reversed-list`` is built: it starts as ``'()``, we then successively take off the heads of the list that was provided and add these heads to the the ``reversed-list`` variable, as detailed by the following:

```lisp

```

[more to come]

----

[^1]: Way back in the prehistoric times when large, building-size computers still roamed the earth and the languages which ran on them were tiny and furry, Lisp came along with only a handful of forms: ``cons`` was one of them, and it was used to construct lists, one cell at a time.