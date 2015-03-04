## Writing Output to a Terminal

It's nice to be able to do formatted output in these example, so the next example shows a simple way to use to use the ``lfe_io:format`` function. Of course, just like all other exported functions, you can test the ``lfe_io:format`` function in the repl:

```lisp
> (lfe_io:format "hello world~n" ())
hello world
ok
> (lfe_io:format "this outputs one LFE term: ~w~n" '(hello))
this outputs one LFE term: hello
ok
> (lfe_io:format "this outputs two LFE terms: ~w~w~n" '(hello world))
this outputs two LFE terms: helloworld
ok
> (lfe_io:format "this outputs two LFE terms: ~w ~w~n" '(hello world))
this outputs two LFE terms: hello world
ok
```

The function ``format/2`` (i.e. ``format`` with two arguments) takes two lists. The first one is nearly always a list written as a string between " ". This list is printed out as it stands, except that each ~w is replaced by a term taken in order from the second list. Each ~n is replaced by a new line. The ``lfe_io:format/2`` function itself returns the atom ``ok`` if everything goes as planned. Like other functions in LFE, it crashes if an error occurs. This is not a fault in LFE, it is a deliberate policy. LFE has sophisticated mechanisms to handle errors which we will show later. As an exercise, try to make ``lfe_io:format`` crash, it shouldn't be difficult. But notice that although ``lfe_io:format`` crashes, the Erlang shell itself does not crash.

```lisp
> (lfe_io:format "this outputs one LFE term: ~w~n" 'hello)
exception error: badarg
  in (: lfe_io fwrite1 "this outputs one LFE term: ~w~n" hello)
  in (lfe_io format 3)
```
