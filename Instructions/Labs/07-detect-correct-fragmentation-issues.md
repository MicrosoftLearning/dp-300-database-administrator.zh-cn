---
lab:
  title: 实验室 7 - 检测并纠正碎片问题
  module: Monitor and optimize operational resources in Azure SQL
---

# 检测并纠正碎片问题

预计用时：20 分钟

学生将获取从课程中获得的信息，以确定 AdventureWorks 中数字转换项目的可交付成果。 通过检查 Azure 门户以及其他工具，学生将确定如何利用本机工具来识别和解决与性能相关的问题。 最后，学生将能够识别数据库中的碎片，并学习恰当解决问题的步骤。

你是数据库管理员，需识别与性能相关的问题并提供可行的解决方案来解决发现的所有问题。 十多年来，AdventureWorks 一直将自行车和自行车零件直接销售给最终用户和分销商。 最近，该公司注意到其用于满足客户要求的产品性能下降了。 你需要使用 SQL 工具来确定性能问题并提出解决这些问题的方法。

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

## 调查索引碎片

1. 选择“新建查询”  。 复制下面的 T-SQL 代码并将其粘贴到“查询”窗口中。 选择“执行”以执行此查询。

    ```sql
    USE AdventureWorks2017

    GO
    
    SELECT i.name Index_Name
     , avg_fragmentation_in_percent
     , db_name(database_id)
     , i.object_id
     , i.index_id
     , index_type_desc
    FROM sys.dm_db_index_physical_stats(db_id('AdventureWorks2017'),object_id('person.address'),NULL,NULL,'DETAILED') ps
     INNER JOIN sys.indexes i ON ps.object_id = i.object_id 
     AND ps.index_id = i.index_id
    WHERE avg_fragmentation_in_percent > 50 -- find indexes where fragmentation is greater than 50%
    ```

    此查询将报告碎片率超过 50% 的任何索引。 查询不应返回任何结果。

1. 索引碎片可能是由多种因素引起的，包含以下因素：

    - 频繁更新表或索引。
    - 经常插入或删除表或索引。
    - 页拆分。

    要提高 Person.Address 表及其索引的碎片级别，需要插入和删除大量记录。 为此，请运行以下查询。

    选择“新建查询”  。 复制下面的 T-SQL 代码并将其粘贴到“查询”窗口中。 选择“执行”以执行此查询。

    ```sql
    USE AdventureWorks2017

    GO
    
    -- Insert 60000 records into the Address table    

    INSERT INTO [Person].[Address] 
        ([AddressLine1], [AddressLine2], [City], [StateProvinceID], [PostalCode], [SpatialLocation], [rowguid], [ModifiedDate])
    SELECT 
        'Split Avenue ' + CAST(v1.number AS VARCHAR(10)), 
        'Apt ' + CAST(v2.number AS VARCHAR(10)), 
        'PageSplitTown', 
        100 + (v1.number % 60),  -- 60 different StateProvinceIDs (100-159)
        '88' + RIGHT('000' + CAST(v2.number AS VARCHAR(3)), 3), -- Structured postal codes
        NULL, 
        NEWID(), -- Ensure unique rowguid
        GETDATE()
    FROM master.dbo.spt_values v1
    CROSS JOIN master.dbo.spt_values v2
    WHERE v1.type = 'P' AND v1.number BETWEEN 1 AND 300 
    AND v2.type = 'P' AND v2.number BETWEEN 1 AND 200;
    GO
    
    -- DELETE 25000 records from the Address table
    DELETE FROM [Person].[Address] WHERE AddressID BETWEEN 35001 AND 60000;

    GO

    -- Insert 40000 records into the Address table
    INSERT INTO [Person].[Address] 
        ([AddressLine1], [AddressLine2], [City], [StateProvinceID], [PostalCode], [SpatialLocation], [rowguid], [ModifiedDate])
    SELECT 
        'Fragmented Street ' + CAST(v1.number AS VARCHAR(10)), 
        'Suite ' + CAST(v2.number AS VARCHAR(10)), 
        'FragmentCity', 
        100 + (v1.number % 60),  -- 60 different StateProvinceIDs (100-159)
        '99' + RIGHT('000' + CAST(v2.number AS VARCHAR(3)), 3), -- Structured postal codes
        NULL, 
        NEWID(), -- Ensure a unique rowguid per row
        GETDATE()
    FROM master.dbo.spt_values v1
    CROSS JOIN master.dbo.spt_values v2
    WHERE v1.type = 'P' AND v1.number BETWEEN 1 AND 200 
    AND v2.type = 'P' AND v2.number BETWEEN 1 AND 200;

    GO
    ```

    通过添加和删除大量记录，此查询将增加 Person.Address 表及其索引的碎片级别。

