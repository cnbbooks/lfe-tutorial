## Modules and Functions

### Creating a Simple Module

A programming language isn't much use if you can only run code from a REPL. So next we will write a small LFE program in a file on the file system. In the same directory that you started the LFE REPL, create a new file called ``tut.lfe`` (the filename is **important**: be sure you type it just as we have) using your favorite text editor.

Here's the code to enter:

```lisp
(defmodule tut
  (export all))

(defun double (x)
  (* 2 x))
```

It's not hard to guess that this "program" doubles the value of numbers. We'll get back to the first two lines later. Let's compile the program. This can be done in the LFE REPL as shown below:

```lisp
> (c "tut.lfe")
#(module tut)
>
```

The ``#(module tut)`` tells you that the compilation was successful. If it said "error" instead, you have made some mistake in the text you entered and there will also be error messages to give you some idea as to what has gone wrong so you can change what you have written and try again.

Now lets run the program.

```lisp
> (tut:double 108)
216
>
```

As expected, ``108`` doubled is 216.

Now let's get back to the first two lines. LFE programs are written in files. Each file contains what we call an *LFE module*. The first line of code in the module tells LFE that we're defining a module and giving it a name:

```lisp
(defmodule tut
```
The name of our module is ``tut`` and the file which is used to store the module must have the same name as the module but with the ``.lfe`` extension. In our case the file name is ``tut.lfe``.

In LFE, whenever we use a function that has been defined in another module, we use the syntax, ``(module:function argument1 argument2 ...)``. So

```lisp
> (tut:double 108)
```

means "call the function ``double`` in the module ``tut`` with the argument of ``108``.

The second line tells LFE which functions we will be exporting -- in this case, all of them (which is only *one* ...):

```lisp
  (export all))
```

If we wanted to be explicit about which functions were to be exported, we would have written:

```lisp
(defmodule tut
  (export (double 1)))
```

That says "in the module ``tut``, please make available the function called ``double`` which takes one argument" (``x`` in our example). By "make available" we mean that this function can be called from outside the module ``tut``.

### A More Complicated Example

Now for a more complicated example, the factorial of a number (e.g. factorial of 4 is 4 * 3 * 2 * 1). Enter the following code in a file called ``tut1.lfe``.

```lisp
(defmodule tut1
  (export (fac 1)))

(defun fac
  ((1) 1)
  ((n) (* n (fac (- n 1)))))
```

Compile the file

```lisp
> (c "tut1.lfe")
#(module tut1)
```

And now calculate the factorial of 4.

```lisp
> (tut1:fac 4)
24
```

The function ``fac`` contains two parts. The first part:

```lisp
  ((1) 1)
```

says that the factorial of 1 is 1. Note that this part is a separate list in the function definition where the first element is a *list* of the arguments to the function and the rest is the body of the function. The second part:

```lisp
  ((n) (* n (fac (- n 1)))))
```

says that the factorial of n is n multiplied by the factorial of n - 1. After this part which is the last part we end the function definition with the closing ``)``.

A function can have many arguments. Let's expand the module ``tut1`` with the rather stupid function to multiply two numbers:

```lisp
(defmodule tut1
  (export (fac 1) (mult 2)))

(defun fac
  ((1) 1)
  ((n) (* n (fac (- n 1)))))

(defun mult (x y)
  (* x y))

```

Note that we have also had to expand the ``(export`` line with the information that there is another function ``mult`` with two arguments. Compile the file:

```lisp
> (c "tut1.lfe")
#(module tut1)
```
and try it out:

```lisp
> (tut1:mult 3 4)
12
```

In the example above the numbers are integers and the arguments in the functions in the code, ``n``, ``x``, ``y`` are called variables. Examples of variables could be ``number``, ``shoe-size``, ``age`` etc.

Note that when a function has only one part and all the arguments are variables then we can use the shorter form we saw in ``double`` and ``mult``. This means that we could also have written ``mult`` as:

```lisp
(defun mult
  ((x y) (* x y)))
```
