+++
title = "Setup Modern Python development"
author = ["shelper"]
lastmod = 2022-06-01T21:50:31-04:00
tags = ["python"]
draft = false
weight = 1002
+++

First, you need to install [pyenv]({{< relref "#pyenv" >}}) so you can have multiple version of python in your system. then for projects that does not need packaging, you can use `pyenv-virtualenv` to setup `requirements.txt` for project based venv.

Second, you need to install [pipx]({{< relref "#pipx" >}}), so it installs some shared tools for the whole system. After setting up `pipx` (using certain version of python specified by `pyenv`), pipx reuses the python interpreter on which pipx was installed, so if you upgrade python and got the previous one removed, pipx will fail. it is a good practice to fix the python interpreter for `pipx` so that when you have your default python changed, your pipx does not break. This is done by `export PIPX_DEFAULT_PYTHON="$HOME/.pyenv/versions/x.y.z/bin/python"`

Third, for python projects you want to pack up for package releasing, you better install [poetry]({{< relref "#poetry" >}}) with `pipx` so that it does the building/packaging for you correctly.

for other commonly used python based tools. `pipx` would often be a better choice for installation too.
