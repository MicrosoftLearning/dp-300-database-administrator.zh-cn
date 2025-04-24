---
lab:
  title: 练习 3 - 使用 Microsoft Entra ID 授予对 Azure SQL 数据库的访问权限
  module: Implement a Secure Environment for a Database Service
---

# 配置数据库身份验证和授权

**预计用时：25 分钟**

学生将运用在课程中学到的信息，在 Azure 门户和 *AdventureWorksLT* 数据库中进行配置，再随后实现安全性。

你已被聘为高级数据库管理员，帮助确保数据库环境的安全。

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

1. 脚本完成后，它将返回资源组名称、SQL Server 名称和数据库名称，以及管理员用户名和密码。 记下这些值，因为稍后在实验室中需要这些值。

---

## 使用 Microsoft Entra 授予对 Azure SQL 数据库的访问权限

可以使用 `CREATE USER [anna@contoso.com] FROM EXTERNAL PROVIDER` T-SQL 语法以包含的数据库用户的身份从 Microsoft Entra 帐户创建登录名。 包含的数据库用户映射到与该数据库关联的 Microsoft Entra 目录中的标识，并且在 `master` 数据库中没有登录名。

在 Azure SQL 数据库中引入 Microsoft Entra 服务器登录名后，可以在 SQL 数据库的虚拟 `master` 数据库中从 Microsoft Entra 主体创建登录名。 可以根据 Microsoft Entra 用户、组和服务主体创建 Microsoft Entra 登录名。** 有关详细信息，请参阅 [Microsoft Entra 服务器主体](/azure/azure-sql/database/authentication-azure-ad-logins)

此外，Azure 门户只能用于创建管理员，且 Azure 基于角色的访问控制角色不会传播到 Azure SQL 数据库逻辑服务器。 必须使用 Transact-SQL (T-SQL) 授予其他服务器和数据库权限。 让我们为 SQL Server 创建 Microsoft Entra 管理员。

1. 如果未提供实验室虚拟机或本地计算机，请启动浏览器会话并导航到 [https://portal.azure.com](https://portal.azure.com/)。 使用 Azure 凭据登录到门户。

1. 在 Azure 门户主页上，搜索并选择“**SQL Servers**”。

1. 选择 SQL Server **dp300-lab-xxxxxxxx**，其中 *xxxxxxxx* 是随机数值字符串。

    > &#128221; 请注意，如果使用此实验室未创建自己的 Azure SQL Server，请选择该 SQL Server 的名称。

1. 在“*概述*”边栏选项卡中，选择 *Microsoft Entra 管理员* 旁边的“**未配置**”。

1. 在下一个屏幕上选择“设置管理员”。

1. 在“**Microsoft Entra ID**”边栏中，搜索登录 Azure 门户时所用的 Azure 用户名，然后单击“**选择**”。

1. 选择“保存”以完成此过程。 这会使用户名成为服务器的 Microsoft Entra 管理员。

1. 选择左侧的“概述”，然后复制该服务器名称。

1. 打开 SQL Server Management Studio (SSMS)，然后选择“**连接**” > “**数据库引擎**”。 在“服务器名称”中，粘贴服务器的名称。 将身份验证类型更改为 **Microsoft Entra MFA**。

1. 选择“连接” 。

## 管理对数据库对象的访问权限

在此任务中，你将管理对数据库及其对象的访问权限。 首先，在 AdventureWorksLT 数据库中创建两个用户。

1. 如果未在 SSMS 中提供实验室虚拟机或本地计算机，请使用 Azure Server 管理员帐户或 Microsoft Entra 管理员帐户登录到 *AdventureWorksLT* 数据库。

1. 使用“对象资源管理器”，并展开“数据库”。

1. 右键单击“AdventureWorksLT”并选择“新建查询”。

1. 在“新建查询”窗口中，将以下 T-SQL 复制并粘贴到其中。 执行查询以创建两个用户。

    ```sql
    CREATE USER [DP300User1] WITH PASSWORD = 'Azur3Pa$$';
    GO

    CREATE USER [DP300User2] WITH PASSWORD = 'Azur3Pa$$';
    GO
    ```

    注意：这些用户是在 AdventureWorksLT 数据库范围内创建的。 接下来，你将创建一个自定义角色并将用户添加到其中。

1. 在该查询窗口中执行以下 T-SQL。

    ```sql
    CREATE ROLE [SalesReader];
    GO

    ALTER ROLE [SalesReader] ADD MEMBER [DP300User1];
    GO

    ALTER ROLE [SalesReader] ADD MEMBER [DP300User2];
    GO
    ```

    接下来，在 SalesLT 架构中创建一个新的存储过程。

1. 在查询窗口中执行以下 T-SQL。

    ```sql
    CREATE OR ALTER PROCEDURE SalesLT.DemoProc
    AS
    SELECT P.Name, Sum(SOD.LineTotal) as TotalSales ,SOH.OrderDate
    FROM SalesLT.Product P
    INNER JOIN SalesLT.SalesOrderDetail SOD on SOD.ProductID = P.ProductID
    INNER JOIN SalesLT.SalesOrderHeader SOH on SOH.SalesOrderID = SOD.SalesOrderID
    GROUP BY P.Name, SOH.OrderDate
    ORDER BY TotalSales DESC
    GO
    ```

    接下来，使用 `EXECUTE AS USER` 语法测试安全性。 这允许数据库引擎在用户的上下文中执行查询。

1. 执行以下 T-SQL。

    ```sql
    EXECUTE AS USER = 'DP300User1'
    EXECUTE SalesLT.DemoProc
    ```

    该操作将失败，并显示以下消息：

    <span style="color:red">消息 229，级别 14，状态 5，过程 SalesLT.DemoProc，第 1 行 [Batch 开始行 0] 对对象“DemoProc”，数据库“AdventureWorksLT”，架构“SalesLT”拒绝 EXECUTE 权限。</span>

1. 接下来，向角色授予权限，使其能够执行存储过程。 执行下面的 T-SQL。

    ```sql
    REVERT;
    GRANT EXECUTE ON SCHEMA::SalesLT TO [SalesReader];
    GO
    ```

    第一个命令将执行上下文还原回数据库所有者。

1. 重新运行前面的 T-SQL。

    ```sql
    EXECUTE AS USER = 'DP300User1'
    EXECUTE SalesLT.DemoProc
    ```

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

在本练习中，你将看到如何使用 Microsoft Entra ID 授予 Azure 凭据访问 Azure 中托管的 SQL Server。 你还使用了 T-SQL 语句来创建新的数据库用户，并授予他们运行存储过程的权限。
