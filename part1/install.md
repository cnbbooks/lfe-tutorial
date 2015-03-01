# Getting LFE


## 1 Going Plaid

<img src="https://raw.github.com/lfe/docs/master/images/plaid.jpg"
     style="float: right;">
This is a quick-start, a blur of color. You're not going to get very many
details here, but you *will* get to jump into the LFE REPL and see a little
code in action.

For those that would enjoy a more gradual introduction (with a bit more Erlang)
thus having the time to see the stars (and not just stunning plaid), you may be
interested in checking out the
<a href="http://docs.lfe.io/user-guide/intro/1.html">User Guide</a>.


### 1.1 LFE

LFE, or "Lisp Flavored Erlang", is a Lisp-2 (similar to Common Lisp in that way) that runs on top of the Erlang VM or "BEAM". It is the oldest sibling of the many BEAM languages that have been released since its inception (e.g., Elixir, Joxa, Luerl, Erlog, Haskerl, etc.).

As a Lisp-2, LFE has different namespaces for functions and variables. Furthermore, LFE has Erlang's notion of arity, so functions with different arity may share the same name.

Before we jump into LFE, though, you'll need to have the minimal set of dependencies installed on your machine.


### 1.2 Dependencies

#### Erlang

First and foremost, you will need Erlang installed.

 * On Mac OS X, this is as easy as executing ```brew install erlang```
 * On Ubuntu ```apt-get install erlang```.

You can also install Erlang from the various pre-built packages
provided on the <a href="http://www.erlang.org/download.html">official Erlang
download page</a> or from the
<a href="https://www.erlang-solutions.com/downloads/download-erlang-otp">Erlang
Solutions page</a> (which supports many more package types).

For those who have the need of installing multiple versions of Erlang, there is also the [kerl](https://github.com/spawngrid/kerl) project.

#### make

In order to use LFE, you will be calling ``make`` to compile it. To cover your bases, you'll want to make sure you have the basic development tools installed for your platform.

Ubuntu:

```bash
$ sudo apt-get install build-essential
```

CentOS/Fedora:

```bash
$ sudo yum groupinstall "Development Tools"
```

For Mac OS X, you will need to install the "Developer Tools" from the AppStore.

#### git

You will also need ``git`` to continue with this quick-start. If ``git`` doesn't come installed on your system and it wansn't installed as part of your systems basic development tools package, you can <a href="http://git-scm.com/downloads">download it here</a>
or install it using your favourite OS package manager.

### 1.3 Getting LFE

The most recent version of LFE is always available here:
 * https://github.com/rvirding/lfe/

With ``git`` and ``erl`` in your ``$PATH``, let's download LFE:

```bash
$ git clone https://github.com/rvirding/lfe.git
$ cd lfe
$ make
```

You'll see the output from LFE getting built, and when it's done, we're ready for the next step in our Lispy Space Adventure!

But first, a message from our sponsors ...


### 1.4 Other Helpful Projects

#### lfetool

[lfetool](https://github.com/lfe/lfetool/) provides convenient means of executing simple commands for such tasks as:
 * creating new projects
 * downloading dependencies
 * over-riding rebar defaults
 * compiling code and tests
 * running tests
 * and much, much more!

#### rebar

[rebar](https://github.com/rebar/rebar/) is a tool for managing project dependencies, compiling them, and
compiling your project's source code. Most recent Erlang projects have a
``rebar.config`` file in them, indicating their support of ``rebar``.

``lfetool`` uses ```rebar``` to build LFE as well as managing dependencies for
your projects, so go ahead and get that set up:
<a href="https://github.com/basho/rebar">download rebar</a> and follow the
instructions in the project ``README`` on Github.

#### kerl

If you would like to easily have multiple versions of Erlang installed (or be
available for installation) on your system,
[kerl](https://github.com/spawngrid/kerl) might be the tool for you. If
you have used such tools as ``virtualenv`` in Python or Ruby's RVM, then you
should be quite comfortable with the concepts behind kerl.

Most people don't need to work with multiple versions of Erlang, however;
please consider carefully before unnecessarily complicating your development
environment.


### Next Stop

Ready for some LFE? Next you'll learn how to create a new LFE project with
just one command ...

