## Timeouts

Before improving the messenger program we will look into some general principles, using the ping pong program as an example. Recall that when "ping" finishes, it tells "pong" that it has done so by sending the atom ``finished`` as a message to "pong" so that "pong" could also finish. Another way to let "pong" finish, is to make "pong" exit if it does not receive a message from ping within a certain time, this can be done by adding a *timeout* to pong as shown in the following example:

```lisp
(defmodule tut19
  (export (start-ping 1) (start-pong 0) (ping 2) (pong 0)))

(defun ping
  ((0 pong-node)
   (lfe_io:format "Ping finished~n" ()))
  ((n pong-node)
   (! (tuple 'pong pong-node) (tuple 'ping (self)))
   (receive
     ('pong (lfe_io:format "Ping received pong~n" ())))
   (ping (- n 1) pong-node)))

(defun pong ()
  (receive
    ((tuple 'ping ping-pid)
     (lfe_io:format "Pong received ping~n" ())
     (! ping-pid 'pong)
     (pong))
    (after 5000
      (lfe_io:format "Pong timed out~n" ()))))

(defun start-pong ()
  (register 'pong (spawn 'tut19 'pong ())))

(defun start-ping (pong-node)
  (spawn 'tut19 'ping (list 3 pong-node)))
```

After we have compiled this and copied the ``tut19.beam`` file to the necessary directories:

On (pong@kosken):

```
(pong@kosken)> (tut19:start-pong)
true
Pong received ping
Pong received ping
Pong received ping
Pong timed out
```

On (ping@gollum):

```
(ping@renat)> (tut19:start-ping 'pong@kosken)
<0.40.0>
Ping received pong
Ping received pong
Ping received pong
Ping finished
```

The timeout is set in:

```lisp
(defun pong ()
  (receive
    ((tuple 'ping ping-pid)
     (lfe_io:format "Pong received ping~n" ())
     (! ping-pid 'pong)
     (pong))
    (after 5000
      (lfe_io:format "Pong timed out~n" ()))))
```

We start the timeout ``(after 5000)`` when we enter ``receive``. The timeout is canceled if ``#(ping ping-pid)`` is received. If ``#(ping ping-pid)`` is not received, the actions following the timeout will be done after 5000 milliseconds. ``after`` must be last in the ``receive``, i.e. preceded by all other message reception specifications in the ``receive``. Of course we could also call a function which returned an integer for the timeout:

```lisp
(after (pong-timeout) 
```

In general, there are better ways than using timeouts to supervise parts of a distributed Erlang system. Timeouts are usually appropriate to supervise external events, for example if you have expected a message from some external system within a specified time. For example, we could use a timeout to log a user out of the messenger system if they have not accessed it, for example, in ten minutes.
