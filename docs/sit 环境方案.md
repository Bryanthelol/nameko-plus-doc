---
# hide:
#   - footer
---


# sit 环境方案

## 流程

前置条件

- feature/feature-a 分支的功能在 test 环境通过验收后

主流程

1. 相关人员发起 Pull Request 到 develop 分支
2. 由相关人员进行 Code Review 
3. Code Review 通过后，相关人员合并当前 Pull Request

      - 此时 CI 收到 develop 分支的 pull_request 事件

          - 跑单元测试（如果有的话）
          - 部署到 sit 服务器（集成测试）
          - 跑代码扫描（如果有的话）

备注：

- develop 分支禁止 Push
- 合代码通过 Pull Request

## drone yaml 配置示例

需要准备的 secret：

- develop_host
- develop_ssh_key
- gitea_token
- sonar_host
- sonar_token

```yaml

---

kind: pipeline
type: docker
name: develop

platform:
  os: linux
  arch: amd64

trigger:
  branch:
    include:
      - develop

clone:
  disable: true

steps:
- name: deploy
  image: appleboy/drone-ssh
  settings:
    host:
      from_secret: develop_host
    username: root
    key:
      from_secret: develop_ssh_key
    port: 22
    script_stop: true
    script:
      - cd /xxx/xxx/xxx
      - git fetch
      - git checkout ${DRONE_BRANCH}
      - git reset --hard ${DRONE_COMMIT}
      - docker-compose down
      - docker-compose up -d --build --force-recreate
      - docker image prune -f
  when:
    event:
      include:
        - pull_request
    action:
      include:
        - opened
        - reopened
        - synchronized
- name: gitea-pr-comment-failure
  image: tsakidev/giteacomment:latest
  settings:
    gitea_token:
      from_secret: gitea_token
    gitea_base_url: http://gitea.example.com
    comment_title: "部署 sit 环境失败"
    comment: "${DRONE_PULL_REQUEST_TITLE} 部署 sit 环境失败"
  when:
    status: 
      - failure
    event:
      include:
        - pull_request
    action:
      include:
        - opened
        - reopened
        - synchronized
- name: gitea-pr-comment-success
  image: tsakidev/giteacomment:latest
  settings:
    gitea_token:
      from_secret: gitea_token
    gitea_base_url: http://gitea.example.com
    comment_title: "部署 sit 环境成功"
    comment: "${DRONE_PULL_REQUEST_TITLE} 部署 sit 环境成功"
  when:
    status: 
      - success
    event:
      include:
        - pull_request
    action:
      include:
        - opened
        - reopened
        - synchronized
- name: clone
  image: alpine/git
  commands:
    - git clone ${DRONE_GIT_HTTP_URL} .
    - git checkout ${DRONE_BRANCH}
  when:
    event:
      include:
        - pull_request
    action:
      include:
        - opened
        - reopened
        - synchronized
- name: code-analysis
  image: aosapps/drone-sonar-plugin
  volumes:
    - name: sonarqube_cache
      path: /root
  settings:
    sonar_host:
      from_secret: sonar_host
    sonar_token:
      from_secret: sonar_token
    sources: .
    level: DEBUG
    showProfiling: true
  when:
    event:
      include:
        - pull_request
    action:
      include:
        - opened
        - reopened
        - synchronized

volumes:
- name: sonarqube_cache
  host:
    path: /xxx/xxx/sonarqube_cache
```