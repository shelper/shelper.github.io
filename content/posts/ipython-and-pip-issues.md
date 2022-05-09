---
title: ipython and pip issues
date: 2016-06-09
tags: ["ipython", "python"]
categories: ["programming"]
link:
description:
---

so I am trying to use ipython as my python shell under emacs, since i started using python in emacs, i always encounter different issues, again and again, so i will use this post as a collection of issues and solutions.

**issue1:**

ipython shell imports quite some packages/modules/functions and as you may know, `import` searches current folder as well as the subfolders of the registered module, if there are file with the same name under the current folder, it overrides the file in the registered module, apparently, this causes problems of "import error, could not find ..."

so,watch out how you name your files, and if it does happen, consider to rename or remove the files

**issue2:**

pip install, i once encountered that pip keeps reinstall packages, without aware of the package is already installed.
the issue is in the pip config file (usually in `~/pip/pip.ini`) i have following line

```
ignore-installed = true
```

apparently, that is what this for, so remove this line solves the problem

**issue3:**

how to setup proxy when you are behind a corporation firewall?
goto the config file again, and put

```
[global]
proxy = url:port
```

this works for many other python based packages/software as well

**issue4**

when using emacs, calling `git` using shell-command seems not working well, you might want to use magit instead.

PS: I will keep updating this post if I find new issues and solutions.
