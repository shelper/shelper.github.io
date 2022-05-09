---
title: python module and packaging
date: 2016-06-13
tags: ["python"]
categories: ["programming"]
link:
description:
---

several things:

1. when using `setuptools.find_packages`, only packages with `__init__.py` in its folder will be found
2. only modules imported in the `__init__.py` will be indexed, means the modules will be added into the dir(module)
   and accessable by .module_name
3. for python 3, if one modules(py file) in the same folder of another module and relies on that module,
   you need to put `from . import module` at the top
4. package name is specified in the setup.py, while the module name can be different from the package name.
   e.g. if you have a structure package/setup.py and package/module/`__init__.py`, and in setup.py you set package name to be `package` then after you install that, you `import module`
   rather than `import package`, it is better to keep the package and module name the same.
5. try to use template with cookiecutter to make your life easier, and at the same time, modify the `__init__.py` when needed
6. exclude the test using `packages=setuptools.find_packages(exclude=["*.tests", "*.tests.*", "tests.*", "tests"])`
7. to include upper level modules, e.g., dir1/dir2/module2.py wants to include dir1/module1.py, do `from import module1`

then use `python setup.py sdist` to generate the zip and egg file and then use `python setup.py install` to install the package
