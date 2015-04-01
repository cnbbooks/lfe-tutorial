## Registered Process Names

In the above example, we first created "pong" so as to be able to give the identity of "pong" when we started "ping". I.e. in some way "ping" must be able to know the identity of "pong" in order to be able to send a message to it. Sometimes processes which need to know each others identities are started completely independently of each other. Erlang thus provides a mechanism for processes to be given names so that these names can be used as identities instead of pids. This is done by using the ``register`` BIF:

```lisp
(register some-atom pid)
```

We will now re-write the ping pong example using this and giving the name ``pong`` to the "pong" process:

```lisp
(defmodule tut20
  (export (start 0) (ping 1) (pong 0)))

(defun ping
  ((0)
   (! 'pong 'finished)
   (lfe_io:format "Ping finished~n" ()))
  ((n)
   (! 'pong (tuple 'ping (self)))
   (receive
     ('pong (lfe_io:format "Ping received pong~n" ())))
   (ping (- n 1))))

(defun pong ()
  (receive
    ('finished 
     (lfe_io:format "Pong finished~n" ()))
    ((tuple 'ping ping-pid)
     (lfe_io:format "Pong received ping~n" ())
     (! ping-pid 'pong)
     (pong))))

(defun start ()
  (let ((pong-pid (spawn 'tut20 'pong ())))
    (register 'pong pong-pid)
    (spawn 'tut20 'ping '(3))))
```

```lisp
> (c "tut20")
#(module tut20)
> (tut20:start)
<0.36.0>
> Pong received ping
Ping received pong
Pong received ping
Ping received pong
Pong received ping
Ping received pong
Ping finished
Pong finished
```

In the ``start/0`` function,

```lisp
(register 'pong pong-pid)
```

registers the "pong" process and gives it the name ``pong``. In the "ping" process we can now send messages to ``pong`` by:

```lisp
(! 'pong (tuple 'ping (self)))
```

so that ``ping/2`` now becomes ``ping/1`` as we don't have to use the argument ``pong-pid``.
