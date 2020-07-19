# Installing LFE


## Downloading the Source

The most recent version of LFE is always available here:
 * https://github.com/rvirding/lfe/

With `git` in your `PATH`, you can download LFE:

```bash
git clone https://github.com/rvirding/lfe.git
cd lfe
```

You have two choices for the type of LFE build you want to do.

Now everything is ready to go!

<div class="alert alert-success">
  <h4 class="alert-heading">
    <i class="fa fa-info-circle" aria-hidden="true"></i>
    Stable
  </h4>
  <p class="mb-0">
    Stable contains a version of LFE that is known to be fully reliable and used in production for years.
  </p>
</div>

To build the latest stable release, make sure you are on the `master` branch:

```bash
git checkout master
```

<div class="alert alert-warning">
  <h4 class="alert-heading">
    <i class="fa fa-exclamation-triangle" aria-hidden="true"></i>
Unstable
  </h4>
  <p class="mb-0">
    Unstable contains LFE that is under active development, has not been released, or has not had extensive use and testing in production environments.
  </p>
</div>

To build the latest unstable LFE, make sure you are on the `develop` branch:

```bash
git checkout develop
```

LFE is just a set of Erlang libraries, so like an Erlang project, the source code needs to be compiled to ``.beam`` files. Running ``make`` in the ``lfe`` directory will do that:

```bash
make
```
