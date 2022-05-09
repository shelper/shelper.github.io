---
title: Orgmode for nikola post
date: 2018-05-29
tags: ["nikola", "orgmode"]
categories: ["programming"]
link:
description:
---

first need to setup the orgmode plugin for nikola refer to
[here](https://plugins.getnikola.com/v7/orgmode/), then put this into
init.el

```{.commonlisp}
;;; set up new nikola new post directly in emacs
;; does not work on windows
(defun publish-blog-post ()
  "nikola github-deploy"
  (interactive)
  (save-buffer)
  (let ((my-blog-repo  "path to your blog repo")))
  ;; (let (my-blog-repo))
  ;; (setq my-blog-repo "~/shelper.github.io")
  (cd my-blog-repo)
  (shell-command "nikola github_deploy"))

(defun new-blog-post (title)
  "new blog post to shelper.github.io"
  (interactive "sEnter post title: ")
  (let ((my-blog-repo  "path to your blog repo")))
  (cd my-blog-repo)
  (setq new-post-cmd (concat "nikola new_post -f orgmode -t " "\"" title "\""))
  (shell-command new-post-cmd)
  (setq new-post-file (concat my-blog-repo "/posts/" (replace-regexp-in-string " " "-" title) ".org"))
  (find-file new-post-file))
```

however, it seems mac does not work for org in nikola
refer to: [here](https://github.com/getnikola/plugins/issues/124)
