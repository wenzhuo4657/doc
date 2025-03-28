
# 命令解释

[docker compose | Docker 文档](https://docs.docker.net.cn/reference/cli/docker/compose/#subcommands)


Usage:  `docker compose [OPTIONS] COMMAND`
例：`docker-componse -f 1.yml  build `  

1，默认执行文件为当前目录下的`compose.yaml`,可使用-f参数指定多个文件或者单个文件，其中相同属性会按照顺序被覆盖。
2，docker-componse中的command子命令当中又有各自的配置，并非是单独的命令。[docker compose 子命令](https://docs.docker.net.cn/reference/cli/docker/compose/#subcommands)

## [OPTIONS]
### [--dry-run](https://docs.docker.net.cn/reference/cli/docker/compose/#use-dry-run-mode-to-test-your-command)

`docker compose --dry-run up --build -d`

该选项的含义是在执行子命令（如 --build -d）之前，进行**试运行**。具体而言，展示子命令执行时的步骤，便于程序员排查问题。



# yml配置

[概述 | Docker 文档](https://docs.docker.net.cn/compose/compose-file/)
[构建规范 | Docker 文档](https://docs.docker.net.cn/compose/compose-file/build/#illustrative-example)
## 顶级元素
```
version:3.8 #已弃用，可以添加但不起作用，执行中告警
name: project-name # 指定项目名称
services: # 服务顶级元素，其键是服务名称的字符串表示形式，其值是服务定义
	web: # 服务名称，支持使用build构建镜像，不配置则从仓库中拉取
		image: big-market:latest
		build:
		  .......
networks: # 定义服务之间的通信
volumes:  # 中立卷，挂载于多个服务当中

		
```


### 服务配置

### 保证服务启动顺序


#### [depends_on](https://docs.docker.net.cn/compose/compose-file/05-services/#depends_on)

一定程度上保证了服务的启动、关闭顺序。
```
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

- Compose 按照依赖项顺序创建服务。在以上示例中，`db` 和 `redis` 在 `web` 之前创建。
    
- Compose 按照依赖项顺序删除服务。在以上示例中，`web` 在 `db` 和 `redis` 之前删除。



在实际运用中，对于应用环境类型的服务和应用类型的服务，尤为重要。

#### healthcheck

部分场景下，应用环境服务容器启动时，但是无法应对应用服务的调用，例如需要初始化sql文件等。而healthcheck提供一种机制进行健康检查，保证的依赖服务为我们期望的一个服务。


![[Pasted image 20241114105910.png]]


```
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost"]  # 健康检查的命令
  interval: 1m30s  # 相邻检查的间隔
  timeout: 10s   # 每次检查的规定时长，超时判定为失败
  retries: 3  # 连续失败x次，将状态变为unhealthy
  start_period: 40s  
  start_interval: 5s
```


#### hostname
配置主机名

# 关于构建镜像

## alpine节省空间

[对Docker基础镜像的思考，该不该选择alpine-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/2168079)
