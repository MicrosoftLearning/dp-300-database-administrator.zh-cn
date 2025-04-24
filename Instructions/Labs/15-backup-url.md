---
lab:
  title: 实验室 15 - 备份到 URL 并从 URL 还原
  module: Plan and implement a high availability and disaster recovery solution
---

# 备份到 URL

预计时间：30 分钟

作为 AdventureWorks 的数据库管理员 (DBA)，你需要将数据库备份到 Azure 中的 URL，并在发生人为错误后从 Azure Blob 存储还原它。

## 设置环境

如果实验室虚拟机已提供并预配置，则应在 **C:\LabFiles** 文件夹中找到已准备好的实验室文件。 *花点时间检查，如果文件已存在，请跳过本部分*。 但是，如果使用自己的计算机或缺少实验室文件，则需要从 *GitHub* 克隆它们才能继续。

1. 如果未提供实验室虚拟机或本地计算机，请启动 Visual Studio Code 会话。

1. 打开命令面板 (Ctrl+Shift+P)，然后键入 **Git: Clone**。 选择 **Git: Clone** 选项。

1. 将以下 URL 粘贴到**存储库 URL**字段中，然后选择 **Enter**。

    ```url
    https://github.com/MicrosoftLearning/dp-300-database-administrator.git
    ```

1. 将存储库保存到实验室虚拟机上的 **C:\LabFiles** 文件夹，如果没有提供本地计算机，则保存到本地计算机上的相应文件夹（如果文件夹不存在，则创建该文件夹）。

## 还原  数据库

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

## 配置“备份到 URL”

1. 如果未提供实验室虚拟机或本地计算机，请启动 Visual Studio Code 会话。

1. 打开位于 **C:\LabFiles\dp-300-database-administrator** 的克隆存储库。

1. 右键单击 **Allfiles** 文件夹，然后选择“**在集成终端中打开**”。 这将在正确的位置打开终端窗口。

1. 在终端中，键入以下内容，然后按 **Enter**。

    ```bash
    az login
    ```

1. 系统将提示打开浏览器并输入代码。 按照说明登录 Azure 帐户。

1. *如果你已有资源组，请跳过此步骤*。 如果没有资源组，请在终端执行以下命令创建一个。 将 *contoso-rgXXX######* 替换为资源组的唯一名称。 该名称在 Azure 中必须唯一。 将位置 (-l) 替换为资源组的位置。

    ```bash
    az group create -n "contoso-rglod#######" -l eastus2
    ```

    将 **######** 替换为一些随机字符。

1. 在终端中，键入以下内容，然后按 **Enter** 创建存储帐户。 确保为存储账户使用唯一的名称。 *名称必须为 3 到 24 个字符，并且只能包含数字和小写字母。* 将 *########* 替换为 8 个随机数字字符。 该名称在 Azure 中必须唯一。 将 contoso-rgXXX###### 替换为所需资源组的名称。 最后，将位置 （-l） 替换为资源组的位置。

    ```bash
    az storage account create -n "dp300bckupstrg########" -g "contoso-rgXXX########" --kind StorageV2 -l eastus2
    ```

1. 接下来，获取存储帐户的密钥，你在后续步骤中用到它们。 使用存储帐户和资源组的唯一名称在终端中执行以下代码。

    ```bash
    az storage account keys list -g contoso-rgXXX######## -n dp300bckupstrg########
    ```

    帐户密钥就在上面的命令的结果中。 请确保使用在上一个命令中使用的名称（位于 -n 之后）和资源组（位于 -g 之后）。 复制“**key1**”的返回值（不包含双引号）。

1. 将 SQL Server 中的数据库备份到 URL 会使用存储帐户中的容器。 在这一步中，你将为备份存储专门创建一个容器。 为此，请执行以下命令。

    ```bash
    az storage container create --name "backups" --account-name "dp300bckupstrg########" --account-key "storage_key" --fail-on-exist
    ```

    其中，**dp300bckupstrg########** 是创建存储帐户时使用的唯一存储帐户名称，**storage_key** 是之前生成的密钥。 输出应返回“true”。

1. 若要验证容器备份是否已正确创建，请执行以下代码：

    ```bash
    az storage container list --account-name "dp300bckupstrg########" --account-key "storage_key"
    ```

    其中，**dp300bckupstrg########** 是创建存储帐户时使用的唯一存储帐户名称，**storage_key** 是生成的密钥。

1. 为安全起见，需要容器级别的共享访问签名 (SAS)。 在终端中执行以下命令：

    ```bash
    az storage container generate-sas -n "backups" --account-name "dp300bckupstrg########" --account-key "storage_key" --permissions "rwdl" --expiry "date_in_the_future" -o tsv
    ```

    其中，**dp300bckupstrg########** 是创建存储帐户时使用的唯一存储帐户名称，**storage_key** 是生成的密钥，**date_in_the_future** 是比现在晚的时间。 date_in_the_future 必须为协调世界时 (UTC)。 例如 **2025-12-31T00:00Z**，表示在 2025 年 12 月 31 日午夜过期。

    输出应返回与以下类似的内容。 复制整个共享访问签名，将它粘贴到记事本中，在下一个任务中会用到它。

    *se=2020-12-31T00%3A00Z&sp=rwdl&sv=2018-11-09&sr=c&sig=rnoGlveGql7ILhziyKYUPBq5ltGc/pzqOCNX5rrLdRQ%3D*

