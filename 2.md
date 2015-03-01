# Creating a New Project


## 2 The LFE REPL

<img src="https://raw.github.com/lfe/docs/master/images/barf.jpg"
     style="float: left; padding-right: 1em;">Did you just say "REPL"? What's a "REPL"?

"REPL" is short for "read-eval-print loop", what other languages sometimes call the "interactive interpreter." Lisp had a REPL (and interpreter) back in the 60s, when the designers of many of today's languages were still in diapers (or hadn't yet arrived on-planet).

The REPL lets you type LFE code and execute it on the spot -- no files, instant feedback.

Wanna give it a try?

### 2.1 Starting the LFE REPL

You're going to like this next part -- super-easy, super-fast:

```bash
$ ./bin/lfe
Erlang/OTP 17 [erts-6.2] [source] [64-bit] [smp:8:8] [async...

LFE Shell V6.2 (abort with ^G)
>
```

### 2.2 Interactive LFE Code

Bingo! You're there. Let's have some fun. How about some math? Let's make LFE display The Answer:

```lisp
> (* 2 21)
42
```

Here's LFE's "Hello, World!":

```lisp
> (io:format "Hello, World!~n")
Hello, World!
ok
```
Here's how you set a variable in the REPL ... and call the ``seq`` function in the Erlang standard library's ``lists`` module:

```lisp
> (set my-list (lists:seq 1 6))
(1 2 3 4 5 6)
```

We just created a sequence of numbers and assigned it to the ``my-list`` variable. Let's sum that list using another function in the ``lists`` module:

```lisp
> (* 2 (lists:sum my-list))
42
```

Wanna get functional? Here's an anonymous function, an accumulator, and a left fold (known as a "reduce" in some languages):

```lisp
> (* 2 (lists:foldl (lambda (n acc) (+ n acc)) 0 my-list))
42
```

We can even define functions and macros in the REPL (which you can't actually do in the Erlang shell!). Let's turn the code above into an easy-to-use function:

```lisp
> (defun my-sum (start stop)
    (let ((my-list (lists:seq start stop)))
      (* 2 (lists:foldl
             (lambda (n acc)
               (+ n acc))
             0 my-list))))
my-sum
```

And try it out!

```lisp
> (my-sum 1 6)
42
```

We're going to skip macros for now ... ;-)


### Next Stop

You can taste it, can't you? That *LFE flavor* coming your way? Yup, you're
right. You're going to be looking at LFE modules next ...
