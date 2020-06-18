# ClusteredWebHosting - A easy way for clustering.

Create your own cluster using ansible.

It was tested with:
-  KVM servers + CentOS 7 minimal.
-  Amazon Linux 2


## Cluster Setup Mode
- (Load Balancer) Haproxy -> (Nodes) Apache (1) + MariaDB Galera Cluster
- (Load Balancer) Haproxy -> (Nodes) Nginx (2) + MariaDB Galera Cluster
- (Load Balancer) Nginx -> (Nodes) Nginx (3) + MariaDB Galera Cluster

## Single Setup Mode
- (Load Balancer) Nginx -> (Nodes) Nginx (10) + MariaDB Galera Cluster

## Overview
- Haproxy / Nginx as a Load Balancer
- MariaDB Galera Cluster (mariabackup or rsync) / Mysql8
- You can have PHP74 and PHP73 on nodes
- Apache (httpd) or Nginx on nodes
- NTP for clock synchronization
- Logrotate
- Storage NFS
- Domains
- Iptables
- Phpmyadmin
- Composer tool

## Servers Requirement (Cluster Mode)
- centOS 7 on all servers (minimal)
- 1 server for Load Balancing
- minimum 1  Servers for Nodes
- minimum 3 Servers for MariaDB
- 1 server with ansible (master)

## Servers Requirement (Single Mode)
- centOS 7 (minimal)
- 1 server

## How to setup

On this setup we use user "root" to manage the servers (you can use ssh key or another user)

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


#### 3. Generate SSH key on Master Server

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

## Cluster Mode 1, 2 ,3

- 1 - (Load Balancer) Haproxy -> (Nodes) Apache + MariaDB Galera Cluster
- 2 - (Load Balancer) Haproxy -> (Nodes) Nginx + MariaDB Galera Cluster
- 3 - (Load Balancer) Nginx -> (Nodes) Nginx + MariaDB Galera Cluster


#### 1. Prepare the config

```
# inventory 

$ vim inventory/cluster

# variables
# inital domain
$ vim playbooks/vars/global-vars.yml

# variables used for setup, by default cluster_setup_mode is 3
$ vim playbooks/vars/setup-cluster-vars.yml

```

### Notes: 

Add your public ssh key to playbooks/authorized_keys

Remember to modify the variables with your data!!!
For MariaDB we need the interface name, in our case "enp0s3". You can get it using "ip a" command

#### 2. Start the SETUP

```
$ ansible-playbook -i inventory/cluster playbooks/new-domain-cluster.yml
```

#### Add a new Domain

```
$ vim playbooks/vars/new-domain-cluster.yml
$ ansible-playbook -i inventory/cluster playbooks/new-domain-cluster.yml

```

#### Remove a Domain

```
$ vim playbooks/vars/new-domain-cluster.yml

$ ansible-playbook -i inventory/cluster playbooks/remove-domain-cluster.yml -e "primary_domain_user=mydomainuser" -e "primary_domain=mydomain.example.com" -e "cluster_setup_mode=3"

```

## Single Mode 10
For this setup you need only one server.

- 10 - (Load Balancer) Haproxy + (Web) Nginx + Mysql 8


#### 1. Prepare the config

```
# inventory 

$ vim inventory/single

# variables
# inital domain
$ vim playbooks/vars/global-vars.yml

$ vim playbooks/vars/setup-single-vars.yml

```

#### 2. Start the SETUP

```
$ ansible-playbook -i inventory/single playbooks/new-domain-single.yml
```

#### Add a new Domain

```
$ vim playbooks/vars/new-domain-single.yml
$ ansible-playbook -i inventory/single playbooks/new-domain-single.yml

```

#### Remove a Domain

```
$ vim playbooks/vars/new-domain-single.yml

$ ansible-playbook -i inventory/single playbooks/remove-domain-single.yml -e "primary_domain_user=mydomainuser" -e "primary_domain=mydomain.example.com" -e "cluster_setup_mode=10"

```

#### Add Mysql Username / Database

```
$ ansible-playbook -i inventory/cluster playbooks/mariadb-galera-add.yml -e "username=testinguser" -e "password=testinguser123" -e "database=testinguser"
```

#### Remove Mysql Username / Database

```
$ ansible-playbook -i inventory/cluster playbooks/mariadb-galera-remove.yml -e "username=testinguser" -e "database=testinguser"
```

## TO DO:
- improve security / vaults