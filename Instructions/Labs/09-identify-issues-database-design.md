---
lab:
  title: 实验室 9 - 找出数据库设计问题
  module: Optimize query performance in Azure SQL
---

# 找出数据库设计问题

预计用时：15 分钟

学生将获取从课程中获得的信息，以确定 AdventureWorks 中数字转换项目的可交付成果。 通过检查 Azure 门户以及其他工具，学生将确定如何利用本机工具来识别和解决与性能相关的问题。 最后，学生将能够评估数据库设计中的规范化、数据类型选择和索引设计问题。

你是数据库管理员，需识别与性能相关的问题并提供可行的解决方案来解决发现的所有问题。 十多年来，AdventureWorks 一直将自行车和自行车零件直接销售给最终用户和分销商。 你的工作是使用本模块中所述的技术来确定查询性能方面的问题并解决问题。

> &#128221; 这些练习要求复制并粘贴 T-SQL 代码。 在执行代码之前，请验证代码是否已正确复制。

## 设置环境

如果实验室虚拟机已提供并预配置，则应在 **C:\LabFiles** 文件夹中找到已准备好的实验室文件。 *花点时间检查，如果文件已存在，请跳过本部分*。 但是，如果使用自己的计算机或缺少实验室文件，则需要从 *GitHub* 克隆它们才能继续。

1. 如果未提供实验室虚拟机或本地计算机，请启动 Visual Studio Code 会话。

1. 打开命令面板 (Ctrl+Shift+P)，然后键入 **Git: Clone**。 选择 **Git: Clone** 选项。

1. 将以下 URL 粘贴到**存储库 URL**字段中，然后选择 **Enter**。

    ```url
    https://github.com/MicrosoftLearning/dp-300-database-administrator.git
    ```

1. 将存储库保存到实验室虚拟机上的 **C:\LabFiles** 文件夹，如果没有提供本地计算机，则保存到本地计算机上的相应文件夹（如果文件夹不存在，则创建该文件夹）。

---

## 还原数据库

如果已还原 **AdventureWorks2017** 数据库，则可以跳过本部分。

1. 如果未提供实验室虚拟机或本地计算机，请启动 SQL Server Management Studio 会话 (SSMS)。

1. SSMS 打开时，默认情况下将显示“**连接到服务器**”对话框。 选择默认实例，然后选择“**连接**”。 可能需要选中“**信任服务器证书**”复选框。

    > &#128221; 请注意，如果使用自己的 SQL Server 实例，则需要使用相应的服务器实例名称和凭据连接到它。

1. 选择**数据库**文件夹，然后选择**新建查询**

1. 在新建查询窗口中，将以下 T-SQL 复制并粘贴到其中。 执行查询以还原数据库。

    ```sql
    RESTORE DATABASE AdventureWorks2017
    FROM DISK = 'C:\LabFiles\dp-300-database-administrator\Allfiles\Labs\Shared\AdventureWorks2017.bak'
    WITH RECOVERY,
          MOVE 'AdventureWorks2017' 
            TO 'C:\LabFiles\AdventureWorks2017.mdf',
          MOVE 'AdventureWorks2017_log'
            TO 'C:\LabFiles\AdventureWorks2017_log.ldf';
    ```

    > &#128221; 必须有一个名为 **C:\LabFiles** 的文件夹。 如果没有此文件夹，请创建它或指定数据库和备份文件的另一个位置。

1. 在“**消息**”选项卡下，应会看到一条消息，指示数据库已成功还原。

## 检查查询并确定问题

1. 选择“新建查询”  。 复制下面的 T-SQL 代码并将其粘贴到“查询”窗口中。 选择“执行”以执行此查询。

    ```sql
    USE AdventureWorks2017

    GO
    
    SELECT BusinessEntityID, NationalIDNumber, LoginID, HireDate, JobTitle
    FROM HumanResources.Employee
    WHERE NationalIDNumber = 14417807;
    ```

1. 在运行查询或按 **Ctrl+M** 之前，选择“**执行**”按钮右侧的“**包含实际执行计划**”图标。 这会在你执行查询时显示执行计划。 选择“执行”以执行查询。

