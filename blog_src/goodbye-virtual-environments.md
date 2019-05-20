* * *

# Goodbye Virtual Environments?

If you’re a Python developer you’ve likely heard of Virtual Environments. A Virtual Environment is “a self-contained directory tree that contains a Python installation for a particular version of Python, plus a number of additional packages.”

Why are they so popular? Well, they solve a problem: no longer are packages installed into a mishmash of global site-packages. Instead, each project can install precise dependency versions into their own “virtual” environments.

However, they introduce some problems as well:

*   **Learning curve:** explaining “virtual environments” to people who just want to jump in and code is not always easy
*   **Terminal isolation:** Virtual Environments are activated and deactivated on a per-terminal basis
*   **Cognitive overhead:** Setting up, remembering installation location, activating/deactivating

To solve some of the above points, new higher level tools such as pipenv, poetry, flit, and hatch have been created. These tools improve the situation by hiding the complexities of pip and Virtual Environments. However, they become complex themselves in order to hide complexity. They also have their own API’s, learning curves, and maintenance burden.

### Enter PEP 582 — Python local packages directory

A Python Enhancement Proposal (PEP) was introduced back in May of 2018 to modify Python itself to solve many of the problems Virtual Environments solve, but in a totally different and much simpler way. Note that it’s still in draft state so the proposal may still change, or it may not even be adopted at all.

