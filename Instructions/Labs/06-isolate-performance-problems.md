---
lab:
  title: 实验室 6 - 通过监视隔离性能问题
  module: Monitor and optimize operational resources in Azure SQL
---

# 通过监视隔离性能问题

预计时间：30 分钟

学生将获取从课程中获得的信息，以确定 AdventureWorksLT 中数字转换项目的可交付成果。 通过检查 Azure 门户以及其他工具，学生将确定如何利用工具来识别和解决与性能相关的问题。

你是数据库管理员，需识别与性能相关的问题并提供可行的解决方案来解决发现的所有问题。 需要通过 Azure 门户来识别性能问题并提出解决问题的方法。

> &#128221; 这些练习要求复制粘贴 T-SQL 代码并使用现有的 SQL 资源。 在执行代码之前，请验证代码是否已正确复制。

## 设置环境

如果实验室虚拟机已提供并预配置，则应在 **C:\LabFiles** 文件夹中找到已准备好的实验室文件。 *花点时间检查，如果文件已存在，请跳过本部分*。 但是，如果使用自己的计算机或缺少实验室文件，则需要从 *GitHub* 克隆它们才能继续。

1. 如果未提供实验室虚拟机或本地计算机，请启动 Visual Studio Code 会话。

1. 打开命令面板 (Ctrl+Shift+P)，然后键入 **Git: Clone**。 选择 **Git: Clone** 选项。

1. 将以下 URL 粘贴到**存储库 URL**字段中，然后选择 **Enter**。

    ```url
    https://github.com/MicrosoftLearning/dp-300-database-administrator.git
    ```

1. 将存储库保存到实验室虚拟机上的 **C:\LabFiles** 文件夹，如果没有提供本地计算机，则保存到本地计算机上的相应文件夹（如果文件夹不存在，则创建该文件夹）。

## 在 Azure 中设置 SQL Server

登录到 Azure 并检查是否已在 Azure 中运行现有的 Azure SQL Server 实例。 *如果已在 Azure 中运行 SQL Server 实例，请跳过本部分*。

1. 如果未提供实验室虚拟机或本地计算机，请启动 Visual Studio Code 会话，然后从上一部分导航到克隆的存储库。

1. 右键单击 **/Allfiles/Labs** 文件夹，然后选择“**在集成终端中打开**”。

1. 使用 Azure CLI 连接到 Azure。 输入以下命令并选择 **Enter**。

    ```bash
    az login
    ```

    > &#128221; 请注意，浏览器窗口将打开。 使用 Azure 凭据登录。

1. 登录到 Azure 后，可以创建资源组（如果尚不存在），并在该资源组下创建 SQL Server 和数据库。 输入以下命令并选择 **Enter**。 *完成该脚本需要几分钟时间。*

    ```bash
    cd ./Setup
    ./deploy-sql-database.ps1
    ```

    > &#128221; 请注意，默认情况下，此脚本将创建名为 **contoso-rg** 的资源组，或使用名称以 *contoso-rg* 开头的资源（如果存在）。 默认情况下，它还会在**美国西部 2** 区域 (westus2) 上创建所有资源。 最后，它将为 **SQL 管理员密码生成随机 12 个字符的密码**。 可以通过将一个或多个参数 **-rgName**、**-location** 和 **-sqlAdminPw** 与自己的值一起使用来更改这些值。 密码必须满足 Azure SQL 密码复杂性要求，长度至少为 12 个字符，并且至少包含 1 个大写字母、1 个小写字母、1 个数字和 1 个特殊字符。

    > &#128221; 请注意，该脚本会将当前的公共 IP 地址添加到 SQL Server 防火墙规则。

1. 脚本完成后，它将返回资源组名称、SQL Server 名称和数据库名称，以及管理员用户名和密码。 *记下这些值，因为稍后在实验室中需要这些值*。

---

## 在 Azure 门户中查看 CPU 利用率

