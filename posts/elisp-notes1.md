<!-- 
.. title: elisp notes1
.. slug: elisp-notes1
.. date: 2016-06-13 10:15:28 UTC
.. tags: emacs, elisp
.. category: programming 
.. link: 
.. description: 
.. type: text
-->

# quote for symbols:

```elisp
(setq symbol)
;; is equivelent to
(set 'symbol)
```
the quote means; keep as it is, dont try to evaluate, so if no quote, the lisp processor will try to evaluate it, means the symbol (or the first element of the list) has to be evaluatable as either a function, or a defined variable

# let vs. let*:
```elisp
(setq y 2)
(let ((y 1)
      (z y))
  (list y z)) ⇒ (1 2)

(setq y 2)
(let* ((y 1)
      (z y))
  (list y z)) ⇒ (1 1)
```

`let*` binds 1 to y immediately, while `let` evaluate old y as 2, then `list` binds 1 to the new y and pring it out 


# colon `:` and `#`:

`:` is used for constant symboles, it is commonly used as attributes for faces 
`#` with quote, i.e., `#'` is equal to `'`, the only difference is with `#`, it provides compiling information once the code is compiled.


TO BE CONTINUED
