## Modules and Functions


## 3 Hitting the Code


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