1. 在结果面板中选择“执行计划”选项卡，导航到执行计划。 你会注意到 **SELECT** 运算符有一个黄色三角形，其中包含感叹号。 这表示存在与运算符关联的警告消息。 将鼠标悬停在警告图标上，以查看消息并阅读警告消息。

    > &#128221; 警告消息指出查询中存在隐式转换。 这意味着 SQL Server 查询优化器必须将查询中某一列的数据类型转换为另一个数据类型才能执行查询。

## 确定解决警告信息的方法

*[HumanResources].[Employee]* 表结构如以下数据定义语言 (DDL) 语句所示。 查看上一个 SQL 查询中对此 DDL 使用的字段，注意它们的类型。

```sql
CREATE TABLE [HumanResources].[Employee](
     [BusinessEntityID] [int] NOT NULL,
     [NationalIDNumber] [nvarchar](15) NOT NULL,
     [LoginID] [nvarchar](256) NOT NULL,
     [OrganizationNode] [hierarchyid] NULL,
     [OrganizationLevel] AS ([OrganizationNode].[GetLevel]()),
     [JobTitle] [nvarchar](50) NOT NULL,
     [BirthDate] [date] NOT NULL,
     [MaritalStatus] [nchar](1) NOT NULL,
     [Gender] [nchar](1) NOT NULL,
     [HireDate] [date] NOT NULL,
     [SalariedFlag] [dbo].[Flag] NOT NULL,
     [VacationHours] [smallint] NOT NULL,
     [SickLeaveHours] [smallint] NOT NULL,
     [CurrentFlag] [dbo].[Flag] NOT NULL,
     [rowguid] [uniqueidentifier] ROWGUIDCOL NOT NULL,
     [ModifiedDate] [datetime] NOT NULL
) ON [PRIMARY]
```

1. 根据执行计划中显示的警告消息，会建议进行哪些更改？

    1. 确定引起隐式转换的字段和原因。
    1. 如果查看查询：

        ```sql
        SELECT BusinessEntityID, NationalIDNumber, LoginID, HireDate, JobTitle
        FROM HumanResources.Employee
        WHERE NationalIDNumber = 14417807;
        ```

        你会注意到，与 **WHERE** 子句中的 *NationalIDNumber* 列进行比较的值是作为数字进行比较的，因为 **14417807** 不在带引号的字符串中。

        检查表结构后，你会发现 *NationalIDNumber* 列使用 **NVARCHAR** 数据类型，而不是 **INT** 数据类型。 这种不一致性会导致数据库优化器将数字隐式转换为 *NVARCHAR* 值，从而通过创建次优计划来提高查询性能。

我们可以实现两种方法来修复隐式转换警告。 我们将在接下来的步骤中逐一调查。

### 更改代码

1. 你将如何更改代码来解决隐式转换？ 更改代码并重新运行查询。

    请记得打开“包含实际的执行计划”(Ctrl+M)（如果尚未打开）。 

    在这种情况下，只需在值的每一侧添加一个引号即可将其从数字更改为字符格式。 使此查询的查询窗口保持打开状态。

    运行更新后的 SQL 查询：

    ```sql
    SELECT BusinessEntityID, NationalIDNumber, LoginID, HireDate, JobTitle
    FROM HumanResources.Employee
    WHERE NationalIDNumber = '14417807';
    ```

    > &#128221; 请注意，警告消息现已消失，并且查询计划已得到改进。 通过更改 *WHERE* 子句，使与 *NationalIDNumber* 列相比的值与表中的列的数据类型相匹配，优化器可以消除隐式转换，生成更优计划。

### 更改数据类型

