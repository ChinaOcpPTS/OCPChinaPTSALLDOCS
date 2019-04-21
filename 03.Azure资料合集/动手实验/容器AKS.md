# **网络部分** #

使用静态IP绑定负载均衡器部署容器服务

## 第一部分：准备 ##
预先准备一个可用的镜像，本教程使用的示例应用程序是一个基本的投票应用。 该应用程序由前端 Web 组件和后端 Redis 实例组成。 Web 组件打包到自定义容器映像中。 Redis 实例使用 Docker 中心提供的未修改的映像。

1.使用 git 可将示例应用程序克隆到开发环境：
>     git clone https://github.com/Azure-Samples/azure-voting-app-redis.git

2.换到克隆目录。使用 Docker Compose，可自动生成容器映像和部署多容器应用程序。使用示例 docker-compose.yaml 文件创建容器映像、下载 Redis 映像和启动应用程序：
>     docker-compose up -d

3.完成后，使用 docker images 命令查看创建的映像。 已下载或创建三个映像。 azure-vote-front 映像包含前端应用程序，并以 nginx-flask 映像为基准。 redis 映像用于启动 Redis 实例。
> docker images

4.查看一下docker ps的进程，并且查看正在运行的应用程序，请在本地 Web 浏览器中输入 http://localhost:8080。 示例应用程序会加载，如以下示例所示：
![](https://github.com/ChinaOcpPTS/OCPChinaPTSALLDOCS/blob/master/03.Azure%E8%B5%84%E6%96%99%E5%90%88%E9%9B%86/%E5%8A%A8%E6%89%8B%E5%AE%9E%E9%AA%8C/%E5%AE%B9%E5%99%A8AKS/Media/1.png)

5.使用 az acr create 命令创建 Azure 容器注册表实例，并提供你自己的注册表名称。 注册表名称在 Azure 中必须唯一，并且包含 5-50 个字母数字字符。 在本教程的剩余部分，请使用 <acrName> 作为容器注册表名称的占位符。 提供自己的唯一注册表名称。 “基本”SKU 是一个针对成本优化的入口点，适用于可以对存储和吞吐量进行均衡考虑的开发目的。
> az acr create --resource-group demoaks --name <acrName--sku Basic
