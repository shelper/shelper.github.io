---
title: set up nikola for markdown
date: 2017-02-09
tags: ["nikola"]
categories: ["programming"]
link:
description:
---

to use nikola you need to install it with some other packages as well
`pip instal nikola markdown ws4py watchdog`

then to enable markdown, you need to change the `conf.py` variable `POSTS` and `PAGES`

```python
POSTS = (
    ("posts/*.md", "posts", "post.tmpl"),
    ("posts/*.rst", "posts", "post.tmpl"),
    ("posts/*.txt", "posts", "post.tmpl"),
    ("posts/*.html", "posts", "post.tmpl"),
)
PAGES = (
    ("pages/*.md", "pages", "story.tmpl"),
    ("pages/*.rst", "pages", "story.tmpl"),
    ("pages/*.txt", "pages", "story.tmpl"),
    ("pages/*.html", "pages", "story.tmpl"),
)

```

you may also need to change the github settings for easy github deploy

Then new post can be added using markdown format by
`nikola new_post -f markdown`
