---
hide:
  - footer
---

# prod 环境方案

## 流程

前置条件

- 测试人员在预发布环境下再次验证功能，团队做上线前的其他准备工作

主流程

1. 功能验证通过，相关人员合并 Pull Request

      - 此时 CI 收到 main 分支的 pull_request 的 close 事件
          - CI 将为本次发布的代码及镜像自动打上版本号并书写 ChangeLog
          - 部署到 prod 服务器（生产环境）

备注：

- main 分支禁止 Push
- 合代码通过 Pull Request

## drone yaml 配置示例

需要准备的 secret：

- staging_host
- staging_ssh_key
- gitea_token

```yaml

---

kind: pipeline
type: docker
name: main-pull-request-prod

platform:
  os: linux
  arch: amd64

trigger:
  branch:
    include:
      - main

clone:
  disable: true

steps:
- name: deploy
  image: appleboy/drone-ssh
  settings:
    host:
      from_secret: staging_host
    username: root
    key:
      from_secret: staging_ssh_key
    port: 22
    script_stop: true
    script:
      - TODO
  when:
    event:
      - pull_request
    action:
      - closed
- name: gitea-pr-comment-failure
  image: tsakidev/giteacomment:latest
  settings:
    gitea_token:
      from_secret: gitea_token
    gitea_base_url: http://gitea.example.com
    comment_title: "部署正式环境失败"
    comment: "${DRONE_PULL_REQUEST_TITLE} 部署正式环境失败"
  when:
    status: 
      - failure
    event:
      include:
        - pull_request
    action:
      - closed
- name: gitea-pr-comment-success
  image: tsakidev/giteacomment:latest
  settings:
    gitea_token:
      from_secret: gitea_token
    gitea_base_url: http://gitea.example.com
    comment_title: "部署正式环境成功"
    comment: "${DRONE_PULL_REQUEST_TITLE} 部署正式环境成功"
  when:
    status: 
      - success
    event:
      include:
        - pull_request
    action:
      - closed

```

