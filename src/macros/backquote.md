## The Backquote Macro

The backquote macro makes it possible to build lists and tuples from templates. Used by itself a backquote is equivalent to a regular quote:

```lisp
lfe> `(a b c)
(a b c)
```

Like a regular quote, a backquote alone protects its arguments from evaluation. The advantage of backquote is that it is possible to turn on evaluation inside forms which are backquoted using ``,`` (comma or "unquote") and ``,@`` (comma-at or "unquote splice").[^1] When something is prefixed with a comma it will be evaluated. For example:

```lisp
lfe> (set (tuple a b) #(1 2))
#(1 2)
lfe> `(a is ,a and b is ,b)
(a is 1 and b is 2)
lfe> `#(a ,a b ,b)
#(a 1 b 2)
```

Quoting works with both lists and tuples. The backquote actually expands to an expression which builds the structure the templates describes. For example, the following

```lisp
`(a is ,a and b is ,b)
```

expands to

```lisp
(list 'a 'is a 'and b 'is b)
```

and

```lisp
`(a . ,a)
```

expands to

```lisp
(cons 'a a)
```

This:

```lisp
`#(a ,a b ,b)
```

expands to

```lisp
(tuple 'a a 'b b)
```

They are very useful in macros as we can write a macro definitions which look like the expansions they produce. For example we could define the ``unless`` from the previous section as:

```lisp
(defmacro unless
  ((cons test body) `(if (not ,test) (progn ,@body))))
```

Here we have extended it allow multiple forms in the body. Comma-at is like comma but splices its argument which should be a list. So for example:

```lisp
lfe> (macroexpand '(unless (test x) (first-do) (second-do)) $ENV)
(if (not (test x)) (progn (first-do) (second-do)))
```

As the backquote macro expands to the expression which would build the template it is also very useful in patterns as we can use a template to describe the pattern. Here is the [Converting Temperature](../sequential/example.md) example rewritten to use backquote in both the patterns and constructors:

```lisp
(defun f->c
  ((`#(,name #(C ,temp)))
    ;; No conversion needed
    `#(,name #(C ,temp)))
  ((`#(,name #(F ,temp)))
    ;; Do the conversion
    `#(,name #(C ,(/ (* (- temp 32) 5) 9)))))

(defun print-temp
  ((`#(,name #(C ,temp)))
    (lfe_io:format "~-15w ~w C~n" `(,name ,temp))))
```

Using the backquote macro also makes it much easier to build expressions which we can evaluate with ``eval``. So if we want to import values into the expression to evaluate we can do it like this:

```lisp
lfe> (set expr '(* x y))                   ;Expression to evaluate
(* x y)
lfe> (eval `(let ((x 17) (y 42)) ,expr))
714
```

----

[^1] In LFE the backquote is a normal macro and is expanded at the same time as other macros. When they are parsed `` `thing`` becomes ``(backquote thing)``, ``,thing`` becomes ``(comma thing)`` and ``,@thing`` becomes ``(comma-at thing)``.
