---
title: 创建Openshift Origin 平台
date: 2016-09-24 19:16:08
tags: 
    - docker
    - openshift
    - centos
    - 云计算
description: 搭建分布式Openshift Origin Centos7 Pass平台
---


## 简介
>目前共提供三种产品：OpenShift Online、OpenShift Enterprise 和 OpenShift Origin。 其中，OpenShift Online 是面向普通开发者和小微企业的线上公有云平台；OpenShift Enterprise 是面向企业的私有云平台；OpenShift Origin 是一个开源项目，是构成前两个的基础。

### 目标
搭建分布式Openshift Orgin 集群,用于测试环境。
### 环境
master:node-test-001, 192.168.3.218
etcd:   node-test-002, 192.168.12.171，Centos7.2
node01:node-test-003, 192.168.12.172，Centos7.2
node02:node-test-004, 192.168.12.173，Centos7.2
### 版本
Linux：Centos v7.2
Docker:v1.10.3
Openshift origin:v1.3.0

## 基础设施准备
### 目标
>准备4台centos 7.3主机 

### 操作系统安装

### 启动iptables,开启selinux，安装NetworkManager,安装docker

``` bash
# 查看操作系统版本
[root@node-test-001 ~]# more /etc/redhat-release
CentOS Linux release 7.2.1511 (Core)

#启动 firewall
[root@node-test-001 ~]# systemctl start firewalld
[root@node-test-001 ~]# systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2016-09-23 22:36:52 EDT; 10h ago
 Main PID: 881 (firewalld)
   CGroup: /system.slice/firewalld.service
           └─881 /usr/bin/python -Es /usr/sbin/firewalld --nofork --nopid

#确认selinux开启
[root@node-test-001 ~]# vi /etc/selinux/config
SELINUX=enforcing
[root@node-test-001 ~]# getenforce
Enforcing

#安装基本包：
[root@node-test-001 ~]# yum install wget git net-tools bind-utils iptables-services bridge-utils bash-completion

#确认安装NetworkManager
[root@node-test-001 ~]# yum install NetworkManager -y
[root@node-test-001 ~]# systemctl enable NetworkManager
[root@node-test-001 ~]# systemctl restart NetworkManager
[root@node-test-001 ~]# systemctl status NetworkManager

#确认安装docker

[root@node-test-001 ~]# sudo tee /etc/yum.repos.d/docker.repo <<-'EOF'
> [dockerrepo]
> name=Docker Repository
> baseurl=https://yum.dockerproject.org/repo/main/centos/7/
> enabled=1
> gpgcheck=1
> gpgkey=https://yum.dockerproject.org/gpg
> EOF
[dockerrepo]
name=Docker Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/7/
enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg


[root@node-test-001 ~]# yum install docker
[root@node-test-001 ~]# systemctl enable docker
[root@node-test-001 ~]# systemctl restart docker
[root@node-test-001 ~]# systemctl status docker
[root@node-test-001 openshift-ansible]# docker version
Client:
 Version:      1.12.1
 API version:  1.24
 Go version:   go1.6.3
 Git commit:   23cf638
 Built:
 OS/Arch:      linux/amd64

Server:
 Version:      1.12.1
 API version:  1.24
 Go version:   go1.6.3
 Git commit:   23cf638
 Built:
 OS/Arch:      linux/amd64

```

### 配置dns
``` bash
#规划如下
10.5.5.100 master.openshift.test.com
10.5.5.101 etcd.openshift.test.com
10.5.5.102 node01.openshift.test.com
10.5.5.103 node02.openshift.test.com

#将上述域名添加/etc/hosts文件
[root@openshift001 ~]# vi /etc/hosts
...
10.5.5.100 master.openshift.test.com
10.5.5.101 etcd.openshift.test.com
10.5.5.102 node01.openshift.test.com
10.5.5.103 node02.openshift.test.com

```

## 部署openshift-origin

### 部署ansible工具

