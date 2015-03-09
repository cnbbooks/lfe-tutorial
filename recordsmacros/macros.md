## Macros

[forthcoming]

### The Backquote Macro

The backquote macro makes it possible to build lists and tuples from templates. Used by itself a backquote is equivalent to a regular quote:

```lisp
> `(a b c)
(a b c)
```

Like a regular quote, a backquote alone protects its arguments from evaluation. The advantage of backquote is that it is possible to turn on evaluation inside it using ``,`` (comma) and ``,@`` (comma-at)[^1]. When something is prefixed with a comma it will be evaluated. For example:

```lisp
> (set (tuple a b) #(1 2))
#(1 2)
> `(a is ,a and b is ,b)
(a is 1 and b is 2)
> `#(a ,a b ,b)
#(a 1 b 2)
```

It works with both lists and tuples. The backquote actually expands to an expression which builds the structure the templates describes. For example `` `(a is ,a and b is ,b)`` expands to ``(list 'a 'is a 'and b 'is b)``, `` `(a . ,a)`` expands to ``(cons 'a a)`` and `` `#(a ,a b ,b)`` expands to ``(tuple 'a a 'b b)``. They are very useful in macros as we can write a macro definitions which look like the expansions they produce. For example we could define ``unless`` as:

```lisp
(defmacro unless
  ((cons test body) `(if (not ,test) ,@body)))
```

Comma-at is like comma but splices its argument which should be a list. As the backquote macro expands to the expression which would build the template it is also very useful in patterns as we can use a template to describe the pattern. Here is the [Converting Temperature](../sequential/example.md) example rewritten to use backquote in both the patterns and constructors:

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

[^1] In LFE the backquote is a normal macro and is expanded at the same time as other macros. When they are parsed `` `thing`` becomes ``(backquote thing)``, ``,thing`` becomes ``(unquote thing)`` and ``,@thing`` becomes ``(unquote-splicing thing)``.