1. 再次执行第一个查询。 现在，你应该能够看到 4 个碎片程度高的索引。

1. 选择“**新建查询**”，然后将以下 T-SQL 代码复制并粘贴到查询窗口中。 选择“执行”以执行此查询。

    ```sql
    SET STATISTICS IO,TIME ON

    GO
        
    USE AdventureWorks2017

    GO
        
    SELECT DISTINCT (StateProvinceID)
        ,count(StateProvinceID) AS CustomerCount
    FROM person.Address
    GROUP BY StateProvinceID
    ORDER BY count(StateProvinceID) DESC;
        
    GO
    ```

    选择 SQL Server Management Studio 的“结果”窗格中的“**消息**”选项卡。 记下查询在 **Address** 表上执行的逻辑读取次数。

## 重新生成碎片索引

1. 选择“**新建查询**”，然后将以下 T-SQL 代码复制并粘贴到查询窗口中。 选择“执行”以执行此查询。

    ```sql
    USE AdventureWorks2017

    GO
    
    ALTER INDEX [IX_Address_StateProvinceID] ON [Person].[Address] REBUILD PARTITION = ALL 
    WITH (PAD_INDEX = OFF, 
        STATISTICS_NORECOMPUTE = OFF, 
        SORT_IN_TEMPDB = OFF, 
        IGNORE_DUP_KEY = OFF, 
        ONLINE = OFF, 
        ALLOW_ROW_LOCKS = ON, 
        ALLOW_PAGE_LOCKS = ON)
    ```

1. 选择“**新建查询**”并执行以下查询，以确认 **IX_Address_StateProvinceID** 索引的碎片率不再超过 50%。

    ```sql
    USE AdventureWorks2017

    GO
        
    SELECT DISTINCT i.name Index_Name
        , avg_fragmentation_in_percent
        , db_name(database_id)
        , i.object_id
        , i.index_id
        , index_type_desc
    FROM sys.dm_db_index_physical_stats(db_id('AdventureWorks2017'),object_id('person.address'),NULL,NULL,'DETAILED') ps
        INNER JOIN sys.indexes i ON (ps.object_id = i.object_id AND ps.index_id = i.index_id)
    WHERE i.name = 'IX_Address_StateProvinceID'
    ```

    比较结果，我们可以看到 **IX_Address_StateProvinceI** 碎片率从 88% 下降到了 0。

1. 重新执行上一部分中的 select 语句。 记下 Management Studio 中“结果”窗格的“消息”选项卡中的逻辑读取次数。 *重新生成 address 表的索引之前，遇到的逻辑读取次数是否有变化*？

    ```sql
    SET STATISTICS IO,TIME ON

    GO
        
    USE AdventureWorks2017

    GO
        
    SELECT DISTINCT (StateProvinceID)
        ,count(StateProvinceID) AS CustomerCount
    FROM person.Address
    GROUP BY StateProvinceID
    ORDER BY count(StateProvinceID) DESC;
        
    GO
    ```

由于索引已重新生成，所以它现在会尽可能高效，并且逻辑读取次数应该会减少。 现在，你已了解到索引维护可能会影响查询性能。

---

## 清理

如果不将数据库或实验室文件用于任何其他目的，则可以清理在此实验室中创建的对象。

### 删除 C:\LabFiles 文件夹

1. 如果未提供实验室虚拟机或本地计算机，请打开“**文件资源管理器**”。
1. 导航到 **C:\\**。
1. 删除 **C:\LabFiles** 文件夹。

### 删除 AdventureWorks2017 数据库

1. 如果未提供实验室虚拟机或本地计算机，请启动 SQL Server Management Studio 会话 (SSMS)。
1. SSMS 打开时，默认情况下将显示“**连接到服务器**”对话框。 选择默认实例，然后选择“**连接**”。 可能需要选中“**信任服务器证书**”复选框。
1. 在“**对象资源管理器**”中，展开 **Databases** 文件夹。
1. 右键单击 **AdventureWorks2017** 数据库并选择“**删除**”。
1. 在“**删除对象**”对话框中，选中“**关闭现有连接**”复选框。
1. 选择“确定”****。

---

你已成功完成本实验室。

在本练习中，你学习了如何重新生成索引和分析逻辑读取来提高查询性能。
