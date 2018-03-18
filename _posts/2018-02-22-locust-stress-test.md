---
layout: post
title: "Stress test using locust"
description: ""
categories: [devops]
tags: []
redirect_from:
  - /2018/02/22/
---

# Install Python and locustio
~~~
brew install python3
python3 --version
pip3 install locustio
~~~

# Writing locustfile
~~~ python
from locust import HttpLocust, TaskSet, task
import random, json
import params.article as article_params

class WebsiteTasks(TaskSet):

    def on_start(self):
        params = self.__signup_params()
        r = self.client.post('/users',
            data=json.dumps(params),
            headers=self.__headers()
        )
        user = json.loads(r.text)['user']
        self.token = user['token']
        return

    @task
    def post article(self):
        self.client.post('/articles',
            data=json.dumps(self._ article_params([])),
            headers=self.__headers(token=self.token)
        )
        return

    @task
    def follow_others(self):
        # source = [18, 19, 20, 21, 22] # local環境用
        source = [11797, 73465, 87761, 82514, 83039]
        self.client.post('/follows',
            data=json.dumps({
                'follow': {
                    'user': {
                        'id': random.SystemRandom().choice(source)
                    }
                }
            }),
            headers=self.__headers(token=self.token),
            name='/follows'
        )

    @task
    def get_timeline_and_comment_and_like(self):
        # 5ページまでのタムラインを見る
        r = self.client.get('/timeline/?page=1',
            headers=self.__headers(token=self.token),
            name='/timeline'
        )
        articles = json.loads(r.text)['articles']
        # 5つ選んでいいね、コメントする
        select_count = min(5, len(articles))
        for article in random.sample(articles, select_count):
            self.__like_on article)
            self.__comment_on article)
        return

    def __like_on article(self, article):
        self.client.post('/articles/%d/likes' % article['id'],
            headers=self.__headers(token=self.token),
            name='/articles/:id/likes'
        )
        return

    def __comment_on article(self, article):
        self.client.post('/articles/%d/comments' % article['id'],
            data=json.dumps({
                'id': article['id'],
                'comment': {
                    'body':'Hello guys!'
                }
            }),
            headers=self.__headers(token=self.token),
            name='/articles/:id/comments'
        )
        return

    def __headers(self, json=True, token=None):
        h = {
            'Accept-Language': 'ja,en;q=0.8,en-US;q=0.6',
            'X-MyApp-Locale': 'ja',
            'User-Agent': 'MyApp/4.5.0 (iPhone7,1; iOS 10.3.1; Locale/ja_JP)'
        }
        if json:
            h['Content-Type'] = 'application/json'
        if token:
            h['Authorization'] = "Token token=%s" % token
        return h

    def __signup_params(self):
        source = 'abcdefghijklmnopqrstuvwxyz'
        name = ''.join([random.choice(source) for x in range(4)])
        email = '%s@example.com' % name
        password = '%s_password' % name
        return { 'user': {
            'name': name,
            'email': email,
            'password': password
        }}

    def _ article_params(self, images=[]):
        title_src1 = ['Hello', 'Good bye', 'Wonderful', 'Fine', 'Beautiful', 'Great', 'Amazing']
        title_src2 = ['Mountain', 'Sunrise', 'Sunset', 'Climbing', 'Tree', 'Rock', 'River', 'Adventure']
        tag_src = [{ 'id': 5 }, { 'id': 45 }, { 'id': 65 }, { 'id': 73 }, { 'id': 80 }]
        return article_params.build(
            title='%s %s' % (random.choice(title_src1), random.choice(title_src2)),
            tag=random.choice(tag_src),
            images=images
        )

class WebsiteUser(HttpLocust):
    task_set = WebsiteTasks
    min_wait = 1000
    max_wait = 1000
~~~

# Debug: Below code works like `binding.pry` in Ruby.
~~~ python
import code; code.interact(local=dict(globals(), **locals()))
~~~

# Run test
~~~
locust -H http://localhost:3000 -f locustfile/file_name.py
~~~
View http://localhost:8089.


[ref](https://stackoverflow.com/questions/35991355/in-locust-how-to-get-a-response-from-one-task-and-pass-it-to-other-task)
[ref](http://www.cnblogs.com/LanTianYou/p/5987741.html)
[doc](https://docs.locust.io/en/latest/quickstart.html)

