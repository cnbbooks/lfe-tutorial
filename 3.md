# LFE Code


## 3 Hitting the Code

<img src="https://raw.github.com/lfe/docs/master/images/smash.jpg"
     style="float: right; padding-left: 1em;">It may not seem like it,
but we're off to a pretty fast start. Time to put the brakes on, though, 'cause you're gonna want to see this next
part in slow motion.

We've just seen some fancy fingerwork in the REPL ... but what about some *real* code? What does a **project** look like? Well, if you want to see a full project, be sure to checkout the [lfetool Quick Start](). Creating a project by hand is too much for a bare-bones quick-start. But we can take a look at an LFE *module*. How does that sound?

### 3.1 Creating a Simple Module

If you want to create your module in the same terminal window, you'll need to quit the LFE REPL:

```lisp
(exit)
ok
>
```

```bash
$
```

Alternatively, you can open a new terminal window, creating your module there, and then come back to your LFE REPL.

In the same directory you started your REPL from, create the following file, named ``sample-module.lfe`` using your favorite editor:

```lisp
(defmodule sample-module
  (export all))

(defun my-sum (start stop)
    (let ((my-list (lists:seq start stop)))
      (* 2 (lists:foldl
             (lambda (n acc)
               (+ n acc))
             0 my-list))))
```

Hey, that function looks familiar! Save the file and exit your editor. If you exited the LFE REPL, go ahead and restart:

```bash
$ ./bin/lfe
```

### 3.2 Using Your Module

Back in your REPL, compile the file you just created:

```lisp
> (c "sample-module.lfe")
#(module sample-module)
```

That's the message that says "Everything compiled! You're ready to go plaid!"

As you noticed above, when we called functions from the Erlang standard library, we had to use the module name and then a colon before the actual function name. You've created a module and compiled it; to call the function in the module, you'll need to do the same thing:

```lisp
> (sample-module:my-sum 1 6)
42
> (sample-module:my-sum 1 60)
3660
> (sample-module:my-sum 1 600)
360600
> (sample-module:my-sum 1 6000)
36006000
```

### 3.3 Going Deep

Here's something a little more involved you may enjoy, from the examples in the
LFE source code:

```lisp
(defmodule messenger-back
 (export (print-result 0) (send-message 2)))

(defun print-result ()
  (receive
    ((tuple pid msg)
      (io:format "Received message: '~s'~n" (list msg))
      (io:format "Sending message to process ~p ...~n" (list pid))
      (! pid (tuple msg))
      (print-result))))

(defun send-message (calling-pid msg)
  (let ((spawned-pid (spawn 'messenger-back 'print-result ())))
    (! spawned-pid (tuple calling-pid msg))))
```

That bit of code demonstrates one of Erlang's core features in lovely Lisp
syntax: message passing. When loaded into the REPL, that example shows some
bidirectional message passing between the LFE shell and a spawned process.

Want to give it a try? Start by compiling the example module:

```lisp
> (c "examples/messenger-back.lfe")
#(module messenger-back)
```

Next, let's send two messages to another Erlang process (in this case, we'll
send it to our REPL process, ``(self)``:

```lisp
> (messenger-back:send-message (self) "And what does it say now?")
#(<0.26.0> "And what does it say now?")
Received message: 'And what does it say now?'
Sending message to process <0.26.0> ...
> (messenger-back:send-message (self) "Mostly harmless.")
#(<0.26.0> "Mostly harmless.")
Received message: 'Mostly harmless.'
Sending message to process <0.26.0> ...
```

In the above calls, for each message sent we got a reply acknowledging the
message (because the example was coded like that). But what about the receiver
itself? What did it, our REPL process, see? We can flush the message
queue in the REPL to find out:

```lisp
> (c:flush)
Shell got {"And what does it say now?"}
Shell got {"Mostly harmless."}
ok
```

If you found this last bit interesting and want to step through a tutorial on
Erlang's light-weight threads in more detail, you may enjoy
[this tutorial](http://docs.lfe.io/tutorials/processes/1.html).


### Next Stop

Next we'll see what other resources are available for learning LFE, wrapping up the quick start and pointing you in some directions for your next LFE adventures ...

