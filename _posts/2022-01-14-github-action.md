---
layout: post
title: github workflows
categories: blog
keywords: github workflows
---

# github workflows

> 每次将本站点仓库推送到github,都会收到  `Build and Deply:All jobs have failed` 的邮件,
> 但是当时找不出原因.最近看到有关`github workflows`的概念,才明白错误发生在workflows上.

### workflow
* 工作流程是您添加到仓库的自动化过程。
* 工作流程由一项或多项 jobs 组成，可以计划或由事件触发。
* 工作流程可用于在 GitHub 上构建、测试、打包、发布或部署项目。

> 原项目中有一个 `Build and Deply`的workflow, 执行任务失败,所以才会发送相应的错误邮件.

> 根目录下的 `.github/workflows/`路径下`.yml`文件定义了工作流.

原文件 `ci.yml`代码为

```yaml
name: Build and Deploy

on:
  push:
    branches: [ master ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2.3.1
        with: 
          persist-credentials: false

      - name: Set Ruby 2.7
        uses: actions/setup-ruby@v1
        with:
          ruby-version: 2.7

      - name: Install and Build
        run: |
          gem install bundler
          bundle install
          bundle exec jekyll build
        
      - name: Deploy
        uses: JamesIves/github-pages-deploy-action@3.6.2
        with:
          ACCESS_TOKEN: ${{ secrets.ACCESS_TOKEN }}
          BRANCH: built
          FOLDER: _site
          CLEAN: true

```
改为 github 上jekyll默认操作

```yaml
name: Jekyll site CI

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
      - name: Build the site in the jekyll/builder container
        run: |
          docker run \
          -v ${{ github.workspace }}:/srv/jekyll -v ${{ github.workspace }}/_site:/srv/jekyll/_site \
          jekyll/builder:latest /bin/bash -c "chmod -R 777 /srv/jekyll && jekyll build --future"

```
github 的 `blank.yml`

```yaml
# This is a basic workflow to help you get started with Actions

name: CI

# Controls when the workflow will run
on:
  # Triggers the workflow on push or pull request events but only for the main branch
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v2

      # Runs a single command using the runners shell
      - name: Run a one-line script
        run: echo Hello, world!

      # Runs a set of commands using the runners shell
      - name: Run a multi-line script
        run: |
          echo Add other actions to build,
          echo test, and deploy your project.

```

> 查看github Actions显示成功.

> 经过摸索,发现: 当github的仓库开启page功能,github会为该仓库自动添加一个Action,名字叫
> `pages build and deployment`,包含两部分工作

1. build.里面主要有一个任务是 build page with jekell
2. deploy.将_site发布到github page, 并绑定自定义域名

> 所以,事实上,自定义的workflow可以删除,并不是该项目必须的.
