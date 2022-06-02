---
title: setup babun on windows for git and ssh
date: 2016-06-09
tags: ["emacs", "windows", "babun", "git", "ssh"]
categories: ["programming"]
link:
description:
---

1. remove %HOME% (you can put it back later)

2. download and install babun

3. setup ssh, if you already have ssh keys in the ~/.ssh, copy to the ~ of babun shell and do

```
eval `ssh-agent -s`
ssh-add
```

if not, copy the ssh pub and private keys to ~/.ssh or use

```
ssh-add path/to/keys
```

4. if you are using proxy behind firewall you need:
   add proxy for babun and change the user -agent (dont know why, but works)

```
export http_proxy=http://gateway.zscaler.net:80
export https_proxy=$http_proxy
export ftp_proxy=$http_proxy
export no_proxy=localhost
...

export USER_AGENT="Chrome/47.0.2526.111"

```

then you can use pact to install packages

```

pact install connect-proxy

```

if ssh need to go through proxy, add the line below to the ~/.ssh/config file

```

ProxyCommand connect-proxy.exe -H gateway.zscaler.net:80 %h %p

```

then add the lines below to .gitconfig

```

[http]
proxy = http://gateway.zscaler.net:80

```

However, this still cannot work for run `git push` etc., in emacs shellNo solution yet, a question of this is asked at [stackoverflow](http://stackoverflow.com/questions/34009160/unable-to-use-git-push-in-emacs-shell-under-windows)

```

```
