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
> (cons 'Madrid tail)
(Madrid London Rome)
```

Let's look at a more involved example where we use ``cons``es to reverse the order of a list:

```lisp
(defmodule tut10
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
> (c "tut10.lfe")
#(module tut10)
> (tut10:reverse (list 1 2 3))
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

### Processing Lists

Now lets get back to the cities and temperatures, but take a more structured approach this time. First let's convert the whole list to Celsius as follows:

```lisp
(defmodule tut11
  (export (format-temps 1)))

(defun format-temps (cities)
  (->c cities))

(defun ->c
  (((cons (tuple name (tuple 'F temp)) tail))
   (let ((converted (tuple name (tuple 'C (/ (* (- temp 32) 5) 9)))))
     (cons converted (->c tail))))
  (((cons city tail))
   (cons city (->c tail)))
  (('())
   '()))
```
 Now let's test this new function:
 
```lisp
> (c "tut11.lfe")
#(module tut11)
> (tut11:format-temps
    '(#(Moscow #(C 10))
      #(Cape-Town #(F 70))
      #(Stockholm #(C -4))
      #(Paris #(F 28))
      #(London #(F 36)))))
(#(Moscow #(C 10))
 #(Cape-Town #(C 21.11111111111111))
 #(Stockholm #(C -4))
 #(Paris #(C -2.2222222222222223))
 #(London #(C 2.2222222222222223)))
```

Let's look at this, bit-by-bit. In the first function:

```lisp
(defun format-temps (cities)
  (->c cities))
```

we see that ``format-temps/1`` calls ``->c/1``. ``->c/1`` takes off the head of the List ``cities`` and converts it to Celsius if needed. The ``cons`` function is used to add the (maybe) newly converted city to the converted rest of the list:

```lisp
(cons converted (->c tail))
```
or

```lisp
(cons city (->c tail))
```

We go on doing this until we get to the end of the list (i.e. the list is empty):

```lisp
  (('())
   '())
```

Now that we have converted the list, we should add a function to print it:

```lisp
(defmodule tut12
  (export (format-temps 1)))

(defun format-temps (cities)
  (print-temps (->c cities)))

(defun ->c
  (((cons (tuple name (tuple 'F temp)) tail))
   (let ((converted (tuple name (tuple 'C (/ (* (- temp 32) 5) 9)))))
     (cons converted (->c tail))))
  (((cons city tail))
   (cons city (->c tail)))
  (('())
   '()))

(defun print-temps
  (((cons (tuple name (tuple 'C temp)) tail))
   (io:format "~-15w ~w c~n" (list name temp))
   (print-temps tail))
  (('())
   'ok))
```

Let's take a look:

```lisp
> (c "tut12.lfe")
#(module tut12)
> (tut12:format-temps
    '(#(Moscow #(C 10))
      #(Cape-Town #(F 70))
      #(Stockholm #(C -4))
      #(Paris #(F 28))
      #(London #(F 36)))))
'Moscow'        10 c
'Cape-Town'     21.11111111111111 c
'Stockholm'     -4 c
'Paris'         -2.2222222222222223 c
'London'        2.2222222222222223 c
ok
```

### Utility Functions Revisited

Remember a few sections back when we created the utility function for finding the maximum value in a list? Let's put that into action now: we want to add a function which finds the cities with the maximum and minimum temperatures. 

```lisp
(defmodule tut13
  (export (format-temps 1)))

(defun format-temps (cities)
  (let* ((converted (->c cities)))
    (print-temps converted)
    (print-max-min (find-max-min converted))))

(defun ->c
  (((cons (tuple name (tuple 'F temp)) tail))
   (let ((converted (tuple name (tuple 'C (/ (* (- temp 32) 5) 9)))))
     (cons converted (->c tail))))
  (((cons city tail))
   (cons city (->c tail)))
  (('())
   '()))

(defun print-temps
  (((cons (tuple name (tuple 'C temp)) tail))
   (io:format "~-15w ~w c~n" (list name temp))
   (print-temps tail))
  (('())
   'ok))

(defun find-max-min
  (((cons city tail))
    (find-max-min tail city city)))

(defun find-max-min
  (((cons head tail) max-city min-city)
   (find-max-min tail
                 (compare-max head max-city)
                 (compare-min head min-city)))
  (('() max-city min-city)
   (tuple max-city min-city)))

(defun compare-max
  (((= (tuple name1 (tuple 'C temp1)) city1)
    (= (tuple name2 (tuple 'C temp2)) city2))
   (if (> temp1 temp2)
       city1
       city2)))

(defun compare-min
  (((= (tuple name1 (tuple 'C temp1)) city1)
    (= (tuple name2 (tuple 'C temp2)) city2))
   (if (< temp1 temp2)
       city1
       city2)))

(defun print-max-min
  (((tuple (tuple max-name (tuple 'C max-temp))
           (tuple min-name (tuple 'C min-temp))))
   (io:format "Max temperature was ~w c in ~w~n" (list max-temp max-name))
   (io:format "Min temperature was ~w c in ~w~n" (list min-temp min-name))))
```

Let's try it out:

```lisp
> (c "tut13.lfe")
#(module tut13)
> (tut13:format-temps
    '(#(Moscow #(C 10))
      #(Cape-Town #(F 70))
      #(Stockholm #(C -4))
      #(Paris #(F 28))
      #(London #(F 36)))))
'Moscow'        10 c
'Cape-Town'     21.11111111111111 c
'Stockholm'     -4 c
'Paris'         -2.2222222222222223 c
'London'        2.2222222222222223 c
Max temperature was 21.11111111111111 c in 'Cape-Town'
Min temperature was -4 c in 'Stockholm'
ok
```

As you may have noticed, that program isn't the most *efficient* way of doing this, since we walk through the list of cities four times. But it is better to first strive for clarity and correctness and to make programs efficient only if really needed.

----

[^1]: Way back in the prehistoric times when large, building-size computers still roamed the earth and the languages which ran on them were tiny and furry, Lisp came along with only a handful of forms: ``cons`` was one of them, and it was used to construct lists, one cell at a time.

[^2]: More information about this ``lists`` module is available [here](http://www.erlang.org/doc/man/lists.html).