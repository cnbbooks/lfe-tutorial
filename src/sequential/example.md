## Example: Converting Temperature

Now for a larger example to consolidate what we have learnt so far. Assume we have a list of temperature readings from a number of cities in the world. Some of them are in Celsius (Centigrade) and some in Fahrenheit (as in the previous list). First let's convert them all to Celsius, then let's print out the data neatly. Save the following code to ``tut8.lfe``:

```lisp
(defmodule tut8
  (export (format-temps 1)))

;; Only this function is exported
(defun format-temps
  ((())
    ;; No output for an empty list
    'ok)
  (((cons city rest))
    (print-temp (f->c city))
    (format-temps rest)))

(defun f->c
  (((tuple name (tuple 'C temp)))
    ;; No conversion needed
    (tuple name (tuple 'C temp)))
  (((tuple name (tuple 'F temp)))
    ;; Do the conversion
    (tuple name (tuple 'C (/ (* (- temp 32) 5) 9)))))

(defun print-temp
  (((tuple name (tuple 'C temp)))
    (lfe_io:format "~-15w ~w C~n" (list name temp))))
```

```lisp
lfe> (c "tut8.lfe")
#(module tut8)
lfe> (tut8:format-temps
    '(#(Moscow #(C 10))
      #(Cape-Town #(F 70))
      #(Stockholm #(C -4))
      #(Paris #(F 28))
      #(London #(F 36)))))
Moscow          10 C
Cape-Town       21.11111111111111 C
Stockholm       -4 C
Paris           -2.2222222222222223 C
London          2.2222222222222223 C
ok
```

Before we look at how this program works, notice that we have added a few comments to the code. A comment starts with a ``;`` character and goes on to the end of the line. [^1] Note as well that the ``(export (format-temps 1))`` line only includes the function ``format-temps/1``, the other functions are local functions, i.e. they are not visible from outside the module ``temp-convert``.

When we call ``format-temps/1`` the first time, the ``city`` variable gets the value ``#(Moscow #(C-10))`` and the remainder of the list is assigned to the ``rest`` variable. Next, the ``f->c/1`` function is called inside the ``print-temp/1`` function, with ``f->c/1`` getting passed ``#(Moscow #(C-10))``.

Note that when we see function calls nested like ``(print-temp (f->c ...))`` -- in other words when one function call is passed as the argument to another function -- we execute (evaluate) them from the inside out. We first evaluate ``(f->c city)`` which gives the value ``#(Moscow #(C 10))`` as the temperature is already in Celsius and then we evaluate ``(print-temp #(Moscow #(C 10)))``. Note that the ``f->c/1`` function  works in a similar way to the ``convert-length/1`` function we wrote in a previous section.

Next, ``print-temp/1`` simply calls ``lfe_io:format/2`` in a similar way to what has been described above. Note that ``~-15w`` says to print the "term" with a field length (width) of 15 and left justify it.[^2]

Now we call ``(format-temps rest)`` with the remainder of the list as an argument. This way of doing things is similar to the loop constructs in other languages. (Yes, this is recursion, but don't let that worry you). So the same ``format-temps/1`` function is called again, this time ``city`` gets the value ``#(Cape-Town #(F 70))`` and we repeat the same procedure as before. We go on doing this until the list becomes empty, i.e. ``()``, which causes the first clause ``(format-temps '())`` to match. This simply "returns" or "results in" the atom ``ok``, so the program ends.

----

[^1] In LFE, the convention is that a comment starting with a single ``;`` is reserved for comments at the end of a line of code; lines which start with a comment use two ``;;``, as above. There are also conventions for ``;;;`` and ``;;;;`` -- to learn more about these, see [Program Development Using LFE - Rules and Conventions](http://docs.lfe.io/prog-rules/1.html) and [LFE Style Guide](http://docs.lfe.io/style-guide/1.html).

[^2] LFE's ``lfe_io:format`` differs from the Erlang standard library ``io:format`` in that ``lfe_io`` displays its results using LFE-formatted data structures in its Lisp syntax; ``io`` uses the standard Erlang syntax. You may use either from LFE. ``lfe_io`` takes the same formatting parameters as ``io``, so there should be no surprises if you're coming from Erlang. For more information, be sure to read the [io:format documentation](http://www.erlang.org/doc/man/io.html#fwrite-1).
