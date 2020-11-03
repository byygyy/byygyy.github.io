---
title: Linux之shell命令别名
date: 2020-03-13 23:45:37
tags: shell
categories: linux
---

为什么每当你执行rm file_name的时候，会提示你确认并输入y后才会执行删除动作，而不忽略或直接删除？为什么每当你执行cp的时候，如果存在重名文件会提示你是否覆盖，而不忽略？ 为什么执行mv命令的时候，目标位置有重名文件会提示你是否覆盖？其实，是因为shell环境给这些命令定义了别名（alias name)。

<!--more-->

我们来查询下当前shell，定义了哪些别名，使用alias查询别名
```
[root@aliyun-hk1 linux-shell-test]# alias
alias cp='cp -i'
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'
alias grep='grep --color=auto'
alias l.='ls -d .* --color=auto'
alias ll='ls -l --color=auto'
alias ls='ls --color=auto'
alias mv='mv -i'
alias rm='rm -i'
alias which='alias | /usr/bin/which --tty-only --read-alias --show-dot --show-tilde'
```

当我们编写ansible脚本或者shell脚本的时候，为了防止目标服务器shell环境别名命令的差异导致的一系列问题，我们可以主动临时禁用别名，使用\cmd或者unalias cmd
```
[root@aliyun-hk1 linux-shell-test]# touch 1
[root@aliyun-hk1 linux-shell-test]# rm 1
rm: remove regular empty file ‘1’? y
[root@aliyun-hk1 linux-shell-test]# \rm 3
[root@aliyun-hk1 linux-shell-test]#
```

在自动化运维时代，我是不推荐定义太多的shell命令别名，除非这个server 是你自己使用的。当然定义少量的命令别名也是有好处理的，将复杂命令定义成别名，方面我们手工运维时提高效率。自动化运维时代嘛，难免有时候还是要手工在server 排查一些问题的。如何定义别名，请参考man alias帮助手册。

最后回答下最开始的提问，其实很简单，因为别名在发挥作用。​