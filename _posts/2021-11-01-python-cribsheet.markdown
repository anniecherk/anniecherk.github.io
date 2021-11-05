---
layout: post
title: "python tooling cribsheet"
date: 2021-11-01
permalink: python-tooling
categories: computers
---
 
🧝🗡️🏹🛡️ it's dangerous to go alone! take this 🐍🛠️🐍🛠️


This page is a brain-dump of the tools & libraries I reach for in python in various situations. It's a living document; I intend to update it as my practices drift. Perhaps you'll find something you find interesting here!

🐍🛠️🐍🛠️🐍🛠️🐍🛠️🐍🛠️🐍🛠️🐍🛠️🐍🛠️🐍🛠️🐍🛠️

# Python tooling:
- [mypy](https://mypy.readthedocs.io/en/stable/): type checker
- [pylint](https://pylint.org/): linter
- [bellybutton](https://github.com/hchasestevens/bellybutton): linter that lets you write custom rules
- [jupyter notebook](https://jupyter.org/): good for quick experimentation
- [black](https://github.com/psf/black): formatter
- [isort](https://github.com/PyCQA/isort): sort imports
- [docformatter](https://pypi.org/project/docformatter/): formats docstrings
- [pyenv](https://github.com/pyenv/pyenv): manages switches between different versions of the python interpretter
- [tox](https://tox.wiki/en/latest/index.html): test code against different python versions
- [pythex](https://pythex.org/): online regex sandbox

# Python libraries:
- [argparse](https://docs.python.org/3/library/argparse.html): for building CLI tools which take arguments
- [click](https://click.palletsprojects.com/en/8.0.x/): another way of building CLI tools which take arguments
- [sqlalchemy](https://www.sqlalchemy.org/): for a programmatic interface to a SQL database
- [fastapi](https://fastapi.tiangolo.com/): for a server
- [pydantic](https://pydantic-docs.helpmanual.io/): for validating data to server endpoints
- [pytest](https://docs.pytest.org/en/6.2.x/): for testing
- [beautifulsoup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/): for webscraping
- [rich](https://github.com/willmcgugan/rich): for pretty-printing to the terminal
- [hypothesis](https://hypothesis.readthedocs.io/en/latest/): for property-based testing
- [sphinx](https://www.sphinx-doc.org/en/master/): for documentation generation
- [altair](https://altair-viz.github.io/): for visualization
- [tqdm](https://github.com/tqdm/tqdm): for progress bars
- [sortedcontainers](http://www.grantjenks.com/docs/sortedcontainers/): for performant sorted containers
- [numpy](https://numpy.org/doc/stable/release/1.20.0-notes.html): for matrix / vector manipulations

🐍🛠️🐍🛠️🐍🛠️🐍🛠️🐍🛠️🐍🛠️🐍🛠️🐍🛠️🐍🛠️🐍🛠️