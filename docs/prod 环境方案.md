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

          - CI 将做语义化发布，对 main 分支最新 commit 自动打上版本号并书写 ChangeLog
          - 这个 releas 会推送到远程仓库
    
      - 此时 CI 收到 main 分支的 tag 事件
        
          - 构建镜像并推送到镜像仓库
          - 部署到 prod 服务器（生产环境）
            - 通知 k8s 更新镜像

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
name: main-merge-pull-request-prod

platform:
  os: linux
  arch: amd64

trigger:
  branch:
    include:
      - main
  event:
    include:  
      - push

clone:
  disable: true

steps:
  - name: semantic-release  
    image: gtramontina/semantic-release:17.4.3
    environment:  
      GITHUB_TOKEN:  
        from_secret: gitea_token  
    entrypoint:  
      - semantic-release
    
---

kind: pipeline
type: docker
name: main-tag-prod

platform:
  os: linux
  arch: amd64

trigger:
  branch:
    include:
      - main
  event:
    include:
      - tag

clone:
  disable: true

steps:
- name: clone-repo
  image: alpine/git
  commands:
    - git clone ${DRONE_GIT_HTTP_URL} .
    - git checkout ${DRONE_BRANCH}
- name: build-and-push-image
  image: plugins/docker
  settings:
    registry:
      from_secret: docker_registry
    username:
      from_secret: docker_username
    password:
      from_secret: docker_password
    repo: gitea.bearcatlog.com/bryant/admin-service
    context: .
    dockerfile: ./Dockerfile
    tags:
    - ${DRONE_TAG}
    # auto_tag: true
    purge: true
    compress: true
# TODO 通知 k8s 更新镜像
```

