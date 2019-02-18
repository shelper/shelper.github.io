<!--
.. date: 2018-05-24 02:01:03 UTC
.. tags: meditation, private
.. category:
.. link:
.. description:
.. type: text
-->

A few things to setup the IPython and Jupyter for a best practise.

### IPython configuration

Change `~/.ipython/profile_default/startup/init.ipy` for their usage
respectively

-   To enable versioning of the python/jupyter environment, you need to
    `pip install version_information` and then add below, afterward for
    each IPython notebook file I put
    `%version_information numpy, scipy, matplotlib, pandas` to display
    my python environment version information

``` {.python .rundoc-block rundoc-language="python" rundoc-session="yes" rundoc-tangle="yes"}
%load_ext version_information
```

-   To enable autoloading of modified modules in IPython, add below

``` {.python .rundoc-block rundoc-language="python" rundoc-session="yes" rundoc-tangle="yes"}
%load_ext version_information
%load_ext autoreload
%autoreload 2
```

-   I also added the lines below just because I use them everytime I
    open IPython

``` {.python .rundoc-block rundoc-language="python" rundoc-session="yes" rundoc-tangle="yes"}
import numpy as np
import matplotlib.pyplot as plt
plt.ion()
```

### Modify Jupyter notebook settings:

-   To enable auto conversion and saving of `.py` and `.html` files from
    the notebook, put the following script to your Jupyter
    configuration: `~/.jupyter/jupyter_notebook_config.py`, more details
    can be found at
    [here](http://jupyter-notebook.readthedocs.io/en/latest/extending/savehooks.html)

``` {.python .rundoc-block rundoc-language="python" rundoc-session="yes" rundoc-tangle="yes"}

# make a script equivalent whenever the notebook is saved
import io
import os
from notebook.utils import to_api_path

_script_exporter = None
_html_exporter = None

def script_post_save(model, os_path, contents_manager, **kwargs):
    """convert notebooks to Python script after save with nbconvert

    replaces `ipython notebook --script`
    """
    from nbconvert.exporters.script import ScriptExporter
    from nbconvert.exporters.html import HTMLExporter

    if model['type'] != 'notebook':
        return

    global _script_exporter

    if _script_exporter is None:
        _script_exporter = ScriptExporter(parent=contents_manager)

    log = contents_manager.log

    base, ext = os.path.splitext(os_path)
    # py_fname = base + '.py'
    script, resources = _script_exporter.from_filename(os_path)
    script_fname = base + resources.get('output_extension', '.txt')
    log.info("Saving script /%s", to_api_path(script_fname, contents_manager.root_dir))

    with io.open(script_fname, 'w', encoding='utf-8') as f:
        f.write(script)

    global _html_exporter
    if _html_exporter is None:
        _html_exporter = HTMLExporter(parent=contents_manager)
    html, resources = _html_exporter.from_filename(os_path)
    html_fname = base + resources.get('output_extension', '.txt')
    log.info("Saving html /%s", to_api_path(html_fname, contents_manager.root_dir))

    with io.open(html_fname, 'w', encoding='utf-8') as f:
        f.write(html)

c.FileContentsManager.post_save_hook = script_post_save
```

### For better visualization and vim keybindings

-   Jupyter theme can be added following
    [here](https://github.com/dunovank/jupyter-themes), I prefer to use
    onedork with different fonts for text and code blocks, so basically
    want you can do is

``` {.bash .rundoc-block rundoc-language="sh" rundoc-session="yes" rundoc-tangle="yes"}
pip install --upgrade jupyterthemes
jt -t onedork -tf georgiaserif -nf droidsans -vim
```
