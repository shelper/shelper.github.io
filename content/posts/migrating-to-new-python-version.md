+++
title = "Migrating to new python version with lots of dependencies"
author = ["shelper"]
tags = ["python"]
draft = false
+++

so i am using poetry to manage my venv for a project that relies on dozens of PYPI packages. i first thought it would be relatively easy to migrate python3.7 to the latest 3.10. but i was wrong, and i spent hours to fix it. Here is what i've learned.

first of all, you need to intaall [pyenv]({{< relref "#pyenv" >}}) for multiple version management, a caveat here is that if you use the suggested method to setup (through `curl https://pyenv.run | bash`), you need to set `git.core.symlinks` to true, 'cus pyenv relies on that for shims. you also need to run `eval "$(pyenv init -)"` to integrate pyenv with shell, then you need to use pyenv to select the python version for your project folder through `pyenv local $python_version` , after that you need to run `poetry env use $python_version` to set the env for [poetry]({{< relref "#poetry" >}})

now you can enter poetry shell and run poetry install to set the project up. however, if you use lots of packages, there might be dependencies to be resovled, and according to [poetry](https://github.com/python-poetry/poetry/issues/2094), this may takes long time. and sometimes even fails. For me i used the latest python3.10, but the numpy i specified in my poetry.lock file is an older version that does not supports 3.10, i then got strange errors that fail the installation.

So the solution is keep the poetry.lock file, but run poetry install, if some package failed, run poetry update package-name. if things get complicated, it maybe is a better idea to add packages gradually.
