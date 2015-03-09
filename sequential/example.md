## Example: Converting Temperature

Now for a larger example to consolidate what we have learnt so far. Assume we have a list of temperature readings from a number of cities in the world. Some of them are in Celsius (Centigrade) and some in Fahrenheit (as in the previous list). First let's convert them all to Celsius, then let's print out the data neatly. Save the following code to ``temp-convert.lfe``:

```lisp
(defmodule temp-convert
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
  (((tuple name (tuple 'C temp))
    ;; No conversion needed
    (tuple name (tuple 'C temp))))
  (((tuple name (tuple 'F temp)))
    ;; Do the conversion
    (tuple name (tuple 'C (/ (* (- temp 32) 5) 9)))))

(defun print-temp
  (((tuple name (tuple 'C temp)))
    (lfe_io:format "~-15w ~w C~n" (list name temp))))
```

```lisp
> (c "temp-convert.lfe")
#(module temp-convert)
> (temp-convert:format-temps
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

Before we look at how this program works, notice that we have added a few comments to the code. A comment starts with a ``;`` character and goes on to the end of the line (in LFE, the convention is that a comment starting with a single ``;`` is reserved for comments at the end of a line of code; lines which start with a comment use two ``;;``, as above). Note as well that the ``(export (format-temps 1))`` line only includes the function ``format-temps/1``, the other functions are local functions, i.e. they are not visible from outside the module ``temp-convert``.
