# Property Lists and Maps

## Property Lists

Property lists in Erlang and LFE are a simple way to create key-value pairs. They have a very simple structure: a list of tuples, where the key is the first element of each tuple and is an atom. Very often you will see "options" for functions provided as property lists (this is similar to how other programming languages use keywords in function arguments).

Property lists can be created with just the basic data structures of LFE:

```lisp
> (set options (list (tuple 'debug 'true)
                     (tuple 'default 42)))
(#(debug true) #(default 42))
```

Or, more commonly, using quoted literals:

```lisp
> (set options '(#(debug true) #(default 42)))
(#(debug true) #(default 42))
```

There are convenience functions provided in the ``proplists`` module. In the last example, we define a default value to be used in the event that the given key is not found in the proplist:

```lisp
> (proplists:get_value 'default options)
42
> (proplists:get_value 'poetry options)
undefined
> (proplists:get_value 'poetry options "Vogon")
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

Let's try our funtion without and then with various options:

```lisp
> (div 1 2)
0.5
> (div 1 2 '(#(ratio true)))
"1/2"
> (div 1 2 '(#(ratio true) #(debug true)))
Returning as ratio ...
"1/2"
```

## Maps

As with property lists, maps are a set of key to value associations. You may create an association from "key" to value 42 in one of two ways: using the LFE core form ``map`` or entering a map literal:

```lisp
> (map "key" 42)
#M("key" 42)
> #M("key" 42)
#M("key" 42)
```

We will jump straight into the deep end with an example using some interesting features. The following example shows how we calculate alpha blending using maps to reference color and alpha channels:

```lisp

```

