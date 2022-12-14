---
lab:
  title: 实验室 3 - 使用 Azure Active Directory 授予对 Azure SQL 数据库的访问权限
  module: Implement a Secure Environment for a Database Service
---

# <a name="configure-database-authentication-and-authorization"></a>配置数据库身份验证和授权

预计时间：**20 分钟**

学生将运用在课程中学到的信息，在 Azure 门户和 AdventureWorks 数据库中进行配置，再随后实现安全性。

你已被聘为高级数据库管理员，帮助确保数据库环境的安全。

注意：这些练习要求你复制并粘贴 T-SQL 代码。 在执行代码之前，请验证代码是否已正确复制。

## <a name="authorize-access-to-azure-sql-database-with-azure-active-directory"></a>使用 Azure Active Directory 授予对 Azure SQL 数据库的访问权限

1. 在实验室虚拟机中，启动浏览器会话并导航到 [https://portal.azure.com](https://portal.azure.com/)。 使用此实验室虚拟机的“资源”选项卡上提供的 Azure 用户名和密码连接到门户。  

    ![图 1](../images/dp-300-module-01-lab-01.png)

1. 在 Azure 门户主页上，选择“所有资源”。

    ![屏幕截图显示在 Azure 门户主页上选择“所有资源”](../images/dp-300-module-03-lab-01.png)

1. 选择 Azure SQL 数据库服务器 dp300-lab-xxxxxx（其中 xxxxxx 是随机字符串），然后选择“Active Directory 管理员”旁边的“未配置”。   

    ![屏幕截图显示选择“未配置”](../images/dp-300-module-03-lab-02.png)

1. 在下一个屏幕上选择“设置管理员”。

    ![屏幕截图显示选择“设置管理员”](../images/dp-300-module-03-lab-03.png)

1. 在“Azure Active Directory”边栏中，搜索登录 Azure 门户时所用的 Azure 用户名，然后单击“选择” 。

1. 选择“保存”以完成此过程。 这样，你的用户名就成了服务器的 Azure Active Directory 管理员，如下所示。

    ![“Active Directory 管理员”页的屏幕截图](../images/dp-300-module-03-lab-04.png)

1. 选择左侧的“概述”，然后复制该服务器名称。

    ![显示从何处复制服务器名称的屏幕截图](../images/dp-300-module-03-lab-05.png)

1. 打开 SQL Server Management Studio，然后选择“连接”****“数据库引擎”。 在“服务器名称”中，粘贴服务器的名称。 将身份验证类型更改为“Azure Active Directory - 使用 MFA 进行通用身份验证”。

    ![“连接到服务器”对话框的屏幕截图](../images/dp-300-module-03-lab-06.png)

    对于“用户名”字段，从“资源”选项卡选择 Azure 用户名  。

1. 选择“连接”  。

> [!NOTE]
> 首次尝试登录到 Azure SQL 数据库时，需要将客户端 IP 地址添加到防火墙。 SQL Server Management Studio 可以为你执行此操作。 使用“资源”选项卡中的 Azure 门户密码，然后选择“登录”，选择 Azure 凭据，然后选择“确定”。
> ![显示添加客户端 IP 地址的屏幕截图](../images/dp-300-module-03-lab-07.png)

## <a name="manage-access-to-database-objects"></a>管理对数据库对象的访问权限

在此任务中，你将管理对数据库及其对象的访问权限。 首先，在 AdventureWorksLT 数据库中创建两个用户。

1. 使用“对象资源管理器”，并展开“数据库”。
1. 右键单击“AdventureWorksLT”并选择“新建查询”。

    ![“新建查询”菜单选项的屏幕截图](../images/dp-300-module-03-lab-08.png)

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

    ![错误消息“对象 DemoProc 的 EXECUTE 权限被拒绝”的屏幕截图](../images/dp-300-module-03-lab-09.png)

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

    ![显示从存储过程返回的数据行的屏幕截图](../images/dp-300-module-03-lab-10.png)

在本练习中，你了解了如何使用 Azure Active Directory 向 Azure 凭据授予访问 Azure 中托管的 SQL Server 的权限。 你还使用了 T-SQL 语句来创建新的数据库用户，并授予他们运行存储过程的权限。
