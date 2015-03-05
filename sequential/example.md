## Example: Converting Temperature

Now for a larger example to consolidate what we have learnt so far. Assume we have a list of temperature readings from a number of cities in the world. Some of them are in Celsius (Centigrade) and some in Fahrenheit (as in the previous list). First let's convert them all to Celsius, then let's print out the data neatly.

```lisp
(defmodule temp-convert
  (export (format-temps 1)))
  
;; Only this function is exported
(defun format-temps
  (('())
    ;; No output for an empty list
    'ok)
  (((cons city rest))
    (print-temp (f->c city))
    (format-temps rest)))

(defun f->c
  ((`#(,name #(C ,temp)))
    ;; No conversion needed
    `#(,name #(C ,temp)))
  ((`#(,name #(F ,temp)))
    ;; Do the conversion
    `#(,name #(C ,(/ (* (- temp 32) 5) 9)))))

(defun print-temp
  ((`#(,name #(C ,temp)))
    (io:format "~-15w ~w C~n" `(,name ,temp))))
```

```lisp
> (format-temps '(#(Moscow #(C 10))
                  #(Cape-Town #(F 70))
                  #(Stockholm #(C -4))
                  #(Paris #(F 28))
                  #(London #(F 36)))))
```