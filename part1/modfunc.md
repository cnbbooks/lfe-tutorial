## Modules and Functions

### Creating a Simple Module

A programming language isn't much use if you can only run code from a REPL. So next we will write a small LFE program in a file on the file system. In the same directory that you started the LFE REPL, reate a new file called ``tut.lfe`` (the filename is **important**: be sure you type it just as we have) using your favorite text editor.

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

