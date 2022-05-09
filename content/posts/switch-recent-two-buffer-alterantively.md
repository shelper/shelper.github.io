<!--
.. date: 2018-05-23 02:01:03 UTC
.. tags: meditation, private
.. category:
.. link:
.. description:
.. type: text
-->

so there is a post [described using key chord for fast buffer switching
between most recent two
buffers](http://emacsredux.com/blog/2013/04/28/switch-to-previous-buffer/).
it works for most cases, but if you are using org babel mode and editing
source code in the \*org src ... \* buffer, then this \*org src ... \*
buffer cannot be accessed by the method described here.

so I made another function to solve this issue, and here it is:

``` {.commonlisp .rundoc-block rundoc-language="emacs-lisp" rundoc-tangle="yes"}
(defun get-next-buffer ()
  (interactive)
  (if (string= (car (helm-buffer-list)) " *Minibuf-1*")
      (switch-to-buffer (car (cdr (helm-buffer-list))))
    (switch-to-buffer (car (helm-buffer-list)))))
(key-chord-define-global "jk" 'get-next-buffer)
```

strangely when switching buffer using \`helm-buffer-lists\`, and if you
exit without selecting any buffer, you gets \`\*Minibuf-1\*\` into the
buffer list and will switch to minibuffer... so here I compared the
first buffer name in the list to \`\*Minibuf-1\*\` and remove it if it
is for \`Minibuf-1\`

in addition i added the function and key binding below to enable:

1.  always change to normal mode when switch-to-buffer
2.  always move to the end of buffer for comint-mode and eshell mode

the code is here:

``` {.commonlisp .rundoc-block rundoc-language="emacs-lisp" rundoc-tangle="yes"}
;; goto buffer end and use insert mode if switch to a shell or comint mode
(defvar last-buffer nil)
(defun post-switch-buffer-func ()
  (if (not (eq (current-buffer) last-buffer))
      (progn
        (setq last-buffer (current-buffer))
        (if (derived-mode-p 'eshell-mode 'comint-mode)
            (progn
              ;; (evil-insert-state) ;uncomment for insert mode when switched to
              (end-of-buffer)))
        (if (derived-mode-p 'prog-mode)
            (evil-normal-state)))))
(add-hook 'post-command-hook 'post-switch-buffer-func)
```
