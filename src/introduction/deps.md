# Dependencies

In order to write LFE scripts, libraries, and applications, you will need to install the following:

 * Erlang
 * `make` and related developer tools
 * `git`

## Installing Erlang

First and foremost, you will need Erlang installed.

 * On Mac OS X, this is as easy as executing ```brew install erlang```
 * On Ubuntu ```apt-get install erlang```.

You can also install Erlang from the various pre-built packages
provided on the <a href="http://www.erlang.org/download.html">official Erlang
download page</a> or from the
<a href="https://www.erlang-solutions.com/downloads/download-erlang-otp">Erlang
Solutions page</a> (which supports many more package types).

For those who have the need of installing multiple versions of Erlang, there is also the [kerl](https://github.com/spawngrid/kerl) project.

## Installing `make`

In order to use LFE, you will be calling `make` to compile it. To cover your bases, you'll want to make sure you have the basic development tools installed for your platform.

Ubuntu:

```bash
sudo apt-get install build-essential
```

CentOS/Fedora:

```bash
sudo yum groupinstall "Development Tools"
```

For Mac OS X, you will need to install the "Developer Tools" from the AppStore.

## Installing `git`

You will also need `git` to continue with this quick-start. If `git` doesn't come installed on your system and it wasn't installed as part of your systems basic development tools package, you can <a href="http://git-scm.com/downloads">download it here</a>
or install it using your favourite OS package manager.
