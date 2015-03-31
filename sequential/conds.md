## Conditionals

In the module ``tut13.lfe``, we saw our first conditional, the ``(if ...)`` form. We're going to spendthe rest of this section discussing ``if``, ``cond``, ``case``, as well as the use of guards and pattern matching to form conditional code branches.

### The ``if`` Form

In the previous section, we wrote the function ``find-max-min/3`` to work out the maximum and minimum temperature. This work was delegated to two helper functions:
 * ``compare-max/2``
 * ``compare-min/2``

In both of those functions, we introduced the new ``if`` form. If works as follows:

```lisp
(if <predicate>
    <expression1>
    <expression2>)
```

where ``<expression 1>`` is executed if ``<predicate>`` evaluates to ``true`` and ``<expression 2>`` is executed if ``<predicate>`` evaluates to ``false``. If you have used other programming languages, then this will be quite familiar to you. If you have not, if should remind you a bit of the logic we looked at when discussing guards.

We can see it in action with the following LFE session in the REPL:

```lisp
> (if (=:= 1 1) "They are equal!" "They are *not* equal!")
"They are equal!"
> (if (=:= 2 1) "They are equal!" "They are *not* equal!")
"They are *not* equal!"
```

Or -- you will be more familiar with this -- our code from the last section:

```lisp
(if (< temp1 temp2)
    city1
    city2)
```
where, if ``temp1`` is less than ``temp2``, the value stored in ``city1`` is returned. 

So the ``if`` form works for two conditions. What about 3? 10? 100? Well, for the situations were we want to check multiple conditions, we'll need the ``cond`` form.

### The ``cond`` Form

```lisp
(cond (<predicate1> <expression1>)
      (<predicate2> <expression2>)
      (<predicate3> <expression3>)
      ...
      (<predicaten> <expressionn>))
```

A given expression is only executed if its accompanying predicate evaluates to ``true``. The ``cond`` returns the value of the expression for the first predicate that evaluates to ``true``. Using ``cond``, our temperature test would look like this:

```lisp
(cond ((< temp1 temp2) city1)
      ((>= temp1 temp2) city2))
```

Here's an example which takes advantage of ``cond`` supporting more than two logic branches:

```lisp
(cond ((> x 0) x)
      ((=:= x 0) 0)
      ((< x 0) (- x)))
```

Note that each predicate is an expression with it's own parentheses around it; on its left is the opening parenthenis for that particular branch of the ``cond``.

Often times when using ``cond`` one needs a "default" or "fall-through" option to be used when no other condition is met. Since it's the last one, and we need it to evaluate to ``true`` we simply set the last condition to ``true`` when we need a default. Here's a rather silly example:

```lisp
(cond ((lists:member x '(1 2 3)) "First three")
      ((=:= x 4) "Is four")
      ((>= x 5) "More than four")
      ('true "You chose poorly"))
```

Any number that is negative will be caught by the last condition.

[add pattern-matching example for cond]

### The ``case`` Form

The ``case`` form is useful for situations where you want to check for multiple possible values of the same expression. Without guards, the general form for ``case`` is the following:

```lisp
(case <expression>
  (<pattern1> <expression1>)
  (<pattern2> <expression2>)
  ...
  (<patternn> <expressionn>))
```

So we could rewrite our ``cond`` above with the following ``case``:

```lisp
(case x
  ((cons head '())
   "Only one element")
  ((list 1 2)
   "Two element list")
  ((list 'a _)
    "List starts with 'a'")
  (_ "Anything goes"))
```

The following will happen with the ``case`` defined above:
 * Any 1-element list will be matched by the first clause.
 * A 2-element list of ``1`` and ``2`` (in that order) will match the second clause.
 * Any list whose first element is the atom ``a`` will match the third caluse.
 * Anything *not* matching the first three clauses will be matched by the fourth.
 
With guars, the case has the following general form:

```lisp
(case <expression>
  (<pattern1> [<guard1>] <expression1>)
  (<pattern2> [<guard2>] <expression2>)
  ...
  (<patternn> [<guardn>] <expressionn>))
```

Let's update the previosu example with a couple guards:

```lisp
(case x
  ((cons head '())
   "Only one element")
  ((list 1 2)
   "Two element list")
  ((list a _) (when (is_atom a))
    "List starts with an atom")
  ((cons _ (cons a _)) (when (is_tuple a))
    "Second element is a tuple")
  (_ "Anything goes"))
```

This changes the logic of the previous example in the following ways:
 * Any list whose first element is an atom will match the third clause.
 * Any list whose second element is a tuple will match the fourth clause.
 * Anything *not* matching the first four clauses will be matched by the fifth.
 
### Function Heads as Conditionals

### The Extended ``cond`` Form

The ``cond`` form has been extended with the extra test ``(?= <pattern> <expression>)`` which tests if the evaluated result of ``<expression>`` matches ``<pattern>``. If so, it binds the variables in ``<pattern>`` which can be used in the ``cond`` form. A optional guard is also allowed here. Here is an example of extended ``cond``:

```lisp
(cond ((foo x) ...)
      ((?= (cons head tail) (when (is_atom head)) (bar y))
       (fubar tail (baz head)))
      ((?= (tuple 'ok x) (baz y))
       (zipit x))
      ... )
```

[update this section]