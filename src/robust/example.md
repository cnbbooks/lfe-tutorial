## Example: Robust Messenger

Now we return to the messenger program and add changes which make it more robust:

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
;;; When the client terminates for some reason
;;; To server: (tuple 'EXIT client-pid reason)
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
  (export (start-server 0) (server 0)
          (logon 1) (logoff 0) (message 2) (client 2)))

;;; Change the function below to return the name of the node where the
;;; messenger server runs

(defun server-node () 'messenger@renat)

;;; This is the server process for the "messenger"
;;; the user list has the format [{ClientPid1, Name1},{ClientPid22, Name2},...]

(defun server ()
  (process_flag 'trap_exit 'true)
  (server ()))

(defun server (user-list)
  (receive
    ((tuple from 'logon name)
     (let ((new-user-list (server-logon from name user-list)))
       (server new-user-list)))
    ((tuple 'EXIT from _)
     (let ((new-user-list (server-logoff from user-list)))
       (server new-user-list)))
    ((tuple from 'message-to to message)
     (server-transfer from to message user-list)
     (lfe_io:format "list is now: ~p~n" (list user-list))
     (server user-list))))

;;; Start the server

(defun start-server ()
  (register 'messenger (spawn 'messenger 'server '())))

;;; Server adds a new user to the user list

(defun server-logon (from name user-list)
  ;; Check if logged on anywhere else
  (if (lists:keymember name 2 user-list)
    (progn                              ;Reject logon
      (! from #(messenger stop user-exists-at-other-node))
      user-list)
    (progn                              ;Add user to the list
      (! from #(messenger logged-on))
      (link from)
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
     (lfe_io:format "~p~n" (list what)))
    (after 5000
      (lfe_io:format "No response from server~n" ())
      (exit 'timeout))))
```

We have added the following changes:

The messenger server traps exits. If it receives an exit signal, ``#(EXIT from reason)`` this means that a client process has terminated or is unreachable because:

- the user has logged off (we have removed the "logoff" message),
- the network connection to the client is broken,
- the node on which the client process resides has gone down, or
- the client processes has done some illegal operation.

If we receive an exit signal as above, we delete the tuple, ``#(from name)`` from the servers ``user-list`` using the ``server-logoff`` function. If the node on which the server runs goes down, an exit signal (automatically generated by the system), will be sent to all of the client processes: ``#(EXIT messenger-pid noconnection)`` causing all the client processes to terminate.

We have also introduced a timeout of five seconds in the ``await-result`` function. I.e. if the server does not reply within five seconds (5000 ms), the client terminates. This is really only needed in the logon sequence before the client and server are linked.

An interesting case is if the client was to terminate before the server links to it. This is taken care of because linking to a non-existent process causes an exit signal, ``#(EXIT from noproc)``, to be automatically generated as if the process terminated immediately after the link operation.
