## Example: Messenger

Now for a larger example. We will make an extremely simple "messenger". The messenger is a program which allows users to log in on different nodes and send simple messages to each other.

Before we start, let's note the following:

- This example will just show the message passing logic no attempt at all has been made to provide a nice graphical user interface - this can of course also be done in LFE - but that's another tutorial.

- This sort of problem can be solved more easily if you use the facilities in OTP, which will also provide methods for updating code on the fly etc. But again, that's another tutorial.

- The first program we write will contain some inadequacies as regards handling of nodes which disappear, we will correct these in a later version of the program.

We will set up the messenger by allowing "clients" to connect to a central server and say who and where they are. I.e. a user won't need to know the name of the Erlang node where another user is located to send a message.

File `messenger.lfe`:

```lisp
;;; Message passing utility.
;;; User interface:
;;; (logon name)
;;;     One user at a time can log in from each Erlang node in the
;;;     system messenger: and choose a suitable name. If the name
;;;     is already logged in at another node or if someone else is
;;;     already logged in at the same node, login will be rejected
;;;     with a suitable error message.
;;; (logoff)
;;;     Logs off anybody at at node
;;; (message to-name message)
;;;     sends message to to-name. Error messages if the user of this
;;;     function is not logged on or if to-name is not logged on at
;;;     any node.
;;;
;;; One node in the network of Erlang nodes runs a server which maintains
;;; data about the logged on users. The server is registered as "messenger"
;;; Each node where there is a user logged on runs a client process registered
;;; as "mess-client"
;;;
;;; Protocol between the client processes and the server
;;; ----------------------------------------------------
;;;
;;; To server: (tuple client-pid 'logon user-name)
;;; Reply #(messenger stop user-exists-at-other-node) stops the client
;;; Reply #(messenger logged-on) logon was successful
;;;
;;; To server: (tuple client-pid 'logoff)
;;; Reply: #(messenger logged-off)
;;;
;;; To server: (tuple client-pid 'logoff)
;;; Reply: no reply
;;;
;;; To server: (tuple client-pid 'message-to to-name message) send a message
;;; Reply: #(messenger stop you-are-not-logged-on) stops the client
;;; Reply: #(messenger receiver-not-found) no user with this name logged on
;;; Reply: #(messenger sent) message has been sent (but no guarantee)
;;;
;;; To client: (tuple 'message-from name message)
;;;
;;; Protocol between the "commands" and the client
;;; ----------------------------------------------
;;;
;;; Started: (messenger:client server-node name)
;;; To client: logoff
;;; To client: (tuple 'message-to to-name message)
;;;
;;; Configuration: change the server-node() function to return the
;;; name of the node where the messenger server runs

(defmodule messenger
  (export (start-server 0) (server 1) (logon 1) (logoff 0)
          (message 2) (client 2)))

;;; Change the function below to return the name of the node where the
;;; messenger server runs

(defun server-node () 'messenger@renat)

;;; This is the server process for the "messenger"
;;; the user list has the format [{ClientPid1, Name1},{ClientPid22, Name2},...]

(defun server (user-list)
  (receive
    ((tuple from 'logon name)
     (let ((new-user-list (server-logon from name user-list)))
       (server new-user-list)))
    ((tuple from 'logoff)
     (let ((new-user-list (server-logoff from user-list)))
       (server new-user-list)))
    ((tuple from 'message-to to message)
     (server-transfer from to message user-list)
     ;;(lfe_io:format "list is now: ~p~n" (list user-list))
     (server user-list))))

;;; Start the server

(defun start-server ()
  (register 'messenger (spawn 'messenger 'server '(()))))

;;; Server adds a new user to the user list

(defun server-logon (from name user-list)
  ;; Check if logged on anywhere else
  (if (lists:keymember name 2 user-list)
    (progn                              ;Reject logon
      (! from #(messenger stop user-exists-at-other-node))
      user-list)
    (progn                              ;Add user to the list
      (! from #(messenger logged-on))
      (cons (tuple from name) user-list))))

;;; Server deletes a user from the user list

(defun server-logoff (pid user-list)
  (lists:keydelete pid 1 user-list))

;;; Server transfers a message between user

(defun server-transfer (from-pid to-name message user-list)
  ;; Check that the user is logged on and who he is
  (case (lists:keyfind from-pid 1 user-list)
    ((tuple from-pid from-name)
     (server-transfer from-pid from-name to-name message user-list))
    ('false
     (! from-pid #(messenger stop you-are-not-logged-on)))))

;;; If the user exists, send the message

(defun server-transfer (from-pid from-name to-name message user-list)
  ;; Find the receiver and send the message
  (case (lists:keyfind to-name 2 user-list)
    ((tuple to-pid to-name)
     (! to-pid (tuple 'message-from from-name message))
     (! from-pid #(messenger sent)))
    ('false
     (! from-pid #(messenger receiver-not-found)))))

;;; User Commands

(defun logon (name)
  (case (whereis 'mess-client)
    ('undefined
     (let ((client (spawn 'messenger 'client (list (server-node) name))))
       (register 'mess-client client)))
    (_ 'already-logged-on)))

(defun logoff ()
  (! 'mess-client 'logoff))

(defun message (to-name message)
  (case (whereis 'mess-client)          ;Test if the client is running
    ('undefined
     'not-logged-on)
    (_  (! 'mess-client (tuple 'message-to to-name message))
        'ok)))

;;; The client process which runs on each server node

(defun client (server-node name)
  (! (tuple 'messenger server-node) (tuple (self) 'logon name))
  (await-result)
  (client server-node))

(defun client (server-node)
  (receive
    ('logoff
     (! (tuple 'messenger server-node) (tuple (self) 'logoff))
     (exit 'normal))
    ((tuple 'message-to to-name message)
     (! (tuple 'messenger server-node)
        (tuple (self) 'message-to to-name message))
     (await-result))
    ((tuple 'message-from from-name message)
     (lfe_io:format "Message from ~p: ~p~n" (list from-name message))))
  (client server-node))

;;; Wait for a response from the server

(defun await-result ()
  (receive
    ((tuple 'messenger 'stop why)       ;Stop the client
     (lfe_io:format "~p~n" (list why))
     (exit 'normal))
    ((tuple 'messenger what)            ;Normal response
     (lfe_io:format "~p~n" (list what)))))
```

