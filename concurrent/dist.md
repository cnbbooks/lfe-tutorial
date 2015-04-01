## Distributed Programming

Now let's re-write the ping pong program with "ping" and "pong" on different computers. Before we do this, there are a few things we need to set up to get this to work. The distributed LFE/Erlang implementation provides a basic security mechanism to prevent unauthorized access to an Erlang system on another computer (*manual*). Erlang systems which talk to each other must have the same *magic cookie*. The easiest way to achieve this is by having a file called ``.erlang.cookie`` in your home directory on all machines which on which you are going to run Erlang systems communicating with each other (on Windows systems the home directory is the directory where pointed to by the $HOME environment variable - you may need to set this. On Linux or Unix you can safely ignore this and simply create a file called ``.erlang.cookie`` in the directory you get to after executing the command ``cd`` without any argument). The ``.erlang.cookie`` files should contain one line with the same atom. For example on Linux or Unix in the OS shell:

```
$ cd
$ cat > .erlang.cookie
this_is_very_secret
$ chmod 400 .erlang.cookie
```

The chmod above make the ``.erlang.cookie`` file accessible only by the owner of the file. This is a requirement.

When you start an LFE/Erlang system which is going to talk to other LFE/Erlang systems, you must give it a name, eg:

```
$ lfe -sname my-name
```

We will see more details of this later (*manual*). If you want to experiment with distributed Erlang, but you only have one computer to work on, you can start two separate Erlang systems on the same computer but give them different names. Each Erlang system running on a computer is called an Erlang node.

(Note: ``erl -sname`` assumes that all nodes are in the same IP domain and we can use only the first component of the IP address, if we want to use nodes in different domains we use ``-name`` instead, but then all IP address must be given in full (*manual*).

Here is the ping pong example modified to run on two separate nodes:

```lisp
(defmodule tut21
  (export (start-ping 1) (start-pong 0) (ping 2) (pong 0)))

(defun ping
  ((0 pong-node)
   (! (tuple 'pong pong-node) 'finished)
   (lfe_io:format "Ping finished~n" ()))
  ((n pong-node)
   (! (tuple 'pong pong-node) (tuple 'ping (self)))
   (receive
     ('pong (lfe_io:format "Ping received pong~n" ())))
   (ping (- n 1) pong-node)))

(defun pong ()
  (receive
    ('finished
     (lfe_io:format "Pong finished~n" ()))
    ((tuple 'ping ping-pid)
     (lfe_io:format "Pong received ping~n" ())
     (! ping-pid 'pong)
     (pong))))

(defun start-pong ()
  (register 'pong (spawn 'tut21 'pong ())))

(defun start-ping (pong-node)
  (spawn 'tut21 'ping (list 3 pong-node)))
```

Let us assume we have two computers called gollum and kosken. We will start a node on kosken called ping and then a node on gollum called pong.

On kosken (on a Linux/Unix system):

```
$ lfe -sname ping
Erlang/OTP 17 [erts-6.0] [source-07b8f44] [64-bit] [smp:8:8] [async-threads:10] [hipe] [kernel-poll:false] [dtrace]

LFE Shell V6.0 (abort with ^G)
(ping@kosken)>
```

On gollum:

```
$ lfe -sname pong
Erlang/OTP 17 [erts-6.0] [source-07b8f44] [64-bit] [smp:8:8] [async-threads:10] [hipe] [kernel-poll:false] [dtrace]

LFE Shell V6.0 (abort with ^G)
(pong@gollum)>
```

Now start the "pong" process on gollum:

```
(pong@gollum)> (tut21:start-pong)
true
```

and start the "ping" process on kosken (from the code above you will see that a parameter of the ``start-ping`` function is the node name of the Erlang system where "pong" is running):

```
(ping@kosken)> (tut21:start-ping 'pong@gollum)
<0.41.0>
(ping@kosken)> Ping received pong
Ping received pong
Ping received pong
Ping finished
```

Here we see that the ping pong program has run, on the "pong" side we see:

```
(pong@gollum)>
Pong received ping
Pong received ping
Pong received ping
Pong finished
```

Looking at the ``tut21`` code we see that the ``pong`` function itself is unchanged, the lines:

```lisp
    ((tuple 'ping ping-pid)
     (lfe_io:format "Pong received ping~n" ())
     (! ping-pid 'pong)
```

work in the same way irrespective of on which node the "ping" process is executing. Thus Erlang pids contain information about where the process executes so if you know the pid of a process, the "!" operator can be used to send it a message if the process is on the same node or on a different node.

A difference is how we send messages to a registered process on another node:

```lisp
   (! (tuple 'pong pong-node) (tuple 'ping (self)))
```

We use a tuple ``#(registered-name node-name)`` instead of just the ``registered-name``.

In the previous example, we started "ping" and "pong" from the shells of two separate Erlang nodes. ``spawn`` can also be used to start processes in other nodes. The next example is the ping pong program, yet again, but this time we will start "ping" in another node:

```lisp
(defmodule tut22
  (export (start 1) (ping 2) (pong 0)))

(defun ping
  ((0 pong-node)
   (! (tuple 'pong pong-node) 'finished)
   (lfe_io:format "Ping finished~n" ()))
  ((n pong-node)
   (! (tuple 'pong pong-node) (tuple 'ping (self)))
   (receive
     ('pong (lfe_io:format "Ping received pong~n" ())))
   (ping (- n 1) pong-node)))

(defun pong ()
  (receive
    ('finished
     (lfe_io:format "Pong finished~n" ()))
    ((tuple 'ping ping-pid)
     (lfe_io:format "Pong received ping~n" ())
     (! ping-pid 'pong)
     (pong))))

(defun start (ping-node)
  (register 'pong (spawn 'tut22 'pong ()))
  (spawn ping-node 'tut22 'ping (list 3 (node))))
```

The function ``node/0`` returns the name of the current node.

Assuming an LFE system called ``ping`` (but not the "ping" process) has already been started on kosken, then on gollum we do:

```
(pong@gollum)> (tut22:start 'ping@kosken)
<8524.50.0>
(pong@gollum)> Pong received ping
Ping received pong
Pong received ping
Ping received pong
Pong received ping
Ping received pong
Ping finished
Pong finished
```

Notice we get all the output on gollum. This is because the io system finds out where the process is spawned from and sends all output there.

Although we do not show here there are no problems with running nodes distributed nodes where some run Erlang and some run LFE. The handling of io will also work the same across mixed LFE and Erlang nodes.
