# Drone 基于Docker的cicd工具

- [关于CICD](#%e5%85%b3%e4%ba%8ecicd)
- [Drone](#drone)
  - [介绍](#%e4%bb%8b%e7%bb%8d)
  - [实践](#%e5%ae%9e%e8%b7%b5)
- [Reference](#reference)
    
## 关于CICD

CI(持续集成)CD(持续交付/部署)，为的是实现自动化部署和测试，减少人工的参与  
具体之后会有另一篇文章讲CICD相关
- CI：远程仓库push代码，触发一系列测试
- CD：将远程的默认分支，部署在生产环境上（开发环境亦可）
- pipeline：构建CICD执行流程的管道  
- 流程
  ![cicd_flow](/images/cicd_flow.png)
- 关于文档的一点吐槽
  - 我也不知道文档是针对0.8版本的，还是1.0版本的，所有blog都是在吐槽，也分不清个所以然
  - 大部分blog都是通过docker-compose来实现，而文档却是建议直接docker进行，所以本文先测试文档，然后再测试compose

## Drone
### 介绍
- 基于docker的CICD技术，每步构建都在docker中进行
- drone server 控制端
  - drone的服务配置，用于对接主流代码仓库（Github，Gitlab等），并且有相应的api
  - 类似gitlab cicd中的webhook作用
  - drone服务并不需要管理账号，而是通过代码托管仓库提供具有权限的账户
- drone agent 客户端
  - 也就是文档中所谓的drone runner
  - 具体执行CI的pipeline
- 开发者需要在项目中包含.drone.yml文件,将代码推送到git仓库,drone就能够自动化的进行编译/测试/发布

### 实践
以Github为例
- 创建OAuth账户
  - developer settings -> OAuth apps -> register new
  - 生成shared secret
  ```
  openssl rand -hex 16
  ```
- docker pull drone/drone:1
- 启动 docker server
  ```
  docker run \
    --volume=/var/lib/drone:/data \
    --env=DRONE_AGENTS_ENABLED=true \
    --env=DRONE_GITHUB_CLIENT_ID={{DRONE_GITHUB_CLIENT_ID}} \
    --env=DRONE_GITHUB_CLIENT_SECRET={{DRONE_GITHUB_CLIENT_SECRET}} \
    --env=DRONE_RPC_SECRET={{DRONE_RPC_SECRET}} \
    --env=DRONE_SERVER_HOST={{DRONE_SERVER_HOST}} \
    --env=DRONE_SERVER_PROTO={{DRONE_SERVER_PROTO}} \
    --publish=80:80 \
    --publish=443:443 \
    --restart=always \
    --detach=true \
    --name=drone \
    drone/drone:1
  ```
  - DRONE_GITHUB_CLIENT_ID和DRONE_GITHUB_CLIENT_SECRET在Github上查看
  - DRONE_RPC_SECRET： 刚刚生成的密钥，用于在rpc和本季之间的认证
  - DRONE_SERVER_PROTO：使用的协议：http/https
  - DRONE_SERVER_HOST：服务器所在ip

- 启动docker-agent
  ```
  docker pull drone/drone-runner-docker:1

  docker run -d \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -e DRONE_RPC_PROTO=https \
    -e DRONE_RPC_HOST=drone.company.com \ 
    -e DRONE_RPC_SECRET=super-duper-secret \
    -e DRONE_RUNNER_CAPACITY=2 \
    -e DRONE_RUNNER_NAME=${HOSTNAME} \
    -p 3000:3000 \
    --restart always \
    --name runner \
    drone/drone-runner-docker:1
  ```
  - DRONE_RPC_PROTO：
  - DRONE_RPC_HOST
  - DRONE_RPC_SECRET
  - DRONE_RUNNER_CAPACITY
  - DRONE_RUNNER_NAME

- 使用docker-compose创建
  ```
  version: '2'

  services:
    drone-server:
      image: drone/drone:1

      ports:
        - 8000:8000
      volumes:
        - /var/lib/drone:/var/lib/drone/
      restart: always
      environment:
        - DRONE_AGENTS_ENABLED=true                                 # 使用Runner
        - DRONE_GITHUB_SERVER=https://github.com                    # github的地址
        - DRONE_GITHUB_CLIENT_ID=${DRONE_GITHUB_CLIENT_ID}          # 上一步获得的ClientID
        - DRONE_GITHUB_CLIENT_SECRET=${DRONE_GITHUB_CLIENT_SECRET}  # 上一步获得的ClientSecret
        - DRONE_RPC_SECRET=${DRONE_RPC_SECRET}                      # RPC秘钥
        - DRONE_SERVER_HOST=${DRONE_SERVER_HOST}                    # RPC域名(在一个实例上可以不用)
        - DRONE_SERVER_PROTO=${DRONE_SERVER_PROTO}                  # git webhook使用的协议(我建议http)
        - DRONE_OPEN=true                                           # 开发drone
        - DRONE_DATABASE_DATASOURCE=/var/lib/drone/drone.sqlite     # 数据库文件
        - DRONE_DATABASE_DRIVER=sqlite3                             # 数据库驱动，我这里选的sqlite
        - DRONE_DEBUG=true                                          # 调试相关，部署的时候建议先打开
        - DRONE_LOGS_DEBUG=true                                     # 调试相关，部署的时候建议先打开
        - DRONE_LOGS_TRACE=true                                     # 调试相关，部署的时候建议先打开
        - DRONE_USER_CREATE=username:Terry，admin:true              # 初始管理员用户
        - TZ=Asia/Shanghai                                          # 时区

    drone-agent:
      image: drone/agent:1

      command: agent
      restart: always
      depends_on:
        - drone-server
      volumes:
        - /var/run/docker.sock:/var/run/docker.sock
      environment:
        - DRONE_RPC_SERVER=http://drone-server  # RPC服务地址
        - DRONE_RPC_SECRET=${DRONE_RPC_SECRET}  # RPC秘钥
        - DRONE_RPC_PROTO=${DRONE_RPC_PROTO}    # RPC协议(http || https)
        - DRONE_RUNNER_CAPACITY=2               # 最大并发执行的 pipeline 数
        - DRONE_DEBUG=true                      # 调试相关，部署的时候建议先打开
        - DRONE_LOGS_DEBUG=true                 # 调试相关，部署的时候建议先打开
        - DRONE_LOGS_TRACE=true                 # 调试相关，部署的时候建议先打开
        - TZ=Asia/Shanghai
    ```





## Reference
- [Drone Document](https://docs.drone.io/)
- [从零构建企业CICD工作流，以springboot+vue项目为例](https://iambigboss.top/post/54178_1_1.html)
- [Drone CI 介绍](https://lework.github.io/2019/08/26/drone-Introduction/)
- [DroneCI+Github入坑指北](https://juejin.im/post/5d97489ee51d457824771d47)

