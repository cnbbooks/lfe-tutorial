## Error Handling

Before we go into details of the supervision and error handling in an LFE system, we need see how LFE processes terminate, or in LFE terminology, *exit*.

A process which executes ``(exit 'normal)`` or simply runs out of things to do has a *normal* exit.

A process which encounters a runtime error (e.g. divide by zero, bad match, trying to call a function which doesn't exist etc) exits with an error, i.e. has an *abnormal* exit. A process which executes ``(exit reason)`` where ``reason`` is any LFE term except the atom ``normal``, also has an abnormal exit.

An LFE process can set up links to other LFE processes. If a process calls ``(link other-pid)`` it sets up a bidirectional link between itself and the process called ``other-pid``. When a process terminates, it sends something called a *signal* to all the processes it has links to.

The signal carries information about the pid it was sent from and the exit reason.

The default behaviour of a process which receives a normal exit is to ignore the signal.

The default behaviour in the two other cases (i.e. abnormal exit) above is to bypass all messages to the receiving process and to kill it and to propagate the same error signal to the killed process' links. In this way you can connect all processes in a transaction together using links and if one of the processes exits abnormally, all the processes in the transaction will be killed. As we often want to create a process and link to it at the same time, there is a special BIF, ``spawn_link`` which does the same as spawn, but also creates a link to the spawned process.

Now an example of the ping pong example using links to terminate "pong":

```lisp
(defmodule tut20
  (export (start 1) (ping 2) (pong 0)))

(defun ping (n pong-pid)
  (link pong-pid)
  (ping1 n pong-pid))

(defun ping1
  ((0 pong-pid)
   (exit 'ping))
  ((n pong-pid)
   (! pong-pid (tuple 'ping (self)))
   (receive
     ('pong (lfe_io:format "Ping received pong~n" ())))
   (ping1 (- n 1) pong-pid)))

(defun pong ()
  (receive
    ((tuple 'ping ping-pid)
     (lfe_io:format "Pong received ping~n" ())
     (! ping-pid 'pong)
     (pong))))

(defun start (ping-node)
  (let ((pong-pid (spawn 'tut20 'pong ())))
    (spawn ping-node 'tut20 'ping (list 3 pong-pid))))
```

```lisp
(s1@bill)lfe> (tut20:start 's2@kosken)
Pong received ping
<5627.43.0>
Ping received pong
Pong received ping
Ping received pong
Pong received ping
Ping received pong
```

This is a slight modification of the ping pong program where both processes are spawned from the same ``start/1`` function, where the "ping" process can be spawned on a separate node. Note the use of the ``link`` BIF. "Ping" calls ``(exit `ping)`` when it finishes and this will cause an exit signal to be sent to "pong" which will also terminate.

It is possible to modify the default behaviour of a process so that it does not get killed when it receives abnormal exit signals, but all signals will be turned into normal messages on the format ``#(EXIT from-pid reason)`` and added to the end of the receiving processes message queue. This behaviour is set by:

```lisp
(process_flag 'trap_exit 'true)
```

There are several other process flags, see *erlang manual*. Changing the default behaviour of a process in this way is usually not done in standard user programs, but is left to the supervisory programs in OTP (but that's another tutorial). However we will modify the ping pong program to illustrate exit trapping.

```lisp
(defmodule tut21
  (export (start 1) (ping 2) (pong 0)))

(defun ping (n pong-pid)
  (link pong-pid)
  (ping1 n pong-pid))

(defun ping1
  ((0 pong-pid)
   (exit 'ping))
  ((n pong-pid)
   (! pong-pid (tuple 'ping (self)))
   (receive
     ('pong (lfe_io:format "Ping received pong~n" ())))
   (ping1 (- n 1) pong-pid)))

(defun pong ()
  (process_flag 'trap_exit 'true)
  (pong1))

(defun pong1 ()
  (receive
    ((tuple 'ping ping-pid)
     (lfe_io:format "Pong received ping~n" ())
     (! ping-pid 'pong)
     (pong1))
    ((tuple 'EXIT from reason)
     (lfe_io:format "Pong exiting, got ~p~n"
		    (list (tuple 'EXIT from reason))))))

(defun start (ping-node)
  (let ((pong-pid (spawn 'tut21 'pong ())))
    (spawn ping-node 'tut21 'ping (list 3 pong-pid))))
```

```lisp
(s1@bill)lfe> (tut21:start 's2@kosken)
<5627.44.0>
Pong received ping
Ping received pong
Pong received ping
Ping received pong
Pong received ping
Ping received pong
Pong exiting, got #(EXIT <5627.44.0> ping)
```
