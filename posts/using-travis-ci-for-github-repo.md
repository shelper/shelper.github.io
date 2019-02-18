<!-- 
.. title: using travis CI for github repo
.. slug: using-travis-ci-for-github-repo
.. date: 2017-10-06 10:56:45 UTC-04:00
.. tags: nikola, 
.. category: 
.. link: 
.. description: 
.. type: text
-->

using Nikola with Travis CI on github.io pages
major reference [here](https://getnikola.com/blog/automating-nikola-rebuilds-with-travis-ci.html)

it is noticeable that instead of running 
```
travis login
travis enable
travis encrypt-file id_rsa --add
```
we should run 
```
travis login
travis encrypt-file id_rsa --add
travis enable
```