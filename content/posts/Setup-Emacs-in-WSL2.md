+++
title = "Emacs in WSL2"
author = ["shelper"]
lastmod = 2022-05-12T13:03:05-04:00
tags = ["emacs"]
draft = false
weight = 1001
+++

Using Emacs on windows is always an imperfect experience, luckly windows10 comes with WSL2 now, which lets you install Emacs in WSL2 and use it on windows now.
I did some hacking to make the experience little bit better, which includes:

1.  open any file on windows in emacs@wsl2
2.  setup org-capture on chrome on windows and capture notes using emacs@wsl2
3.  open links in org-mode in emacs@wsl2 using windows default apps. e.g., open https links using chrome on windows, open document links in MS-office on windows, etc.
4.  copy paste sharing between windows and wsl2
5.  fix for some keyboard shortcuts that's been screened by windows by default
6.  setup outlook to create outlook links(uuid based) for emails/meetings/contacts in outlook, then open links in org-mode using outlook
7.  insert files into emacs@wsl2

lets go into the steps for each of the above fixes


## precondition {#precondition}

On windows 10 you can install [GWSL](https://github.com/Opticos/GWSL-Source) to enable emacs@wsl2 with gui, or you can setup your own VcXsrv so you can open emacs@wsl2 with gui.


## open files on windows using emacs@wsl2 {#open-files-on-windows-using-emacs-wsl2}

in order to add a context menu to open windows files in WSL2, i created a python file with content below:

```python
import sys
import subprocess
import os
import re

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


if __name__ == "__main__":

    file_name = (
        '"' + sys.argv[1] + '"'
        if sys.argv[1].startswith("org-protocol")
        else os.path.abspath(sys.argv[1]).replace("\\", "/").replace("C:", "/mnt/c")
    )

    wsl_path_prefix = "//wsl$/Ubuntu/"
    if file_name.startswith(wsl_path_prefix):
        file_name = file_name.replace(wsl_path_prefix, "/")

    command = " ".join(
        [
            "C:/Windows/System32/wsl.exe",
            "-d",
            "Ubuntu",
            "emacsclient",
            "-n",
            file_name,
        ]
    )
    subprocess.Popen(command, shell=True)

    w = WindowMgr()
    w.find_window_wildcard(".*Doom Emac")
    w.set_foreground()
```

-   then i setup the context menu by the following reg file:

    > Windows Registry Editor Version 5.00
    >
    > [HKEY_CLASSES_ROOT\\\*\shell\Open in WSL Emacs]
    > @="WSL Emacs"
    > "icon"="\\"C:\\\Users\\\username\\\scoop\\\apps\\\emacs\\\current\\\share\\\icons\\\hicolor\\\scalable\\\apps\\\emacs.ico\\""
    >
    > [HKEY_CLASSES_ROOT\\\*\shell\Open in WSL Emacs\Command]
    > @="\\"C:\\\Users\\\username\\\scoop\\\apps\\\python\\\current\\\pythonw.exe\\" \\"C:\\\Path\\&rarr;\\\wsl_emacs.py\\" \\"%1\\""

This works fine on windows as long you have `pythonw.exe` and you have the icon file specified above exists
if you use `python.exe` instead of `pythonw.exe`, it still works but will always generate a cmd window when you call emacs@wsl2.


## setup org capture  for emacs@wsl2 {#setup-org-capture-for-emacs-wsl2}

if you already have the above script `wsl_emacs.py` in place, then it is relatively simple, just run the reg file below

> Windows Registry Editor Version 5.00
>
> [HKEY_CLASSES_ROOT\org-protocol]
> @="URL:Org Protocol"
> "URL Protocol"=""
>
> [HKEY_CLASSES_ROOT\org-protocol\shell]
>
> [HKEY_CLASSES_ROOT\org-protocol\shell\open]
>
> [HKEY_CLASSES_ROOT\org-protocol\shell\open\command]
> @="\\"C:\\\Users\\\username\\\scoop\\\apps\\\python\\\current\\\pythonw.exe\\" \\"C:\\\Path\\&rarr;\\\wsl_emacs.py\\" \\"%1\\""

of coures you need to install the [org-capture extension](https://chrome.google.com/webstore/detail/org-capture/kkkjlfejijcjgjllecmnejhogpbcigdc), and add your own capture template for org-capture, see more details at [here](https://orgmode.org/worg/org-contrib/org-protocol.html)


## open links in org-mode in emacs@wsl2 using windows apps {#open-links-in-org-mode-in-emacs-wsl2-using-windows-apps}

this is done by installing the [wslu](https://github.com/wslutilities/wslu) on your wsl2, and then you can config your org-mode by:

```emacs-lisp
;; "Set org-file-apps based on the system type"
(when IS-WSL
  (setq org-file-apps '((remote . emacs)
                        (auto-mode . emacs)
                        (directory . emacs)
                        ("\\.mm\\'" . "wslview \"%s\"")
                        ("\\.x?html?\\'" . "wslview \"%s\"")
                        ("\\.pptx?\\'" . "wslview \"%s\"")
                        ("\\.xlsx?\\'" . "wslview \"%s\"")
                        ("\\.docx?\\'" . "wslview \"%s\"")
                        ("\\.pdf\\'" .  "wslview \"%s\"")))
  ;; need to set explorer for open weblinks and htmls seperately
  (setq browse-url-generic-program "/mnt/c/Program Files \(x86\)/Google/Chrome/Application/chrome.exe")
  )
```


## copy paste sharing between wsl2 and windows host {#copy-paste-sharing-between-wsl2-and-windows-host}

This can be easily done using GWSL above


## fix keyboard shortcut for C-shift-0 {#fix-keyboard-shortcut-for-c-shift-0}

I don't know why but it seems windows blocks this keybind to be used.
this is annoying, what i did is i remap this C-shift-0 to something i dont use with powertoys on windows, e.g., i remap C-shift-0 to C-shift-., then i have the following line to set my emacs keybinding

```emacs-lisp
;; Windows 10 blocks Ctrl-Shift-0, So we using powertoy to cheat the system
;; when we press "C-)", it becomes "C->"
(cond (IS-WSL (map! "C->" #'sp-forward-slurp-sexp))
      (t (map! "C-)" #'sp-forward-slurp-sexp)))
```


## Outlook for Org-mode {#outlook-for-org-mode}

-   add macro to outlook for copying link to objects in outlook
    -   check developer in `File > Options > Customize Ribbon > Main Tabs > Developer`
    -   in developer tab create new VBA macro, and input:

        ```nil
        Sub AddLinkToMessageInClipboard()
            Dim objMail As Object
            'was earlier Outlook.MailItem
            Dim doClipboard As New DataObject
            Dim message As String

            'One and ONLY one message muse be selected
            If Application.ActiveExplorer.Selection.Count <> 1 Then
                MsgBox ("Select one and ONLY one message.")
                Exit Sub
            End If

            Set objMail = Application.ActiveExplorer.Selection.Item(1)

            If objMail.Class = olMail Then
                doClipboard.SetText "[[outlook:" + objMail.EntryID + "][MESSAGE: " + objMail.Subject + " (" + objMail.SenderName + ")]]"
            ElseIf objMail.Class = olAppointment Then
                doClipboard.SetText "[[outlook:" + objMail.EntryID + "][MEETING: " + objMail.Subject + " (" + objMail.Organizer + ")]]"
            ElseIf objMail.Class = olTask Then
                doClipboard.SetText "[[outlook:" + objMail.EntryID + "][TASK: " + objMail.Subject + " (" + objMail.Owner + ")]]"
            ElseIf objMail.Class = olContact Then
                doClipboard.SetText "[[outlook:" + objMail.EntryID + "][CONTACT: " + objMail.Subject + " (" + objMail.FullName + ")]]"
            ElseIf objMail.Class = olJournal Then
                doClipboard.SetText "[[outlook:" + objMail.EntryID + "][JOURNAL: " + objMail.Subject + " (" + objMail.Type + ")]]"
            ElseIf objMail.Class = olNote Then
                doClipboard.SetText "[[outlook:" + objMail.EntryID + "][NOTE: " + objMail.Subject + " (" + " " + ")]]"
            Else
                doClipboard.SetText "[[outlook:" + objMail.EntryID + "][ITEM: " + objMail.Subject + " (" + objMail.MessageClass + ")]]"
            End If

            doClipboard.PutInClipboard
        End Sub
        ```
    -   in visual basic editor window select `Tools > References`, if the option is disabled (grayed) then try to close all office apps and restart it, then click `browse...` and add `FM20.dll`
    -   then run `SELFCERT.exe` in office folder and create personal certificate
    -   in visual basic editor window select `Tools > Digital signature`, choose the created certificate.
    -   you should be able to run the macro now, you can further add the macro to the quick access toolbar through `File > Options > QuickAccessToolbar > choose commands from: Macro > your Macro`
-   Create registry entry for handling links to outlook object:

    ```nil
    Windows Registry Editor Version 5.00

    [HKEY_CLASSES_ROOT\outlook]
    @="URL:Outlook Folders"
    "URL protocol"=""

    [HKEY_CLASSES_ROOT\outlook\DefaultIcon]
    @="C:\\Program Files\\Microsoft Office\\root\\Office16\\OUTLOOK.EXE"

    [HKEY_CLASSES_ROOT\outlook\shell]

    [HKEY_CLASSES_ROOT\outlook\shell\open]

    [HKEY_CLASSES_ROOT\outlook\shell\open\command]
    @="\"C:\\Program Files\\Microsoft Office\\root\\Office16\\OUTLOOK.EXE\" /select \"%1\""
    ```
-   Setup org-mode
    in your own emacs configuration put:

    ```emacs-lisp
    (org-link-set-parameters "outlook" :follow (lambda (id)
                                                 (cond (IS-WSL
                                                        (shell-command (concat "wslview " "outlook:" id)))
                                                       (IS-WINDOWS
                                                        (w32-shell-execute "open" (concat "outlook:" id)))
                                                       )))
    ```


## insert windows files into emacs@wsl2 {#insert-windows-files-into-emacs-wsl2}

since emacs runs in wsl, we cannot drag and drop files between windows and emacs@wsl2. so i wrote a simple script to get the path of files in windows, and convert it to path in wsl2

```python
import os
import sys

file_path = os.path.abspath(sys.argv[-1]).replace("\\", "/").replace("C:", "/mnt/c")

wsl_path_prefix = "//wsl$/Ubuntu/"
if file_path.startswith(wsl_path_prefix):
    file_path = file_path.replace(wsl_path_prefix, "/")

command = "echo " + file_path.strip() + "| clip"
os.system(command)
```

then I create the context menu `WSL Path` by the registry file

> Windows Registry Editor Version 5.00
>
> [HKEY_CLASSES_ROOT\\\*\shell\Copy WSL path]
> @="WSL Path"
>
> [HKEY_CLASSES_ROOT\\\*\shell\Copy WSL path\Command]
> @="\\"C:\\\Users\\\username\\\scoop\\\apps\\\python\\\current\\\pythonw.exe\\" \\"C:\\\path\\&rarr;\\\wsl_path.py\\" \\"%1\\""
