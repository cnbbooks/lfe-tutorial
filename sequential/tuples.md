## Tuples

Now the ``tut2`` program is hardly good programming style. Consider:

```lisp
(tut2:convert 3 'inch)
```

Does this mean that 3 is in inches? or that 3 is in centimeters and we want to convert it to inches? So LFE has a way to group things together to make things more understandable. We call these tuples. Tuples are contructed and matched using ``(tuple ...)`` and literal tuples written with ``#( ... )``.

So we can write ``#(inch 3)`` to denote 3 inches and ``#(centimeter 5)`` to denote 5 centimeters. Now let's write a new program which converts centimeters to inches and vice versa. (file ``tut3.lfe``).

```lisp
(defmodule tut3
  (export (convert-length 1)))

(defun convert-length
  (((tuple 'centimeter x)) (tuple 'inch (/ x 2.54)))
  (((tuple 'inch y)) (tuple 'centimeter (* y 2.54))))
```

Compile and test:

```lisp
(c "tut3.lfe")                 
#(module tut3)
> (tut3:convert-length #(inch 5))
#(centimeter 12.7)
> (tut3:convert-length (tut3:convert-length #(inch 5)))
#(inch 5.0)
```

Note that in the last call we convert 5 inches to centimeters and back again and reassuringly get back to the original value. I.e the argument to a function can be the result of another function. Pause for a moment and consider how that line (above) works. The argument we have given the #function ``#(inch 5)`` is first matched against the first head clause of ``convert_length`` i.e. in ``((tuple 'centimeter x))`` where it can be seen that the pattern ``(tuple 'centimeter x)`` does not match ``#(inch 5)`` (the *head* is the bit first bit in the clause with a list of argument patterns). This having failed, we try the head of the next clause i.e. ``((tuple 'inch y))``, this pattern matches ``#(inch 5)`` and y gets the value 5.

We have shown tuples with two parts above, but tuples can have as many parts as we want and contain any valid LFE **term**. For example, to represent the temperature of various cities of the world we could write

```lisp
#(moscow #(c -10))
#(cape_town #(f 70))
#(paris #(f 28))
```

Tuples have a fixed number of things in them. We call each thing in a tuple an *element*. So in the tuple ``#(moscow #(c -10))``, element 1 is ``moscow`` and element 2 is ``#(c -10)``. I have chosen ``c`` meaning Celsius (or Centigrade) and ``f`` meaning Fahrenheit.
