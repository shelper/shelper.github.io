+++
title = "setup hugo with github pages on github"
author = ["shelper"]
date = 2022-05-09
lastmod = 2022-05-09T14:44:28-04:00
tags = ["area", "blog", "hugo", "github"]
draft = false
weight = 1001
+++

I followed this [site](https://gohugo.io/hosting-and-deployment/hosting-on-github/) for setting up hugo blogging on <https://github.com/> using github.io, the instruction is little bit confusion, so just to clarify a little bit here.

1.  the instructions there uses `gh-pages` branch to publish your site, this is done by the action [peaceiris/actions-gh-pages@v3](https://github.com/peaceiris/actions-gh-pages).
    however, for user/organization.github.io pages, the default branch for publishing is `main`. so if you use this action, you need to change your page branch to `gh-pages` and set the directory to `/root`.

2.  if you are using project_name.github.io for a project, then [github]({{< relref "#d41d8c" >}}) defaults to `gh-pages` branch so you dont need to do anything.

3.  you dont need to use [peaceiris/actions-gh-pages@v3](https://github.com/peaceiris/actions-gh-pages) action if you setup your hugo build action appropriately.

also if you are using ox-hugo to publish your blog, and you are using subtree per blog post, then  you need to have `EXPORT_FILE_NAME` property setted to the file name to be exported as `.md` file and you cannot have links in the headlines of the subtree to be exported. otherwise, it will cause error of "org-find-olp: Heading not found on level ..."
