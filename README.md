# Jinja

## CS593 Homework 3

This branch fixes [issue #2069](https://github.com/pallets/jinja/issues/2069)
from the Jinja `3.1.6` release commit
`15206881c006c79667fe5154fe80c01c65410679`. An exhaustive conditional
defines its output locally. The Meta API should report the names in the
condition and any other context dependencies.

### Set up the checkout

Run these commands from the repository root. The patch was verified with
Python 3.11.4 and Pytest 8.3.4.

```sh
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -e . -r requirements/tests.txt
```

### Reproduce the original failure

Switch to the test-only commit. It contains the regression test on the
original 3.1.6 source. The editable install above follows the checkout.

```sh
git switch --detach e364b0223708f56f9396227d1b1496a743257b34
python -m pytest -q tests/test_api.py::TestMeta::test_undeclared_variables_with_assignment_in_every_branch
```

The assertion fails with the extra name `output`. The expected set is
`{"control"}`.

### Validate the patch

```sh
git switch codex/hw3-jinja-2069
python -m pytest -q tests/test_api.py::TestMeta::test_undeclared_variables_with_assignment_in_every_branch
python -m pytest -q tests/test_api.py tests/test_idtracking.py tests/test_core_tags.py
python -m pytest -q
```

The regression test passes, the targeted suite passes 115 tests, and the
full suite passes 922 tests. The validation cases cover multiple `elif`
arms, missing assignments, nested conditionals, reads before assignment, and
context fallback during rendering.

The Meta API adds path-sensitive tracking on top of Jinja's compiler frames.
It intersects assignments across branch outcomes and retains names read
before a guaranteed assignment. The renderer keeps its original frame
initialization behavior.

## About Jinja

Jinja is a fast, expressive, extensible templating engine. Special
placeholders in the template allow writing code similar to Python
syntax. Then the template is passed data to render the final document.

It includes:

-   Template inheritance and inclusion.
-   Define and import macros within templates.
-   HTML templates can use autoescaping to prevent XSS from untrusted
    user input.
-   A sandboxed environment can safely render untrusted templates.
-   AsyncIO support for generating templates and calling async
    functions.
-   I18N support with Babel.
-   Templates are compiled to optimized Python code just-in-time and
    cached, or can be compiled ahead-of-time.
-   Exceptions point to the correct line in templates to make debugging
    easier.
-   Extensible filters, tests, functions, and even syntax.

Jinja's philosophy is that while application logic belongs in Python if
possible, it shouldn't make the template designer's job difficult by
restricting functionality too much.


## In A Nutshell

```jinja
{% extends "base.html" %}
{% block title %}Members{% endblock %}
{% block content %}
  <ul>
  {% for user in users %}
    <li><a href="{{ user.url }}">{{ user.username }}</a></li>
  {% endfor %}
  </ul>
{% endblock %}
```

## Donate

The Pallets organization develops and supports Jinja and other popular
packages. In order to grow the community of contributors and users, and
allow the maintainers to devote more time to the projects, [please
donate today][].

[please donate today]: https://palletsprojects.com/donate

## Contributing

See our [detailed contributing documentation][contrib] for many ways to
contribute, including reporting issues, requesting features, asking or answering
questions, and making PRs.

[contrib]: https://palletsprojects.com/contributing/
