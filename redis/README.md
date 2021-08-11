使用说明
=========

## 适用场景

单节点的redis批量安装

一主多从的复制集的安装

redis-sentinel集群的安装



## Requirements

目前仅适用centos7系列的环境

## Role Variables

```shell
#单节点使用变量
REDIS_PORT: 6379
REDIS_PASSWD: 1qaz2wsx
REDIS_HOME: /usr/local/redis
#主从使用变量
redis_role: master/slave #定义位置INVENTORY文件
redis_master_host: xxx #EXTRA_VARS变量，使用-e传入
redis_master_port: xxx #EXTRA_VARS变量，使用-e传入
#sentinel变量
SENTINEL_PORT: 16379  #定义位置INVENTORY文件
SENTINEL_DATA: /usr/local/redis/sentinel_data

```


## INVENTORY文件定义

本实例未做免密，正式环境请自行免密

### redis单节点安装

测试环境一台安装多个redis

```shell
[redis-server]
redis-server01 ansible_ssh_host=192.168.1.12  ansible_ssh_pass=xxx REDIS_PORT=6379
redis-server02 ansible_ssh_host=192.168.1.12  ansible_ssh_pass=xxx REDIS_PORT=6380
redis-server02 ansible_ssh_host=192.168.1.12  ansible_ssh_pass=xxxx REDIS_PORT=6381
```

### redis单节点安装

正式环境安装多个redis

```shell
[redis-server]
redis-server01 ansible_ssh_host=192.168.1.12  ansible_ssh_pass=xxx
redis-server02 ansible_ssh_host=192.168.1.13  ansible_ssh_pass=xxx
redis-server02 ansible_ssh_host=192.168.1.14  ansible_ssh_pass=xxx
#REDIS_PORT变量可使用-e传入也可在default/main.yml中定义。
```

### 主从复制环境安装

测试环境一台服务器

```shell
[redis-server]
redis-server01 ansible_ssh_host=192.168.1.12  ansible_ssh_pass=xxx REDIS_PORT=6379 redis_role=master
redis-server02 ansible_ssh_host=192.168.1.12  ansible_ssh_pass=xxx REDIS_PORT=6380 redis_role=slave
redis-server03 ansible_ssh_host=192.168.1.12  ansible_ssh_pass=xxx REDIS_PORT=6381 redis_role=slave
#需要定义变量redis_role用来判断哪个节点是master；因为是单台服务器因此需要分别定义REDIS_PORT；
```

正式环境的主从

```shell
[redis-server]
redis-server01 ansible_ssh_host=192.168.1.12  ansible_ssh_pass=xxx  redis_role=master
redis-server02 ansible_ssh_host=192.168.1.13  ansible_ssh_pass=xxx redis_role=slave
redis-server03 ansible_ssh_host=192.168.1.14  ansible_ssh_pass=xxx redis_role=slave
#REDIS_PORT变量可使用-e传入也可在default/main.yml中定义。
```

### sentinel集群安装

测试环境一台服务器

```shell
[redis-server]
redis-server01 ansible_ssh_host=192.168.1.12  ansible_ssh_pass=xxx REDIS_PORT=6379 SENTINEL_PORT=16379 redis_role=master sentinel_status=install
redis-server02 ansible_ssh_host=192.168.1.12  ansible_ssh_pass=xxx REDIS_PORT=6380 SENTINEL_PORT=16380 redis_role=slave sentinel_status=install
redis-server03 ansible_ssh_host=192.168.1.12  ansible_ssh_pass=xxx REDIS_PORT=6381 SENTINEL_PORT=26380  redis_role=slave sentinel_status=install
#需要定义变量redis_role用来判断哪个节点是master；定义sentinel_status=install 表明安装sentinel，也可以使用-e传入；因为单台服务器所以要区分SENTINEL_PORT，因此要分别定义
```

正式环境的sentinel

```shell
[redis-server]
redis-server01 ansible_ssh_host=192.168.1.12  ansible_ssh_pass=xxx redis_role=master sentinel_status=install
redis-server02 ansible_ssh_host=192.168.1.13  ansible_ssh_pass=xxx redis_role=slave sentinel_status=install
redis-server03 ansible_ssh_host=192.168.1.14  ansible_ssh_pass=xxx redis_role=slave sentinel_status=install
#REDIS_PORT和SENTINEL_PORT变量可使用-e传入也可在default/main.yml中定义。
```

Playbook
----------------

    - hosts: redis-serve
      roles:
         - redis
         

运行实例

```shell
#安装sentinel集群
ansible-playbook -i hosts_sentinel   -e redis_master_host=192.168.1.12 -e redis_master_port=16379  install.yml 
```
![image](https://user-images.githubusercontent.com/38902618/128983725-c76357fc-1cf0-4024-be8e-a8d7136a2e61.png)
![image](https://user-images.githubusercontent.com/38902618/128983877-a76aa61a-7d5c-4503-a38f-f11f8fdc1a97.png)

![image](https://user-images.githubusercontent.com/38902618/128984098-372663a5-ecd2-4a2f-a330-45003823682b.png)


License
-------

BSD

Author Information
------------------

1830624909@qq.com
