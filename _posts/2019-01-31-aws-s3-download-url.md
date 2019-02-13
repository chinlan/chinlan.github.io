---
layout: post
title: "AWS S3: 發行一定時間內有效的下載用url"
description: ""
categories: [rails]
tags: [aws s3]
redirect_from:
  - /2019/01/31/
---

[ref1](https://qiita.com/takeyuweb/items/b32dd7487d724faac1fe)

[ref2](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/S3/Presigner.html)

使用`Aws::S3::Client.new`
~~~ruby
s3 = Aws::S3::Client.new
signer = Aws::S3::Presigner.new(client: s3)
signer.presigned_url(:get_object,
                     bucket: 'your-bucket',
                     key: 'path/to/object')
~~~

專案中使用的gem:
~~~ruby
gem 'aws-sdk-rails'
gem 'aws-sdk-s3'
gem 'aws-sdk-sqs'
~~~
