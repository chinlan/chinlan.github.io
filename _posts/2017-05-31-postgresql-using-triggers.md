---
layout: post
title: "Postgresql using Triggers"
description: ""
categories: [commands]
tags: [postgresql]
redirect_from:
  - /2017/05/31/
---

這陣子把公司專案裡之前留下來的triggers和相關的functions全部清除，紀錄一下相關commands:

trigger和function的關係：

用trigger來引發function



列出資料庫內所有triggers:
~~~
select * from pg_trigger;
~~~

列出資料庫內所有functions:
~~~
\df 
~~~

列出function原始碼內容（可以編輯）
~~~
\ef function-name  
~~~
離開編輯
~~~
:q 
~~~
顯示所有trigger的詳細資料包括table_name
~~~
SELECT event_object_schema,

       event_object_table,

       trigger_schema,

       trigger_name

FROM information_schema.triggers
~~~
