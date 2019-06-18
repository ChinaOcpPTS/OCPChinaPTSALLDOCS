
## WIP - Azure 监控平台 Whitepaper & Handson

---
### HOL01 了解Azure中监控数据平台的层次，及各层次能够收集到的数据

Azure中监控的数据主要为 `Metrics` & `Logs`， 监控包括 `Tenant(租户)` & `Subscription（）` & `IaaS()` & `Container()` & `PaaS（）` & `客户自定义数据`等不同层级的资源，
  提供一致性的`监控` & `分析` & `可视化` & `警报`等功能。

![image](./images/monitor/mon01.png)

### 数据的种类

- `Metrics` 是时序数据，包含一个时间戳和数值，形如`"Percentage CPU": {"average": 0.57,"timeStamp":"2019-06-13T13:00:00+00:00"}`，定期收集，能够快速反应环境中出现的问题，如“CPU过高”，可以根据合理的`Metrics`设置报警，以便在环境出现问题时快速响应； `Metrics`的访问可以
  集中通过 `Metrics Explorer`进行查询， 可以通过`Azure Dashboard`创建一个定制化的大屏，能够体现资源整体的运行情况；
  
  `Metrics` 主要包含三个维度
  
  - `平台指标 （无需任何配置）` ：  反映资源的运行状况和性能；
  - `Guest OS Metrics (特指VM，需要配置诊断设置)` ：  通过Extension的Agent收集；
  - `容器 Metrics （特指AKS，需要额外开启Addons）` : 通过Agent进行收集；
  - `额外需要配置诊断设置的 Azure服务`
  - `应用程序数据 （需要集成Application Insights）`
  - `自定义指标 (需遵循平台配置指南)` ：  收集非平台的指标，比如用户在虚机中安装了MySQL，收集MySQL的一些指标；或用户想收集本地虚机的指标等；
  
  `Metrics` 大多数资源的指标数据保留93天，用户可以导出到存储账户进行存档保存；
  
  更多详细信息 ： 
  - [Azure中的Metrics介绍](https://docs.microsoft.com/zh-cn/azure/azure-monitor/platform/data-platform-metrics)  
  - [Azure平台中不同服务支持的 Metrics](https://docs.microsoft.com/zh-cn/azure/azure-monitor/platform/metrics-supported)

- `Logs` 是详细的信息记录，包含更多的信息，主要用于对出现的问题进行具体的分析；日志不同于指标，虽然按照时间顺序进行汇总，但不是按固定事件间隔进行收集发送； 建议将所近期日志
      （一个月或三个月）放入 `Log Analytics` 中保存分析，这样可以多维度的分析问题或获取见解，将长期数据（三个月以上）放入存储账户进行存储，以符合不同要求的合规性，审查或用于长期
    分析。 
  
  Azure Monitor中提供了 `Log Analytics` 的服务，用于对日志进行查询分析； `Log Analytics` 使用Kusto语言进行查询； `Log Analytics`的日志数据均存储在`Log Analytics workspace`
      中，且 `Application Insights` & `Azure Security Center` & `Azure Sentinel` 收集的日志存储在其内置的workspace中，但可以跨workspace进行分析；
    
  Azure中的日志主要包括 管理层的 `Azure AD Audit Logs` & `Azure Activity Log`; 资源层面的 `支持Diagnostics Settings的资源的诊断日志` & `虚拟机 Extension收集的性能及自定义日志`
    & `容器的日志数据` & `Application Insights中收集的应用程序请求&异常等详细数据， 使用情况等`； 支持通过多种手段将第三方日志导入到 `Log Analytics`;
  
    一般情况下，资源产生的日志数据与Azure Monitor之间会有 2-5 Mins的延迟，主要是因为
  
    更多详细信息：
    - [Azure中日志介绍](https://docs.microsoft.com/zh-cn/azure/azure-monitor/platform/data-platform-logs)
    - [Azure中查询语言入门](https://docs.microsoft.com/zh-cn/azure/azure-monitor/log-query/get-started-queries)
    - [影响日志数据传入Azure Monitor的因素](https://docs.microsoft.com/zh-cn/azure/azure-monitor/platform/data-ingestion-time)

### 能够收集的不同层面的资源

一般情况下，一家公司是处于一个Tenant中，通过创建不同的订阅，并在订阅下创建不同的资源。

![image](./images/monitor/mon02.png)

- Azure Tenant相关的数据 ：主要为 __*Azure Active Directory 审核日志*__ ，Azure AD的全局管理员才能够进行设置，具体设置可参照 [将 Azure AD 日志集成与 Azure Monitor 日志](https://docs.microsoft.com/zh-cn/azure/active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics), 将日志导入到特定的Log Analytics workspace中，并依照存储建议对数据进行存档；

- Azure Subscription相关数据：主要为 __*Activity Log*__ & __*Service Health*__

  - Activity Log ：收集所有针对资源的操作记录及资源的运行状况，例如：创建一个虚拟机，产生相关的活动日志记录虚拟机的创建过程及创建者；当平台对服务进行维护或某一资源的状态发生改变，也会创建一条活动日志记录发生的变化，如下所示：

    ![image](./images/monitor/mon03.png)

    ![image](./images/monitor/mon04.png)

    用户可以针对不同类别的Activity Log进行告警设置，及早知道环境中发生的变化；可参照 [收集和分析 Azure Monitor 的 Log Analytics 工作区中的 Azure 活动日志](https://docs.microsoft.com/zh-cn/azure/azure-monitor/platform/activity-log-collect)将活动日志配置到特定的Log Analytics workspace中，并依照存储建议对数据进行存档; 默认 Activity Log的保存期为90天。
  
  - Azure Service Health : 服务运行状况的数据实际上是存放在活动日志中，用户可以登陆到特定页面 `Monitor - Service Health` 中了解到包括近一段环境中出现的服务相关的问题及RCA报告，平台计划的Maintenance等，并可设置响应的告警，以便第一时间知道平台的哪个服务出了问题，详细介绍请参照 [使用 Azure 门户查看服务运行状况通知](https://docs.microsoft.com/zh-cn/azure/azure-monitor/platform/service-notifications)

    ![image](./images/monitor/mon05.png)

- Azure Resources ：主要为 __*Metrics*__ & __*Logs*__ ,另外包含针对于虚机的 `Guest OS` & `Azure Monitor for Container`

  - Metrics : 如上面介绍，指标是能够反应Azure服务可用性及性能的参考；大家比较好理解的是对虚机进行指标收集，Metrics除了支持IaaS资源外，还支持平台中的PaaS服务，且使用第一方的方式收集Metrics，更为简单，快速，稳定；部分IaaS服务&PaaS服务需要开启诊断日志，以支持指标的收集，用户在创建资源的时候记得打开诊断日志，以便更好的了解创建的服务；

  - Logs ：主要针对诊断日志；Azure资源的诊断日志默认是不开启的，需要在创建过程中或使用过程中开启，且指定到特定的Log Analytics workspace中，并依照存储建议对数据进行存档；并不是所有的服务都支持诊断日志，具体支持列表请参考 [Azure 诊断日志支持的服务、架构和类别](https://docs.microsoft.com/zh-cn/azure/azure-monitor/platform/diagnostic-logs-schema)

  - Guest OS : 通过不同的 Extension 来收集 Guest OS 的指标数据，主要针对于 Azure VM 及 On-Prem VM

    - Azure Diagnostics Extension : 主要收集 Azure VM 中的指标数据；

    - Log Analytics Agent : 主要通过Agent收集 Windows/Linux 自定义数据，VM可以为Azuge VM，也可以是本地 On-Prem VM

    - Azure Monitor for VM ：主要提供对于Azure虚机的运行状况指标，提供针对于Azure/非Azure虚机的性能及Service Map指标；运行状况条件指标存储在 Azure Monitor 中时间序列数据库、 收集性能和依赖关系数据存储在 Log Analytics workspace 中。
  
  - Azure Monitor for Container : 提供针对AKS数据的收集，主要收集AKS集群的指标并发送到Azure Monitor，可在 Metrics Explorer中进行查询；提供针对AKS集群的日志数据，包括实时的Pod日志；

- Application 数据 ：Azure Monitor中的Application Insights是一款智能APM工具，能够提供对支持的框架开发的应用程序进行数据的收集，且不论应用程序部署在Azure还是本地；Application Insights安装检测包后，会收集与应用程序的性能和运行相关的指标和日志，并发送到Azure，保存在Application Insights Instance专属的Log Analytics workspace中；

#### 环境准备

Step 1

本次实验，是从既有的其他实验中，选取了两个实验部署脚本，目的是尽可能多的cover到不同类型的Azure服务，例如：虚机，存储，网络，PaaS，容器等。关于如何部署实验环境，可以参照：

- [Before the HOL - Security baseline on Azure](https://github.com/microsoft/MCW-Security-baseline-on-Azure/blob/master/Hands-on%20lab/Before%20the%20HOL%20-%20Security%20baseline%20on%20Azure.md)

- [Tailwind Traders Website](https://github.com/Microsoft/TailwindTraders-Website)

目前实验环境暂时支持 Global Azure 的部署，如果需要部署在 Azure Mooncake, 需要重新 consolidate 下现有的部署脚本。

Step 2 准备 Ansible 环境

```
# 准备WSL或一台Linux虚机，例如：Ubuntu 16.04

# 安装 Ansible on Ubuntu 16.04 with Azure Module
sudo apt-get update && sudo apt-get install -y libssl-dev libffi-dev python-dev python-pip
sudo pip install ansible[azure]

# 创建 Azure Service Principle
az ad sp create-for-rbac

# 在当前目录创建文件 
touch ~/.azure/credentials

# 在文件中填入以下值
[default]
subscription_id=<your-subscription_id>
client_id=<security-principal-appid>
secret=<security-principal-password>
tenant=<security-principal-tenant>
```

由于Ansible不是本次Handson的主要介绍内容，更多资料请参照：

- [Using Ansible with Azure](https://docs.microsoft.com/en-us/azure/ansible/ansible-overview)

- [Ansible中支持的Azure Module](https://docs.ansible.com/ansible/latest/modules/list_of_cloud_modules#azure)

- [一些Azure相关的Ansible Playbook](https://github.com/Azure-Samples/ansible-playbooks)

__*注意 ：*__ 使用Ansible是为了更好的对部署的资源进行管理，与模板分离；很多时候我们部署的模板都是一个，但变量的名称各不相同，通过Azure Ansible的vars，结合Ansible Playbook可以更好的协调 Azure ARM Template 及 部署资源之间的管理。

#### Challenge 00 创建 `Initiative - 计划策略`来检验某一订阅下是否正确设置并打开了数据收集

#### Chn01 规划创建使用的 Log Analytics workspace

本次实验，将会通过 `ARM Template` 结合 `Ansible`部署出环境需要的 `Log Analytics workspace`.

本次实验的规划思路为：

- 订阅级别的 Activity Log 建议放在单独的 Log Analytics workspace 中

- 订阅下的 Azure Resources，建议以 Project 为单位进行划分

本次实验，将创建名为 `activityLogWS` & `projectOne` 的两个 workspace。

所有Workspace的创建，均通过 Ansible Playbook，所有需要创建的 workspace，均定义在 [loganalytics.yml](./files/monitor/monitor-ansible/vars/loganalytics.yml), 例如：

 ![image](./images/monitor/mon06.JPG)

```
# 进入 Ansible Playbook 目录
cd ./files/monitor/monitor-ansible

# 预先定义好要创建的workspace后，运行playbook
ansible-playbook ./deploy-loganalytics.yml
```

参考资料 : [使用 Azure CLI 2.0 创建 Log Analytics 工作区](https://docs.microsoft.com/zh-cn/azure/azure-monitor/learn/quick-create-workspace-cli)

#### Challenge 01 配置将 Activity Log 发送至 Log Analytics workspace

#### Challenge 02 配置开启 HOL资源中的诊断日志，并将诊断日志配置到 Log Analytics workspace

#### Challenge 03 安装并收集 On-Prem 中虚机的性能指标及模拟安装MySQL，收集MySQL的指标

#### Challenge 04 针对 Azure VM & 非 Azure VM 开启 Azure Monitor for VM，并创建 Service Map

#### Challenge 05 针对 AKS 监控数据的收集，包括AKS-Engine

#### Challenge 06 针对自定义数据，如何通过Azure Monitor REST API，将指标及日志发送至Azure Monitor

---

### HOL 设计定制化的监控指标大屏


### HOL 完善环境中的警报机及后期采取的行动

警报的作用是在问题出现的第一时间知晓，及时处理，尽可能避免大范围的影响。

Tips:

- 动态阈值 / 不同警报的处理方式 / Webhook连接高级管理系统 / Automation&Logic APP完成自动化运维 / Workbook


Chn： 通过Activity Log中的Resource Health及 Service Health, 及早了解平台的动向


### HOL 监控数据生命周期管理建议及数据安全性说明

### HOL 通过资源管理器模板 Enable 监控

### HOL 与 Azure DevOps 结合的 持续监控（Continus Monitor）

DevOps Pipelines 中的 Continus Monitor



### HOL 规划监控数据的生命周期

出于符合性、审核或脱机报告目的，对资源的性能或运行状况历史记录进行 https://docs.microsoft.com/zh-cn/azure/azure-monitor/learn/tutorial-archive-data


---
### 参考资料

- [Azure Monitor 数据源](https://docs.microsoft.com/zh-cn/azure/azure-monitor/platform/data-sources#operating-system-guest)

- [Cloud Governance Tools及需求mapping](https://azure.microsoft.com/en-gb/product-categories/management-tools/)

---



{
  "appId": "3bbcad52-5b8d-4093-b392-d5a11c25fe4a",
  "displayName": "azure-cli-2019-06-17-08-27-07",
  "name": "http://azure-cli-2019-06-17-08-27-07",
  "password": "3395c6ab-3547-408c-80ba-24c33333dcc0",
  "tenant": "72f988bf-86f1-41af-91ab-2d7cd011db47"
}
sysadm