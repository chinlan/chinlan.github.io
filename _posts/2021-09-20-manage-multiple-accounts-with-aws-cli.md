---
layout: post
title: "Manage multiple accounts with aws cli"
description: ""
categories: [aws]
tags: []
redirect_from:
  - /2021/09/20/
---
To list configured profiles:
```
cat ~/.aws/credentials
```
```
cat ~/.aws/config
```
```
aws configure list
```
```
aws configure list --profile default
```

To add new profile (Max 2):
```
aws configure --profile new-profile-name
```

ref:
- https://shakib37.medium.com/manage-aws-cli-for-multiple-accounts-e2c414006191
- https://stackoverflow.com/questions/593334/how-to-use-multiple-aws-accounts-from-the-command-line#:~:text=You%20can%20work%20with%20two,region%2C%20so%20have%20them%20ready.&text=You%20can%20then%20switch%20between,the%20profile%20on%20the%20command.
