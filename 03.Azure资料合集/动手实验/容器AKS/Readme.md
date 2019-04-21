# **网络部分** #

使用静态IP绑定负载均衡器部署容器服务

## 第一部分：准备 ##
预先准备一个可用的镜像，本教程使用的示例应用程序是一个基本的投票应用。 该应用程序由前端 Web 组件和后端 Redis 实例组成。 Web 组件打包到自定义容器映像中。 Redis 实例使用 Docker 中心提供的未修改的映像。

1.使用 git 可将示例应用程序克隆到开发环境：

    git clone https://github.com/Azure-Samples/azure-voting-app-redis.git

2.换到克隆目录。使用 Docker Compose，可自动生成容器映像和部署多容器应用程序。使用示例 docker-compose.yaml 文件创建容器映像、下载 Redis 映像和启动应用程序：

    docker-compose up -d

3.完成后，使用 docker images 命令查看创建的映像。 已下载或创建三个映像。 azure-vote-front 映像包含前端应用程序，并以 nginx-flask 映像为基准。 redis 映像用于启动 Redis 实例。

    docker images

4.查看一下docker ps的进程，并且查看正在运行的应用程序，请在本地 Web 浏览器中输入 http://localhost:8080。 示例应用程序会加载，如以下示例所示：
![](https://github.com/ChinaOcpPTS/OCPChinaPTSALLDOCS/blob/master/03.Azure%E8%B5%84%E6%96%99%E5%90%88%E9%9B%86/%E5%8A%A8%E6%89%8B%E5%AE%9E%E9%AA%8C/%E5%AE%B9%E5%99%A8AKS/Media/1.png)

5.使用 az acr create 命令创建 Azure 容器注册表实例，并提供你自己的注册表名称。 注册表名称在 Azure 中必须唯一，并且包含 5-50 个字母数字字符。 在本教程的剩余部分，请使用 <acrName> 作为容器注册表名称的占位符。 提供自己的唯一注册表名称。 “基本”SKU 是一个针对成本优化的入口点，适用于可以对存储和吞吐量进行均衡考虑的开发目的。

    az acr create --resource-group demoaks --name <acrName> --sku Basic

6.若要使用 ACR 实例，必须先登录。 使用 az acr login 命令并提供一个唯一名称，该名称是在上一步提供给容器注册表的

    az acr login --name <acrName>

## 第二部分：主要操作部分 ##
1.首先获取节点的资源组名称，因为节点所在的资源组名称与AKS创建的资源组名称并不是一致的。

    az aks show --resource-group demoaks --name demoaks --query nodeResourceGroup -o tsv

2.现在，使用 az network public ip create 命令创建静态公用 IP 地址。 指定上一命令中获取的节点资源组名称，然后指定 IP 地址资源的名称，如 demoaksPublicIP。同时记下输出的IP地址，已备后用。

    az network public-ip create --resource-group MC_demoaks_demoaks_chinaeast2 --name demoaksPublicIP --allocation-method static

3.当然稍后可以使用 az network public-ip list 命令获取公用 IP 地址。 指定节点资源组的名称和创建的公共 IP 地址，然后查询 ipAddress，如以下示例中所示：

    az network public-ip show --resource-group MC_demoaks_demoaks_chinaeast2 --name demoaksPublicIP --query ipAddress --output  tsv

4.查询一下资源组中可用的ACR资源，记住ACR的地址和名字

    az acr list --resource-group demoaks --query "[].{acrLoginServer:loginServer}" --output table

5.登录ACR

    az acr login --name <acrName>


6.现在，请使用容器注册表的 acrloginServer 地址标记本地 azure-vote-front 映像。 若要指示映像版本，请将 :v1 添加到映像名称的末尾：

    docker tag azure-vote-front <acrLoginServer>/azure-vote-front:v1

7.若要验证是否已应用标记，请再次运行 docker images。 系统会使用 ACR 实例地址和版本号对映像进行标记。

    docker images


8.将映像推送到注册表.生成并标记映像后，将 azure-vote-front 映像推送到 ACR 实例。 使用 docker push 并提供自己的适用于映像名称的 acrLoginServer 地址，如下所示：

    docker push <acrLoginServer>/azure-vote-front:v1

9.列出注册表中的映像。若要返回已推送到 ACR 实例的映像列表，请使用 az acr repository list 命令。 按如下所示提供自己的 <acrName>：

    az acr repository list --name <acrName--output table


10.在目录中编辑yaml文件

    vi azure-vote-all-in-one-redis.yaml

11.将 microsoft 替换为 ACR 登录服务器名称。 映像名称位于清单文件的第 47 行。把IP地址更改为之前建好的静态地址，在文件64行。 以下示例展示了默认映像名称：

    containers:
      name: azure-vote-front
      image: microsoft/azure-vote-front:v1
    
      loadBalancerIP: 40.73.86.77
      type: LoadBalancer

12.提供自己的 ACR 登录服务器名称，使清单文件如以下示例所示：

    containers:
      -name: azure-vote-front
      image: <acrName>.azurecr.io/azure-vote-front:v1

13.部署应用程序容器，请使用 kubectl apply 命令。 此命令分析清单文件并创建定义的 Kubernetes 对象。 指定示例清单文件，如以下示例所示：

    kubectl apply -f azure-vote-all-in-one-redis.yaml

14.应用程序运行时，Kubernetes 服务将向 Internet 公开应用程序前端。 此过程可能需要几分钟才能完成。若要监视进度，请将 kubectl get service 命令与 --watch 参数配合使用。查看是否服务得到了之前创建的外部地址。如果得到了外部地址，通过浏览器打开相关IP，查看应用是否可用。

    kubectl get service azure-vote-front –watch
![](https://github.com/ChinaOcpPTS/OCPChinaPTSALLDOCS/blob/master/03.Azure%E8%B5%84%E6%96%99%E5%90%88%E9%9B%86/%E5%8A%A8%E6%89%8B%E5%AE%9E%E9%AA%8C/%E5%AE%B9%E5%99%A8AKS/Media/2.png)

15.删除服务，Pod等

    kubectl delete deployments azure-vote-front
    kubectl delete deployments azure-vote-back
    kubectl delete services azure-vote-front
    kubectl delete services azure-vote-back
 

----------


# **存储部分** #
1.使用 kubectl get sc 命令查看预先创建的存储类。 以下示例显示了 AKS 群集中可用的预先创建存储类：
    kubectl get sc

2.创建永久性卷声明.永久卷声明 (PVC) 用于基于存储类自动预配存储。 在这种情况下，PVC 可以使用预先创建的存储类之一创建标准或高级 Azure 托管磁盘。创建名为 azure-premium.yaml 的文件，并将其复制到以下清单中。 该声明请求名为 azure-managed-disk、大小为 5 GB、具有 ReadWriteOnce 访问权限的磁盘。 managed-premium 存储类指定为存储类。使用 kubectl apply 命令创建永久性卷声明，并指定 azure-premium.yaml 文件：

    kubectl apply -f azure-premium.yaml
[<yaml文件地址>](https://github.com/ChinaOcpPTS/OCPChinaPTSALLDOCS/blob/master/03.Azure%E8%B5%84%E6%96%99%E5%90%88%E9%9B%86/%E5%8A%A8%E6%89%8B%E5%AE%9E%E9%AA%8C/%E5%AE%B9%E5%99%A8AKS/azure-premium.yaml)

3.创建永久性卷声明并成功预配磁盘以后，即可创建可以访问磁盘的 Pod。 以下清单创建的基本 NGINX Pod 使用名为 azure-managed-disk 的永久性卷声明将 Azure 磁盘装载到 /mnt/azure 路径。创建名为 azure-pvc-disk.yaml 的文件，并将其复制到以下清单中。

    kubectl apply -f azure-pvc-disk.yaml
[<yaml文件地址>](https://github.com/ChinaOcpPTS/OCPChinaPTSALLDOCS/blob/master/03.Azure%E8%B5%84%E6%96%99%E5%90%88%E9%9B%86/%E5%8A%A8%E6%89%8B%E5%AE%9E%E9%AA%8C/%E5%AE%B9%E5%99%A8AKS/azure-pvc-disk.yaml)

4.现在你有一个正在运行的 Pod，其中 Azure 磁盘被装载到 /mnt/azure 目录中。 通过 kubectl describe pod mypod 检查 Pod 时可以看到此配置，如以下精简示例所示：

    kubectl describe pod mypod

5.删除Pod，和创建的永久申明卷。

    kubectl delete pods mypod
    kubectl delete pvc azure-managed-disk
