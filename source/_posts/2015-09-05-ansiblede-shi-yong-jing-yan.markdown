---
layout: post
title: "Ansible 使用经验"
date: 2015-09-05 18:09
comments: true
categories: System
description: ansible role
---


当你只有一两台服务器的情况下，可以直接登上服务器，手敲命令完成软件部署，代码发布等工作。但假如你有10台，100台的时候，这种方式不仅浪费大量时间，而且给人为犯错带来了可能。于是我们选择 Ansible 来做自动化批量操作。
    
之前有记录一些 Ansible 入门的使用,请看[这里](http://wiki.zheng-ji.info/Sys/ansible.html), 这半年的积累, 总结一些实用的经验, 记录了一把。

* [配置 ansible.cfg 文件](#第一节)
* [使用 ansible role 来区分业务](#第二节)
* [files 目录的路径定位](#第三节)
* [使用 tags 区分不同操作](#第四节)
* [规划 ansible roles 的 tasks 文件](#第五节)
* [ansible-play-book 一些常用的选项](#第六节)

<h3 id="第一节">更好地配置文件</h3>

我们会如下配置 /etc/ansible/host, 特意指明用户与 端口

```
[web-cluster]
<node-1-IP> ansible_ssh_port=<Your Port> ansible_ssh_user=zj
<node-2-IP> ansible_ssh_port=<Your Port> ansible_ssh_user=zj
```

在 /etc/ansible/ansible.cfg 文件里
我们特意提及了 ansible-role 的配置，未来我们会使用这个东西

```
roles_path    = /home/zj/my-ansible/roles
```


<h3 id="第二节">使用 ansible role 来区分业务</h3>

打开 ansible 部署脚本的文件夹, 目录树如下


```
cd /home/zj/my-ansible/
haproxy
     - entry.yaml
roles
     - haproxy
        - files
        - handlers
        - vars
        - tasks
```

我用一个管理 haproxy 的例子来讲解这种方式。
在 roles 目录下创建 haproxy, 如上所示，需要有四个目录;

* files 目录下放置需要被传输到远端的文件;
* vars  目录下有一个 main.yml 文件,可以定义一些通用的配置变量，可以在 ansbile 脚本中使用;
* handlers 目录下有一个 main.yml, 可以定义一些通用的操作，比如重启服务等;
* tasks 目录下是我们编写 main.yml 脚本，执行业务逻辑的地方;

> 那么 ansible role 的入口在哪呢？

在 ~/my-ansible/haproxy/entry.yml 中，指定了roles的角色，如此一来， 
ansible-playbook 就会去 /home/zj/my-ansible/roles/haproxy 准备执行 tasks/main.yml 

```
- hosts: web-cluster
  roles:
    - haproxy
```

<h3 id="第三节">files 目录的路径定位</h3>

摘取 ~/my-ansible/roles/haproxy/tasks/main.yml

```
- name: copy haproxy conf
  copy: src=haproxy.cfg dest=/etc/haproxy/haproxy.cfg owner=root group=root
  sudo: yes
```

这里的 src=haproxy.cfg 意味着 ~/my-ansible/roles/haproxy/files/haproxy.cfg

<h3 id="第四节">使用 tags 区分不同操作</h3>

```
- name: install ppa
  shell: add-apt-repository -y ppa:vbernat/haproxy-1.5
  sudo: yes
  tags:
    - install-haproxy
```

以下命令，是使用 tags 参数区分操作的例子

```
cd ~/my-ansible/haproxy
ansible-playbook entry.yml -v -K --tags "install-haproxy"
```

<h3 id="第五节">规划 ansible roles 的 tasks 目录</h3>

tasks 目录有一个主执行文件 main.yml, 因为业务操作步骤太多，导致 main.yml 文件很长，那么可读性就下降了。为此，我们使用了 include 语法。

cat ~/my-ansible/roles/haproxy/tasks/main.yml

```
- include: 'install-haproxy.yml'
```

include 上述文件，这样 main.yml 就显得简洁，我们可以将相关的操作写在对应的 yml 文件里

cat ~/my-ansible/roles/haproxy/tasks/install-haproxy.yml

```
- name: copy haproxy conf
  copy: src=haproxy.cfg dest=/etc/haproxy/haproxy.cfg owner=root group=root
  sudo: yes
  tags:
     - install-haproxy
```
tags 最好也与该 yml 文件名一致，清晰分明

<h3 id="第六节">ansible-play-book 一些常用的选项</h3>


* -K 需要 sudo 权限去客户机执行命令，会提示你输入密码
* -v 可以输出冗余的执行过程
* --check 可以测试脚本执行情况，但实际并未在远程机器执行
* --tags 提示 ansible-play-book 调用哪些 tags 命令

使用过ansible roles 之后，最大的体会是操作调理化，甚至编程化，合理的利用 handler, vars, 能更加优雅抽象。

----

上述的例子在 Github 有代码, 结合本文阅读可能更容易上手
[Link](https://github.com/zheng-ji/ToyCollection/tree/master/my-ansible) 


