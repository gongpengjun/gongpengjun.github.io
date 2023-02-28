---
layout: post
title: docker-comopse运行etcd集群
date: 2023-02-28 09:00:00
categories: devops
---

doccker-compose运维etcd集群。

### 1、单实例etcd集群

doccker-compose管理的单实例etcd集群，支持数据持久化，支持公网访问，支持开机启动

配置文件`docker-compose.yml`:

```yaml
services:
    etcd-1:
        image: quay.io/coreos/etcd:v3.5.7
        entrypoint: /usr/local/bin/etcd
        command:
            - '--name=etcd-1'
            - '--initial-advertise-peer-urls=http://etcd-1:2380'
            - '--listen-peer-urls=http://0.0.0.0:2380'
            - '--listen-client-urls=http://0.0.0.0:2379'
            - '--advertise-client-urls=http://etcd-1:2379'
            - '--heartbeat-interval=250'
            - '--election-timeout=1250'
            - '--data-dir=/etcd-data'
            - '--initial-cluster=etcd-1=http://etcd-1:2380'
            - '--initial-cluster-state=new'
            - '--initial-cluster-token=mys3cr3ttok3n'
        restart: always
        ports:
            - "12379:2379"
        volumes:
            - ./data/etcd1:/etcd-data:rw
        networks:
            - bridge-network

networks:
    bridge-network:
        driver:  bridge
        enable_ipv6: false
```

说明：

- 使用官方镜像：quay.io/coreos/etcd:v3.5.7
- 使用bind volume，即映射宿主机目录`./data/etcd1`到容器`/etcd-data`目录，配合`--data-dir=/etcd-data`
- 禁用IPv6网络`enable_ipv6: false`，以便重启时不会报port已被占用错误
- 支持开机启动 `restart: always`
- 端口映射`"12379:2379"`将宿主机端口`12379`映射到容器`2379`端口，给公网访问使用



实际使用：

```shell
# 启动etcd cluster
$ docker-compose up -d
Creating network "etcd-single-node_bridge-network" with driver "bridge"
Creating etcd-single-node_etcd-1_1 ... done

# 查看容器状态
$ docker-compose ps
          Name                         Command               State                       Ports
-------------------------------------------------------------------------------------------------------------------
etcd-single-node_etcd-1_1   /usr/local/bin/etcd --name ...   Up      0.0.0.0:2379->2379/tcp, 0.0.0.0:2380->2380/tcp

# 查看data目录树，etcd已经初始化好了数据目录
$ tree
.
├── README.md
├── data
│   └── etcd1
│       └── member
│           ├── snap
│           │   └── db
│           └── wal
│               ├── 0.tmp
│               └── 0000000000000000-0000000000000000.wal
└── docker-compose.yml

5 directories, 5 files

# 集群只有一个etcd节点
$ etcdctl -w=table member list
+------------------+---------+--------+--------------------+--------------------+------------+
|        ID        | STATUS  |  NAME  |     PEER ADDRS     |    CLIENT ADDRS    | IS LEARNER |
+------------------+---------+--------+--------------------+--------------------+------------+
| b8c6addf901e4e46 | started | etcd-1 | http://etcd-1:2380 | http://etcd-1:2379 |      false |
+------------------+---------+--------+--------------------+--------------------+------------+

# 数据读写
$ etcdctl get secret
$ etcdctl put secret baby-im
OK
$ etcdctl get secret
secret
baby-im
```

### 2、3实例etcd集群

doccker-compose管理的3实例etcd集群，支持数据持久化，支持公网访问

配置文件`docker-compose.yml`:

```yaml
x-variables:
    flag_initial_cluster_token: &flag_initial_cluster_token '--initial-cluster-token=mys3cr3ttok3n'
    common_settings: &common_settings
        image: quay.io/coreos/etcd:v3.5.7
        entrypoint: /usr/local/bin/etcd
        ports:
            - 2379

services:
    etcd-1:
        <<: *common_settings
        command:
            - '--name=etcd-1'
            - '--initial-advertise-peer-urls=http://etcd-1:2380'
            - '--listen-peer-urls=http://0.0.0.0:2380'
            - '--listen-client-urls=http://0.0.0.0:2379'
            - '--advertise-client-urls=http://etcd-1:2379'
            - '--heartbeat-interval=250'
            - '--election-timeout=1250'
            - '--data-dir=/etcd-data'
            - '--initial-cluster=etcd-1=http://etcd-1:2380,etcd-2=http://etcd-2:2380,etcd-3=http://etcd-3:2380'
            - '--initial-cluster-state=new'
            - *flag_initial_cluster_token
        ports:
            - "12379:2379"
            - "12380:2380"
        volumes:
            - ./data/etcd1:/etcd-data:rw
        networks:
            - bridge-network

    etcd-2:
        <<: *common_settings
        command:
            - '--name=etcd-2'
            - '--initial-advertise-peer-urls=http://etcd-2:2380'
            - '--listen-peer-urls=http://0.0.0.0:2380'
            - '--listen-client-urls=http://0.0.0.0:2379'
            - '--advertise-client-urls=http://etcd-2:2379'
            - '--heartbeat-interval=250'
            - '--election-timeout=1250'
            - '--data-dir=/etcd-data'            
            - '--initial-cluster=etcd-1=http://etcd-1:2380,etcd-2=http://etcd-2:2380,etcd-3=http://etcd-3:2380'
            - '--initial-cluster-state=new'
            - *flag_initial_cluster_token
        ports:
            - "22379:2379"
            - "22380:2380"
        volumes:
            - ./data/etcd2:/etcd-data:rw
        networks:
            - bridge-network

    etcd-3:
        <<: *common_settings
        command:
            - '--name=etcd-3'
            - '--initial-advertise-peer-urls=http://etcd-3:2380'
            - '--listen-peer-urls=http://0.0.0.0:2380'
            - '--listen-client-urls=http://0.0.0.0:2379'
            - '--advertise-client-urls=http://etcd-3:2379'
            - '--heartbeat-interval=250'
            - '--election-timeout=1250'
            - '--data-dir=/etcd-data'
            - '--initial-cluster=etcd-1=http://etcd-1:2380,etcd-2=http://etcd-2:2380,etcd-3=http://etcd-3:2380'
            - '--initial-cluster-state=new'
            - *flag_initial_cluster_token
        ports:
            - "32379:2379"
            - "32380:2380"
        volumes:
            - ./data/etcd3:/etcd-data:rw
        networks:
            - bridge-network

networks:
    bridge-network:
        driver:  bridge
        enable_ipv6: false
```

实际使用：

```shell
$ docker-compose up -d
Creating network "etcd-cluster_default" with the default driver
Creating etcd-cluster_etcd-3_1 ... done
Creating etcd-cluster_etcd-2_1 ... done
Creating etcd-cluster_etcd-1_1 ... done
$ docker-compose ps
        Name                       Command               State                 Ports
--------------------------------------------------------------------------------------------------
etcd-cluster_etcd-1_1   /usr/local/bin/etcd --name ...   Up      0.0.0.0:32773->2379/tcp, 2380/tcp
etcd-cluster_etcd-2_1   /usr/local/bin/etcd --name ...   Up      0.0.0.0:32771->2379/tcp, 2380/tcp
etcd-cluster_etcd-3_1   /usr/local/bin/etcd --name ...   Up      0.0.0.0:32772->2379/tcp, 2380/tcp
$ docker exec -it etcd-cluster_etcd-1_1 etcdctl --write-out=table member list
+------------------+---------+--------+--------------------+--------------------+------------+
|        ID        | STATUS  |  NAME  |     PEER ADDRS     |    CLIENT ADDRS    | IS LEARNER |
+------------------+---------+--------+--------------------+--------------------+------------+
| 88d11e2649dad027 | started | etcd-2 | http://etcd-2:2380 | http://etcd-2:2379 |      false |
| b8c6addf901e4e46 | started | etcd-1 | http://etcd-1:2380 | http://etcd-1:2379 |      false |
| c3697a4fd7a20dcd | started | etcd-3 | http://etcd-3:2380 | http://etcd-3:2379 |      false |
+------------------+---------+--------+--------------------+--------------------+------------+
```

### 参考

- [docker-compose-etcd-single-node](https://github.com/gongpengjun/docker-compose-etcd-single-node)
- [docker-compose-etcd-cluster](https://github.com/gongpengjun/docker-compose-etcd-cluster)