I’ll let [PEP 582](https://www.python.org/dev/peps/pep-0582/) speak for itself:

> This PEP proposes to add to Python a mechanism to automatically recognize a __pypackages__directory and prefer importing packages installed in this location over user or global site-packages. This will avoid the steps to create, activate or deactivate “virtual environments”. Python will use the__pypackages__ from the base directory of the script when present.

This proposal effectively works around all the complexity of Virtual Environments and their higher level counterparts simply by searching a local path for additional packages.

### Try it Today

It even comes with a [reference CPython implementation](https://github.com/kushaldas/cpython/tree/pypackages).

If you don’t have the time or desire to build a CPython binary, you can try a proof of concept Python wrapper I made called [pythonloc](https://github.com/cs01/pythonloc) (for “python local”). It is a Python package (less than 100 lines of code) that does exactly what PEP 582 describes.

`pythonloc` runs Python, but will import packages from `__pypackages__` , if present. It also ships with `piploc` which is the same as `pip` but installs/uninstalls to `__pypackages__`.

Here is an example.

<pre name="4c43" id="4c43" class="graf graf--pre graf-after--p">> piploc install requests
Successfully installed certifi-2018.11.29 chardet-3.0.4 idna-2.8 requests-2.21.0 urllib3-1.24.1</pre>

<pre name="49b4" id="49b4" class="graf graf--pre graf-after--pre">> tree -L 4
.
└── __pypackages__
    └── 3.6
        └── lib
            ├── certifi
            ├── certifi-2018.11.29.dist-info
            ├── chardet
            ├── chardet-3.0.4.dist-info
            ├── idna
            ├── idna-2.8.dist-info
            ├── requests
            ├── requests-2.21.0.dist-info
            ├── urllib3
            └── urllib3-1.24.1.dist-info</pre>

<pre name="f022" id="f022" class="graf graf--pre graf-after--pre">> python -c "import requests; print(requests)"
Traceback (most recent call last):
  File "<string>", line 1, in <module>
ModuleNotFoundError: No module named 'requests'</pre>

<pre name="4ab2" id="4ab2" class="graf graf--pre graf-after--pre">> pythonloc -c "import requests; print(requests)"
<module 'requests' from '/tmp/demo/__pypackages__/3.6/lib/requests/__init__.py'></pre>

_Note: This is identical to what you might find in your site packages directory, i.e._`_~/.local/lib/python3.6/site-packages_`_._

As you can see `piploc` installed `requests` to `__pypackages__/3.6/lib/requests` . Running `python` demonstrated that it did not find `requests` (which is expected since it doesn’t search `__pypackages__`).

To make Python find it, you can run `pythonloc`, which is roughly the same as running `PYTHONPATH=.:__pypackages__:$PYTHONPATH python`. This searches `__pypackages__` and finds the`requests` installation. 🎉

You can try `pythonloc` today by running

<pre name="96b8" id="96b8" class="graf graf--pre graf-after--p">pip install --user pythonloc</pre>

and can learn more at [https://github.com/cs01/pythonloc](https://github.com/cs01/pythonloc).

### Installing Multiple Dependencies or Lockfiles

If you have the source code available and it has a `setup.py` file, you can run

<pre name="ea0e" id="ea0e" class="graf graf--pre graf-after--p">piploc install .</pre>

then run `pythonloc` and have all your dependencies available.

If you have a `requirements.txt` file, you can run

<pre name="5044" id="5044" class="graf graf--pre graf-after--p">piploc install -r requirements.txt</pre>

If you are using pipenv you can generate a `requirements.txt` file with

```
pipenv lock --requirements
```

And finally if you are using poetry you can generate `requirements.txt` with

```
poetry run pip freeze > requirements.txt
```

### Freezing Dependencies

Okay so we can install from various sources, but what if we’re developing and want to _generate_ a list of dependencies.

A new workflow you could use with the advent of `__pypackages__` is to work around creating a list of dependencies and actually commit `__pypackages__` _itself_ to source control. Doing that would virtually guarantee you’re using the same versions because, well… you’re using the exact same source code.

Assuming you don’t want to do that, you could run `piploc freeze` . But this presents a problem. It shows all installed python packages: those in `site-packages` as well as in `__pypackages__`. This probably isn’t want you want because it includes more than what you installed to `__pypackages__` .

You likely only want to output the packages installed to `__pypackages__` . That is exactly what `pipfreezeloc` does.

<pre name="d24a" id="d24a" class="graf graf--pre graf-after--p">pipfreezeloc
requests==2.21.0</pre>

It is the equivalent of `pip freeze` but only outputs packages in `__pypackages__`. This is required because there is no built-in way to do this with pip. For example, the command `pip freeze --target __pypackages__` does not exist.

So instead of running

```
pip freeze > requirements.txt
```

you would run

```
pipfreezeloc > requirements.txt
```

### Update (2/19/19): What about …?

Changes to packaging and workflows should be scrutinized heavily since they have an outsized impact on the language’s adoption and community momentum. I wanted to take a moment to respond to some of the concerns I have seen raised. _(Note: I, like you, am just an observer to the whole PEP process. I have no say in whether it is adopted or not.)_

**What about entry points?**

Entry points are CLI tools that are made available by packages. Examples of these are black, tox, pytest, pipenv, flake8, mypy, gdbgui, and many more. With pip and virtualenv, when packages are installed, these associated entry points are automatically available on your $PATH. On the other hand, with `piploc` they are installed to `__pypackages__/3.6/lib/bin` and are **not** on your $PATH.

To solve this, you can type the path out yourself, but that’s rather inconvenient; `__pypackages/3.6/lib/bin/pytest` is a lot to type!

An existing Python tool called [pipx](https://github.com/pipxproject/pipx) (also something I maintain) has been modified to search `__pypackages__` for the binary you want to run, and is now included when you install `pythonloc`. This is similar to JavaScript’s [npx](https://www.npmjs.com/package/npx) tool (which you likely have installed on your computer; try typing `npx` in your terminal).

So instead of running

```
BINARY [ARGS] # i.e. pytest --help
```

you would run

```
pipx run BINARY [ARGS] # i.e. pipx run pytest --help
```

**What about Pipenv (or poetry, or pip)?**

Despite its name, I don’t think Pipenv really wants to be limited to working with pip and virtual environments. Higher level tools like Pipenv and poetry tools really want to enable _desirable workflows_. Right now the tools to accomplish this are pip and virtual environments. But pipenv and others could be adapted to install and manage the contents of`__pypackages__` to enable those convenient higher level workflows rather than hiding away virtual environments.

In other words, virtual environments are an implementation detail that are not required, and the implementation of these tools could switch to using `__pypackages__`.

Indeed, having a higher level tool to resolve dependencies and install precise versions into `__pypackages__` will be a highly desired feature of these types of tools. This is exactly how `yarn` and `npm` work in the JavaScript world, and there is no reason the Python equivalents couldn’t be made to work with `__pypackages__`.

**What about the fallback to site-packages? (added 2/26/19)**

The PEP describes the paths searched by python to resolve packages.

> If a __pypackages__ directory is found in the current working directory, then it will be included in sys.path after the current working directory and just before the system site-packages.

Basically `__pypackages__` is prepended to the search paths, and if a package isn’t found, the other locations are then searched. I think this is an unfortunate choice because it leads to uncertainty of which version of a package is being used.

What if I forgot to install `requests` to `__pypackages__` and the site-package version is used? And what if there is an API change in that version that is incompatible with my code? Yes, `node` works this way, but there is no requirement that we copy `node` exactly. I would prefer behavior such that if`__pypackages__` exists, it is the _only_ path searched by python, and if a package is not found there an error is raised.

**What about cross platform?**

Installing packages does not always result in identical installations across platforms. Some packages have cross platform differences (i.e. pip installs on Windows vs mac vs unix do not necessarily install the same code). This isn’t really a problem unless you are distributing your application’s source tree and `__pypackages__` directory directly. To fix this, it may be helpful if PEP 582 namespaced the installation target based on OS as well, i.e. `__pypackages__/windows/3.6/lib` or `__pypackages__/unix/3.6/lib` , etc.

**What about bloat?**

This approach is similar to JavaScript’s`node_modules` , which is notorious for installing tons of dependencies for simple npm packages. I don’t think this applies here for three reasons. 1.) There will be no more or less bloat than there was with virtual environments (actually there will be a little less since venvs install python and pip executables). 2.) Python’s standard library is relatively complete so reliance on PyPI to fill in the gaps is minimal. 3.) JavaScript is heavily transpiled leading to all kinds of packages and linters being installed. This is uncommon in the Python world.

**What about …?**

There are probably dozens of other concerns, but these are the major ones I am aware of. I don’t really see anything popping up that can’t be coded around and captured in documentation. However the devil is in the details, and PEPs do lock you in basically forever. So playing around with prototypes and getting as much feedback as possible is extremely valuable.

### Conclusion

PEP 582 is a draft proposal that introduces a new way to install and isolate packages without Virtual Environments. It also eliminates indirection between project location and environment location, since the installation location is always in the same directory as the project — at `__pypackages__`.

`[pythonloc](https://github.com/cs01/pythonloc)` (and `piploc`, `pipfreezeloc`) is a proof of concept Python implementation of PEP 582 available today.

What do you think? Should PEP 582 be approved? _(note I am not a decision maker in this process, just an interested observer)_ Are Virtual Environments going to be relied on less ? Does `pythonloc` improve your workflow?