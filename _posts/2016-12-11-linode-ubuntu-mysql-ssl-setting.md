---
layout: post
title: "Linode ubuntu: mysql ssl setting"
description: ""
categories: [devops]
tags: [linode, ubuntu, mysql, ssl]
redirect_from:
  - /2016/12/11/
---
two users:
- root
- mysqluser

context:
- 兩台server，一台做前台deploy，一台做後台deploy加上資料庫，後台使用者也需要連到資料庫

Ref: http://xmodulo.com/enable-ssl-mysql-server-client.html

# ssl grant
~~~
GRANT ALL PRIVILEGES ON staging.* TO ’mysqluser’@’%’ IDENTIFIED BY ’mysqluser_password’ REQUIRE SSL;
~~~

做完這個設定，拿掉/etc/mysql/my.cnf 設定檔中bind 127.0.0.1的設定，就限定mysqluser一定要用ssl設定了

但剛做完這個設定時，mysqluser可以從遠端連入，卻不能從localhost連入，
會出access denied錯誤

之後加上client-cert.pem/client-req.pem/client-key.pem，/etc/mysql/my.cnf 設定檔中加上
~~~
[client]
ssl-ca=/path/to/ca-cert.pem
ssl-cert=/path/to/client-cert.pem
ssl-key=/path/to/client-key.pem
~~~
就可以讓mysqluser@localhost連入資料庫了

# 前台機器要連到後台mysql
~~~
sudo apt-get install mysql-client
~~~
把後台的client-cert.pem/client-req.pem/client-key.pem 複製過來

# % vs localhost
％是代表沒有指定ip，從任何ip連過來都接受的意思

Ref: https://dev.mysql.com/doc/refman/5.7/en/problems-connecting.html

原本有mysqluser@%和mysqluser@localhost兩個，但砍掉localhost那個以後，連線也沒什麼問題

所以之前的本地連線問題應該完全是ssl 客戶端設定沒做的原因

在本地用mysqluser登入後：
!(https://hackpad-attachments.s3.amazonaws.com/allodola.hackpad.com_cDwB4Fs8r1D_p.578563_1479290483141_undefined)

# mysql command
- 列出使用者名，主機
~~~
select user, host from mysql.user;
~~~
- 列出某個使用者的grants
~~~
show grants for ’mysqluser’@’localhost’;
~~~
- 刪除某個使用者
~~~
DROP USER ’someuser’@’localhost’;
~~~
- 取消某使用者全部的權限
~~~
revoke all privileges on *.* from ’someuser’@’host’;
~~~
- 檢查密碼政策
~~~
SHOW VARIABLES LIKE ’validate_password%’;
~~~
- 設定密碼政策為低
~~~
SET GLOBAL validate_password_policy=LOW;
~~~
- 或者也可直接在my.cnf中設定
~~~
[mysqld]
validate_password_policy=LOW
~~~
- 登入
~~~
mysql -u username -p
mysql -u username -P port -h host -p
~~~
