## WIP - Azure Governance Services Hands on Lab

---

## WIP - 动手实验 Azure Resource Graph

Azure Resource Graph 是 Azure 中的一项服务，旨在通过提供高效和高性能的资源浏览来扩展 Azure 资源管理器，它能够跨给定的订阅组进行大规模查询，使你能够有效地管理环境。

Azure Resource Graph 是 Azure 中提供的一种对资源进行查询整理的方式，可以根据需要，定制化查询结果，并针对查询结果做相应的操作，比如 `形成可视化的图表`, `导出成Excel表格`等，方便了运维人员对于云端资源的管理。

Azure Resource Graph 始终保持与环境中资源更新的同步，资源管理器会将资源的更新同步给 Resource Graph；Azure Resource Graph 使用 `Kusto` 作为查询语言，以确保用户对于 Azure 云端查询语言的一致性。

### Lab 1 通过 Azure Portal，运行自己的第一个查询

Azure Portal 针对 Azure Resource Graph 提供了便捷的查询工具 `Azure Resource Graph Explorer`，可以允许用户进行快速的查询，并将查询结果可视化到 Dashboard中，便于直观的了解环境中的资源。

点击进入  `Azure Resource Graph Explorer`

![image](./images/191011/101101.png)

### 参考资料

- [Azure Resource Graph 服务概述](https://docs.microsoft.com/zh-cn/azure/governance/resource-graph/overview)


---