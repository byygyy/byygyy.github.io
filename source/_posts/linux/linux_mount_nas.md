---
title: linux mount the nas or windows share
date: 2019-11-21 18:27:37
tags: nas
categories: linux
---

### refer to:
https://www.seagate.com/cn/zh/support/kb/how-to-mount-nfs-and-cifs-file-systems-on-linux-with-the-seagate-blackarmor-nas-209791en/
http://linux.vbird.org/linux_server/0330nfs.php
https://www.tutorialspoint.com/unix_commands/showmount.htm

### How to mount nas to a linux folder:
```bash
mount -t nfs server_fqdn:/prod/ddddd_dr1   /nas/name1
```

### How to list nas resource at linus server:
```bash
showmount -d server_fqdn
```
