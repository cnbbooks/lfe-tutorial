## The LFE REPL

Most operating systems have a command interpreter or shell -- Unix and Linux have many, while Windows has the Command Prompt. Likewise, Erlang has a shell where you can directly write bits of Erlang code and evaluate (run) them to see what happens. LFE has more than a shell: it's a full REPL (*read-eval-print loop*) like other Lisps, and it can do more than the Erlang shell can (including defining proper functions and Lisp macros).

### Starting the LFE REPL

In your system terminal window where you changed directory to the clone of the LFE repository, you can start the LFE REPL by typing the following:

```bash
$ ./bin/lfe
```

At which point you will see output something like this:
```
Erlang/OTP 17 [erts-6.2] [source] [64-bit] [smp:8:8] ...

LFE Shell V6.2 (abort with ^G)
>
```

### Interactive LFE Code

Now let's multiply two numbers in the REPL by typing ``(* 2 21)`` at the ``> `` prompt:

```lisp
> (* 2 21)
```
Lisp stands for "LISt Processor" because nearly everything in Lisp is reall just a list of things -- including the code itself. The lists in Lisps are created with parentheses, just like the expression above. As you can see, the multiplication operator goes first, followed by its arguments -- this is called *prefix notation*, due to the operator coming first.

In order to tell the REPL that you want it to evaluate your LFE code, you need to his the ``<ENTER>`` or ``<RETURN>`` key.

```lisp
42
>
```

It has correctly given you the answer: 42. 


Now let's try a more complex calculation:

```lisp
> (* 2 (+ 1 2 3 4 5 6))
```
This expression has one nested inside the other. The first one to be executed is the inner-most, in this case, the addition operation. Just like we saw before, the operator comes first (the "addition" operator, in this case) and then all of the numbers to be added.

Hit ``<ENTER>`` to get your answer:

```lisp
42
> 
```

### Leaving the REPL

To exit the REPL and shutdown the underlying Erlang system which started when you executed ``./bin/lfe``, simply exit:

```lisp
> (exit)
```
```lisp
ok
```
At which point you will be presented with your regular system terminal prompt.

There are two other ways in which you may leave the REPL:
 * Hitting ``^c`` twice in a row, or
 * Hitting ``^g`` then ``q``