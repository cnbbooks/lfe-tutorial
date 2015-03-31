## Conditionals

In the module ``tut13.lfe``, we saw our first conditional, the ``(if ...)`` form. We're going to spendthe rest of this section discussing ``if``, ``cond``, ``case``, as well as the use of guards and pattern matching to form conditional code branches.

### The ``if`` Form

In the previous section, we wrote the function ``find-max-min/3`` to work out the maximum and minimum temperature. This work was delegated to two helper functions:
 * ``compare-max/2``
 * ``compare-min/2``

In both of those functions, we introduced the new ``if`` form. If works as follows:

```lisp
(if <predicate>
    <expression 1>
    <expression 2>)
```

where ``<expression 1>`` is executed if ``<predicate>`` evaluates to ``true`` and ``<expression 2>`` is executed if ``<predicate>`` evaluates to ``false``. If you have used other programming languages, then this will be quite familiar to you. If you have not, if should remind you a bit of the logic we looked at when discussing guards.

We can see it in action with the following LFE session in the REPL:

```lisp
> (if (=:= 1 1) "They are equal!" "They are *not* equal!")
"They are equal!"
> (if (=:= 2 1) "They are equal!" "They are *not* equal!")
"They are *not* equal!"
```

Or, as you may be more familiar with this, our code from the last section:

```lisp
(if (< temp1 temp2)
    city1
    city2)
```
where, if ``temp1`` is less than ``temp2``, the value stored in ``city1`` is returned. 
What about the situations were we want to check for multiple conditions? For that, you'll need the ``cond`` form.

### The ``cond`` Form




### The ``case`` Form

### Guards and Patterns as Conditionals

### The Extended ``cond`` Form

The ``cond`` form has been extended with the extra test ``(?= <pattern> <expression>)`` which tests if the evaluated result of ``<expression>`` matches ``<pattern>``. If so, it binds the variables in ``<pattern>`` which can be used in the ``cond`` form. A optional guard is also allowed here. Here is an example of extended ``cond``:

```lisp
(cond ((foo x) ...)
      ((?= (cons x xs) (when (is_atom x)) (bar y))
       (fubar xs (baz x)))
      ((?= (tuple 'ok x) (baz y))
       (zipit x))
      ... )
```