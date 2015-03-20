## Atoms

Atoms are another data type in LFE. They are words, for example ``charles``, ``centimeter``, ``inch`` and ``ok``. Atoms are similar to symbols in Lisp except that are simply names, nothing else. They are not like variables which can have a value.

Enter the next program (file: ``tut4.lfe``) which could be useful for converting from inches to centimeters and vice versa:

```lisp
(defmodule tut4
  (export (convert 2)))

(defun convert
  ((m 'inch) (/ m 2.54))
  ((n 'centimeter) (* n 2.54)))
```

Compile and test:

```lisp
> (c "tut4.lfe")
#(module tut4)
> (tut4:convert 3 'inch)
1.1811023622047243
> (tut4:convert 7 'centimeter)
17.78
```

Notice that atoms and variables look the same so we have to tell LFE when we want it to be an atom. We do this by *quoting* the atom with a ``'``, for example ``'inch`` and ``'centimeter``. We have to do this both when we use it as argument in a function definition and when we use it when calling a function, otherwise LFE will assume that it is a variable.

Also notice that we have introduced decimals (floating point numbers) without any explanation, but I guess you can cope with that.

See what happens if I enter something other than centimeter or inch in the convert function:

```lisp
> (tut4:convert 3 'miles)
exception error: function_clause
  in (: tut4 convert 3 miles)
```

The two parts of the convert function are called its *clauses*. Here we see that ``miles`` is not part of either of the clauses. The LFE system can't **match** either of the clauses so we get an error message ``function_clause``. The shell formats the error message nicely, but to see the actual error tuple we can do:

```lisp
> (catch (tut4:convert 3 'miles))
#(EXIT
  #(function_clause
    (#(tut2 convert (3 miles) (#(file "./tut2.lfe") #(line 4)))
     #(lfe_eval eval_expr 2 (#(file "src/lfe_eval.erl") #(line 160)))
     #(lfe_shell eval_form_1 2 (#(file "src/lfe_shell.erl") #(line 268)))
     #(lists foldl 3 (#(file "lists.erl") #(line 1261)))
     #(lfe_shell server_loop 1 (#(file "src/lfe_shell.erl") #(line 101))))))
```
