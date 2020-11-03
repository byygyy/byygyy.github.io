---
title: CentOS8使用podman常见错误解决
date: 2020-04-23 18:48:37
tags: podman
categories: docker
---

podman 遇到there might not be enough IDs available in the namespace。
<!--more-->


# 1.podman 遇到there might not be enough IDs available in the namespace

## 1.1发现错误：
使用rootless用户pull ubuntu镜像，竟然报这种错误。
```
[javadm@instance-2 ~]$ docker pull ubuntu
ERRO[0000] cannot find mappings for user javadm: No subuid ranges found for user "javadm" in /etc/subuid
ERRO[0000] cannot find mappings for user javadm: No subuid ranges found for user "javadm" in /etc/subuid
Trying to pull registry.access.redhat.com/ubuntu...
  name unknown: Repo not found
Trying to pull registry.fedoraproject.org/ubuntu...
  manifest unknown: manifest unknown
Trying to pull registry.centos.org/ubuntu...
  manifest unknown: manifest unknown
Trying to pull docker.io/library/ubuntu...
Getting image source signatures
Copying blob 6154df8ff988 done
Copying blob d51af753c3d3 done
Copying blob fee5db0ff82f done
Copying blob fc878cd0a91c done
Copying config 1d622ef86b done
Writing manifest to image destination
Storing signatures
  Error processing tar file(exit status 1): there might not be enough IDs available in the namespace (requested 0:42 for /etc/gshadow): lchown /etc/gshadow: invalid argument
Error: error pulling image "ubuntu": unable to pull ubuntu: 4 errors occurred:
        * Error initializing source docker://registry.access.redhat.com/ubuntu:latest: Error reading manifest latest in registry.access.redhat.com/ubuntu: name unknown: Repo not found
        * Error initializing source docker://registry.fedoraproject.org/ubuntu:latest: Error reading manifest latest in registry.fedoraproject.org/ubuntu: manifest unknown: manifest unknown
        * Error initializing source docker://registry.centos.org/ubuntu:latest: Error reading manifest latest in registry.centos.org/ubuntu: manifest unknown: manifest unknown
        * Error committing the finished image: error adding layer with blob "sha256:d51af753c3d3a984351448ec0f85ddafc580680fd6dfce9f4b09fdb367ee1e3e": Error processing tar file(exit status 1): there might not be enough IDs available in the namespace (requested 0:42 for /etc/gshadow): lchown /etc/gshadow: invalid argument

```

## 1.2解决问题：
**1./etc/subuid和/etc/subgid 增加子用户映射**
````
[root@instance-2 ~]# echo javadm:200000:300006666 >> /etc/subuid
[root@instance-2 ~]# echo javadm:300000:400006666 >> /etc/subgid
[root@instance-2 ~]# cat /etc/subuid
robin:100000:65536
javadm:200000:300006666
[root@instance-2 ~]# cat /etc/subgid
robin:100000:65536
javadm:300000:400006666

````
验证发现还报错：
```
[root@instance-2 ~]# su - javadm
Last login: Fri Apr 24 13:15:11 UTC 2020 on pts/0
[javadm@instance-2 ~]$ docker pull ubuntu
Trying to pull registry.access.redhat.com/ubuntu...
  name unknown: Repo not found
Trying to pull registry.fedoraproject.org/ubuntu...
  manifest unknown: manifest unknown
Trying to pull registry.centos.org/ubuntu...
  manifest unknown: manifest unknown
Trying to pull docker.io/library/ubuntu...
Getting image source signatures
Copying blob fee5db0ff82f done
Copying blob fc878cd0a91c done
Copying blob 6154df8ff988 done
Copying blob d51af753c3d3 done
Copying config 1d622ef86b done
Writing manifest to image destination
Storing signatures
  Error processing tar file(exit status 1): there might not be enough IDs available in the namespace (requested 0:42 for /etc/gshadow): lchown /etc/gshadow: invalid argument
Error: error pulling image "ubuntu": unable to pull ubuntu: 4 errors occurred:
        * Error initializing source docker://registry.access.redhat.com/ubuntu:latest: Error reading manifest latest in registry.access.redhat.com/ubuntu: name unknown: Repo not found
        * Error initializing source docker://registry.fedoraproject.org/ubuntu:latest: Error reading manifest latest in registry.fedoraproject.org/ubuntu: manifest unknown: manifest unknown
        * Error initializing source docker://registry.centos.org/ubuntu:latest: Error reading manifest latest in registry.centos.org/ubuntu: manifest unknown: manifest unknown
        * Error committing the finished image: error adding layer with blob "sha256:d51af753c3d3a984351448ec0f85ddafc580680fd6dfce9f4b09fdb367ee1e3e": Error processing tar file(exit status 1): there might not be enough IDs available in the namespace (requested 0:42 for /etc/gshadow): lchown /etc/gshadow: invalid argument
```

