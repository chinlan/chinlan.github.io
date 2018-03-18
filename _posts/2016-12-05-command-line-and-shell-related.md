---
layout: post
title: "Command line and shell related"
description: ""
categories: [command]
tags: []
redirect_from:
  - /2016/12/05/
---

# 中斷push
中斷push   --->   ctrl + c     下一次push還是會從頭開始push

# echo
append with echo
~~~
echo "sysctl -w lalala=1" >> /etc/sysctl.conf
~~~

not append but replace
~~~
echo "sysctl -w lalala=1" > /etc/sysctl.conf
~~~

# ubuntu server user related
列出所有使用者及其權限
~~~
cat /etc/passwd
~~~
刪除某個使用者，然後刪除其家目錄下的檔案夾
~~~
deluser username
rm -r /home/username
~~~
找到某個特定的資料夾
~~~
find / -type d -name ‘httpdocs’
~~~

# sudo
sudo 只適用於program，而cd是個內建指令，並不是program

所以sudo cd /path 不能用

可以先sudo -i 變成以root身份登入，

然後再cd /path


# 清空某資料夾
~~~
rm /path/to/directory/*
~~~

# 複製整個資料夾
~~~
cp -r dir1 dir2
~~~
要加-r才能cp directory

# open gem
~~~
export BUNDLER_EDITOR=subl

bundle open [:gem]
~~~
