# 在docker中配置agent环境

## docker常用命令

在windwos下配置nodejs环境，更推荐使用docker。因此在此处简单介绍一下docker的常用命令

### docker镜像操作

```bash
docker images                        # 列出本地镜像
docker pull <镜像名>[:标签]           # 拉取镜像
docker build -t <镜像名> .           # 构建镜像
docker rmi <镜像ID>                  # 删除镜像
docker tag <镜像ID> <新标签>         # 打标签
docker push <仓库名/镜像名>          # 推送镜像
docker save -o <文件.tar> <镜像>     # 导出镜像
docker load -i <文件.tar>            # 导入镜像
```
### docker容器操作
```bash
docker run <镜像>                    # 创建并启动容器
docker run -d <镜像>                 # 后台运行
docker run -it <镜像> /bin/bash      # 交互式运行
docker run -p 8080:80 <镜像>         # 端口映射
docker run -v /host:/container <镜像> # 挂载卷

docker ps                           # 查看运行中容器
docker ps -a                        # 查看所有容器

docker start <容器ID>                # 启动
docker stop <容器ID>                 # 停止
docker restart <容器ID>              # 重启
docker rm <容器ID>                   # 删除（先停止）
docker rm -f <容器ID>                # 强制删除

docker exec -it <容器ID> /bin/bash   # 进入容器
docker logs <容器ID>                 # 查看日志
docker logs -f <容器ID>              # 实时跟踪日志

docker inspect <容器ID>              # 查看详情
docker top <容器ID>                  # 查看进程
docker stats                        # 资源使用统计
```

### 操作案例

```bash
docker run -d --name myapp -p 8080:80 nginx   # 后台运行 nginx
docker ps                                      # 查看是否运行
docker exec -it myapp /bin/bash               # 进入容器操作
```

### docker-compose.yml

可以事先在项目文件下写好该文件，然后docker启动的时候就可以按照该文件的内容启动对应的文件以及映射。具体案例如下：
```yml
version: '3.8'  # 版本号

services:       # 服务定义
  web:
    image: nginx:latest
    container_name: my-nginx
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html
    networks:
      - my-network

networks:       # 网络定义
  my-network:
    driver: bridge

volumes:        # 卷定义
  my-data:
    driver: local
```

