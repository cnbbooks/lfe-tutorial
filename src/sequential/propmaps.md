## Property Lists and Maps

### Property Lists

Property lists in Erlang and LFE are a simple way to create key-value pairs (we actually saw them in the last section, but didn't mention it). They have a very simple structure: a list of tuples, where the key is the first element of each tuple and is an atom. Very often you will see "options" for functions provided as property lists (this is similar to how other programming languages use keywords in function arguments).

Property lists can be created with just the basic data structures of LFE:

```lisp
lfe> (set options (list (tuple 'debug 'true)
                     (tuple 'default 42)))
(#(debug true) #(default 42))
```

Or, more commonly, using quoted literals:

```lisp
lfe> (set options '(#(debug true) #(default 42)))
(#(debug true) #(default 42))
```

There are convenience functions provided in the `proplists` module. In the last example, we define a default value to be used in the event that the given key is not found in the proplist:

```lisp
lfe> (proplists:get_value 'default options)
42
lfe> (proplists:get_value 'poetry options)
undefined
lfe> (proplists:get_value 'poetry options "Vogon")
"Vogon"
```

Be sure to read the [module documentation](http://www.erlang.org/doc/man/proplists.html) for more information. Here's an example of our options in action:

```lisp
(defun div (a b)
  (div a b '()))

(defun div (a b opts)
  (let ((debug (proplists:get_value 'debug opts 'false))
        (ratio? (proplists:get_value 'ratio opts 'false)))
    (if (and debug ratio?)
        (io:format "Returning as ratio ...~n"))
    (if ratio?
        (++ (integer_to_list 1) "/" (integer_to_list 2))
        (/ a b))))
```

Let's try our function without and then with various options:

```lisp
lfe> (div 1 2)
0.5
lfe> (div 1 2 '(#(ratio true)))
"1/2"
lfe> (div 1 2 '(#(ratio true) #(debug true)))
Returning as ratio ...
"1/2"
```

### Maps

As with property lists, maps are a set of key to value associations. You may create an association from "key" to value 42 in one of two ways: using the LFE core form `map` or entering a map literal:

```lisp
lfe> (map "key" 42)
#M("key" 42)
lfe> #M("key" 42)
#M("key" 42)
```

We will jump straight into the deep end with an example using some interesting features. The following example shows how we calculate alpha blending using maps to reference colour and alpha channels. Save this code as the file `tut7.lfe` in the directory from which you have run the LFE REPL:

```lisp
(defmodule tut7
  (export (new 4) (blend 2)))

(defmacro channel? (val)
  `(andalso (is_float ,val) (>= ,val 0.0) (=< ,val 1.0)))

(defmacro all-channels? (r g b a)
  `(andalso (channel? ,r)
            (channel? ,g)
            (channel? ,b)
            (channel? ,a)))

(defun new
  ((r g b a) (when (all-channels? r g b a))
   (map 'red r 'green g 'blue b 'alpha a)))

(defun blend (src dst)
  (blend src dst (alpha src dst)))

(defun blend
  ((src dst alpha) (when (> alpha 0.0))
   (map-update dst
               'red (/ (red src dst) alpha)
               'green (/ (green src dst) alpha)
               'blue (/ (blue src dst) alpha)
               'alpha alpha))
  ((_ dst _)
   (map-update dst 'red 0 'green 0 'blue 0 'alpha 0)))

(defun alpha
  (((map 'alpha src-alpha) (map 'alpha dst-alpha))
   (+ src-alpha (* dst-alpha (- 1.0 src-alpha)))))

(defun red
  (((map 'red src-val 'alpha src-alpha)
    (map 'red dst-val 'alpha dst-alpha))
   (+ (* src-val src-alpha)
      (* dst-val dst-alpha (- 1.0 src-alpha)))))

(defun green
  (((map 'green src-val 'alpha src-alpha)
    (map 'green dst-val 'alpha dst-alpha))
   (+ (* src-val src-alpha)
      (* dst-val dst-alpha (- 1.0 src-alpha)))))

(defun blue
  (((map 'blue src-val 'alpha src-alpha)
    (map 'blue dst-val 'alpha dst-alpha))
   (+ (* src-val src-alpha)
      (* dst-val dst-alpha (- 1.0 src-alpha)))))
```

Now let's try it out, first compiling it:

```lisp
lfe> (c "tut7.lfe")
#(module tut7)
lfe> (set colour-1 (tut7:new 0.3 0.4 0.5 1.0))
#M(alpha 1.0 blue 0.5 green 0.4 red 0.3)
lfe> (set colour-2 (tut7:new 1.0 0.8 0.1 0.3))
#M(alpha 0.3 blue 0.1 green 0.8 red 1.0)
lfe> (tut7:blend colour-1 colour-2)
#M(alpha 1.0 blue 0.5 green 0.4 red 0.3)
lfe> (tut7:blend colour-2 colour-1)
#M(alpha 1.0 blue 0.38 green 0.52 red 0.51)
```

This example warrants some explanation.

First we define a couple macros to help with our guard tests. This is only here for convenience and to reduce syntax cluttering. Guards can be only composed of a limited set of functions, so we needed to use macros that would compile down to just the functions allowed in guards. A full treatment of Lisp macros is beyond the scope of this tutorial, but there is a lot of good material available online for learning macros, including Paul Graham's book "On Lisp."

```lisp
(defun new
  ((r g b a) (when (all-channels? r g b a))
   (map 'red r 'green g 'blue b 'alpha a)))
```

The function `new/4` [^1] creates a new map term with and lets the keys `red`, `green`, `blue` and `alpha` be associated with an initial value. In this case we only allow for float values between and including 0.0 and 1.0 as ensured by the `all-channels?` and `channel?` macros.

By calling `blend/2` on any colour term created by `new/4` we can calculate the resulting colour as determined by the two maps terms.

The first thing `blend/2` does is to calculate the resulting alpha channel.

```lisp
(defun alpha
  (((map 'alpha src-alpha) (map 'alpha dst-alpha))
   (+ src-alpha (* dst-alpha (- 1.0 src-alpha)))))
```

We fetch the value associated with key `alpha` for both arguments using the `(map 'alpha <var>)` pattern. Any other keys in the map are ignored, only the key `alpha` is required and checked for.

This is also the case for functions `red/2`, `blue/2` and `green/2`.

```lisp
(defun red
  (((map 'red src-val 'alpha src-alpha)
    (map 'red dst-val 'alpha dst-alpha))
   (+ (* src-val src-alpha)
      (* dst-val dst-alpha (- 1.0 src-alpha)))))
```

The difference here is that we check for two keys in each map argument. The other keys are ignored.

Finally we return the resulting colour in `blend/3`.

```lisp
(defun blend
  ((src dst alpha) (when (> alpha 0.0))
   (map-update dst
               'red (/ (red src dst) alpha)
               'green (/ (green src dst) alpha)
               'blue (/ (blue src dst) alpha)
               'alpha alpha))
```

We update the `dst` map with new channel values. The syntax for updating an existing key with a new value is done with `map-update` form.

----

[^1] Note the use of the slash and number after the function name. We will be discussing this more in a future section, though before we get there you will see this again. Until we get to the full explanation, just know that the number represents the arity of a given function and this helps us be explicit about which function we mean.
