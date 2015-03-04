## Message Passing

In the following example we create two processes which send messages to each other a number of times.

```lisp
(defmodule tut15
  (export (start 0) (ping 2) (pong 0)))

(defun ping
  ((0 pong-pid)
   (! pong-pid 'finished)
   (lfe_io:format "ping finished~n" ()))
  ((n pong-pid)
   (! pong-pid (tuple 'ping (self)))
   (receive
     ('pong (lfe_io:format "Ping received pong~n" ())))
   (ping (- n 1) pong-pid)))

(defun pong ()
  (receive
    ('finished 
     (lfe_io:format "Pong finished~n" ()))
    ((tuple 'ping ping-pid)
     (lfe_io:format "Pong received ping~n" ())
     (! ping-pid 'pong)
     (pong))))

(defun start ()
  (let ((pong-pid (spawn 'tut15 'pong ())))
    (spawn 'tut15 'ping (list 3 pong-pid))))
```

```lisp
> (c "tut15.lfe")
#(module tut15)
> (tut15:start)
<0.36.0>
> Pong received ping
Ping received pong
Pong received ping
Ping received pong
Pong received ping
Ping received pong
ping finished
Pong finished
```

The function ``start`` first creates a process, let's call it "pong":

```lisp
(let ((pong-pid (spawn 'tut15 'pong ())))
```

This process executes ``(tut15:pong)``. ``pong-pid`` is the process identity of the "pong" process. The function ``start`` now creates another process "ping".

```lisp
(spawn 'tut15 'ping (list 3 pong-pid))))
```

this process executes:

```lisp
(tut15:ping (list 3 pong-pid))
```

<0.36.0> is the return value from the ``start`` function.

The process "pong" now does:

```lisp
(receive
  ('finished 
   (lfe_io:format "Pong finished~n" ()))
  ((tuple 'ping ping-pid)
   (lfe_io:format "Pong received ping~n" ())
   (! ping-pid 'pong)
   (pong)))
```

The ``receive`` construct is used to allow processes to wait for messages from other processes. It has the format:

```lisp
(receive
  (pattern1
   actions1)
  (pattern2
   actions2)
  ....
  (patternN
   actionsN))
```

Messages between LFE processes are simply valid LFE terms. I.e. they can be lists, tuples, integers, atoms, pids etc.

Each process has its own input queue for messages it receives. New messages received are put at the end of the queue. When a process executes a ``receive``, the first message in the queue is matched against the first pattern in the ``receive``, if this matches, the message is removed from the queue and the actions corresponding to the the pattern are executed.

However, if the first pattern does not match, the second pattern is tested, if this matches the message is removed from the queue and the actions corresponding to the second pattern are executed. If the second pattern does not match the third is tried and so on until there are no more pattern to test. If there are no more patterns to test, the first message is kept in the queue and we try the second message instead. If this matches any pattern, the appropriate actions are executed and the second message is removed from the queue (keeping the first message and any other messages in the queue). If the second message does not match we try the third message and so on until we reach the end of the queue. If we reach the end of the queue, the process blocks (stops execution) and waits until a new message is received and this procedure is repeated.

Of course the LFE implementation is "clever" and minimizes the number of times each message is tested against the patterns in each ``receive``.

Now back to the ping pong example.

"Pong" is waiting for messages. If the atom ``finished`` is received, "pong" writes "Pong finished" to the output and as it has nothing more to do, terminates. If it receives a message with the format:

```lisp
#(ping ping-pid)
```

