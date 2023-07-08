---
hide:
  - footer
---

# CICD概述

## 目标

将团队开发、测试、发布的流程全部自动化，开发人员只需要认真提交代码就可以完成日常的所有 DevOps 工作。

## 工具

- Gitea 代码托管/镜像仓库
- Drone CI/CD 平台
- Kubernetes 容器编排系统

## 策略

1. 单人单分支手动发布
2. 团队多分支 GitFlow 工作流
3. 团队多分支 semantic-release 语义化发布
4. 通知 Kubernetes 全自动发布

目前采用 gitflow 工作流

## 备忘

pull request 事件的 action 包含如下：

- assigned
- unassigned
- labeled
- unlabeled
- opened
- edited
- closed
- reopened
- synchronize
- converted_to_draft
- ready_for_review
- locked
- unlocked
- review_requested
- review_request_removed
- auto_merge_enabled
- auto_merge_disabled

