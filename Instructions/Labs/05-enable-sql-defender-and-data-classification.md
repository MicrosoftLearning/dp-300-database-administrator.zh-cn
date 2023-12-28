---
lab:
  title: 实验室 5 - 启用 Microsoft Defender for SQL 和数据分类
  module: Implement a Secure Environment for a Database Service
---

# 启用 Microsoft Defender for SQL 和数据分类

预计时间：**20 分钟**

学生将利用从课程中获取的信息，在 Azure 门户和 AdventureWorks 数据库中进行配置，随后实现安全性。

你已被聘为高级数据库管理员，帮助确保数据库环境的安全。 这些任务侧重于 Azure SQL 数据库。

## 启用 Microsoft Defender for SQL

1. 在实验室虚拟机中，启动浏览器会话并导航到 [https://portal.azure.com](https://portal.azure.com/)。 使用此实验室虚拟机的“资源”选项卡上提供的 Azure 用户名和密码连接到门户。

    ![图 1](../images/dp-300-module-01-lab-01.png)

1. 在 Azure 门户顶部的搜索框中搜索“SQL Server”，然后从选项列表中单击“SQL Server”。

    ![自动生成的社交媒体文章说明的屏幕截图](../images/dp-300-module-04-lab-1.png)

1. 选择要转到详细信息页的服务器名称 dp300-lab-XXXXXXXX（可能为 SQL Server 分配了不同的资源组和位置）。

    ![自动生成的社交媒体文章说明的屏幕截图](../images/dp-300-module-04-lab-2.png)

1. 在 Azure SQL server 的主边栏选项卡中，导航到“安全”部分，并选择“Microsoft Defender for Cloud”。

    ![选择 Microsoft Defender for Cloud 选项的屏幕截图](../images/dp-300-module-05-lab-01.png)

    在“Microsoft Defender for Cloud”页面上，选择“启用 Microsoft Defender for SQL”。

1. 成功启用 Azure Defender for SQL 后，将显示以下通知消息。

    ![选择“配置”选项的屏幕截图](../images/dp-300-module-05-lab-02_1.png)

1. 在 Microsoft Defender for Cloud 页上，选择“配置”链接（可能需要刷新页面才能看到此选项）

    ![选择“配置”选项的屏幕截图](../images/dp-300-module-05-lab-02.png)

1. 在“服务器设置”页面上，注意将“MICROSOFT DEFENDER FOR SQL”下的切换开关设置为“开”。

## 启用数据分类

1. 在 Azure SQL Server 的主边栏选项卡中，导航到“设置”部分，选择“SQL 数据库”，然后选择数据库名称 。

    ![屏幕截图显示选择 AdventureWOrksLT 数据库](../images/dp-300-module-05-lab-04.png)

1. 在 AdventureWorksLT 数据库的主边栏选项卡上，导航到“安全性”部分，然后选择“数据发现和分类”。

    ![屏幕截图显示“数据发现和分类”](../images/dp-300-module-05-lab-05.png)

1. 在“数据发现和分类”屏幕上，你会看见一条信息性消息：“当前通过 SQL 信息保护策略，我们发现了 15 列分类建议”。 选择此链接。

    ![屏幕截图显示“分类建议”](../images/dp-300-module-05-lab-06.png)

1. 在下一个“数据发现和分类”屏幕中，选中“全选”旁的复选框，然后选择“接受所选建议”，最后选择“保存”以将分类保存到数据库中。

    ![屏幕截图显示“接受所选建议”](../images/dp-300-module-05-lab-07.png)

1. 返回到“数据发现和分类”屏幕，请注意，已将 15 列成功分类到在 5 个不同的表中。

    ![屏幕截图显示“接受所选建议”](../images/dp-300-module-05-lab-08.png)

在本练习中，你通过启用 Microsoft Defender for SQL 增强了 Azure SQL 数据库的安全性。 你基于 Azure 门户建议创建了分类列。
