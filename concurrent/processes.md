## Processes

One of the main reasons for using LFE/Erlang instead of other functional languages is Erlang/LFE's ability to handle concurrency and distributed programming. By concurrency we mean programs which can handle several threads of execution at the same time. For example, modern operating systems would allow you to use a word processor, a spreadsheet, a mail client and a print job all running at the same time. Of course each processor (CPU) in the system is probably only handling one thread (or job) at a time, but it swaps between the jobs a such a rate that it gives the illusion of running them all at the same time. It is easy to create parallel threads of execution in an LFE program and it is easy to allow these threads to communicate with each other. In LFE we call each thread of execution a *process*.

(Aside: the term "process" is usually used when the threads of execution share no data with each other and the term "thread" when they share data in some way. Threads of execution in LFE share no data, that's why we call them processes).

The LFE BIF ``spawn`` is used to create a new process: ``(spawn module exported-function list-of-arguments)``. Consider the following module:

```lisp
(defmodule tut14
  (export (start 0) (say-something 2)))

(defun say-something
  ([what 0] 'done)
  ([what times]
   (lfe_io:format "~p~n" (list what))
   (say-something what (- times 1))))

(defun start ()
  (spawn 'tut14 'say-something '(hello 3))
  (spawn 'tut14 'say-something '(goodbye 3)))
```

```lisp
> (c "tut14.lfe")
#(module tut14)
> (tut14:say-something 'hello 3)
hello
hello
hello
done
```

We can see that function ``say-something`` writes its first argument the number of times specified by second argument. Now look at the function ``start``. It starts two LFE processes, one which writes "hello" three times and one which writes "goodbye" three times. Both of these processes use the function ``say-something``. Note that a function used in this way by ``spawn`` to start a process must be exported from the module (i.e. in the (export ... ) at the start of the module).

```lisp
> (tut14:start)
<0.37.0>
hello
goodbye
> hello
goodbye
hello
goodbye
```

Notice that it didn't write "hello" three times and then "goodbye" three times, but the first process wrote a "hello", the second a "goodbye", the first another "hello" and so forth. But where did the <0.37.0> come from? The return value of a function is of course the return value of the last "thing" in the function. The last thing in the function ``start`` is

```lisp
(spawn 'tut14 'say-something '(goodbye 3))
```

``spawn`` returns a *process identifier*, or *pid*, which uniquely identifies the process. So <0.37.0> is the pid of the spawn function call above. We will see how to use pids in the next example.

Note as well that we have used ~p instead of ~w in ``lfe_io:format``. To quote the manual: "~p Writes the data with standard syntax in the same way as ~w, but breaks terms whose printed representation is longer than one line into many lines and indents each line sensibly. It also tries to detect lists of printable characters and to output these as strings".
