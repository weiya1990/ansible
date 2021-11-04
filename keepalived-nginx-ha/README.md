ansible安装nginx高可用VIP
=========

[TOC]

## 环境依赖：

- keepalived
- nginx
- sendmail
- mailx
- SMTP服务器信息
- Centos7系统

hosts文件配置
------------
```bash
[master]
127.0.0.1  keepalived_role=master
[backup]
192.168.1.12 interface=br0
```
## playbook文件

```bash
[root@localhost ansible]# cat keepalived-nginx.yml 
- hosts:
  - master
  - backup
  roles:
  - keepalived-nginx-ha
 
```

## 变量定义default/main.yml

```bash
[root@localhost keepalived-nginx-ha]# cat defaults/main.yml 
---
# defaults file for keepalived-nginx-ha
interface: "ens33" 

virtual_router_id:
  master: 13
 
virtual_ipaddress: 192.168.1.110  #VIP

#接收邮件地址
receiver_mail: "1830624909@qq.com weiy@xxx.cn"

#smtp配置
smtp_config:
  from: wy1830624909@163.com
  smtp: smtp.163.com
  smtp_auth_user: wy1830624909@163.com
  smtp_auth_password: xxxx
  smtp_auth: login

#内核参数配置
sysctl:
  net.ipv4.tcp_syncookies: 0 #（有攻击才开启）
  #表示开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭； #此参数是为了防止洪水攻击的，但对于大并发系统，要禁用此设置
  net.nf_conntrack_max: 655360
  net.netfilter.nf_conntrack_tcp_timeout_established: 1200
  net.ipv4.tcp_tw_recycle: 1
  #这个参数用于设置启用timewait快速回收。但是NAT网络模式下打开有可能会导致tcp连接错误，慎重。 要开启 需要添加 添加 net.ipv4.tcp_timestamps=0
  net.ipv4.tcp_tw_reuse: 1
  #参数设置为 1 ，表示允许将TIME_WAIT状态的socket重新用于新的TCP链接，这对于服务器来说意义重大，因为总有大量TIME_WAIT状态的链接存在；（已测，可以大量减少TIME_WAIT）
  net.ipv4.tcp_fin_timeout: 25
  net.ipv4.tcp_orphan_retries: 1
  net.ipv4.tcp_max_orphans: 8192
  net.ipv4.ip_local_port_range: 1024 65535
  #定义UDP和TCP链接的本地端口的取值范围。（这个对于客户端（压测的时候）和nginx反向代理或者负载均衡影响比较大，对后端服务器感觉影响不是很大（有待验证））
  fs.file-max: 1048576 #表示单个进程较大可以打开的句柄数；重要否则nginx并发数无法上去 （已测）
  net.ipv4.tcp_max_tw_buckets: 6000 #表示系统同时保持TIME_WAIT的最大数量，如果超过这个数字，TIME_WAIT将立刻被清除并打印警告信息。默 认为180000，改为6000。对于Apache、Nginx等服务器，上几行的参数可以很好地减少TIME_WAIT套接字数量，但是对于Squid，效果却不大。此项参数可以控制TIME_WAIT的最大数量，避免Squid服务器被大量的TIME_WAIT拖死。

```
## VIP切换邮件通知
![image](https://user-images.githubusercontent.com/38902618/140270231-46437b01-5956-411e-87c6-9261f3607842.png)
