---
layout: post
title: "Often used commands"
description: ""
categories: [command]
tags: [postgresql, mysql]
redirect_from:
  - /2016/10/17/
---
# start posgresql
~~~~~~~~~~~~
pg_ctl -D /usr/local/var/postgres -l /usr/local/var/postgres/server.log start
~~~~~~~~~~~~~~~~~~

# stop postgresql
~~~~~~~~~~~~
pg_ctl -D /usr/local/var/postgres stop -s -m fast
~~~~~~~~~~~~~~~~~~

# start mysql
~~~~~~~~~~~~
mysql.server start
~~~~~~~~~~~~~~~~~~

# stop mysql
~~~~~~~~~~~~
mysql.server stop
~~~~~~~~~~~~~~~~~~
