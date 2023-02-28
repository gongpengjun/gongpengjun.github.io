---
layout: post
title: Docker常用命令
date: 2023-02-27 09:00:00
categories: devops
---

### 1、查看Docker Volume内幕

查看名为`todo-db`的docker volume的信息。

```shell
$ docker volume create todo-db
todo-db
$ docker volume inspect todo-db
[
    {
        "CreatedAt": "2023-02-28T07:11:09Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/todo-db/_data",
        "Name": "todo-db",
        "Options": {},
        "Scope": "local"
    }
]
```

`Mountpoint`是该volume在宿主机的文件系统的路径

在Mac上，Docker是运行在一个LinuxKit的虚拟机里面的，所以Docker的宿主机是LinuxKit，

所以进入Mac版Docker LinuxKit虚拟机查看：

```shell
$ docker run -it --rm --privileged --pid=host alpine:edge nsenter -t 1 -m -u -n -i sh
/ # ls /var/lib/docker/volumes/todo-db/_data
hello.txt
```

参考：

- [Docker/Guides/Get started/Part 5: Persist the DB](https://docs.docker.com/get-started/05_persisting_data/)
- [Getting a Shell in the Docker Desktop Mac VM](https://gist.github.com/BretFisher/5e1a0c7bcca4c735e716abf62afad389)
- [Justin Cormack (Docker Maintainer)'s nsenter1 Repo](https://github.com/justincormack/nsenter1)
- [Under the Hood: Demystifying Docker Desktop For Mac](https://collabnix.com/how-docker-for-mac-works-under-the-hood/)