**2.再次修改/etc/subuid和/etc/subgid**
```
[javadm@instance-2 ~]$ cat /etc/subuid
robin:100000:65536
javadm:200000:300006666

[javadm@instance-2 ~]$ cat /etc/subgid
robin:100000:65536
javadm:400000000:400006666
```

错误依旧：
```
[javadm@instance-2 ~]$ docker pull ubuntu
Trying to pull registry.access.redhat.com/ubuntu...
  name unknown: Repo not found
Trying to pull registry.fedoraproject.org/ubuntu...
  manifest unknown: manifest unknown
Trying to pull registry.centos.org/ubuntu...
  manifest unknown: manifest unknown
Trying to pull docker.io/library/ubuntu...
Getting image source signatures
Copying blob 6154df8ff988 done
Copying blob fc878cd0a91c done
Copying blob fee5db0ff82f done
Copying blob d51af753c3d3 done
Copying config 1d622ef86b done
Writing manifest to image destination
Storing signatures
  Error processing tar file(exit status 1): there might not be enough IDs available in the namespace (requested 0:42 for /etc/gshadow): lchown /etc/gshadow: invalid argument
Error: error pulling image "ubuntu": unable to pull ubuntu: 4 errors occurred:
        * Error initializing source docker://registry.access.redhat.com/ubuntu:latest: Error reading manifest latest in registry.access.redhat.com/ubuntu: name unknown: Repo not found
        * Error initializing source docker://registry.fedoraproject.org/ubuntu:latest: Error reading manifest latest in registry.fedoraproject.org/ubuntu: manifest unknown: manifest unknown
        * Error initializing source docker://registry.centos.org/ubuntu:latest: Error reading manifest latest in registry.centos.org/ubuntu: manifest unknown: manifest unknown
        * Error committing the finished image: error adding layer with blob "sha256:d51af753c3d3a984351448ec0f85ddafc580680fd6dfce9f4b09fdb367ee1e3e": Error processing tar file(exit status 1): there might not be enough IDs available in the namespace (requested 0:42 for /etc/gshadow): lchown /etc/gshadow: invalid argument

```

