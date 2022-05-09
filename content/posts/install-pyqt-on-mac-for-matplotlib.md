---
title: install pyqt on mac for matplotlib
date: 2016-06-13
tags: ["python", "matplotlib", "pyqt"]
categories: ["programming"]
link:
description:
---

so i was trying to use matplotlib for fast image ploting and updating, and I found a post [here](http://bastibe.de/2013-05-30-speeding-up-matplotlib.html).
the matplotlib uses different backend to render the images, and due to the differences between different backend, some functions are only available for specific backend, for instance, the `fig.canvas.update()` is a fast way to render new images to the axes, but is only availabe to PyQt4.

so i was trying to install pyqt4 using homebrew, it failed by saying the osx version is pre-released version and is not supported, and arises 2 errors during make process.

this is due to the cached info for homebrew is not up-to-date, so you need to run brew update.

however, this also arises an error of "not writable directory: /usr/local, so you need to `sudo chown - $user:admin /usr/local`, then update shall work, then run `brew install pyqt (add --with-python3 if need to support python3.x)`.

now the remaining problem is the pyqt package cannot be imported in python shell because homebrew gives the below message:

> Python modules have been installed and Homebrew's site-packages is not
> in your Python sys.path, so you will not be able to import the modules
> this formula installed. If you plan to develop with these modules,
> please run:
> echo 'import site; site.addsitedir("/usr/local/lib/python2.7/site-packages")' >> /path/to/your/python/folder/site-packages/homebrew.pth

you may run `import site; site.getsitepackages()` to find out the path to your python site-packages folder,
of course, you need to change python2.7 to python3.x for python3 support.

Another thing is use `python3 -m pip` instead of `pip` so to ensure the package will be installed in the default site-packages folder, actually `python3 -m pip install pyside` successfully installed pyside for python3

however, i found pyside 1.2.4 does not work due to the @rpath issue, cannot find a solution for it. so i installed pyside 1.2.2 and downloaded [https://raw.githubusercontent.com/PySide/pyside-setup/1.2.2/pyside_postinstall.py], after running this script, the pyside is succesfully installed and can be imported