1. 如果未提供实验室虚拟机或本地计算机，请启动浏览器会话并导航到 [https://portal.azure.com](https://portal.azure.com/)。 使用 Azure 凭据登录到门户。

1. 在 Azure 门户顶部的搜索框中搜索“*SQL servers*”，然后从选项列表中选择“**SQL Server**”。

1. 选择 SQL Server **dp300-lab-xxxxxxxx**，其中 *xxxxxxxx* 是随机数值字符串。

    > &#128221; 请注意，如果使用此实验室未创建自己的 Azure SQL Server，请选择该 SQL Server 的名称。

1. 在 Azure SQL server 主页的“**安全**”下，选择“**网络**”。

1. 在“**网络**”页上，验证当前公共 IP 是否已添加到“**防火墙规则**”列表，如果不是，选择 **“+ 添加客户端 IPv4 地址”（IP 地址）**，然后选择“**保存**”。

1. 在 Azure SQL Server 的主边栏选项卡中，导航到“**设置**”部分，选择“**SQL 数据库**”，然后选择 **AdventureWorksLT** 数据库。

1. 在左侧导航栏中，选择“查询编辑器(预览版)”。

    注意：此功能为预览版。

1. 选择 SQL Server 管理员用户名，输入密码或 Microsoft Entra 凭据（如果已分配连接到数据库）。
    - 服务器名称：&lt;在此处粘贴你的 Azure SQL 数据库服务器名称&gt;
    - 身份验证：SQL Server 身份验证
    - **服务器管理员登录：** Azure SQL 数据库服务器管理员登录
    - **密码：** Azure SQL 数据库服务器管理员密码

1. 在“查询 1”中，键入以下查询，然后选择“运行” ：

    ```sql
    DECLARE @Counter INT 
    SET @Counter=1
    WHILE ( @Counter <= 10000)
    BEGIN
        SELECT 
             RTRIM(a.Firstname) + ' ' + RTRIM(a.LastName)
            , b.AddressLine1
            , b.AddressLine2
            , RTRIM(b.City) + ', ' + RTRIM(b.StateProvince) + '  ' + RTRIM(b.PostalCode)
            , CountryRegion
            FROM SalesLT.Customer a
            INNER JOIN SalesLT.CustomerAddress c 
                ON a.CustomerID = c.CustomerID
            RIGHT OUTER JOIN SalesLT.Address b
                ON b.AddressID = c.AddressID
        ORDER BY a.LastName ASC
        SET @Counter  = @Counter  + 1
    END
    ```

1. 等待查询完成。

1. 再次运行查询*两*次，以在数据库上生成一些 CPU 负载。

1. 在 AdventureWorksLT 数据库的边栏选项卡上，选择“监视”部分的“指标”图标  。

    如果弹出消息“*“未保存的更改将被丢弃*”，请选择“**确定**”。

1. 更改“指标”菜单选项以反映 CPU 百分比，然后选择“平均值聚合”。这将显示给定时间范围的平均 CPU 百分比。

1. 观察一段时间内的 CPU 平均值。 在运行查询时，应注意到图形末尾的 CPU 使用率峰值。

## 识别 CPU 占用率高的查询

1. 在 AdventureWorks 数据库的边栏选项卡的“智能性能”部分，找到“Query Performance Insight”图标  。

1. 选择“重置设置”。

1. 选择图下方的网格中的查询。 如果未看到我们之前多次运行的查询，请等待最多 2 到 5 分钟，然后选择“**刷新**”。

    > &#128221; 如果列出了多个查询，请选择每个查询来观察结果。 请注意可用于每个查询的大量信息。

1. 对于之前运行的查询，请注意总持续时间超过一分钟，运行时间约为 3000 次。

1. 根据运行的查询查看**查询详细信息**页上的 SQL 文本，你会注意到**查询详细信息**仅包含 **SELECT** 语句，而不包含 **WHILE** 循环或其他语句。 之所以发生这种情况，是因为 **Query Performance Insight** 依赖于**查询存储**中的数据，该数据仅跟踪 **SELECT、INSERT、UPDATE、DELETE、MERGE** 和 **BULK INSERT** 等数据操作语言 (DML) 语句，同时忽略数据定义语言 (DDL) 语句。

并非所有性能问题都与单个查询执行的高 CPU 使用率有关。 在这种情况下，查询被执行了数千次，这也可能导致 CPU 使用率高。

---

## 清理资源

如果不将 Azure SQL Server 用于任何其他目的，则可以清理在此实验室中创建的资源。

### 删除资源组

如果为此实验室创建了一个新的资源组，则可以删除资源组以移除在此实验室中创建的所有资源。

1. 在 Azure 门户中，从左侧导航窗格中选择“**资源组**”，或在搜索栏中搜索“**资源组**”，并从结果中选择资源组。

1. 转到为此实验室所创建的资源组。 资源组将包含在此实验室中包含的 Azure SQL 服务器和其他资源。

1. 在顶部菜单中选择“删除资源组”。

1. 在“**删除资源组**”对话框中，输入资源组的名称进行确认，然后选择“**删除**”。

1. 等待资源组被删除。

1. 关闭 Azure 门户。

### 仅删除实验室资源

如果未为此实验室创建新的资源组，并且想要使资源组及其以前的资源保持不变，仍可以删除在此实验室中创建的资源。

1. 在 Azure 门户中，从左侧导航窗格中选择“**资源组**”，或在搜索栏中搜索“**资源组**”，并从结果中选择资源组。

1. 转到为此实验室所创建的资源组。 资源组将包含在此实验室中包含的 Azure SQL 服务器和其他资源。

1. 选择前面在实验室中指定的 SQL Server 名称为前缀的所有资源。

1. 从顶部菜单中选择**删除**。

1. 在“**删除资源**”对话框中，键入“**删除**”并选择“**删除**”。

1. 再次选择“**删除**”，确认删除资源。

1. 等待资源删除完毕。

1. 关闭 Azure 门户。

### 删除 LabFiles 文件夹

如果为此实验室创建了一个新的 LabFiles 文件夹，并且不再需要它，则可以删除 LabFiles 文件夹以移除在此实验室中创建的所有文件。

1. 如果未提供实验室虚拟机或本地计算机，请打开文件资源管理器并导航到 **C:\\** 驱动器。
1. 右键单击 **LabFiles** 文件夹，然后选择“**删除**”。
1. 选择“**是**”，确认删除文件夹。

---

你已成功完成本实验室。

在本练习中，你学习了如何探索 Azure SQL 数据库的服务器资源，以及如何通过 Query Performance Insight 识别潜在的查询性能问题。
