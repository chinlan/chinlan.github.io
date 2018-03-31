---
layout: post
title: "AWS: ecs-deploy"
description: ""
categories: [devops]
tags: [AWS]
redirect_from:
  - /2018/03/26/
---

公司專案使用aws ecs，進行deploy前的安裝筆記：

- 安裝需要的工具：

~~~~
brew install awscli
~~~~

aws configure:

填入access key, secret, 等等config資料
（要先在aws主控台生成access key, secret)

~~~
brew install jq
~~~

- 安裝 ecs-deploy:

~~~
curl https://raw.githubusercontent.com/silinternational/ecs-deploy/master/ecs-deploy | sudo tee -a /usr/bin/ecs-deploy

sudo chmod +x /usr/bin/ecs-deploy
~~~

官方doc 是寫上面這樣，但是因為[沒辦法安裝到/usr/bin](https://github.com/tstack/lnav/issues/295)，所以要把`/usr/bin/ecs-deploy`改成`/usr/local/bin/ecs-deploy`
