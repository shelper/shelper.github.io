---
title: using emacs for blog post using nikola and github pages
date: 2016-05-23
tags: ["emacs"]
categories: ["programming"]
link:
description:
---

so it is a long story, to make it short, here is how it works step by step:

- install nikola using `pip install nikola markdown webassets`
- init nikola site by `nikola init --quiet folder_name`
- change the configurations of the default to your configurations just like below diff file shows:

```diff
--- default conf.py
+++ customized conf.py
@@ -17,16 +17,16 @@ import time


# Data about this site
-BLOG_AUTHOR = "Your Name"  # (translatable)
-BLOG_TITLE = "Demo Site"  # (translatable)
+BLOG_AUTHOR = "Shelper"  # (translatable)
+BLOG_TITLE = "The Way As It Is"  # (translatable)
# This is the main URL for your site. It will be used
# in a prominent link. Don't forget the protocol (http/https)!
-SITE_URL = "https://example.com/"
+SITE_URL = "http://shelper.github.io/"
# This is the URL where Nikola's output will be deployed.
# If not set, defaults to SITE_URL
# BASE_URL = "https://example.com/"
-BLOG_EMAIL = "joe@demo.site"
-BLOG_DESCRIPTION = "This is a demo site for Nikola."  # (translatable)
+BLOG_EMAIL = "shelper@github"
+BLOG_DESCRIPTION = "A Site for Meaningful Things"  # (translatable)

# Nikola is multilingual!
#
@@ -171,11 +171,13 @@ POSTS = (
    ("posts/*.rst", "posts", "post.tmpl"),
    ("posts/*.txt", "posts", "post.tmpl"),
    ("posts/*.html", "posts", "post.tmpl"),
+    ("posts/*.md", "posts", "post.tmpl"),
)
PAGES = (
    ("stories/*.rst", "stories", "story.tmpl"),
    ("stories/*.txt", "stories", "story.tmpl"),
    ("stories/*.html", "stories", "story.tmpl"),
+    ("stories/*.md", "stories", "story.tmpl"),
)


@@ -546,11 +548,11 @@ REDIRECTIONS = []
# https://getnikola.com/handbook.html#deploying-to-github
# For user.github.io OR organization.github.io pages, the DEPLOY branch
# MUST be 'master', and 'gh-pages' for other repositories.
-# GITHUB_SOURCE_BRANCH = 'master'
-# GITHUB_DEPLOY_BRANCH = 'gh-pages'
+GITHUB_SOURCE_BRANCH = 'source'
+GITHUB_DEPLOY_BRANCH = 'master'

# The name of the remote where you wish to push to, using github_deploy.
-# GITHUB_REMOTE_NAME = 'origin'
+GITHUB_REMOTE_NAME = 'origin'

# Whether or not github_deploy should commit to the source branch automatically
# before deploying.
@@ -829,7 +831,7 @@ COMMENT_SYSTEM = "disqus"
# depends on what comment system you use. The default is
# "nikolademo" which is a test account for Disqus. More information
# is in the manual.
-COMMENT_SYSTEM_ID = "nikolademo"
+COMMENT_SYSTEM_ID = "shelper"

# Enable annotations using annotateit.org?
# If set to False, you can still enable them for individual posts and pages
```

- follow nikola handbook on how to use github pages [deploy to github](https://getnikola.com/handbook.html#id44)
- setup emacs to edit your post and here is my functions:

```lisp
(setq my-blog-repo "~/develop/shelper.github.io")
(defun my-new-blog-post (title)
"new blog post to shelper.github.io"
(interactive "sEnter post title: ")
(cd my-blog-repo)
(setq new-post-cmd (concat "nikola new_post -f markdown -t " "\"" title "\""))
(shell-command new-post-cmd)
  (setq new-post-file (concat my-blog-repo
                          "/posts/"
                          (replace-regexp-in-string " " "-" title)
                          ".md"))
    (find-file new-post-file))
(global-set-key (kbd "C-c C-p") 'my-new-blog-post)
```

- realtime .md file html preview using [flymd](https://github.com/mola-T/flymd), I setup the flymd as below:

```lisp
;;; realtime markdown rendering
(defun my-flymd-browser-function (url)
  (if (string-equal system-type "windows-nt")
      (setq url (concat "file:/" url)))
  (if (string-equal system-type "darwin")
      (let ((process-environment (browse-url-process-environment)))
        (apply 'start-process
                (concat "firefox " url)
                nil
                "/usr/bin/open"
                (list "-a" "firefox" url)))
    (let ((browse-url-browser-function 'browse-url-firefox))
      (browse-url url))))
(setq flymd-browser-open-function 'my-flymd-browser-function)
(add-hook 'markdown-mode-hook 'flymd-flyit)
```

another thing that needs your attention is that you cannot indent code blocks, so code blocks have to start with no heading spaces or TABs
