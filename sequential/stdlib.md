## The Erlang Standard Library

### ``man`` Pages and Online Docs
Erlang has a lot of standard modules to help you do things which are directly callable from LFE. For example, the module ``io`` contains a lot of functions to help you perform various acts of formatted input/output. Depending upon your Erlang installation, you may have man pages available. From your operating system shell, you can found out by typing ``erl -man <module name>`` like so:

```bash
$ erl -man io
```

If you have man pages installed, that command would give output along these lines:

```
ERLANG MODULE DEFINITION                                    io(3)

MODULE
     io - Standard I/O Server Interface Functions

DESCRIPTION
     This module provides an  interface  to  standard  Erlang  IO
     servers. The output functions all return ok if they are suc-
     ...
```

If your installation of Erlang doesn't have man pages, you can always find what you're looking for on the documentation web site. Here is the online manpage for the [io module](http://erlang.org/doc/man/io.html).

### Module and Function Tab-Completion in the REPL

From the LFE REPL, you have some other nice options for standard library discovery. Start up LFE to take a look:

```
$ ./bin/lfe
Erlang/OTP 17 [erts-6.3] [source] [64-bit] [smp:8:8] ...

>
```

Now, at the prompt, hit your ``<TAB>`` key. You should see something like this:

```
application               application_controller    application_master
beam_lib                  binary                    c
code                      code_server               edlin
edlin_expand              epp                       erl_distribution
erl_eval                  erl_parse                 erl_prim_loader
erl_scan                  erlang                    error_handler
error_logger              error_logger_tty_h        erts_internal
ets                       file                      file_io_server
file_server               filename                  gb_sets
gb_trees                  gen                       gen_event
gen_server                global                    global_group
group                     heart                     hipe_unified_loader
inet                      inet_config               inet_db
inet_parse                inet_udp                  init
io                        io_lib                    io_lib_format
kernel                    kernel_config             lfe_env
lfe_eval                  lfe_init                  lfe_io
lfe_shell                 lists                     net_kernel
orddict                   os                        otp_ring0
prim_eval                 prim_file                 prim_inet
prim_zip                  proc_lib                  proplists
ram_file                  rpc                       standard_error
supervisor                supervisor_bridge         sys
unicode                   user_drv                  user_sup
zlib
```

These are all the modules available to you by default in the LFE REPL. Now type ``(g`` and hit ``<TAB>``:

```lisp
> (g
```
```
gb_sets         gb_trees        gen             gen_event
gen_server      global          global_group    group
```
Let's keep going! Continue typing a full module, and then hit ``<TAB>`` again:

```lisp
> (gb_trees:
```
```
add/2            add_element/2    balance/1        del_element/2
delete/2         delete_any/2     difference/2     empty/0
filter/2         fold/3           from_list/1      from_ordset/1
insert/2         intersection/1   intersection/2   is_disjoint/2
is_element/2     is_empty/1       is_member/2      is_set/1
is_subset/2      iterator/1       largest/1        module_info/0
module_info/1    new/0            next/1           singleton/1
size/1           smallest/1       subtract/2       take_largest/1
take_smallest/1  to_list/1        union/1          union/2
```
Now you can see all the *functions* that are available in the module you have selected. This is a great feature, allowing for easy use as well as exploration and discovery.