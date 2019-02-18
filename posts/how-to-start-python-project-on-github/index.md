<!-- 
.. title: how to start python project on github
.. slug: how-to-start-python-project-on-github
.. date: 2016-06-13 10:06:28 UTC
.. tags: python, github, git
.. category: programming 
.. link: 
.. description: 
.. type: text
-->


steps are:

   1. install cookiecutter: `python -m pip install cookiecutter`
   2. run cookiecutter to initiate a project: `cookiecutter('https://github.com/audreyr/cookiecutter-pypackage.git')` under the project folder
   3. on github, start a new repo `python_prj`
   4. run `git init` in the local project folder and commit
   5. run `git remote add origin git@github.com:username/projectname.git`
    * using git@git requires to setup the ssh key pair, this can be found by easily google it or under FAQ of github
   6. run `git push -u origin master`

that is it, enjoy.

if you use pycharm it will add a folder called .idea under the root folder, this should also add to git repor if needed.

