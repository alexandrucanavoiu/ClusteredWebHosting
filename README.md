# ClusteredWebHosting - A easy way for clustering.

Create your own cluster using ansible.

It was tested with KVM servers + CentOS 7 minimal.


## Cluster Setup Mode
- (Load Balancer) Haproxy -> (Nodes) Apache + Lighttpd (1) + MariaDB Galera Cluster
- (Load Balancer) Haproxy -> (Nodes) Nginx (2) + MariaDB Galera Cluster
- (Load Balancer) Nginx -> (Nodes) Nginx (3) + MariaDB Galera Cluster

## Overview
- Haproxy / Nginx as a Load Balancer
- MariaDB Galera Cluster (mariabackup or rsync)
- You can have PHP74 and PHP73 on nodes
- Apache (httpd) + Lighttpd or Nginx on nodes
- NTP for clock synchronization
- Logrotate
- Storage NFS
- Domains
- Iptables
- Phpmyadmin
- Composer tool

## Servers Requirement
- centOS 7 on all servers (minimal)
- 1 server for Load Balancing
- minimum 1  Servers for Nodes
- minimum 3 Servers for MariaDB
- 1 server with ansible (master)

## How to setup

On this setup we use user "root" to manage the servers.

#### 1. Install "ansible" on Master server
```
$ yum install ansible
```

#### 2. Clone the repo in your /etc/ansible

#### 3. Prepare the "hosts" file from /etc/ansible/

In our case:

Load Balancing: lb.example.org
Nodes: s1.example.org, s2.example.org, s2.example.org, s3.example.org
NFS (storage): lb.example.org
MariaDB: db1.example.org, db2.example.org, db3.example.org

#### 4. Prepare the local "hosts" file from /etc/

```
[root@master ansible]# cat /etc/hosts | grep example.org

192.168.1.101 lb.example.org
192.168.1.102 s1.example.org
192.168.1.103 s2.example.org
192.168.1.104 s3.example.org
192.168.1.105 db1.example.org
192.168.1.106 db2.example.org
192.168.1.107 db3.example.org

[root@master ansible]#
````


#### 3. Generate SSH key on Master Server (without password)

```sh
$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

#### 4. Copy the ssh key to servers

```
$ ssh-copy-id root@lb.example.org
$ ssh-copy-id root@s1.example.org
$ ssh-copy-id root@s2.example.org
$ ssh-copy-id root@s3.example.org
$ ssh-copy-id root@db1.example.org
$ ssh-copy-id root@db2.example.org
$ ssh-copy-id root@db3.example.org
```

#### 5. Prepare the config

```
$ vim /etc/ansible/group_vars/all.yml
$ vim /etc/ansible/group_vars/lb.yml
```

Remember to modify the variables with your data!!!
For MariaDB we need the interface name, in our case "enp0s3". You can get it using "ip a" command

#### 6. Start the SETUP

```
$ cd /etc/ansible
$ ansible-playbook setup.yml
```


#### TO DO:
- improve security
- playbook to add/remove domains
- playbook to add/remove users/databases