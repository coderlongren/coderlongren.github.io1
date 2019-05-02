---
title: Docker  Hello World
date: 2018-4-25 14:18:40
categories:
	- Docker
tags:
	- Docker
---
摘要：Docker
<!-- more -->
刚在 Win10装好Docker，第一条命令当然是看一下成功了没？ 

	PS C:\Users\86298> docker -v
	Docker version 18.03.0-ce, build 0520e24
	PS C:\Users\86298> docker
OK -->  
运行下Hello world呢? 
	PS C:\Users\86298> docker run ubuntu:15.10 /bin/echo "hello world"

这里命令 `docker run ubuntu:15.10 /bin/echo "hello world"`
* docker: 就是用Docker二进制文件来执行命令

* run:与前面的 docker 组合来运行一个容器。

* ubuntu:15.10指定要运行的镜像，Docker首先从本地主机上查找镜像是否存在，如果不存在，Docker 就会从镜像仓库 Docker Hub 下载公共镜像。

* /bin/echo "Hello world": 在启动的容器里执行的命令
下面就是从源pull iamge的过程

	Unable to find image 'ubuntu:15.10' locally
	15.10: Pulling from library/ubuntu
	7dcf5a444392: Pull complete
	759aa75f3cee: Pull complete
	3fa871dc8a2b: Pull complete
	224c42ae46e7: Pull complete
	Digest: sha256:02521a2d079595241c6793b2044f02eecf294034f31d6e235ac4b2b54ffc41f3
	Status: Downloaded newer image for ubuntu:15.10
	hello world

 	docker run -i -t ubuntu:15.10 /bin/bash

模拟和Docker容器交互式
	-t:在新容器内指定一个伪终端或终端。

    -i:允许你对容器内的标准输入 (STDIN) 进行交互。  
下面你就使用这台ubuntu15.10主机了。

	root@de839238d52e:/# ls
	bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
	root@de839238d52e:/# echo "hello world"
	hello world
	root@de839238d52e:/# whoami
	root
	root@de839238d52e:/# top
	top - 06:25:36 up 9 min,  0 users,  load average: 0.32, 0.31, 0.18
	Tasks:   2 total,   1 running,   1 sleeping,   0 stopped,   0 zombie
	%Cpu(s):  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
	KiB Mem:   2027764 total,  1805460 used,   222304 free,     9344 buffers
	KiB Swap:  1048572 total,        0 used,  1048572 free.  1378564 cached Mem

	   PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
	     1 root      20   0   18208   3348   2872 S   0.0  0.2   0:00.03 bash
	    11 root      20   0   19912   2464   2096 R   0.0  0.1   0:00.00 top

##  构建自己的Docker镜像
`Dockerfile`
```
FROM    centos:6.7
MAINTAINER      Fisher "fisher@sudops.com"

RUN     /bin/echo 'root:123456' |chpasswd
RUN     useradd runoob
RUN     /bin/echo 'runoob:123456' |chpasswd
RUN     /bin/echo -e "LANG=\"en_US.UTF-8\"" >/etc/default/local
EXPOSE  22
EXPOSE  80
CMD     /usr/sbin/sshd -D
```
使用 Dockerfile 文件，通过 docker build 命令来构建一个镜像。s
	docker build -t runoob/centos:6.7 .
