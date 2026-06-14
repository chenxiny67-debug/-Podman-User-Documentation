| **<font style="background-color:rgba(255, 255, 255, 0);">文件</font>** | **<font style="background-color:rgba(255, 255, 255, 0);">作用</font>** |
| --- | --- |
| Dockerfile | <font style="background-color:rgba(255, 255, 255, 0);">构建镜像、基础环境及依赖</font> |
| docker-compose.yml | <font style="background-color:rgba(255, 255, 255, 0);">启动多容器、配置网络</font> |
| .dockerignore | <font style="background-color:rgba(255, 255, 255, 0);">忽略不需要的文件和目录</font> |
| .gitlab-ci.yml | <font style="background-color:rgba(255, 255, 255, 0);">GitLab 自动化流水线阶段、脚本任务</font> |


# Podman
## Podman 连接 Desktop 
1. 宿主机 [参考](https://desktop.podman.org.cn/docs/podman/podman-remote)

```bash
sudo apt update
sudo apt install -y podman

# 添加 Flathub 仓库
flatpak remote-add --if-not-exists --user flathub \
https://flathub.org/repo/flathub.flatpakrepo

# 安装 Podman Desktop
flatpak install --user flathub io.podman_desktop.PodmanDesktop

# 更新 Podman Desktop
flatpak update --user io.podman_desktop.PodmanDesktop

# 创建 ed25519算法密钥 ，给予server
ssh-keygen -t ed25519
ssh-copy-id -i ~/.ssh/id_ed25519.pub root@123.57.64.74

```

2. 服务器

```bash
sudo apt install -y podman

# 服务器启用接口可让远程客户端控制 Podman
systemctl --user enable podman.socket
systemctl --user start podman.socket
```

3. 宿主机

```bash
# SSH 添加到本地 Podman 的连接列表里 
podman system connection add yan-servers \
--identity ~/.ssh/id_ed25519 \
ssh://root@123.57.64.74

# 查看
podman system connection list     

# 删除
podman system connection rm yan-servers 
```

# Docker
## 镜像操作
+ [<font style="background-color:rgba(255, 255, 255, 0);">仓库地址</font>](https://hub.docker.com/explore)

```bash
podman pull </>    # 下载镜像
podman images      # 查看已下载镜像
podman rmi </>     # 删除镜像
```

## 容器操作
```bash
podman ps         # 查看正在运行的
podman stats      # 查看资源占比

podman start <容器名>     # 启动容器
podman stop <容器名>      # 停止容器
podman restart <容器名>   # 重启容器
podman rm <容器名>        # 删除容器
podman inspect <容器名>   # 容器信息

podman logs <容器名>                 # 查看日志
podman logs -f --tail 100 <容器名>   # 查看日志最后100行 并开始实时跟随
podman exec -it <容器名> sh          # 进入容器终端
podman exec <容器名> <命令>           # 使用命令
```

| **<font style="background-color:rgba(255, 255, 255, 0);">重启策略</font>** | **<font style="background-color:rgba(255, 255, 255, 0);">报错崩溃</font>** | **<font style="background-color:rgba(255, 255, 255, 0);">正常退出</font>** | **<font style="background-color:rgba(255, 255, 255, 0);">手动 stop</font>** | **<font style="background-color:rgba(255, 255, 255, 0);">宿主机重启后</font>** |
| --- | --- | --- | --- | --- |
| --restart always | <font style="background-color:rgba(255, 255, 255, 0);">是</font> | <font style="background-color:rgba(255, 255, 255, 0);">是</font> | <font style="background-color:rgba(255, 255, 255, 0);"></font><font style="background-color:rgba(255, 255, 255, 0);">不</font> | <font style="background-color:rgba(255, 255, 255, 0);">不</font> |
| <font style="background-color:rgba(255, 255, 255, 0);">--restart unless-stopped</font> | <font style="background-color:rgba(255, 255, 255, 0);">是</font> | <font style="background-color:rgba(255, 255, 255, 0);">是</font> | <font style="background-color:rgba(255, 255, 255, 0);"></font><font style="background-color:rgba(255, 255, 255, 0);">不</font> | <font style="background-color:rgba(255, 255, 255, 0);">不</font> |
| --restart on-failure | <font style="background-color:rgba(255, 255, 255, 0);">是</font> | <font style="background-color:rgba(255, 255, 255, 0);">不</font> | <font style="background-color:rgba(255, 255, 255, 0);"></font><font style="background-color:rgba(255, 255, 255, 0);">不</font> | <font style="background-color:rgba(255, 255, 255, 0);">是</font> |


```bash
podman run -d \         # 后台运行
  --name nginx \        # 容器名字
  --restart always \    # 重启策略
  --memory=512m \       # 最多512mb内存
  --cpus=1.0 \          # 最多1核cpu算力
  --pids-limit=1024 \   # 最多同时1024 个进程/线程
  -p 80:80 \            # 宿主机与容器端口

  # 挂载卷 
  -v nginx_conf:/etc/nginx/conf.d:ro \
  -v nginx_html:/usr/share/nginx/html:ro \
  -v nginx_logs:/var/log/nginx \
  -v nginx_ssl:/etc/ssl/nginx:ro \
  
  # 镜像
  nginx:stable-alpine3.23
```

## 卷 / 网络管理
```bash
# 卷操作
podman volume create <名>      # 创建卷
podman volume inspect <被选名>  # 查看卷详情
podman volume ls               # 列出所有卷
podman volume prune -a         # 删除未使用的卷（谨慎执行）

# 网络操作
podman network create <名子>   # 创建子网
podman network ls              # 查看所有网络
podman network rm <被选名>      # 删除子网
podman network inspect <被选名> # 查看网络详情（如子网段、网关）

podman network connect <网络名> <容器名称或 ID>    # 添加
podman network disconnect <网络名称> <容器名称或 ID> # 退出
```

## 导入 / 导出
```bash
# 镜像导出为 tar 包
podman save -o <镜像名>.tar <镜像名:标签>
podman load -i <镜像名>.tar               # 导入


# 容器导出为 tar 包
podman export -o <容器名>.tar <容器名或ID>
podman import <容器名>.tar <新镜像名:标签>  # 导入

# 容器快照 (正在运行)
podman commit <容器名> <新镜像名>
```

# GitLab
**<font style="background-color:rgba(255, 255, 255, 0);">CI (持续集成)</font>**<font style="background-color:rgba(255, 255, 255, 0);">： </font>`<font style="background-color:rgba(255, 255, 255, 0);">代码提交</font>`<font style="background-color:rgba(255, 255, 255, 0);"> → </font>`<font style="background-color:rgba(255, 255, 255, 0);">自动构建</font>`<font style="background-color:rgba(255, 255, 255, 0);"> → </font>`<font style="background-color:rgba(255, 255, 255, 0);">自动测试</font>`

**<font style="background-color:rgba(255, 255, 255, 0);">CD (持续交付)</font>**<font style="background-color:rgba(255, 255, 255, 0);">： </font>`<font style="background-color:rgba(255, 255, 255, 0);">自动部署到预生产/测试环境</font>`<font style="background-color:rgba(255, 255, 255, 0);"> → </font>`**<font style="background-color:rgba(255, 255, 255, 0);">人工批准/点击发布</font>**`<font style="background-color:rgba(255, 255, 255, 0);"> → </font>`<font style="background-color:rgba(255, 255, 255, 0);">自动部署到生产环境</font>`

<font style="background-color:rgba(255, 255, 255, 0);"></font>

<font style="color:#DF2A3F;background-color:rgba(255, 255, 255, 0);">访问 - SSH密钥 -添加</font>

```bash
# 多行的SSH私钥无损压缩成一行纯字母数字字符串

base64 -w0 ~/.ssh/id_ed25519
```

+ <font style="background-color:rgba(255, 255, 255, 0);">推送现有</font>

```bash
git config --local user.name "User Q-Hook"
git config --local user.email "241695-Q-Hook@users.noreply.jihulab.com"

git add .
commit -m "nestjs new"

git remote add origin git@jihulab.com:Q-Hook/q-hook-nestjs.git
git push --set-upstream origin --all
```

```yaml
# 克隆
git clone git@jihulab.com:Q-Hook/q-hook-project.git

```

## 构建 gitlab-ci.yml
```yaml
nestjs:
  # 做前
  before_script:
    - echo "安装依赖"

  # 执行命令
  script:
    - echo "执行命令"

  # 做完
  after_script:
    - echo "删除没必要的文件"
```

### stage 顺序
+ 同一个 stage 内 job 是“并行执行”，可以用 needs 先执行

```yaml
stages:
  - nginx
  - nestjs

nginx:
  stage: nginx
  needs:
    - vue
  script:
    - echo "nginx 测试中"

vue:
  stage: nginx
  script:
    - echo "vue 测试中"

nestjs:
  stage: nestjs
  script:
    - echo "nestjs 测试中"
```

### 引入外部脚本
```git
echo "脚本测试成功"
```

```yaml
stages:
  - nestjs

nestjs:
  stage: nestjs
  script:
    - chmod +x ./run.sh
    - ./run.sh
```

### 变量
+ 设置隐藏变量： 设置 → CI/CD → 变量 → 添加变量 

```yaml
stages:
  - nginx
  - nestjs

# 全局变量
variables:
  APP_NAME: "请稍候"

nginx:
  stage: nginx
  script:
    - echo "nginx 测试中 ${APP_NAME}"

# 局部变量 仅对当前 job 生效
nestjs:
  stage: nestjs
  variables:     
    NODE_ENV: test
  script:
    - echo $NODE_ENV
```

### 拉取镜像
```yaml
stages:
  - nginx
  - nestjs
  
# 全局使用镜像
image: nginx:stable
nginx-test:
  stage: nginx
  script:
    - nginx -v
    - nginx -V

# 单独使用镜像
nginx:
  stage: nginx
  image: nginx:stable-alpine3.23-perl
  script:
    - nginx -v
    - nginx -V
```

## 安装 GitLab Runner
+ [文档连接](https://docs.gitlab.com/runner/install/linux-repository/)

```bash
# 下载仓库配置脚本
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" -o script.deb.sh
sudo os=ubuntu dist=noble bash script.deb.sh # 检查脚本
sudo apt update
sudo apt install gitlab-runner # 安装

# 查看版本
gitlab-runner -v
```

接收和执行GitLab的CI/CD作业的进程

+ 设置 → CI/CD → Runner → 创建项目Runner → <font style="color:#DF2A3F;">填写标签名</font>
+ 注意：其他可用的项目运行器 可直接添加

```bash
gitlab-runner register # 1 注册
https://jihulab.com    # 2 公有云实例, 或者：私有 http://<服务器IP>
                       # 3 身份验证令牌 token
                       # 4 维护备注
                       # 5 选择执行器 shell ，docker 等等
```

+ 配置镜像源

```bash
# sudo vim /etc/containers/registries.conf

unqualified-search-registries = ["docker.io"]

[[registry]]
prefix = "docker.io"
location = "docker.io"
```

```bash
stages:
  - nginx
  
nginx_test:
  stage: nginx
  tags:
    - ubuntu # 指定标签
  script:
    - echo "nginx 安装中"
    - podman run -d --name nginx -p 80:80 nginx:stable
    - sleep 2
    - podman exec nginx nginx -v
    - podman exec nginx nginx -V
    - echo "测试完毕 删除中"
    - podman stop nginx
    - podman rm nginx

```

+ 手动验证 runner 是否可以选中作业

```bash
gitlab-runner run
```

+ 下载镜像

### Dockerfile
```bash
podman pull node:24.15-alpine
```

```bash
FROM node:24.15-alpine AS builder

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY . .

RUN ./node_modules/.bin/nest build

FROM node:24.15-alpine AS production

WORKDIR /app

COPY package*.json ./

RUN npm ci --only=production

COPY --from=builder /app/dist ./dist

EXPOSE 3000

CMD ["node", "dist/main"]
```

### .gitlab-ci.yml
```bash
stages:
  - build
  - deploy

build_image:
  stage: build
  tags:
    - ubuntu
  script:
    - podman build -t nestjs-app:latest .
    - podman save -o nestjs-image.tar nestjs-app:latest

  # 只保留一个小时
  artifacts:
    paths:
      - nestjs-image.tar
    expire_in: 1 hour
    
deploy_prod:
  stage: deploy
  tags:
    - ubuntu
  script:
  
    # 删除原有的
    - podman stop nestjs-prod 2>/dev/null || true
    - podman rm -f nestjs-prod 2>/dev/null || true

    # 导入镜像
    - podman load -i nestjs-image.tar
    - podman run -d --name nestjs-prod -p 3000:3000 --restart always nestjs-app:latest
    - sleep 2
    
    - rm -f nestjs-image.tar
    - podman image prune -f
```

### .dockerignore
```bash
test/
.dockerignore
.gitlab-ci.yml
.git/
*.md
Dockerfile
```



## Docker 仓库
### 推送
+ 部署 → 容器镜像仓库 

```bash
stages:
  - nginx
nginx:
  stage: nginx
  # 标签
  tags:
    - ubuntu
  before_script:
  # 登录
    - podman login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD registry.jihulab.com
  script:
  # 构建 
    - podman build -t registry.jihulab.com/q-hook/q-hook-project:1.0 -f ./Dockerfile .
  # 推送 
  # 注意：版本号要必须同时改
    - podman push registry.jihulab.com/q-hook/q-hook-project:1.0

```

```bash
FROM nginx:stable-alpine3.23

RUN echo '<h1>Hello from JiHu GitLab!</h1>' > /usr/share/nginx/html/index.html
```

### 拉取
```bash
stages:
  - nginx
nginx:
  stage: nginx
  tags:
    - ubuntu
  before_script:
    - podman login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD registry.jihulab.com
  script:
    - podman pull registry.jihulab.com/q-hook/q-hook-project:1.0
    - podman run -d -p 80:80 --name nginx registry.jihulab.com/q-hook/q-hook-project:1.0
```

### 自增版本号
```bash
cat package-lock.json | jq -r .version

# 使用
  before_script:
    - export IMAGE_VERSION=$(cat package.json | jq -r .version)
```

```bash
sudo apt install podman-compose
```

```yaml
version: "3"
services:
  nginx:
    image: registry.jihulab.com/q-hook/q-hook-project:0.1.0
    ports: 
      - "80:80"
    container_name: nginx
    restart: always

```

```bash
podman-compose up -d
```

# ssh
```yaml
stages:
  - deploy_to_dev

deploy_to_dev:
  stage: deploy_to_dev
  before_script:
    - eval $(ssh-agent -s)
    - echo "$SERVERS_KEY" | tr -d '\r' | ssh-add

  script:
    - ssh -o StrictHostKeyChecking=no $SERVERS_USER@$SERVERS_URL "ls -la"
```

### 推送 image
```yaml
stages:
  - deploy_to_dev

deploy_to_dev:
  stage: deploy_to_dev
  before_script:
    - eval $(ssh-agent -s)
    - ssh-add <(echo "$SERVERS_KEY")  

  script:
    - ssh -o StrictHostKeyChecking=no $SERVERS_USER@$SERVERS_URL "
      podman run -p 8000:80 -d --name nginx registry.jihulab.com/q-hook/q-hook-project:0.1.0
      "

```

```yaml
stages:
  - deploy_to_dev

deploy_to_dev:
  stage: deploy_to_dev
  before_script:
    - eval $(ssh-agent -s)
    - ssh-add <(echo "$SERVERS_KEY")  

  script:
    - scp -o StrictHostKeyChecking=no ./docker-compose.yml $SERVERS_USER@$SERVERS_URL:/root
    - ssh -o StrictHostKeyChecking=no $SERVERS_USER@$SERVERS_URL "
      podman compose down &&
      podman compose up -d 
      "

```

```yaml
version: "3"
services:
  nginx:
    image: registry.jihulab.com/q-hook/q-hook-project:0.1.0
    ports: 
      - "8000:80"
    container_name: nginx
    restart: always

```

### 部署 Staging 服务器
```yaml
stages:
  - deploy_to_dev
  - deploy_to_Staging

deploy_to_dev:
  stage: deploy_to_dev
  before_script:
    - eval $(ssh-agent -s)
    - ssh-add <(echo "$SERVERS_KEY")  

  script:
    - scp -o StrictHostKeyChecking=no ./docker-compose.yml $SERVERS_USER@$SERVERS_URL:/root
    - ssh -o StrictHostKeyChecking=no $SERVERS_USER@$SERVERS_URL "
      export COMPOSE_PROJECT_NAME=dev
      export APP_PORT=8000
      podman compose down &&
      podman compose up -d 
      "

deploy_to_Staging:
  stage: deploy_to_Staging
  before_script:
    - eval $(ssh-agent -s)
    - ssh-add <(echo "$SERVERS_KEY")  

  script:
    - scp -o StrictHostKeyChecking=no ./docker-compose.yml $SERVERS_USER@$SERVERS_URL:/root
    - ssh -o StrictHostKeyChecking=no $SERVERS_USER@$SERVERS_URL "
      export COMPOSE_PROJECT_NAME=Staging
      export APP_PORT=8080
      podman compose down &&
      podman compose up -d 
      "
```

```yaml
version: "3.9"
services:
  nginx:
    image: registry.jihulab.com/q-hook/q-hook-project:0.1.0
    ports: 
      - "${APP_PORT}:80"
    restart: always

```

#### 添加环境
```yaml
stages:
  - deploy_to_dev
  - deploy_to_Staging

deploy_to_dev:
  stage: deploy_to_dev
  before_script:
    - eval $(ssh-agent -s)
    - ssh-add <(echo "$SERVERS_KEY")  

  script:
    - scp -o StrictHostKeyChecking=no ./docker-compose.yml $SERVERS_USER@$SERVERS_URL:/root
    - ssh -o StrictHostKeyChecking=no $SERVERS_USER@$SERVERS_URL "
      export COMPOSE_PROJECT_NAME=dev
      export APP_PORT=8000
      podman compose down &&
      podman compose up -d 
      "
  environment:
    name: dev
    url: http://$SERVERS_URL:$APP_PORT

deploy_to_Staging:
  stage: deploy_to_Staging
  before_script:
    - eval $(ssh-agent -s)
    - ssh-add <(echo "$SERVERS_KEY")  

  script:
    - scp -o StrictHostKeyChecking=no ./docker-compose.yml $SERVERS_USER@$SERVERS_URL:/root
    - ssh -o StrictHostKeyChecking=no $SERVERS_USER@$SERVERS_URL "
      export COMPOSE_PROJECT_NAME=Staging
      export APP_PORT=8080
      podman compose down &&
      podman compose up -d 
      "
  environment:
    name: Staging
    url: http://$SERVERS_URL:$APP_PORT
```

### 去重
```yaml
stages:
  - deploy_to_dev
  - deploy_to_staging

# 1. 定义一个隐藏模板 (以 . 开头)
.deploy_base:
  before_script:
    - eval $(ssh-agent -s)
    - ssh-add <(echo "$SERVERS_KEY")
  script:
    - scp -o StrictHostKeyChecking=no ./docker-compose.yml $SERVERS_USER@$SERVERS_URL:/root/
    - ssh -o StrictHostKeyChecking=no $SERVERS_USER@$SERVERS_URL "
        export COMPOSE_PROJECT_NAME=$CI_ENVIRONMENT_NAME &&
        export APP_PORT=$DEPLOY_PORT &&
        podman compose down &&
        podman compose up -d
      "
  environment:
    name: $CI_ENVIRONMENT_SLUG
    url: http://$SERVERS_URL:$DEPLOY_PORT

# 2. Dev 环境直接继承模板，只改变量
deploy_to_dev:
  extends: .deploy_base
  stage: deploy_to_dev
  variables:
    DEPLOY_PORT: "8000"
  environment:
    name: dev
    url: http://$SERVERS_URL:8000

# 3. Staging 环境直接继承模板，只改变量
deploy_to_staging:
  extends: .deploy_base
  stage: deploy_to_staging
  variables:
    DEPLOY_PORT: "8080"
  environment:
    name: staging
    url: http://$SERVERS_URL:8080

```

# 手动部署 production 环境
+ 加上 when: manual 手动执行

```yaml
stages:
  - deploy_to_dev
  - deploy_to_staging

.deploy_base:
  before_script:
    - eval $(ssh-agent -s)
    - ssh-add <(echo "$SERVERS_KEY")
  script:
    - scp -o StrictHostKeyChecking=no ./docker-compose.yml $SERVERS_USER@$SERVERS_URL:/root/
    - ssh -o StrictHostKeyChecking=no $SERVERS_USER@$SERVERS_URL "
        export COMPOSE_PROJECT_NAME=$CI_ENVIRONMENT_NAME &&
        export APP_PORT=$DEPLOY_PORT &&
        podman compose down &&
        podman compose up -d
      "
  environment:
    name: $CI_ENVIRONMENT_SLUG
    url: http://$SERVERS_URL:$DEPLOY_PORT


deploy_to_dev:
  needs:
    - run-test-on-dev
  extends: .deploy_base
  stage: deploy_to_dev
  variables:
    DEPLOY_PORT: "8000"
  environment:
    name: dev
    url: http://$SERVERS_URL:8000

run-test-on-dev:
  stage: deploy_to_dev
  script:
    - echo "Running tests on dev environment..."


deploy_to_staging:
  needs:
    - run-test-on-staging
  extends: .deploy_base
  stage: deploy_to_staging
  variables:
    DEPLOY_PORT: "8080"
  environment:
    name: staging
    url: http://$SERVERS_URL:8080


run-test-on-staging:
  stage: deploy_to_staging
  script:
    - echo "Running tests on staging environment..."


deploy_to_staging:
  extends: .deploy_base
  stage: deploy_to_staging
  needs:
    - run-test-on-staging
  when: manual             #  staging 手动触发
  allow_failure: false     #  阻塞后续流水线（如有）
  variables:
    DEPLOY_PORT: "8088"
  environment:
    name: staging
    url: http://$SERVERS_URL:8088
```

```yaml
version: "3.9"
services:
  nginx:
    image: registry.jihulab.com/q-hook/q-hook-project:0.1.0
    ports: 
      - "${APP_PORT}:80"
    restart: always

```

# 产物-日志
```bash
output.txt
```

```bash
stages:
  - nginx
nginx:
  stage: nginx
  tags:
    - ubuntu
  before_script:
    - podman login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD registry.jihulab.com
  variables:
    IMAGE_TAG: "1.1"
  script:
    - podman build -t registry.jihulab.com/q-hook/q-hook-project:${IMAGE_TAG} -f ./Dockerfile . 2>&1 | tee output.txt
    - podman push registry.jihulab.com/q-hook/q-hook-project:${IMAGE_TAG}

  artifacts:
    paths:
      - output.txt
    when: always      # 失败时也要上传日志
    expire_in: 1 day
```

```bash
FROM docker.io/library/nginx:stable-alpine3.23

RUN echo '<h1>Hello from JiHu GitLab!</h1>' > /usr/share/nginx/html/index.html
```

