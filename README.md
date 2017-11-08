## Introduction

**Docker** is great tool to management linux containers. It brings DevOps to next level, from development to production environment. And of course, before deploy anything to production, software should be tested carefully and **automatically**.

That's why **Drone**, a new lightweight CI server built-on top Go lang and Docker, will help us to resolve the testing problems in simple and fast way.

## Setup

This guide will assume you already have Docker and Docker Compose tool. And of course, root permission ;)

**Step 1** : Clone my example docker-compose here : https://github.com/khanhicetea/drone-ci

```bash
$ git clone https://github.com/khanhicetea/drone-ci
$ cd drone-ci
$ cp .env.example .env
```

**Step 2** : Update your setting in `.env` file

**Step 3** : Run drone via docker-compose

```bash
$ source .env
$ sudo docker-compose up -d
```

**Step 4** : Go to your Drone url (remember use https url), then authorize with Github provider.

## Usage

In example repo, I created a sample `.drone.sample.yml` file so you can follow the structure to create own file.

I will explain some basics here

```yaml
clone:
  git:
    image: plugins/git
    depth: 5

pipeline:
  phpunit:
    image: php:7
    commands:
      - /bin/sh conflict_detector.sh
      - /bin/sh phplinter.sh app lib test
      - composer install -q --prefer-dist
      - test/db/import testdatabase root passwd testdb test.sql
      - php -d memory_limit=256M vendor/bin/phpunit --no-coverage --colors=never
  
  notify:
    image: plugins/slack
    webhook: [your_slack_webhook_url]
    channel: deployment
    username: DroneCI
    when:
      status: success
  
  notify-bug:
    image: plugins/slack
    webhook: [your_slack_webhook_url]
    channel: bugs
    username: DroneCI
    when:
      status: failure
      branch: production

services:
  testdatabase:
    image: mysql:5.7
    detach: true
    environment:
      - MYSQL_DATABASE=testdb
      - MYSQL_ROOT_PASSWORD=passwd
      - MYSQL_ALLOW_EMPTY_PASSWORD=yes
```

**This file consists 3 sections :**

- clone : To clone the source code and prepare for `pipeline` step. This section will be run first
- services : Declare your docker services (databases, ip server) which source code connect to. This section will be run at sametime with `pipeline` (after `clone`)
- pipeline : Testing pipe, where you put testing logic here.

**In this `pipeline`, I made a example PHP testing through these steps :**

1. Check conflicts in code (grep for `>>>> HEAD` string)
2. Run PHP linter in application codes
3. Run Composer to install all dependencies
4. Import testing database to mysql services (using `testdatabase` hostname to connect service)
5. Run testing script via `phpunit` tool

Then, notify testing result via Slack channel ! ;)