**3.修改user.max_user_namespaces并且大于最大的UID**
```
  121  cd /etc/sysctl.d/
  122  ll
  123  touch podman.conf
  124  echo user.max_user_namespaces = 900000000 >>podman.conf
  125  cat podman.conf
  126  reboot
```
错误依旧：
```
[javadm@instance-2 ~]$ docker pull ubuntu
Trying to pull registry.access.redhat.com/ubuntu...
  name unknown: Repo not found
Trying to pull registry.fedoraproject.org/ubuntu...
  manifest unknown: manifest unknown
Trying to pull registry.centos.org/ubuntu...
  manifest unknown: manifest unknown
Trying to pull docker.io/library/ubuntu...
Getting image source signatures
Copying blob fee5db0ff82f done
Copying blob fc878cd0a91c done
Copying blob d51af753c3d3 done
Copying blob 6154df8ff988 done
Copying config 1d622ef86b done
Writing manifest to image destination
Storing signatures
  Error processing tar file(exit status 1): there might not be enough IDs available in the namespace (requested 0:42 for /etc/gshadow): lchown /etc/gshadow: invalid argument
Error: error pulling image "ubuntu": unable to pull ubuntu: 4 errors occurred:
        * Error initializing source docker://registry.access.redhat.com/ubuntu:latest: Error reading manifest latest in registry.access.redhat.com/ubuntu: name unknown: Repo not found
        * Error initializing source docker://registry.fedoraproject.org/ubuntu:latest: Error reading manifest latest in registry.fedoraproject.org/ubuntu: manifest unknown: manifest unknown
        * Error initializing source docker://registry.centos.org/ubuntu:latest: Error reading manifest latest in registry.centos.org/ubuntu: manifest unknown: manifest unknown
        * Error committing the finished image: error adding layer with blob "sha256:d51af753c3d3a984351448ec0f85ddafc580680fd6dfce9f4b09fdb367ee1e3e": Error processing tar file(exit status 1): there might not be enough IDs available in the namespace (requested 0:42 for /etc/gshadow): lchown /etc/gshadow: invalid argument
```

**4.再想办法**
```
[javadm@instance-2 ~]$ getcap /usr/bin/newuidmap
/usr/bin/newuidmap = cap_setuid+ep
podman system migrate
```

**5.关闭selinux再试**
```
[root@instance-2 ~]# setenforce 0
[root@instance-2 ~]# su - javadm
Last login: Fri Apr 24 14:21:15 UTC 2020 on pts/0
[javadm@instance-2 ~]$ getenforce
Permissive
```

**6.回退subuid和subgid的修改**
```

[javadm@instance-2 ~]$ cat /etc/subuid
robin:100000:65536
javadm:200000:300006666
[javadm@instance-2 ~]$ cat /etc/subgid
robin:100000:65536
javadm:400000000:400006666
[javadm@instance-2 ~]$

```

**7.做一些更改，最重要的**
```
echo user.max_user_namespaces=900000000  >> /etc/sysctl.d/userns.conf

**[javadm@instance-2 ~]$ cat /etc/subuid
robin:100000:65536
javadm:165536:65536
[javadm@instance-2 ~]$ cat /etc/subgid
robin:100000:65536
javadm:165536:65536**
[javadm@instance-2 ~]$

podman system migrate
```

错误依旧。
```
[javadm@instance-2 ~]$ docker pull ubuntu
Trying to pull registry.access.redhat.com/ubuntu...
  name unknown: Repo not found
Trying to pull registry.fedoraproject.org/ubuntu...
  manifest unknown: manifest unknown
Trying to pull registry.centos.org/ubuntu...
  manifest unknown: manifest unknown
Trying to pull docker.io/library/ubuntu...
Getting image source signatures
Copying blob 6154df8ff988 done
Copying blob d51af753c3d3 done
Copying blob fc878cd0a91c done
Copying blob fee5db0ff82f done
Copying config 1d622ef86b done
Writing manifest to image destination
Storing signatures
1d622ef86b138c7e96d4f797bf5e4baca3249f030c575b9337638594f2b63f01
[javadm@instance-2 ~]$

```

