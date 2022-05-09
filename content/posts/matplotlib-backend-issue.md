<!-- 
.. title: matplotlib backend issue
.. slug: matplotlib-backend-issue
.. date: 2016-06-13 10:06:28 UTC
.. tags: python, matplotlib
.. category: programming 
.. link: 
.. description: 
.. type: text
-->

using different backends for matplotlib might cause some unexpected issues:
today, i am trying to run a program that uses the following function: 

```python
plt.figure()
figManager = plt.get_current_fig_manager()
figManager.window.showMaximized()
```

however, i found when using the backend of Tk, it gives error, saying get_current_fig_manager is not available, so when i switch to qt4egg, everything works well. 

how to switch the backend for matplotlib? first find the configure directory for matplotlib on your computer

```python
import matplotlib as plt
plt.get_configdir()
```

then find/generate a file named matplotlibrc, add line: 
```
backend      : qt4agg
```

reference [here](http://matplotlib.org/users/customizing.html)

another issue with using qt4 as backend is below from [stackoverflow](http://stackoverflow.com/questions/33984798/matplotlib-using-qt4-backend-has-issue-with-cached-renderer)
I was trying to use matplotlib to replot an image in a faster way, so instead of replotting everything, I use set_data method of the AxesImage class as below:

```python
import numpy as np
import time
import matplotlib.pyplot as plt
    
fig, ax = plt.subplots()
img = plt.imshow(np.random.rand(100, 100))
    
img.set_data(np.random.rand(100, 100))
ax.draw_artist(ax.patch)
ax.draw_artist(img)
fig.canvas.update()
fig.canvas.flush_events()
```

I encountered this error:

> Traceback (most recent call last):  ...
>     ax.draw_artist(ax.patch)   File "/Library/Frameworks/Python.framework/Versions/3.4/lib/python3.4/site-packages/matplotlib/axes/_base.py",
> line 2319, in draw_artist
>     raise AttributeError(msg) AttributeError: draw_artist can only be used after an initial draw which caches the render

However, if I run the script line by line in python shell (IPython), it works and nothing is wrong. So what is the mystery behind this cached renderer? 

EDIT: add a line `fig.canvas.draw()` solves the problem, now the remaining question is why running it line by line in IPython shell does not cause the same error?

