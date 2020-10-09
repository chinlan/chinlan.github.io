---
layout: post
title: "NextJS: set custom environment"
description: ""
categories: [devops]
tags: [frontend, nextJS, react]
redirect_from:
  - /2020/09/21/
---
Goal: NextJS only support `development`, `test`, and `production` environment, the goal is adding a `staging` environment setting.

Solution 1: Using `dotenv`
`npm install dotenv`

next.config.js
~~~js
require("dotenv").config({ path: `${process.env.ENVIRONMENT}` });

......
  env: {
    'BASE_URL': process.env.BASE_URL,
    'API_BASE_URL': process.env.API_BASE_URL,
    'ENVIRONMENT': process.env.ENVIRONMENT,
  }
......
~~~

package.json
~~~js
"scripts": {
  "dev": "ENVIRONMENT=env/.env.dev next dev -p 4000",
  "build:staging": "ENVIRONMENT=env/.env.staging next build",
  "start:staging": "ENVIRONMENT=env/.env.staging next start -p 4000",
  "build": "ENVIRONMENT=env/.env.production next build",
  "start": "ENVIRONMENT=env/.env.production next start -p 4000"
}
~~~

Add env files for each environments.
NOTE: Do not put env files under root directory, or will conflict with nextJS default behaviour.

env/.example.env.dev
~~~js
BASE_URL=http://localhost:3000
API_BASE_URL=http://web:3000
~~~
env/.example.env.staging
~~~js
BASE_URL=https://staging.xxx.com
API_BASE_URL=https://staging.xxx.com
~~~
env/.example.env.production
~~~js
BASE_URL=https://xxx.com
API_BASE_URL=https://xxx.com
~~~

Revise the code to use `process.env.BASE_URL` and `process.env.API_BASE_URL`.

Last, because we use docker, the env file will need to be copied to Docker image during building:

.circleci/production/Dockerfile-frontend-deploy
~~~
FROM node:14-alpine
RUN apk update && apk upgrade && \
    apk add --no-cache bash git openssh

COPY . /frontend
COPY ./env/.example.env.production /frontend/env/.env.production
WORKDIR /frontend

RUN npm i

RUN npm run build

EXPOSE 4000

CMD ["npm", "start"]
~~~

.circleci/staging/Dockerfile-frontend-deploy
~~~
FROM node:14-alpine
RUN apk update && apk upgrade && \
    apk add --no-cache bash git openssh

COPY . /frontend
COPY ./env/.example.env.staging /frontend/env/.env.staging
WORKDIR /frontend

RUN npm i

RUN npm run build:staging

EXPOSE 4000

CMD ["npm", "run", "start:staging"]
~~~

The contents contained in the env files are only url infos which is still ok to be exposured in git, but if later we need to add something secretly, this is not ideal.

Solution 2: Put env setting into Terraform ECS task-definition

To do this, most change made in solution 1 is not needed.

Still need to claim the env variables in setting file:
next.config.js
~~~js
......
     env: {
            'BASE_URL': process.env.BASE_URL,
            'API_BASE_URL': process.env.API_BASE_URL,
          }
......
~~~

Still need to use `process.env.BASE_URL` and `process.env.API_BASE_URL` in the code.

Scripts can stay simple:
package.json
~~~js
......
 "scripts": {
    "dev": "next dev -p 4000",
    "build": "next build",
    "start": "next start -p 4000"
  },
......
~~~

Add env variables into Terraform ECS frontend task definition:
~~~
......
        "environment": [
          {
            "name": "BASE_URL",
            "value": "${var.base_url}"
          },
          {
            "name": "API_BASE_URL",
            "value": "${var.api_base_url}"
          }
        ],
......
~~~

Now the env variables is defined as container env variables, but in our circleCI script, `npm build` is done during building the image, at that time, container env variables is not accessable since no container is started.
(It cannot read container env variable when it is `started`, the reading can only be done during `build`)
So we need to move the `npm build` process out of image building process, make it build and start after the container is started.

.circleci/staging/Dockerfile-frontend-deploy
~~~
FROM node:14-alpine
RUN apk update && apk upgrade && \
    apk add --no-cache bash git openssh

COPY . /frontend
WORKDIR /frontend

