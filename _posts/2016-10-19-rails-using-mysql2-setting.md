---
layout: post
title: "Rails: using mysql2: setting"
description: ""
categories: [rails]
tags: [mysql]
redirect_from:
  - /2016/10/19/
---
開始一個新專案，database.yml設定使用mysql2

gemfile要安裝mysql2

然後進入mysql資料庫console

照著在database.yml裡面設定的username和database建立以下資料庫和使用者
~~~
mysql -u root -p

CREATE DATABASE xxxx_dev;

CREATE USER 'xxxx_dev'@'localhost' IDENTIFIED BY 'xxxx_dev_password';

USE xxxx_dev;
GRANT ALL privileges on xxxx_dev.* to xxxx_dev@localhost identified by 'xxxx_dev';

FLUSH PRIVILEGES;
~~~
測試資料庫也是照著同樣作法

接著mysql.server start

應該就可以rails s

看到專案的首頁了

# 如果不幸忘記了mysql root密碼
1. 先停止mysql
~~~
sudo mysqld stop
~~~
2. 使用safe mode啟動mysql
~~~
mysqld_safe --skip-grant-tables
~~~
3. 以root連線至mysql
~~~
mysql --user=root mysql
~~~
4. 重設root密碼
~~~
update user set Password=PASSWORD(’new-password’) where user=’root’;
flush privileges;
exit;
~~~
5. 重啟mysql