## 创建凭据

现在功能已配置，你可以将备份文件生成为 Azure 存储帐户中的 Blob。

1. 启动 SQL Server Management Studio (SSMS)。

1. 系统将提示你连接到 SQL Server。 确保选中“Windows 身份验证”，然后选择“连接” 。

1. 选择“新建查询”  。

1. 使用以下 Transact-SQL 语句创建将用于访问云中的存储的凭据。 填写相应的值，然后选择“执行”。

    ```sql
    IF NOT EXISTS  
    (SELECT * 
        FROM sys.credentials  
        WHERE name = 'https://<storage_account_name>.blob.core.windows.net/backups')  
    BEGIN
        CREATE CREDENTIAL [https://<storage_account_name>.blob.core.windows.net/backups]
        WITH IDENTITY = 'SHARED ACCESS SIGNATURE',
        SECRET = '<key_value>'
    END;
    GO  
    ```

    其中，**<storage_account_name>** 的两个匹配项都是创建的唯一存储帐户名称，**<key_value>** 是在上一个任务结束时生成的值，类似于下面的内容：

    *se=2020-12-31T00%3A00Z&sp=rwdl&sv=2018-11-09&sr=c&sig=rnoGlveGql7ILhziyKYUPBq5ltGc/pzqOCNX5rrLdRQ%3D*

1. 可以在 SSMS 的“对象探索”中导航到 **“安全性”->“凭据”**，检查是否成功创建了凭据。

1. 如果你键入错误，需要重新创建凭据，你可以通过以下命令将其删除，确保更改存储帐户的名称：

    ```sql
    -- Only run this command if you need to go back and recreate the credential! 
    DROP CREDENTIAL [https://<storage_account_name>.blob.core.windows.net/backups]  
    ```

## 将数据库备份到 URL

1. 使用 SSMS，用 Transact-SQL 中的以下命令将数据库 **AdventureWorks2017** 备份到 Azure：

    ```sql
    BACKUP DATABASE AdventureWorks2017   
    TO URL = 'https://<storage_account_name>.blob.core.windows.net/backups/AdventureWorks2017.bak';
    GO 
    ```

    其中，<storage_account_name> 是创建和使用的唯一存储帐户名称。 

    如果发生错误，请检查在凭据创建过程中是否键入了错误的内容，以及所有内容是否已成功创建。

## 通过 Azure CLI 验证备份

要确保文件确实在 Azure 中，可以使用存储资源管理器（预览版）或 Azure Cloud Shell。

1. 回到 Visual Studio Code 终端，运行此 Azure CLI 命令：

    ```bash
    az storage blob list -c "backups" --account-name "dp300bckupstrg########" --account-key "storage_key" --output table
    ```

    请确保使用在前面命令中使用的相同的唯一存储帐户名称（在 --account-name 之后）和帐户密钥（在 --account-key 之后） 。

    我们可以确认已成功生成备份文件。

## 通过存储浏览器验证备份

1. 在浏览器窗口中，转到 Azure 门户，搜索并选择“**存储帐户**”。

1. 选择为备份创建的唯一存储帐户名称。

1. 在左侧导航栏中，选择“**存储浏览器**”。 展开“Blob 容器”。

1. 选择“备份”。

1. 请注意，备份文件存储在容器中。

## 从 URL 还原

此任务将演示如何从 Azure Blob 存储还原数据库。

1. 从 SQL Server Management Studio (SSMS) 中，选择“新建查询”，然后粘贴并执行以下查询 。

    ```sql
    USE AdventureWorks2017;
    GO
    SELECT * FROM Person.Address WHERE AddressId = 1;
    GO
    ```

1. 运行此命令更改该客户的地址。

    ```sql
    UPDATE Person.Address
    SET AddressLine1 = 'This is a human error'
    WHERE AddressId = 1;
    GO
    ```

1. 重新运行步骤 1，验证地址是否已更改。 现在请设想，如果有人在没有 WHERE 子句或有错误 WHERE 子句的情况下更改了数千或数百万行。 其中一个解决方案涉及到从上次可用备份还原数据库。

1. 若要还原数据库以使其返回到错误更改客户名称之前的位置，请执行以下命令。

    > &#128221; **SET SINGLE_USER WITH ROLLBACK IMMEDIATE 语法**，打开的事务将全部回滚。 这可以防止由于活动连接而导致还原失败。

    ```sql
    USE [master]
    GO

    ALTER DATABASE AdventureWorks2017 SET SINGLE_USER WITH ROLLBACK IMMEDIATE
    GO

    RESTORE DATABASE AdventureWorks2017 
    FROM URL = 'https://<storage_account_name>.blob.core.windows.net/backups/AdventureWorks2017.bak'
    GO

    ALTER DATABASE AdventureWorks2017 SET MULTI_USER
    GO
    ```

    其中，<storage_account_name> 是创建的唯一存储帐户名称。

1. 重新运行步骤 1，验证客户名称是否已还原。

请务必了解组件和交互，以备份到 Azure Blob 存储服务或从中进行还原。

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

你现在已看到，你可以在 Azure 中将数据库备份到 URL，并根据需要还原。
