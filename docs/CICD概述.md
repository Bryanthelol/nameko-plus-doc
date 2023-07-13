<!-- ---
hide:
  - footer
--- -->

# CICD概述

## 目标

将团队开发、测试、发布的流程全部自动化。开发人员只需要关注自己的核心任务：按照工作流规范，写好代码，写好 Commit，提交代码就可以完成日常的所有 DevOps 工作。

## 工具

- Gitea 代码托管/镜像仓库
- Drone CI/CD 平台
- docker compose （test，sit，staging 环境）
- Kubernetes 容器编排系统（prod 环境）

## 策略

1. 单人单分支手动发布
2. 团队多分支 GitFlow 工作流
3. 团队多分支 semantic-release 语义化发布（需要代码托管平台为 github / gitlab）
4. 通知 Kubernetes 发布

目前采用 gitflow 工作流

TODO

- 缺乏一个 hotfix 分支策略
- 缺乏一个发布分支策略
- 缺乏一个回滚策略
  
    - feature 分支的回滚
        - 可以 reset ，然后 push -f
    - develop 和 main 分支
        - 只能 revert （对应的是 push 事件）

## 备忘

git emoji

https://github.com/tony-yin/git-emoji