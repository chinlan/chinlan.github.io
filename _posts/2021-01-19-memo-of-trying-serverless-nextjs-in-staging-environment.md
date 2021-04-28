---
layout: post
title: "Memo of Trying serverless-next.js in staging environment"
description: ""
categories: [serverless]
tags: []
redirect_from:
  - /2021/01/18/
---

# stagingでDEPLOYしてみたメモ：
- terraformみたいにremote stateを管理してくれる機能がありませんので、deploy.shの中でそれらしいことをしています
- custom origin (defaultのS3以外のorigin）を削除したいときは、serverless.staging.yml中の設定を削除しただけでは、このoriginに関連するCloudFrontのbehavioursだけが削除されます、origin自体はaws consoleで手動削除することが必要です。
- distributionIdを書かないと、新しい CloudFront distributionが作成されます、書くと書いてるdistributionを更新します
- 現在はCloudFrontにstandard logを設定しています（s3へ保存）これはdefaultの設定ではない、debugのために追加しました、削除します？
- static_pagesは全部localeをpathの中に入れないとダメになります
その原因はserverless-nextjsの[未対応issue](https://github.com/serverless-nextjs/serverless-next.js/issues/721)です：next10から入ってるi18nextの変化にまだ対応していません
なので、stagingのjob-frontendとjobのコードを修正して、一時的にpathにlocaleを入れるように対応して試しました。

# インフラ管理は：
serverless-next.js -> CloudFront, s3 origin, lambda@edge (作成、更新だけ、削除は手動）
terraform -> 他全部

## 以下は初回の作成をもとに予想する流れです、検証に伴い更新する可能性あり：
# stagingでserverless-next.jsが管理するものを削除せず場合のup, down:
[up]
- terraform apply
- 既存CloudFront distributionをenabledにします
- aws console で古いCloudFront alb origin・behavioursを削除して、
circleCIのenvを新しいalbエンドポイントに変えて(https://付き)、
job-frontend -> stagingへコードをプッシュしますか、前回のbuildをrebuildします。
job -> stagingへコードをプッシュしてbuildします。(codeDeployまで成功したbuildをrebuildすることを避けます、codeDeployあたりでDeploymentLimitExceededExceptionが起こります)

[down]
- terraform destroy
- 既存CloudFront distributionをdisabledにします

# stagingでserverless-next.jsが管理するものを削除する場合のup, down:
[up]
- stagingへコードをプッシュします(serverless.staging.ymlにdistributionIdなし)(originsの部分はコメントアウト、まだterraform applyしてないから、albがありません)
- 作られたdistributionのIdをterraformのvariablesに入れて、terraform applyします
- circleCIのenvを新しいalbエンドポイントに変えて(https://付き)、
コメントアウトされたoriginsの部分を復元して、stagingへもう一度コードをプッシュします。

[down]
- terraform destroy
- 手動でCloudFront, Lambda@edge, s3を全部削除します




Code Example:
my-next-app/bin/serverless/serverless.staging.yml
~~~
my-next-app:
  component: "@sls-next/serverless-component"
  inputs:
    build:
      env:
        BASE_URL: ${env.STAGING_BASE_URL}
        API_BASE_URL: ${env.STAGING_API_BASE_URL}
    domain: ["staging", "${env.DOMAIN}"]
    bucketName: "careerchat-serverless-staging"
    name:
      defaultLambda: "careerchat-serverless-default-func-staging"
    memory:
      defaultLambda: 2048
    timeout:
      defaultLambda: 30
    cloudfront:
      distributionId: ${env.STAGING_DISTRIBUTION_ID}
      aliases: ["${env.STAGING_DOMAIN}"]
      defaults:
        forward:
          headers:
            - "Host"
            - "X-CSRF-Token"
          # cookies:
          #   - "_jobb_session"
          # cookies: "none"
      _next/data/*:
        forward:
          headers:
            - "Host"
            - "X-CSRF-Token"
          # cookies:
          #   - "_jobb_session"
          # cookies: "none"
      origins:
        - url: ${env.STAGING_ALB_URL}
          protocolPolicy: "match-viewer"
          pathPatterns:
            "/api/*":
              minTTL: 0
              defaultTTL: 10
              maxTTL: 10
              allowedHttpMethods:
                - "HEAD"
                - "GET"
                - "DELETE"
                - "POST"
                - "PUT"
                - "PATCH"
                - "OPTIONS"
              forward:
                headers:
                  - "Host"
                  - "X-CSRF-Token"
                cookies: "all"
                # cookies:
                #   - "_jobb_session"
                # cookies: "none"
                queryString: true
              viewerProtocolPolicy: "redirect-to-https"
            "/admins":
              minTTL: 0
              defaultTTL: 10
              maxTTL: 10
              forward:
                headers:
                  - "Host"
                  - "X-CSRF-Token"
                cookies: "all"
                # cookies:
                #   - "_jobb_session"
                # cookies: "none"
                queryString: false
              viewerProtocolPolicy: "redirect-to-https"
            "/admins/*":
              minTTL: 0
              defaultTTL: 10
              maxTTL: 10
              allowedHttpMethods:
                - "HEAD"
                - "GET"
                - "DELETE"
                - "POST"
                - "PUT"
                - "PATCH"
                - "OPTIONS"
              forward:
                headers:
                  - "Host"
                  - "X-CSRF-Token"
                cookies: "all"
                # cookies:
                #   - "_jobb_session"
                # cookies: "none"
                queryString: false
              viewerProtocolPolicy: "redirect-to-https"
            "/admin_users/*":
              minTTL: 0
              defaultTTL: 10
              maxTTL: 10
              allowedHttpMethods:
                - "HEAD"
                - "GET"
                - "DELETE"
                - "POST"
                - "PUT"
                - "PATCH"
                - "OPTIONS"
              forward:
                headers:
                  - "Host"
                  - "X-CSRF-Token"
                cookies: "all"
                # cookies:
                #   - "_jobb_session"
                # cookies: "none"
                queryString: true
              viewerProtocolPolicy: "redirect-to-https"
            "/users/*":
              minTTL: 0
              defaultTTL: 10
              maxTTL: 10
              allowedHttpMethods:
                - "HEAD"
                - "GET"
                - "DELETE"
                - "POST"
                - "PUT"
                - "PATCH"
                - "OPTIONS"
              forward:
                headers:
                  - "Host"
                  - "X-CSRF-Token"
                cookies: "all"
                # cookies:
                #   - "_jobb_session"
                # cookies: "none"
                queryString: true
              viewerProtocolPolicy: "redirect-to-https"
            "/rails/*":
              minTTL: 0
              defaultTTL: 10
              maxTTL: 10
              allowedHttpMethods:
                - "HEAD"
                - "GET"
                - "DELETE"
                - "POST"
                - "PUT"
                - "PATCH"
                - "OPTIONS"
              forward:
                headers:
                  - "Host"
                  - "X-CSRF-Token"
                cookies: "all"
                # cookies:
                #   - "_jobb_session"
                # cookies: "none"
                queryString: true
              viewerProtocolPolicy: "redirect-to-https"
            "/sitemap":
              minTTL: 0
              defaultTTL: 10
              maxTTL: 10
              forward:
                headers:
                  - "Host"
                  - "X-CSRF-Token"
                cookies: "all"
                # cookies:
                #   - "_jobb_session"
                # cookies: "none"
                queryString: false
              viewerProtocolPolicy: "redirect-to-https"
            "/healthcheck":
              minTTL: 0
              defaultTTL: 10
              maxTTL: 10
              forward:
                headers:
                  - "Host"
                  - "X-CSRF-Token"
                cookies: "all"
                # cookies:
                #   - "_jobb_session"
                # cookies: "none"
                queryString: false
              viewerProtocolPolicy: "redirect-to-https"
      certificate:
        cloudFrontDefaultCertificate: false
        acmCertificateArn: ${env.ACM_SSL_CERTIFICATE_ARN}
~~~

my-next-app/bin/deploy.sh
~~~sh
#!/bin/bash

set -e

export BUCKET_NAME=careerchat-serverless-$STAGE

# 念のためクリーンアップ
if [ -e ./serverless.yml ]; then
    rm ./serverless.yml
fi
if [ -d ./.serverless ]; then
    rm -rf ./.serverless
fi
if [ -d ./.serverless_nextjs ]; then
    rm -rf ./.serverless_nextjs
fi

# serverlessの設定ファイルをコピー
cp ./bin/serverless/serverless.$STAGE.yml ./serverless.yml

# S3からデプロイ設定ファイルを復元
aws s3 cp s3://$BUCKET_NAME/.serverless ./.serverless --recursive

# デプロイ
npx serverless

# S3にデプロイ設定ファイルを退避
aws s3 cp ./.serverless s3://$BUCKET_NAME/.serverless --recursive

# 後片付け
rm ./serverless.yml
rm -rf ./.serverless
rm -rf ./.serverless_nextjs
~~~

my-next-app/.circleci/config.yml
~~~
---
version: 2.1

orbs:
  ......
  aws-cli: circleci/aws-cli@1.3.1

jobs:
  build:
    ......
  deploy:
    parameters:
      env:
        type: enum
        enum: ["staging", "production"]
      aws-access-key-id:
        type: env_var_name
      aws-secret-access-key:
        type: env_var_name
      aws-region:
        type: env_var_name
    working_directory: ~/NinNinInc/job-frontend
    docker:
      - image: circleci/node:10.21-browsers
    steps:
      - aws-cli/setup:
          aws-access-key-id: << parameters.aws-access-key-id >>
          aws-secret-access-key: << parameters.aws-secret-access-key >>
          aws-region: << parameters.aws-region >>
      - checkout
      - run: npm i
      - run: chmod +x ./bin/deploy.sh
      - run: STAGE=<< parameters.env >> ./bin/deploy.sh

workflows:
  build-and-deploy:
    jobs:
      - build:
          post-steps:
            ......
      - deploy: &deploy
          name: deploy-to-staging
          env: staging
          aws-access-key-id: AWS_ACCESS_KEY_ID
          aws-secret-access-key: AWS_SECRET_ACCESS_KEY
          aws-region: AWS_DEFAULT_REGION
          requires:
            - build
          filters:
            branches:
              only: serverless
  ......

~~~