**8.最终解决**
```
[javadm@localhost ~]$ echo javadm:410000000:500000000 >> /etc/subuid
[javadm@localhost ~]$ echo javadm:410000000:500000000 >> /etc/subgid
[javadm@localhost ~]$ podman system migrate
[javadm@localhost ~]$ podman info
host:
  BuildahVersion: 1.12.0-dev
  CgroupVersion: v1
  Conmon:
    package: conmon-2.0.6-1.module_el8.1.0+298+41f9343a.x86_64
    path: /usr/bin/conmon
    version: 'conmon version 2.0.6, commit: 2721f230f94894671f141762bd0d1af2fb263239'
  Distribution:
    distribution: '"centos"'
    version: "8"
  IDMappings:
    gidmap:
    - container_id: 0
      host_id: 400001528
      size: 1
    - container_id: 1
      host_id: 410000000
      size: 500000000
    uidmap:
    - container_id: 0
      host_id: 300005526
      size: 1
    - container_id: 1
      host_id: 410000000
      size: 500000000
  MemFree: 61030400
  MemTotal: 500600832
  OCIRuntime:
    name: runc
    package: runc-1.0.0-64.rc9.module_el8.1.0+298+41f9343a.x86_64
    path: /usr/bin/runc
    version: 'runc version spec: 1.0.1-dev'
  SwapFree: 2124136448
  SwapTotal: 2147479552
  arch: amd64
  cpus: 1
  eventlogger: file
  hostname: localhost.localdomain
  kernel: 4.18.0-80.el8.x86_64
  os: linux
  rootless: true
  slirp4netns:
    Executable: /usr/bin/slirp4netns
    Package: slirp4netns-0.4.2-3.git21fdece.module_el8.1.0+298+41f9343a.x86_64
    Version: |-
      slirp4netns version 0.4.2+dev
      commit: 21fdece2737dc24ffa3f01a341b8a6854f8b13b4
  uptime: 16m 24.88s
registries:
  blocked: null
  insecure: null
  search:
  - registry.access.redhat.com
  - registry.fedoraproject.org
  - registry.centos.org
  - docker.io
store:
  ConfigFile: /home/javadm/.config/containers/storage.conf
  ContainerStore:
    number: 0
  GraphDriverName: overlay
  GraphOptions:
    overlay.mount_program:
      Executable: /usr/bin/fuse-overlayfs
      Package: fuse-overlayfs-0.7.2-5.module_el8.1.0+298+41f9343a.x86_64
      Version: |-
        fuse-overlayfs: version 0.7.2
        FUSE library version 3.2.1
        using FUSE kernel interface version 7.26
  GraphRoot: /home/javadm/.local/share/containers/storage
  GraphStatus:
    Backing Filesystem: xfs
    Native Overlay Diff: "false"
    Supports d_type: "true"
    Using metacopy: "false"
  ImageStore:
    number: 0
  RunRoot: /tmp/run-300005526
  VolumePath: /home/javadm/.local/share/containers/storage/volumes
  
[javadm@iZj6cdyw9ivwn9a3j8q0nzZ ~]$  docker pull ubuntu
Trying to pull registry.access.redhat.com/ubuntu...
  name unknown: Repo not found
Trying to pull registry.fedoraproject.org/ubuntu...
  manifest unknown: manifest unknown
Trying to pull registry.centos.org/ubuntu...
  manifest unknown: manifest unknown
Trying to pull docker.io/library/ubuntu...
Getting image source signatures
Copying blob fee5db0ff82f skipped: already exists
Copying blob fc878cd0a91c skipped: already exists
Copying blob 6154df8ff988 skipped: already exists
Copying blob d51af753c3d3 skipped: already exists
Copying config 1d622ef86b done
Writing manifest to image destination
Storing signatures
1d622ef86b138c7e96d4f797bf5e4baca3249f030c575b9337638594f2b63f01
[javadm@iZj6cdyw9ivwn9a3j8q0nzZ ~]$ docker image list
REPOSITORY                   TAG      IMAGE ID       CREATED        SIZE
docker.io/library/ubuntu     latest   1d622ef86b13   33 hours ago   76.3 MB
[javadm@iZj6cdyw9ivwn9a3j8q0nzZ ~]$
```

## 3.总结问题
经过重复测试后，发现解决这种问题还是要先解决namespace分配的问题，正确的步骤应该这样。

