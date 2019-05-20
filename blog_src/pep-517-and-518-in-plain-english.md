# PEP 517 and 518 in Plain English

PEP 517 and 518 are related and their combined changes are as follows.

A new standardized file named `pyproject.toml` will be read by `pip` for a section called `[build-system]` . This file specifies two things

1.  How to build a package (“wheel”) from source code (“sdist”) (PEP 517)
2.  Which packages need to be installed before trying to build. These are the build requirements. (PEP 518)

Here is an example of what`pyproject.toml` looks like

<pre name="7a9d" id="7a9d" class="graf graf--pre graf-after--p">[build-system]
requires = ["requests"]  # PEP 518 - what is required to build
build-backend = “tool.module:myfunction”  # PEP 517 - what function to call to build</pre>

#### Implications

This is a big deal because previously `pip` is literally hardcoded to run `python setup.py` to build packages when installing them from source. **PEP 517 and 518 break that convention and let pyproject.toml tell pip how to build wheels and install them.**

Most packages on PyPI are “wheels”, meaning they won’t be affected by this, but plenty still arent. Aditionally when you install from source code or a GitHub url you **are** installing from source.

Starting in pip version 19 when you run `pip install` on a folder with the above`pyproject.toml` in it, pip will first install `requests` (per PEP 518) then call`myfunction()` in `./tool/module.py` (per PEP 517).

Below you can see what happens with a version of pip that does not have this change implemented. You can see that setup.py is hardcoded into pip!

<pre name="dc42" id="dc42" class="graf graf--pre graf-after--p">> pip install -e .
Directory '.' is not installable. File 'setup.py' not found.</pre>

#### Extensibility

`pyproject.toml` is allowed to have arbitrary fields in it so other tools can make use of it.

For example `[poetry](https://github.com/sdispater/poetry)` adds project metadata like author and license in it, project dependencies, along with the PEP 517 and 518 fields for build system.

<pre name="ece3" id="ece3" class="graf graf--pre graf-after--p">[tool.poetry]
name = "myproject"
version = "0.1.0"
description = ""
authors = ["Chad Smith <[cs01@users.noreply.github.com](mailto:cs01@users.noreply.github.com)>"]</pre>

<pre name="7a05" id="7a05" class="graf graf--pre graf-after--pre">[tool.poetry.dependencies]
python = "^3.7"
somepackage = "^0.13.1"</pre>

<pre name="c9a9" id="c9a9" class="graf graf--pre graf-after--pre">[build-system]
requires = ["poetry>=0.12"]
build-backend = "poetry.masonry.api"</pre>

### Status

PEP 518 already landed in pip as of version 10\. PEP 517 was implemented in pip in November 2018 ([pull request](https://github.com/pypa/pip/pull/5743), [GitHub issue](https://github.com/pypa/pip/issues/5407)) and should be released in version 19 of pip in January 2019.

Some projects are already prepared for this when it occurs. For example when you run`poetry init` , it’ll create a `pyproject.toml` file that’s compliant with PEP 518 and 517\. [[1](https://poetry.eustace.io/docs/pyproject/#poetry-and-pep-517)]

<pre name="8ab8" id="8ab8" class="graf graf--pre graf-after--p">[build-system]
requires = ["poetry>=0.12"]
build-backend = "poetry.masonry.api"</pre>

This means when `pip` has PEP 517 implemented in it, you can install poetry-managed project directly from source code. For example `pip install <github url>` or `pip install -e .`.

### Summary

Starting in version 19, `pip` will no longer be forced to use the hardcoded path `setup.py` to build installable packages from source code. More tools to build packages besides `setuptools` will be created and adopted in the ecosystem, such as `poetry` and `flit`.