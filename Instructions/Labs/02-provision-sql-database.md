---
lab:
  title: 实验室 2 - 预配 Azure SQL 数据库
  module: Plan and Implement Data Platform Resources
---

# 预配 Azure SQL 数据库

预计用时：40 分钟

学生将配置部署具有虚拟网络终结点的 Azure SQL 数据库所需的基本资源。 与 SQL 数据库的连接将通过实验室虚拟机上的 SQL Server Management Studio（如有）或本地计算机设置进行验证。

作为 AdventureWorks 的数据库管理员，你将建立一个新的 SQL 数据库，其中包括一个虚拟网络终结点，以增加和简化部署的安全性。 将使用 SQL Server Management Studio，评估使用 SQL Notebook 进行数据查询和结果保留的情况。

## 在 Azure 门户上导航

1. 如果提供了实验室虚拟机，则在本地计算机上打开浏览器窗口。

1. 导航到[https://portal.azure.com](https://portal.azure.com/)的 Azure 门户。 使用 Azure 帐户或提供的凭据（如有）登录 Azure 门户。

1. 在 Azure 门户顶部的搜索框中搜索“*资源组*”，然后从选项列表中选择“**资源组**”。

1. 在“**资源组**”页上 (如果提供)，选择以 *contoso-rg* 开头的资源组。 如果此资源组不存在，请在本地区域中创建一个名为 *contoso-rg* 的新资源组，或使用现有资源组并记下其所在的区域。

## 创建虚拟网络

1. 在 Azure 门户主页中，选择左侧菜单。  

1. 在左侧导航窗格中，选择“**虚拟网络**”  

1. 单击“**+ 创建**”，打开“**创建虚拟网络**”页面。 在“基本信息”选项卡上，完成以下信息：

    - 订阅：&lt;你的订阅&gt;
    - **资源组：** 以 *DP300* 开头的资源组或之前选择的资源组
    - 名称：lab02-vnet
    - 区域：选择你创建了资源组的同一区域

1. 选择“**查看 + 创建**”，查看新虚拟网络的设置，然后选择“**创建**”。

## 在 Azure 门户中预配 Azure SQL 数据库

1. 在 Azure 门户顶部的搜索框中搜索“*SQL 数据库*”，然后从选项列表中选择“**SQL 数据库**”。

1. 在**SQL 数据库**边栏选项卡上，选择 **+ 创建**

1. 在“**创建 SQL 数据库**”页上，选择“**基本信息**”选项卡上的以下选项，然后选择“**下一步: 网络**”。

    - **订阅：**&lt;你的订阅&gt;
    - **资源组：** 以 *DP300* 开头的资源组或之前选择的资源组
    - 数据库名称：AdventureWorksLT
    - **服务器：** 选择“**新建**”链接。 此时将打开“创建 SQL 数据库服务器”页面。 提供服务器详细信息，如下所示：
        - **服务器名称：** dp300-lab-&lt;your initials (lower case)&gt;，如有需要，可以随机输入 5 位数字（服务器名称必须全局唯一）
        - 区域：&lt;你的本地区域，与你的资源组的选定区域相同，否则可能会失败&gt;
        - 身份验证方法：使用 SQL 身份验证
        - 服务器管理员登录名：dp300admin
        - **密码：** 选择复杂密码并记下它
        - **确认密码：** 选择以前选择的相同密码
    - 选择“**确定**”，以返回到“**创建 SQL 数据库**”页。
    - 将“**想要使用 SQL 弹性池?**”设置为“**否**”。
    - 工作负载环境：开发
    - 在“**计算 + 存储**”选项上，选择“**配置数据库**”链接。 在**配置**页上的**服务层**下拉列表中，选择**基本**，然后选择**应用**。

1. 对于“**备份存储冗余**”选项，请保留默认值：**异地冗余备份存储**。

1. 然后选择“**下一步: 网络**”。

1. 在“**网络**”选项卡上的“**网络连接**”选项中，选择“**专用终结点**”单选按钮。

1. 然后选择“**专用终结点**”选项下的“**+ 添加专用终结点**”链接。

1. 完成右侧窗格的“创建专用终结点”，如下所示：

    - 订阅：&lt;你的订阅&gt;
    - **资源组：** 以 *DP300* 开头的资源组或之前选择的资源组
    - 区域：&lt;你的本地区域，与你的资源组的选定区域相同，否则可能会失败&gt;
    - 名称：DP-300-SQL-Endpoint
    - 目标子资源：SqlServer
    - 虚拟网络：lab02-vnet
    - 子网：lab02-vnet/default (10.x.0.0/24)
    - 与专用 DNS 区域集成：是
    - 专用 DNS 区域：保留默认值
    - 检查设置，然后选择“**确定**”。  

1. 新终结点将显示在“专用终结点”列表中。

1. 依次选择“**下一步: 安全性**”和“**下一步: 其他设置**”。  

1. 在“其他设置”页上，选择“使用现有数据”选项上的“示例”  。 如果为示例数据库显示弹出消息，请选择“确定”。

1. 选择“查看 + 创建”  。

1. 选择“创建”之前查看设置****。

1. 部署完成后，选择**转到资源**。

## 启用对 Azure SQL 数据库的访问

1. 在“**SQL 数据库**”页中，选择“**概述**”部分，然后在顶部选择服务器名称的链接。

1. 在 SQL Server 导航边栏选项卡上，选择“安全性”部分下的“网络” 。

1. 在**公共访问**选项卡上，选择**选定的网络**。

1. 选择“**+ 添加客户端 IPv4 地址**”。 这将添加防火墙规则，以允许当前 IP 地址访问 SQL Server。

1. 选中“**允许 Azure 服务和资源访问此服务器**”属性。

1. 选择“保存”。

---

## 连接到 Server Management Studio 中的 Azure SQL 数据库

1. 在 Azure 门户的左侧导航窗格中，选择“**SQL 数据库**”。 然后选择 **AdventureWorksLT** 数据库。

1. 在“**概述**”页上，复制“**服务器名称**”值。

1. 从实验室虚拟机（如果提供）或本地计算机（如果不提供）启动 SQL Server Management Studio。

1. 在“**连接到服务器**”对话框中，粘贴从 Azure 门户复制的“**服务器名称**”值。

1. 在“**身份验证**”下拉列表中，选择“**SQL Server 身份验证**”。

1. 在“**登录**”字段中，输入 **dp300admin**。

1. 在“**密码**”字段中，输入在 SQL Server 创建过程中选择的密码。

1. 选择“连接” 。

1. SQL Server Management Studio 将连接到 Azure SQL 数据库服务器。 可以展开服务器，然后展开“**数据库”** 节点以查看 *AdventureWorksLT* 数据库。

## 使用 SQL Server Management Studio 查询 Azure SQL Database

1. 右键单击 *AdventureWorksLT* 数据库并选择“**新建查询**”。

1. 将以下语句粘贴到查询窗口中：

    ```sql
    SELECT TOP 10 cust.[CustomerID], 
        cust.[CompanyName], 
        SUM(sohead.[SubTotal]) as OverallOrderSubTotal
    FROM [SalesLT].[Customer] cust
        INNER JOIN [SalesLT].[SalesOrderHeader] sohead
             ON sohead.[CustomerID] = cust.[CustomerID]
    GROUP BY cust.[CustomerID], cust.[CompanyName]
    ORDER BY [OverallOrderSubTotal] DESC
    ```

1. 选择工具栏中的“**执行**”按钮以执行查询。

1. 在“**结果**”窗格中查看查询结果。

1. 右键单击 *AdventureWorksLT* 数据库并选择“**新建查询**”。

1. 将以下语句粘贴到查询窗口中：

    ```sql
    SELECT TOP 10 cat.[Name] AS ProductCategory, 
        SUM(detail.[OrderQty]) AS OrderedQuantity
    FROM salesLT.[ProductCategory] cat
        INNER JOIN [SalesLT].[Product] prod
            ON prod.[ProductCategoryID] = cat.[ProductCategoryID]
        INNER JOIN [SalesLT].[SalesOrderDetail] detail
            ON detail.[ProductID] = prod.[ProductID]
    GROUP BY cat.[name]
    ORDER BY [OrderedQuantity] DESC
    ```

1. 选择工具栏中的“**执行**”按钮以执行查询。

1. 在“**结果**”窗格中查看查询结果。

1. 关闭 SQL Server Management Studio。 当系统提示是否保存更改时，请选择“**否**”。

---

## 清理资源

如果不将虚拟机用于任何其他目的，则可以清理在此实验室中创建的资源。

### 删除资源组

如果为此实验室创建了一个新的资源组，则可以删除资源组以移除在此实验室中创建的所有资源。

1. 在 Azure 门户中，从左侧导航窗格中选择“**资源组**”，或在搜索栏中搜索“**资源组**”，并从结果中选择资源组。

1. 转到为此实验室所创建的资源组。 资源组将包含在此实验室中创建的虚拟机和其他资源。

1. 在顶部菜单中选择“删除资源组”。

1. 在“**删除资源组**”对话框中，输入资源组的名称进行确认，然后选择“**删除**”。

1. 等待资源组被删除。

1. 关闭 Azure 门户。

### 仅删除实验室资源

如果未为此实验室创建新的资源组，并且想要使资源组及其以前的资源保持不变，仍可以删除在此实验室中创建的资源。

1. 在 Azure 门户中，从左侧导航窗格中选择“**资源组**”，或在搜索栏中搜索“**资源组**”，并从结果中选择资源组。

1. 转到为此实验室所创建的资源组。 资源组将包含在此实验室中创建的虚拟机和其他资源。

1. 选择前面在实验室中指定的 SQL Server 名称为前缀的所有资源。 此外，请选择创建的虚拟网络和专用 DNS 区域。

1. 从顶部菜单中选择**删除**。

1. 在“**删除资源**”对话框中，键入“**删除**”并选择“**删除**”。

1. 再次选择“**删除**”，确认删除资源。

1. 等待资源删除完毕。

1. 关闭 Azure 门户。

---

你已成功完成本实验室。

在本练习中，你了解了如何使用虚拟网络终结点部署 Azure SQL 数据库。 你还成功地连接到使用 SQL Server Management Studio 创建的 SQL 数据库。
