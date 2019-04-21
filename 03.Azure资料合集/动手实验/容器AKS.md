# **网络部分** #

使用静态IP绑定负载均衡器部署容器服务

## 第一部分：准备 ##
预先准备一个可用的镜像，本教程使用的示例应用程序是一个基本的投票应用。 该应用程序由前端 Web 组件和后端 Redis 实例组成。 Web 组件打包到自定义容器映像中。 Redis 实例使用 Docker 中心提供的未修改的映像。

1.	使用 git 可将示例应用程序克隆到开发环境：
    

    git clone https://github.com/Azure-Samples/azure-voting-app-redis.git


2.	切换到克隆目录。使用 Docker Compose，可自动生成容器映像和部署多容器应用程序。使用示例 docker-compose.yaml 文件创建容器映像、下载 Redis 映像和启动应用程序：

    docker-compose up -d



