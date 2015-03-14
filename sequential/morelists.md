## More About Lists

### The ``cons`` Form
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
(defmodule tut7
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
> (c "tut7.lfe")
#(module tut7)
> (tut7:reverse (list 1 2 3))
(3 2 1)
```

Consider how ``reversed-list`` is built: it starts as ``'()``, we then successively take off the heads of the list that was provided and add these heads to the the ``reversed-list`` variable, as detailed by the following:

```lisp
(reverse (cons 1 '(2 3)) '()) => (reverse '(2 3) (cons 1 '()))
(reverse (cons 2 '(3)) '(1)) => (reverse '(3) (cons 2 '(1)))
(reverse (cons 3 '()) (2 1)) => (reverse '() (cons 3 '(2 1)))
(reverse '() '(3 2 1)) => '(3 2 1)
```

The Erlang module ``lists`` contains a lot of functions for manipulating lists, for example for reversing them -- our work above was done for demonstration and pedagogical purposes. For serious applications, one should prefer functions in the Erlang standard library.[^2]

Now lets get back to the cities and temperatures, but take a more structured approach this time. First let's convert the whole list to Celsius as follows:

```lisp
(defmodule tut8
  (export all))

...
```
 Now let's test this new function:
 
```lisp

```

[more to come]

### Processing Lists

[forthcoming]

### Utility Functions Revisited

[forthcoming]

----

[^1]: Way back in the prehistoric times when large, building-size computers still roamed the earth and the languages which ran on them were tiny and furry, Lisp came along with only a handful of forms: ``cons`` was one of them, and it was used to construct lists, one cell at a time.

[^2]: More information about this ``lists`` module is available [here](http://www.erlang.org/doc/man/lists.html).