---
layout: post
title: "Ansible自动化脚本集"
date: 2015-04-05 20:22
comments: true
categories: Server
description: 自动化运维
---


管理众多服务器，如果没有自动化的手段，会被累死。

Ansilbe 曾经在 [WIKI](http://wiki.zheng-ji.info/Sys/ansible.html) 里有记载，在一定程度上解放了我.

我想记录下平时写的一些`Ansible playbook`,因此这个文章将会持续更新。


* 批量修改密码　`change-password.yaml`

```
---
- hosts: my-linode-host
  sudo: yes
  user: zhengji
  tasks:
    - name: update password
    user: name=guest password=xxxxx
```

这里的密码是用md5sum 生成的哈希串，只有这样才能让`Ansible`识别.

执行

```
ansible-playbook change-password.yaml -K
```

---

* 批量重启服务


```
- hosts: my-linode-host
  sudo: yes
  tasks:
    - name:  supervisorct restart fxserver
    shell: supervisorctl restart fxserver
    notify:
        - Echo hello
    #触发调度
    handlers:
        - name: Echo hello
        shell: uptime
```

---

待更新
