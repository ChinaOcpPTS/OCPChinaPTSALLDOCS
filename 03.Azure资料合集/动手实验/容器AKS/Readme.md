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

---

# ** DevOps部分 ** #

本次实验，将实现应用容器化的CI/CD流程，通过预先写好的Dockerfile，结合Azure DevOps，自动化Build容器化镜像，将容器化镜像发布到私有ACR中，并将容器镜像部署到AKS集群中。

本次实验的程序为一个在线投票系统，具体架构图如下：

![](./Media/devops/x01.png)

系统共有三部分组成：
- 前端Web Application ：使用Python开发；
- 应用服务 Calculator microservice ：使用Java开发；
- 缓存服务 Cache microservice ：使用 Redis 实现；

本次实验所使用的 [Github Repo](https://github.com/ericzhao0821/python-voting-web-app) 
 
本次实验Azure资源创建部分，尽量使用 `Azure CLI` ； Azure DevOps部分，将会使用 `DevOps Portal`进行创建。

### 创建实验所需资源

#### 创建Azure DevOps的Organization & Project

Azure DevOps的Dashboard请参照 <https://dev.azure.com> ，请使用相应的`Global Azure`账户进行登陆。

1. 创建Organization `zjdevopsdemo01`

![](./Media/devops/x02.png)

2. 创建 Project `devopsdemo01`

![](./Media/devops/x03.png)

3. 将实验中所使用的Github Repo克隆至Azure Repos

Azure DevOps中也会提供代码管理功能，即服务 `Azure Repos`。本次实验，所有代码的修改，都将通过Azure Repos进行，Azure Repos也支持Git的操作方式，可以通过本地客户端，连接Azure Repos，并使用熟悉的Git命令进行操作。

点击 `Repos`，选择 `Import`，来导入实验用到的Github项目

![](./Media/devops/x04.png)

可以看到，导入的项目内容如下：

![](./Media/devops/x05.png)

#### 配置Azure DevOps，连接Azure China

由于目前Azure DevOps服务是由Global Azure提供，但提供对于Azure China的支持，即可以使用Global Azure的Azure DevOps来构建基于Azure China资源的自动化运维解决方案。 如果需要Azure DevOps与Azure China资源进行连接，需要进行手动的配置。

1. 创建 Service Principal

`az ad sp create-for-rbac -n "zjdevopssp01" --role contributor`

![](./Media/devops/x06.png)

2. 建立针对于Azure China的Endpoint，并创建连接

进入Project `devopsdemo01`，点击`Project Settings`，选择`Pipelines - Service connections`，点击`New service connection`，选择`Azure Resource Manager`

![](./Media/devops/x07.png)

点击`use the full version of the service connection dialog`，创建`Azure China Cloud`，并填写相对应的Service Principal，点击`Verify connection`，确保连接成功。

![](./Media/devops/x08.png)

#### 创建实验所需的Azure资源

1. 创建 Resource Group `zjdemo01`

`az group create -n zjdemo01 -l chinaeast2`

![](./Media/devops/x09.png)

2. 创建Azure Kubernetes Service集群 `zjaksdemo01`

`az aks create -n zjaksdemo01 -g zjdemo01 --node-vm-size Standard_DS2_v2 --node-count 2 --kubernetes-version 1.12.6 --disable-rbac`

![](./Media/devops/x10.png)

3. 创建 Azure Container Registry `zjacrdemo01`

`az acr create -n zjacrdemo01 -g zjdemo01 --sku Standard --admin-enabled -l chinaeast2`
 
![](./Media/devops/x11.png)

### 通过 Azure DevOps，构建CI/CD Pipelines

__**注意**__ 新版本Azure DevOps默认会开启YAML编辑页面，本次实验仍然以图形化界面为主，关闭YAML页面可通过如下操作：


点击右上角的用户，选择`Preview features`，disable选项`New YAML pipeline creation experience`即可
 
![](./Media/devops/x12.png)

#### 构建 Builds Pipelines，通过Dockerfile，Build容器镜像，并上传到私有镜像仓库

1. 选择`Pipelines -> Builds`，点击`New pipeline` 

Source选择：选择前面导入的Azure Repos `devopsdemo01`
 
![](./Media/devops/x13.png)

Azure DevOps Pipelines提供了多种内置的模板供用户使用，本次实验，选取`Empty job`进行构建

![](./Media/devops/x14.png)

将创建好的Pipeline `devopsdemo01-CI`，Agent pool更改为`Hosted Ubuntu 1604`

2. 添加 Task `Maven calculator-api/pom.xml`

点击 `Agent job 1` 右侧的`+`，选择 Maven Task添加，Maven Task的作用主要将应用层计算服务的源码构建成可部署的Jar程序。Maven POM file，从Azure Repos `devopsdemo01/calculator-api/pom.xml`中选择，MavenTask的配置信息如下：

![](./Media/devops/x15.png)

3. 添加 Task `Build out demo images with Docker Compose`

点击 `Agent job 1` 右侧的`+`，选择Docker Compose Task添加。Docker Compose Task的作用是通过写好的Docker Compose文件，构建出实验中需要的容器化镜像，Task的配置信息如下：
 
![](./Media/devops/x16.png)
![](./Media/devops/x17.png)

4. 添加 Task `Push demo images to ACR`

点击 `Agent job 1` 右侧的`+`，选择Docker Compose Task添加。Docker Compose Task的作用是将上一步构建好的两个容器镜像 `zjacrdemo01.azurecr.cn/azure-vote-front:latest`，`zjacrdemo01.azurecr.cn/azure-calculator-api:latest` 添加到ACR，Task的配置信息如下：

![](./Media/devops/x18.png)
![](./Media/devops/x19.png)
 
添加完所有Task后，点击`Save & Queue`，手动触发Build的Pipeline，验证所有Tasks都达到预期

![](./Media/devops/x20.png)
![](./Media/devops/x21.png)
 
可以看到，所有Tasks都已运行成功，并成功的将build好的images上传到了ACR中。

5. Enable continuous integration

通过Enable持续集成，后续当有代码更新，会自动触发此Build Pipeline，将更新后的代码打包成新的镜像，并上传到ACR

![](./Media/devops/x22.png)

#### 构建 Release Pipeline，将构建好的容器镜像，部署到AKS集群中

1. 选择`Releases`，点击`New pipeline`，创建新的Release Pipeline，选择`Empty job` Template进行构建，创建好后，将Release Pipeline名字更改为`devopsdemo01-CD`

2. 添加Artifacts，并Enable continuous deployment配置

Artifacts的配置如下，主要是为后面Release Pipeline中的Tasks提供执行文件的支持

![](./Media/devops/x23.png)

为Artifacts Enable Continuous Delivery选项

![](./Media/devops/x24.png)

3. 添加 Task `Deploy out demo services to AKS`

点击`Agent Job`右侧的`+`，选择 Task `Deploy to Kubernetes`，将为此实验写好的YAML，部署到AKS中，YAML中包括了实验中希望创建的服务，Task配置如下：
 
![](./Media/devops/x25.png)
![](./Media/devops/x26.png)
![](./Media/devops/x27.png)
 
配置好后，可以手动触发Release，进行验证。点击右上角`+Release`，选择`Create Release`，进行触发

![](./Media/devops/x28.png)

可以看到，Release Pipeline已经可以正常运行。

4. 验证AKS集群中，实验中涉及到的服务的创建状况

获取AKS的Credentials信息

` az aks get-credentials -n zjaksdemo01 -g zjdemo01 `

查看集群中创建的Pod资源

` kubectl get pod `

![](./Media/devops/x29.png)

这里发现，实际运行服务的Pod并没有创建成功，通过查询可以看到，失败原因主要是因为容器镜像无法正确下载，容器镜像名称并非我们上传到ACR的名字。

` kubectl describe pod azure-vote-front-b56b4686b-7lwt8 `

![](./Media/devops/x30.png)

#### 修复环境中目前的问题，模拟更新文件触发CI/CD

1. 查看部署的文件`azure-vote-all-in-one-redis.yaml`，发现ACR的地址并未进行更改，使用的是个错误的ACR地址
 
![](./Media/devops/x31.png)
![](./Media/devops/x32.png)

2. 将ACR的地址改为实际实验中创建的ACR地址，同时拉取ACR镜像需要Credentials信息，但部署文件中的`imagePullSecrets`信息并不准确，需要同时更新`imagePullSecrets`信息

![](./Media/devops/x33.png)
![](./Media/devops/x34.png)

3. 更改文件后，会自动触发CI/CD的Pipeline，将更新后的信息部署到现有环境中，待Pipelines执行完成后，重新检验AKS环境中资源的创建情况：

` kubectl get pod `

![](./Media/devops/x35.png)

这里发现，Pod `azure-vote-back-746d4bc54b-2h8md` 仍然处于failed状态，查看情况发现，是因为没有成功拉取到 Image `redis`。

![](./Media/devops/x36.png)
 
这个问题主要源于，国内对于 dockerhub 镜像的拉取会有些网络连接的问题，我们可以通过一些proxy进行帮助。

4. 更改`azure-vote-all-in-one-redis.yaml`中，redis的Image地址，然后重新触发CI/CD Pipelines

![](./Media/devops/x37.png)

5. 待Pipeline运行完毕，重新检验AKS环境中资源的创建情况：

` kubectl get pod `

![](./Media/devops/x38.png)

目测环境中一切正常，查询投票系统对外提供的服务信息并进行验证

`kubectl get svc`

![](./Media/devops/x39.png)

访问 http:// 40.73.102.194，可以访问如下页面

![](./Media/devops/x40.png)

本次实验到此成功结束 ！

---
