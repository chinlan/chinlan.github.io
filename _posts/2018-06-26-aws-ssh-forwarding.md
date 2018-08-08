---
layout: post
title: "AWS: ssh forwarding"
description: ""
categories: [AWS]
tags: [ssh]
redirect_from:
  - /2018/06/26/
---

Production：

使用ecs，兩台ec2 instances，一台各有兩個container跑main application，進入production的ec2必須透過bastion的ec2 instance，如果像development環境那樣直接
~~~
ssh ec2-user@<dev public ip> -i ~/myPrivateKey.pem
~~~
這個密鑰不會被forward到bastion，當在bastion中要進一步ssh到ecs的instance時，就會因為沒有密鑰而失敗。

所以要在local先設定ssh forwarding:

[ref](https://aws.amazon.com/tw/blogs/security/securely-connect-to-linux-instances-running-in-a-private-amazon-vpc/)
~~~
ssh-add -K myPrivateKey.pem
~~~
這樣就把密鑰加到keychain了
~~~
ssh-add –L
~~~
然後確認一下是否有成功

有列出剛剛加的密鑰就成功了
~~~
ssh –A ec2-user@<bastion-public-IP-address>
~~~
進入bastion
~~~
ssh user@<instance-private-IP-address>
~~~
從bastion進一步ssh 到ecs instance，注意這邊因為是從內部網路連，所以要使用private ip

連進去以後就可以`docker ps`找到main application的container

然後進行需要的rake task或thor task了