1. 还可以通过更改表结构来修复隐式转换警告。

    若要尝试修复索引，请将以下查询复制并粘贴到新的查询窗口中，以更改列的数据类型。 尝试执行查询，方法是选择“执行”或按 <kbd>F5</kbd>。

    ```sql
    ALTER TABLE [HumanResources].[Employee] ALTER COLUMN [NationalIDNumber] INT NOT NULL;
    ```

    将 NationalIDNumber 列数据类型更改为 INT 将解决转换问题。 但是，此更改引入了数据管理员需要解决的另一个问题。 运行前面的查询会导致出现以下错误消息：

    <span style="color:red">Msg 5074、Level 16、Sate 1、Line1 索引“AK_Employee_NationalIDNumber”依赖于列“NationalIDNumber Msg 4922、Level 16、State 9、Line 1 ALTER TABLE ALTER COLUMN NationalIDNumber 失败，因为一个或多个对象访问此列</span>

    NationalIDNumber 列是已存在的非聚集索引的一部分，必须重新生成/重新创建索引才能更改数据类型。 **这可能会导致生产环境中的停机时间延长，从而凸显在设计中选择正确的数据类型的重要性。**

1. 若要解决此问题，请将下面的代码复制并粘贴到查询窗口中，然后通过选择“执行”来执行它。

    ```sql
    USE AdventureWorks2017

    GO
    
    --Dropping the index first
    DROP INDEX [AK_Employee_NationalIDNumber] ON [HumanResources].[Employee]

    GO

    --Changing the column data type to resolve the implicit conversion warning
    ALTER TABLE [HumanResources].[Employee] ALTER COLUMN [NationalIDNumber] INT NOT NULL;

    GO

    --Recreating the index
    CREATE UNIQUE NONCLUSTERED INDEX [AK_Employee_NationalIDNumber] ON [HumanResources].[Employee]( [NationalIDNumber] ASC );

    GO
    ```

1. 可运行以下查询来确认已成功更改数据类型。

    ```sql
    SELECT c.name, t.name
    FROM sys.all_columns c INNER JOIN sys.types t
        ON (c.system_type_id = t.user_type_id)
    WHERE OBJECT_ID('[HumanResources].[Employee]') = c.object_id
        AND c.name = 'NationalIDNumber'
    ```

1. 现在，让我们检查执行计划。 重新运行不带引号的原始查询。

    ```sql
    USE AdventureWorks2017
    GO

    SELECT BusinessEntityID, NationalIDNumber, LoginID, HireDate, JobTitle
    FROM HumanResources.Employee
    WHERE NationalIDNumber = 14417807;
    ```

     检查查询计划；请注意，现在可使用整数按 NationalIDNumber 进行筛选，而没有隐式转换警告。 现在，SQL 查询优化器可以生成并执行最理想的计划。

>&#128221; 虽然更改列的数据类型可以解决隐式转换问题，但它并非总是最佳解决方案。 在这种情况下，将 *NationalIDNumber* 列的数据类型更改为 **INT** 数据类型将导致生产停机，因为必须删除并重新创建该列上的索引。 在进行任何更改之前，请务必考虑更改列的数据类型对现有查询和索引的影响。 此外，可能还有其他查询依赖于 *NationalIDNumber* 列作为 **NVARCHAR** 数据类型，因此更改数据类型可能会中断这些查询。

---

## 清理

如果不将数据库或实验室文件用于任何其他目的，则可以清理在此实验室中创建的对象。

### 删除 C:\LabFiles 文件夹

1. 如果未提供实验室虚拟机或本地计算机，请打开“**文件资源管理器**”。
1. 导航到 **C:\\**。
1. 删除 **C:\LabFiles** 文件夹。

## 删除 AdventureWorks2017 数据库

1. 如果未提供实验室虚拟机或本地计算机，请启动 SQL Server Management Studio 会话 (SSMS)。
1. SSMS 打开时，默认情况下将显示“**连接到服务器**”对话框。 选择默认实例，然后选择“**连接**”。 可能需要选中“**信任服务器证书**”复选框。
1. 在“**对象资源管理器**”中，展开 **Databases** 文件夹。
1. 右键单击 **AdventureWorks2017** 数据库并选择“**删除**”。
1. 在“**删除对象**”对话框中，选中“**关闭现有连接**”复选框。
1. 选择“确定”****。

---

你已成功完成本实验室。

在本练习中，你学习了如何识别由隐式数据类型转换导致的查询问题，还学习了如何解决该问题来改进查询计划。