ADD .circleci/staging/frontend-entrypoint.sh /
RUN chmod +x /frontend-entrypoint.sh
ENTRYPOINT ["/frontend-entrypoint.sh"]
~~~

.circleci/staging/frontend-entrypoint.sh
~~~
#!/bin/sh
set -ex

npm i

npm run build

npm run start
~~~

.circleci/production/Dockerfile-frontend-deploy
~~~
FROM node:14-alpine
RUN apk update && apk upgrade && \
    apk add --no-cache bash git openssh

COPY . /frontend
WORKDIR /frontend

ADD .circleci/production/frontend-entrypoint.sh /
RUN chmod +x /frontend-entrypoint.sh
ENTRYPOINT ["/frontend-entrypoint.sh"]
~~~

.circleci/production/frontend-entrypoint.sh
~~~
#!/bin/sh
set -ex

npm i

npm run build

npm run start
~~~

[Reference](https://stackoverflow.com/questions/59462614/how-to-use-diferent-env-files-with-nextjs)

Build after container is start will need set higher memory in ECS task_definition container_definition.

In the end we set the env variables in circleCI env and move the build process back to Dockerfile.

.circleci/config.yml
~~~
workflows:
  build-and-deploy:
    jobs:
      - build
      - aws-ecr/build-and-push-image:
          name: build-staging-image
          account-url: AWS_ECR_ACCOUNT_URL
          aws-access-key-id: AWS_ACCESS_KEY_ID
          aws-secret-access-key: AWS_SECRET_ACCESS_KEY
          region: AWS_DEFAULT_REGION
          dockerfile: .circleci/staging/Dockerfile-frontend-deploy
          extra-build-args: "--build-arg BASE_URL=${STAGING_BASE_URL} --build-arg API_BASE_URL=${STAGING_API_BASE_URL}"
          repo: "job-docker_frontend_deploy"
          tag: "latest,${CIRCLE_SHA1}"
          requires:
            - build
          filters:
            branches:
              only: staging
      - aws-ecs/deploy-service-update:
          name: deploy-to-staging
          family: "ninninjob-frontend-staging"
          cluster-name: "ninninjob-staging"
          skip-task-definition-registration: true
          force-new-deployment: true
          requires:
            - build-staging-image
          filters:
            branches:
              only: staging
      - aws-ecr/build-and-push-image:
          name: build-production-image
          account-url: AWS_ECR_ACCOUNT_URL
          aws-access-key-id: AWS_ACCESS_KEY_ID
          aws-secret-access-key: AWS_SECRET_ACCESS_KEY
          region: AWS_DEFAULT_REGION
          dockerfile: .circleci/production/Dockerfile-frontend-deploy
          extra-build-args: "--build-arg BASE_URL=${BASE_URL} --build-arg API_BASE_URL=${API_BASE_URL}"
          repo: "job-docker_frontend_deploy_production"
          tag: "latest,${CIRCLE_SHA1}"
          requires:
            - build
          filters:
            branches:
              only: production
      - aws-ecs/deploy-service-update:
          name: deploy-to-production
          family: "ninninjob-frontend-production"
          cluster-name: "ninninjob-production"
          skip-task-definition-registration: true
          force-new-deployment: true
          requires:
            - build-production-image
          filters:
            branches:
              only: production
~~~

.circleci/staging/Dockerfile-frontend-deploy
~~~
FROM node:14-alpine
RUN apk update && apk upgrade && \
    apk add --no-cache bash git openssh

ARG BASE_URL
ARG API_BASE_URL

COPY . /frontend
WORKDIR /frontend

RUN npm i

RUN BASE_URL=${BASE_URL} API_BASE_URL=${API_BASE_URL} npm run build

EXPOSE 4000

CMD ["npm", "run", "start"]
~~~

.circleci/production/Dockerfile-frontend-deploy
~~~
FROM node:14-alpine
RUN apk update && apk upgrade && \
    apk add --no-cache bash git openssh

ARG BASE_URL
ARG API_BASE_URL

COPY . /frontend
WORKDIR /frontend

RUN npm i

RUN BASE_URL=${BASE_URL} API_BASE_URL=${API_BASE_URL} npm run build

EXPOSE 4000

CMD ["npm", "run", "start"]
~~~



