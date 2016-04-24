---
layout: post
title: "Ansible Dynamic Inventory"
date: 2016-04-24 20:53
comments: true
categories: System
description: ansible
---

Ansible 在使用的过程中，如果机器数量比较固定，且变更不多的情况下，可在 /etc/ansible/hosts 文件里面配置固定的组合机器IP， 并给他起组的别名，执行 Ansible 脚本便可以通过别名找到相应的机器。

```
[webservers]
111.222.333.444 ansible_ssh_port=888
```

假如你有很多台机器，且机器经常变更导致IP时常变换，你还想把IP逐个写入 /etc/ansible/hosts 就不现实了。你也许会问，若不把 IP 写进 /etc/ansible/hosts，那不是没法用 Ansible 指挥这些机器？
感谢 Ansible Dynamic Inventory， 如果我们能通过编程等手段获取变更机器的IP，我们还是有办法实现的。

### Dynamic Inventory 的原理

* 通过编程的方式,也就是动态获取机器的 json 信息;
* Ansible 通过解析这串 json 字符串;

```
ansible -i yourprogram.py -m raw  -a 'cd /home'
```

Ansible Dynamic Inventory 对程序返回的 json 的转义是这样的：

```
{"devtest-asg": {"hosts": ["172.31.21.164"], "vars": {"ansible_ssh_port": 12306}}}
```

翻译一下就是  /etc/ansible/hosts 中的:

```
[devtest-asg]
172.31.21.164 ansible_ssh_port=12306
```

### 一个实战的例子

官方文档对 Inventory 仅作概念性描述，阅读完后仍是一头雾水，不知如何下手。 让我们用一个例子来豁然开朗吧。 我们使用 AWS 的 AutoScaling Group，以下简称 ASG，ASG 会在某种自定义的条件下会自动开启和关闭机器，这给我们在辨别IP，定位机器的时候造成困扰。因此我们需要 Ansible Dynamic Inventory

我们使用 AWS 的 boto 库来获取 ASG 的实例信息.以下程序(get_host.py)中要实现的方法就是列出返回机器信息的 json 串。

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import json
import boto
import boto.ec2
import boto.ec2.autoscale

AWS_REGION = 'BBB'
AWS_ACCESS_KEY = 'xxxx'
AWS_SECRET_KEY = 'yyy'

result = {}
def getData():
    conn_as = boto.ec2.autoscale.connect_to_region(
            'cn-north-1',
            aws_access_key_id=AWS_ACCESS_KEY,
            aws_secret_access_key=AWS_SECRET_KEY)
    group = conn_as.get_all_groups(names=['devtest-asg'])[0]
    conn_ec2 = boto.ec2.connect_to_region(
            AWS_REGION,
            aws_access_key_id=AWS_ACCESS_KEY,
            aws_secret_access_key=AWS_SECRET_KEY)

    instance_ids = [i.instance_id for i in group.instances]
    reservations = conn_ec2.get_all_instances(instance_ids)
    instances = [i for r in reservations for i in r.instances]

    result['devtest-asg'] = {}
    result['devtest-asg']['hosts'] = []
    for r in reservations:
        for i in r.instances:
            result['devtest-asg']['hosts'].append('%s' % i.private_ip_address)
            result['devtest-asg']['vars'] = {'ansible_ssh_port': 36000}

def getlists():
    getData()
    print json.dumps(result)

getlists()
```

执行以下命令就可以愉快地使用 Ansible 了，其中 devtest-asg 是 ASG 的别名：

```
ansible -i get_host.py  devtest-asg -m raw -a 'ls /'
```
