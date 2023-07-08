---
hide:
  - footer
---

# test 环境方案

## 流程

前置操作
- git clone 项目到本地
- git checkout develop

1. 从 develop 创建一个分支来完成新功能的开发, git checkout -b feature/feature-a
2. 代码添加修改后
   1. git add . 
   2. git commit -m "feature: hello world feature A"
3. 将代码推送到代码库的对应分支
   1. git push origin feature/feature-a
4. feature/feature-a 分支 push 到远程仓库后
   1. 自动部署到 test 服务器（功能测试）
   2. 跑单元测试（如果有的话）
   3. 跑代码分析（如果有的话）


## drone yaml 配置示例

```yaml

---

kind: pipeline
type: docker
name: feature

platform:
  os: linux
  arch: amd64

trigger:
  branch:
    include:
      - feature/*

clone:
  disable: true

steps:
- name: deploy
  image: appleboy/drone-ssh
  settings:
    host:
      from_secret: test_host
    username: root
    key:
      from_secret: test_ssh_key
    port: 22
    script_stop: true
    script:
      - cd /xxx/xxx/xxx
      - git fetch
      - git reset --hard ${DRONE_COMMIT}
      - docker-compose down
      - docker-compose up -d --build --force-recreate
      - docker image prune -f
  when:
    event:
      include:
        - push
        - custom
- name: clone
  image: alpine/git
  commands:
    - git clone ${DRONE_GIT_HTTP_URL} .
    - git checkout ${DRONE_BRANCH}
  when:
    event:
      include:
        - push
        - custom
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
        - push
        - custom

volumes:
- name: sonarqube_cache
  host:
    path: /xxx/xxx/sonarqube_cache
```