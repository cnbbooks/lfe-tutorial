## Eval

The form ``eval`` takes an LFE data structure and evaluates it as an expression and then returns the value:

```lisp
> (eval 15)
15
> (eval '(+ 1 2 3 4))
10
> (eval '(list 'a 'b 'c))
(a b c)
```

Using ``eval`` is one way way to merge lists and code. However, it is not a very good way:

- It is inefficient as the input expression is evaluated by ``lfe_eval`` the LFE interpreter. This is much slower than running compiled code.

- The expression is evaluated without a lexical context. So calling ``eval`` inside a ``let`` does not allow the evaluated expression to refer to variables bound by the ``let``:

```lisp
> (set expr '(* x y))                   ;Expression to evaluate
(* x y)
> (let ((x 17) (y 42)) (eval expr))
exception error: #(unbound_symb x)

```

Well, this is not quite true. If we "reverse" the code and build a ``let`` expression which imports and binds the variables and then call ``eval`` on it we can access the variables:

```lisp
> (eval (list 'let (list (list 'x 17) (list 'y 42)) expr))
714
```
