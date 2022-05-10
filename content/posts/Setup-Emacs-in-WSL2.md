+++
title = "Emacs in WSL2"
author = ["shelper"]
lastmod = 2022-05-10T10:44:47-04:00
tags = ["emacs"]
draft = false
weight = 1001
+++

this is for setting up emacs in [WSL2]({{< relref "#d41d8c" >}})

-   in order to add a context menu to open windows files in WSL2, i created a python file with content below:

    ```python
    import sys
    import subprocess
    import os


    if __name__ == "__main__":
        file_name = sys.argv[1]
        file_name = os.path.abspath(file_name)

        file_name = file_name.replace("C:\\", "/mnt/c/")
        file_name = file_name.replace("\\", "/")
        print(file_name)
        command = [
            "C:/Windows/System32/wsl.exe",
            "-d",
            "Ubuntu",
            "emacsclient",
            "-n",
            file_name,
        ]
        subprocess.Popen(command, shell=True)
    ```

-   then i setup the context menu by the following reg file:

    > Windows Registry Editor Version 5.00
    >
    > [HKEY_CLASSES_ROOT\\\*\shell\Open in WSL Emacs]
    > @="WSL Emacs"
    > "icon"="\\"C:\\\Users\\\MPNV38\\\scoop\\\apps\\\emacs\\\current\\\share\\\icons\\\hicolor\\\scalable\\\apps\\\emacs.ico\\""
    >
    > [HKEY_CLASSES_ROOT\\\*\shell\Open in WSL Emacs\Command]
    > @="\\"C:\\\Users\\\MPNV38\\\scoop\\\apps\\\python\\\current\\\pythonw.exe\\" \\"C:\\\PortableApps\\\MyTools\\\wsl_emacs.py\\" \\"%1\\""

This works fine on windows as long you have `pythonw.exe` and you have the icon file specified above exists

by the method above, you may still have issue to bring the emacs window to focus, For that you can use the [Python]({{< relref "#d41d8c" >}}) `wingui32` package, see the implementation below:

```python
import win32gui

class WindowMgr:
    """Encapsulates some calls to the winapi for window management"""

    def __init__(self):
        """Constructor"""
        self._handle = None

    def find_window(self, class_name, window_name=None):
        """find a window by its class_name"""
        self._handle = win32gui.FindWindow(class_name, window_name)

    def _window_enum_callback(self, hwnd, wildcard):
        """Pass to win32gui.EnumWindows() to check all the opened windows"""
        if re.match(wildcard, str(win32gui.GetWindowText(hwnd))) is not None:
            self._handle = hwnd

    def find_window_wildcard(self, wildcard):
        """find a window whose title matches the wildcard regex"""
        self._handle = None
        win32gui.EnumWindows(self._window_enum_callback, wildcard)

    def set_foreground(self):
        """put the window in the foreground"""
        win32gui.SetForegroundWindow(self._handle)
# at the end of your __main__ section, you can then bring the window back to focus with window #+name:
if __name == "__main__":
    # ...
    subprocess.Popen(command, shell=True)

    w = WindowMgr()
    w.find_window_wildcard(".*Doom Emac")
    w.set_foreground()
```