``` bash
#添加epel源，安装ansible

#添加epel源，安装ansible
[root@node-test-001 ~]# wget http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
[root@node-test-001 ~]# rpm -ivh epel-release-latest-7.noarch.rpm
[root@node-test-001 ~]# yum install ansible pyOpenSSL

#配置文件
[root@node-test-001 ~]# rpm -qc ansible
/etc/ansible/ansible.cfg
/etc/ansible/hosts

#各个主机添加ssh信任
[root@node-test-001 ~]# ssh-keygen
[root@node-test-001 ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub  10.5.5.100
[root@node-test-001 ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub  10.5.5.101
[root@node-test-001 ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub  10.5.5.102
[root@node-test-001 ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub  10.5.5.103

#ssh 测试
[root@node-test-001 ~]# ssh 10.5.5.101
Last login: Sat Sep 24 12:47:17 2016 from master.openshift.test.com
[root@node-test-002 ~]# exit

[root@node-test-001 ~]# git clone https://github.com/openshift/openshift-ansible
[root@node-test-001 ~]# cd openshift-ansible
[root@node-test-001 openshift-ansible]# ls -lrt
total 216
-rw-r--r--.  1 root root    811 Sep 24 12:53 Dockerfile
-rw-r--r--.  1 root root   2090 Sep 24 12:53 DEPLOYMENT_TYPES.md
-rw-r--r--.  1 root root   1020 Sep 24 12:53 BUILD.md
-rw-r--r--.  1 root root   3154 Sep 24 12:53 README.md
-rw-r--r--.  1 root root   8265 Sep 24 12:53 README_AEP.md
-rw-r--r--.  1 root root  10759 Sep 24 12:53 LICENSE
-rw-r--r--.  1 root root   4510 Sep 24 12:53 README_CONTAINERIZED_INSTALLATION.md
-rw-r--r--.  1 root root   6374 Sep 24 12:53 README_AWS.md
-rw-r--r--.  1 root root    536 Sep 24 12:53 README_ANSIBLE_CONTAINER.md
-rw-r--r--.  1 root root   3163 Sep 24 12:53 README_openstack.md
-rw-r--r--.  1 root root   6389 Sep 24 12:53 README_libvirt.md
-rw-r--r--.  1 root root   4958 Sep 24 12:53 README_GCE.md
-rw-r--r--.  1 root root   2301 Sep 24 12:53 Vagrantfile
-rw-r--r--.  1 root root   2431 Sep 24 12:53 README_vagrant.md
drwxr-xr-x.  3 root root     45 Sep 24 12:53 ansible-profile
drwxr-xr-x.  2 root root     20 Sep 24 12:53 bin
-rw-r--r--.  1 root root    650 Sep 24 12:53 ansible.cfg.example
drwxr-xr-x.  2 root root     23 Sep 24 12:53 callback_plugins
drwxr-xr-x.  2 root root     92 Sep 24 12:53 docs
drwxr-xr-x.  2 root root    103 Sep 24 12:53 filter_plugins
drwxr-xr-x.  2 root root     79 Sep 24 12:53 git
drwxr-xr-x.  7 root root     78 Sep 24 12:53 inventory
drwxr-xr-x.  2 root root     25 Sep 24 12:53 lookup_plugins
drwxr-xr-x.  2 root root     42 Sep 24 12:53 library
-rw-r--r--.  1 root root 117577 Sep 24 12:53 openshift-ansible.spec
drwxr-xr-x.  9 root root     91 Sep 24 12:53 playbooks
drwxr-xr-x.  2 root root     33 Sep 24 12:53 test
drwxr-xr-x. 62 root root   4096 Sep 24 12:53 roles
drwxr-xr-x.  8 root root   4096 Sep 24 12:53 utils


```

### 部署openshift-origin

``` bash

#修改ansible的hosts文件
[root@node-test-001 ~]# vi /etc/ansible/hosts 
# Create an OSEv3 group that contains the masters, nodes, and etcd groups
[OSEv3:children]
masters
nodes
etcd
 
# Set variables common for all OSEv3 hosts
[OSEv3:vars]
ansible_ssh_user=root
deployment_type=origin
 
[masters]
master.openshift.test.com
 
# host group for etcd
[etcd]
etcd.openshift.test.com
 
# host group for nodes, includes region info
[nodes]
master.openshift.test.com openshift_node_labels="{'region': 'infra', 'zone': 'default'}"
node01.openshift.test.com openshift_node_labels="{'region': 'primary', 'zone': 'east'}"
node02.openshift.test.com openshift_node_labels="{'region': 'primary', 'zone': 'west'}"
 
 
#测试ping
[root@node-test-001 ~]# ansible all -m ping
node02.openshift.test.com | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
node01.openshift.test.com | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
etcd.openshift.test.com | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
master.openshift.test.com | SUCCESS => {
    "changed": false,
    "ping": "pong"
}

#自动部署
[root@node-test-001 ~]# ansible-playbook ~/openshift-ansible/playbooks/byo/config.yml
....
etcd.openshift.test.com    : ok=101  changed=34   unreachable=0    failed=0
localhost                  : ok=14   changed=8    unreachable=0    failed=0
master.openshift.test.com  : ok=406  changed=100  unreachable=0    failed=0
node01.openshift.test.com  : ok=147  changed=44   unreachable=0    failed=0
node02.openshift.test.com  : ok=147  changed=44   unreachable=0    failed=0



```