To use this program you need to:

- configure the server_node() function
- copy the compiled code (`essenger.beam` to the directory on each computer where you start Erlang.

In the following example of use of this program, I have started nodes on four different computers, but if you don't have that many machines available on your network, you could start up several nodes on the same machine.

We start up four Erlang nodes, messenger@super, c1@bilbo, c2@kosken, c3@gollum.

First we start up a the server at messenger@super:

```lisp
(messenger@super)lfe> (messenger:start-server)
true
```

Now Peter logs on at c1@bilbo:

```lisp
(c1@bilbo)lfe> (messenger:logon 'peter)
true
logged-on
```

James logs on at c2@kosken:

```lisp
(c2@kosken)lfe> (messenger:logon 'james)
true
logged-on
```

and Fred logs on at c3@gollum:

```lisp
(c3@gollum)lfe> (messenger:logon 'fred)
true
logged-on
```

Now Peter sends Fred a message:

```lisp
(c1@bilbo)lfe> (messenger:message 'fred "hello")
ok
sent
```

And Fred receives the message and sends a message to Peter and logs off:

```lisp
Message from peter: "hello"
(c3@gollum)lfe> (messenger:message 'peter "go away, I'm busy")
ok
sent
(c3@gollum)lfe> (messenger:logoff)
logoff
```

James now tries to send a message to Fred:

```lisp
(c2@kosken)lfe> (messenger:message 'fred "peter doesn't like you")
ok
receiver-not-found
```

But this fails as Fred has already logged off.

First let's look at some of the new concepts we have introduced.

There are two versions of the `server-transfer` function, one with four arguments (`server-transfer/4`) and one with five (`server_transfer/5`). These are regarded by LFE as two separate functions.

Note how we write the `server` function so that it calls itself, `(server user-list)` and thus creates a loop. The Erlang-LFE compiler is "clever" and optimises the code so that this really is a sort of loop and not a proper function call. But this only works if there is no code after the call, otherwise the compiler will expect the call to return and make a proper function call. This would result in the process getting bigger and bigger for every loop.

We use functions in the `lists` module. This is a very useful module and a study of the manual page is recommended (erl -man lists). `(lists:keymember key position lists)` looks through a list of tuples and looks at `position` in each tuple to see if it is the same as `key`. The first element is position 1. If it finds a tuple where the element at `position` is the same as `key`, it returns `true`, otherwise `false`.

```lisp
> (lists:keymember 'a 2 '(#(x y z) #(b b b) #(b a c) #(q r s)))
true
> (lists:keymember 'p 2 '(#(x y z) #(b b b) #(b a c) #(q r s)))
false
```

`lists:keydelete` works in the same way but deletes the first tuple found (if any) and returns the remaining list:

```lisp
> (lists:keymember 'a 2 '(#(x y z) #(b b b) #(b a c) #(q r s)))
(#(x y z) #(b b b) #(q r s))
```

`lists:keyfind` is like `lists:keymember`, but it returns the tuple found or the atom `false`:

```lisp
> (lists:keyfind 'a 2 '(#(x y z) #(b b b) #(b a c) #(q r s)))
#(b a c)
> (lists:keyfind 'p 2 '(#(x y z) #(b b b) #(b a c) #(q r s)))
false
```

There are a lot more very useful functions in the `lists` module.

An LFE process will (conceptually) run until it does a `receive` and there is no message which it wants to receive in the message queue. I say "conceptually" because the LFE system shares the CPU time between the active processes in the system.

A process terminates when there is nothing more for it to do, i.e. the last function it calls simply returns and doesn't call another function. Another way for a process to terminate is for it to call `exit/1`. The argument to `exit/1` has a special meaning which we will look at later. In this example we will do `(exit 'normal)` which has the same effect as a process running out of functions to call.

The BIF `(whereis registered-name)` checks if a registered process of name `registered-name` exists and return the pid of the process if it does exist or the atom `undefined` if it does not.

You should by now be able to understand most of the code above so I'll just go through one case: a message is sent from one user to another.

The first user "sends" the message in the example above by:

```lisp
(messenger:message 'fred "hello")
```

After testing that the client process exists:

```lisp
(whereis 'mess-client)
```

and a message is sent to `mess-client`:

```lisp
(! 'mess-client #(message-to fred "hello"))
```

The client sends the message to the server by:

```lisp
(! #(messenger messenger@renat) (tuple (self) 'message-to 'fred "hello"))
```

and waits for a reply from the server.

The server receives this message and calls:

```lisp
(server-transfer from 'fred "hello" user-list)
```

which checks that the pid `from` is in the `user-list`:

```lisp
(lists:keyfind from 1 user-list)
```

If `keyfind` returns the atom `false`, some sort of error has occurred and the server sends back the message:

```lisp
(! from-pid #(messenger stop you-are-not-logged-on))
```

which is received by the client which in turn does `(exit 'normal)` and terminates. If `keyfind` returns `(tuple from name)` we know that the user is logged on and is his name (`peter`) is in variable `name`. We now call:

```lisp
(server-transfer from 'peter 'fred "hello" user-list)
```

Note that as this is `server-transfer/5` it is not the same as the previous function `server_transfer/4`. We do another `keyfind` on `user-list` to find the pid of the client corresponding to `fred`:

```lisp
(lists:keyfind 'fred 2 user-list)
```

This time we use argument 2 which is the second element in the tuple. If this returns the atom `false` we know that fred is not logged on and we send the message:

```lisp
(! from-pid #(messenger receiver-not-found)
```

which is received by the client, if `keyfind` returns:

```lisp
(tuple to-pid 'fred)
```

we send the message:

```lisp
(! to-pid #(message-from peter "hello"))
```

to Fred's client and the message:

```lisp
(! from-pid #(messenger sent))
```

to peter's client.

Fred's client receives the message and prints it:

```lisp
((tuple 'message-from from-name message)
 (lfe_io:format "Message from ~p: ~p~n" (list from-name message))))
```

and Peter's client receives the message in the `await-result` function.
