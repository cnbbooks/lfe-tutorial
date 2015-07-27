## Records

A record is defined as:

```lisp
(defrecord <name-of-record> <field-name-1> <field-name-2> ...)
```
The ``defrecord`` macro creates a number of new macro for creating, matching and accessing the fields of the record.

For example:

```lisp
(defrecord message-to to-name message)
```

The record data is a tuple which is exactly equivalent to:

```lisp
#(message-to <to-name> <message>)
```

Creating record is done with a ``make-<record-name>`` macro, and is best illustrated by an example:

```lisp
(make-message-to message "hello" to-name 'fred)
```

This will create the tuple:

```lisp
#(message-to fred "hello")
```

Note that you don't have to worry about the order you assign values to the various parts of the records when you create it. The advantage of using records is that by placing their definitions in header files you can conveniently define interfaces which are easy to change. For example, if you want to add a new field to the record, you will only have to change the code where the new field is used and not at every place the record is referred to. If you leave out a field when creating a record, it will get the value of the atom ``undefined``. (*manual*)

Pattern matching with records is very similar to creating records. For example inside a function clause, case or receive:

```lisp
(match-message-to to-name the-name message the-message)
```

will match a ``message-to`` record and extract the ``to-name`` field in to the variable ``the-name`` and the ``message`` field in to the variable ``the-message``. It is equivalent to writing:

```lisp
(tuple 'message-to the-name the-message)
```

Accessing the fields of a record is done through macros which are created when the record is defined. For example the variable ``my-message`` contains a ``message-to`` record then

```lisp
(message-to-message my-message)
```

will return the value of the ``message`` field of ``my-message`` and

```lisp
(set-message-to-message my-message "goodbye")
```

will return a new record where the ``message`` field now has the value ``"goodbye"``.
