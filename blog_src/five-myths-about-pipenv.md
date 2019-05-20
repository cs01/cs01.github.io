* * *

# Five Myths About Pipenv

## Myth 1: “If I don’t use Pipenv or feel like it improves my workflow, I’m doing it wrong!”

Pipenv is designed for a very specific use-case: application dependency management and associated workflows. In other words it’s a fancy replacement for requirements.txt that automates virtual environment management. Even if your use-case matches what it’s designed for, nobody is saying you **must** use it, or looking down on you if you don’t. In fact core Python developers and the Pipenv team have workflows that don’t involve Pipenv. [[1](https://hynek.me/articles/python-app-deps-2018/)][[2](https://twitter.com/ncoghlan_dev/status/1068151014946103296)]

* * *

## Myth 2: “Pipenv is the officially recommended Python packaging tool from Python.org”

This myth comes from [older versions](https://github.com/pypa/pipenv/commit/6e06fc451767a57e6fccb828c74a1412f6cef687) of Pipenv’s documentation. The claim is misleading because it sounds like a recommendation from the core Python team but is actually referring to a recommendation from the Python Packaging Authority (PyPA), a volunteer organization separate from core Python with no qualifications for membership [[1]](https://www.reddit.com/r/Python/comments/8jd6aq/why_is_pipenv_the_recommended_packaging_tool_by/dyzduci). Pipenv now more accurately markets itself as a tool for application dependency management, not for all Python packaging. Pipenv is still recommended in the PyPA’s documentation (which is slightly [controversial](https://hynek.me/articles/python-app-deps-2018/)) among other tools, but it’s not **_the_**packaging tool for all of Python.

* * *

## Myth 3: “Pipenv is the Python equivalent to npm”

Pipenv can manage, isolate, and lock dependencies for applications similarly to npm. However npm can also manage dependencies for libraries and publish to npm’s package center, so Pipenv only offers a subset of npm’s functionality.

* * *

## Myth 4: “Pipenv makes pip and venv obsolete / Pipenv is the future of dependency management.”

Pipenv relies on both pip and virtual environments. There are still many workflows that require the use of pip and virtual environments. These tools work well, are supported on multiple operating systems and environments, and are not going away. Also, with new standards recently provisionally accepted for package building ([PEP 517](https://www.python.org/dev/peps/pep-0517/), [PEP 518](https://www.python.org/dev/peps/pep-0518/)) there is certainly going to be more innovation in this space going forward (see [Hatch](https://github.com/ofek/hatch), [Poetry](https://github.com/sdispater/poetry), [flit](https://github.com/takluyver/flit), [black’s configuration](https://github.com/ambv/black/#pyprojecttoml)). And there’s [PEP 585](https://www.python.org/dev/peps/pep-0582/) which drastically reduces the need for virtual environments; it will “automatically recognize a `__pypackages__` directory and prefer importing packages installed in this location over user or global site-packages” (if it’s accepted).

* * *

## Myth 5: “pyenv and Pipenv are similar tools”

The only thing similar about these tools are their names. pyenv manages Python versions, while Pipenv manages application dependencies.