**3.1检查现有用户的UID和GID，并且找出最大的ID。**
```
[javadm@iZj6cdyw9ivwn9a3j8q0nzZ ~]$ cat /etc/passwd|awk -F ':' '{print $3,$4}'|sort
0 0
1 1
11 0
12 100
14 50
193 193
2 2
28 28
300005526 400001528
3 4
4 7
5 0
59 59
6 0
65534 65534
7 0
72 72
74 74
8 12
81 81
93 93
992 988
993 989
994 990
995 992
996 993
997 994
998 996
999 997
[javadm@iZj6cdyw9ivwn9a3j8q0nzZ ~]$ cat /etc/group|awk -F ':' '{print $3}'|sort
0
1
10
100
11
12
15
18
19
190
193
2
20
21
22
28
3
33
35
36
39
4
400001528
5
50
54
59
6
63
65534
7
72
74
8
81
9
93
988
989
990
991
992
993
994
995
996
997
998
999
[javadm@iZj6cdyw9ivwn9a3j8q0nzZ ~]$

```
>用户和组配置文件中最大的ID是400001528

**3.2检查现有的/etc/subuid和/etc/subgid**
```
[vagrant@localhost ~]$ cat /etc/subuid
vagrant:100000:65536
[vagrant@localhost ~]$ cat /etc/subgid
vagrant:100000:65536
```
>最大的ID是100000+65536=165536

**3.3为javadm用户配置/etc/subuid和/etc/subgid**
```
[javadm@localhost ~]$ echo javadm:410000000:500000000 >> /etc/subuid
[javadm@localhost ~]$ echo javadm:410000000:500000000 >> /etc/subgid
[javadm@localhost ~]$ podman system migrate
[javadm@localhost ~]$ podman info
```
>子用户使用的subuid初始值应该超出现有被使用的范围，之前找到的最大值是400001528。所以这里我们从410000000开始，最后一位是计数器设置要大于容器内用户的UID/GID，这里设置500000000。


**3.4我们再来看下官方文档的说明：**
Upgrade to rootless containers
If you have upgraded from RHEL 7, you must configure subuid and subgid values manually for any existing user you want to be able to use rootless podman.

Using an existing user name and group name (for example, jill), set the range of accessible user and group IDs that can be used for their containers. Here are a couple of warnings:

Don’t include the rootless user’s UID and GID in these ranges
If you set multiple rootless container users, use unique ranges for each user
We recommend 65536 UIDs and GIDs for maximum compatibility with existing container images, but the number can be reduced
Never use UIDs or GIDs under 1000 or reuse UIDs or GIDs from existing user accounts (which, by default, start at 1000)

Here is an example:

`# echo "jill:165537:65536" >> /etc/subuid`
`# echo "jill:165537:65536" >> /etc/subgid`
The user/group jill is now allocated 65535 user and group IDs, ranging from 165537-231072. That user should be able to begin running commands to work with containers now.

**3.5容器启动后验证uidmap**
```
[javadm@iZj6cdyw9ivwn9a3j8q0nzZ ~]$ podman unshare cat /proc/self/uid_map
         0  300005526          1
         1  410000000  500000000
```
>容器中的用户uid 1对应宿主机的410000000，uid 2对应宿主机410000000-1+2，容器中的应用应用uid 300005526，对应宿主机410000000-1+300005526。以此类推。容器中最大用户ID不能超过500000000，符合我们的预期，验证通过。

**参考资料**
From Docker To Podman [link](http://www.ranta.ch/posts/FromDockerToPodman/)
Why can’t rootless Podman pull my image [link](https://www.redhat.com/sysadmin/rootless-podman)
there might not be enough IDs available in the namespace (system migrate doesn't work1)  [link](https://github.com/containers/libpod/issues/4921)
Rootless Podman on CentOS [link](http://blog.zencoffee.org/2020/02/rootless-podman-on-centos/)
Running rootless Podman as a non-root user [link](https://www.redhat.com/sysadmin/rootless-podman-makes-sense)
start to use podman [link](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/building_running_and_managing_containers/starting-with-containers_building-running-and-managing-containers)