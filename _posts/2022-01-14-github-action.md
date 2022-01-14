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
