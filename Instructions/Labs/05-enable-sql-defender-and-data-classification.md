---
lab:
  title: 实验室 5 - 启用 Microsoft Defender for SQL 和数据分类
  module: Implement a Secure Environment for a Database Service
---

# 启用 Microsoft Defender for SQL 和数据分类

预计时间：30 分钟

学生将利用从课程中获取的信息，在 Azure 门户和 AdventureWorks 数据库中进行配置，随后实现安全性。

你已被聘为高级数据库管理员，帮助确保数据库环境的安全。 这些任务侧重于 Azure SQL 数据库。

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

## 启用 Microsoft Defender for SQL

1. 如果未提供实验室虚拟机或本地计算机，请启动浏览器会话并导航到 [https://portal.azure.com](https://portal.azure.com/)。 使用 Azure 凭据登录到门户。

1. 在 Azure 门户顶部的搜索框中搜索“*SQL servers*”，然后从选项列表中选择“**SQL Server**”。

1. 选择 SQL Server **dp300-lab-xxxxxxxx**，其中 *xxxxxxxx* 是随机数值字符串。

    > &#128221; 请注意，如果使用此实验室未创建自己的 Azure SQL Server，请选择该 SQL Server 的名称。

1. 在“*概述*”边栏选项卡中，选择 *Microsoft Defender for SQL* 旁边的“**未配置**”。

1. 选择右上角的 **X** 以关闭 *Microsoft Defender for Cloud*“概述”窗格。

1. 在 *Microsoft Defender for SQL* 下选择“**启用**”。

1. 在生产环境中，应列出多个建议。 需要选择“**查看 Defender for Cloud 中的所有建议**”，然后查看针对 Azure SQL Server 列出的所有 *Microsoft Defender* 建议，并根据情况实现这些建议。

## 漏洞评估

1. 在 Azure SQL Server 的主边栏选项卡中，导航到“**设置**”部分，选择“**SQL 数据库**”，然后选择数据库名称 **AdventureWorksLT**。

1. 选择“**安全性**”下的 **Microsoft Defender for Cloud** 设置。

1. 选择右上角的 **X** 以关闭“*Microsoft Defender for Cloud*“概述”窗格，并查看 `AdventureWorksLT` 数据库的 **Microsoft Defender for Cloud** 仪表板。

1. 若要开始查看漏洞评估功能，请在“漏洞评估结果”下，选择“查看漏洞评估的其他结果”。  

1. 选择“扫描”以获取最新的漏洞评估结果。 此过程将需要一些时间，且漏洞评估会扫描数据库。

1. 每个安全风险都有风险级别（高、中或低）和其他信息。 现行规则基于 [Internet 安全中心](https://www.cisecurity.org/benchmark/microsoft_sql_server/?azure-portal=true)提供的基准。 在“结果”选项卡中，选择一个漏洞****。 记下漏洞的 **ID**，例如 **VA1143** （如果已列出）。

1. 根据安全检查，将提供备选视图和建议。 查看所提供的信息。 对于此安全检查，可以选择“将所有结果添加为基线”按钮，然后选择“是”以设置基线。 我们现在已拥有基线，此安全检查将在结果与基线不同的所有将来扫描中失败。 选择右上角的 X 关闭特定规则的窗格****。  

1. 让我们再次运行”**扫描**“，确认所选漏洞现在显示为”*已通过*“安全性检查。

    如果选择前面已通过的安全检查，应该能够看到配置的基线。 如果将来发生任何更改，则漏洞评估扫描会选取它，且安全检查将失败。  

## 高级威胁防护

1. 选择右上角的“X”以关闭“漏洞评估”窗格并返回到数据库的“Microsoft Defender for Cloud”仪表板。 在“安全事件和警报”下，不应显示任何项。 这意味着“高级威胁防护”未检测到任何问题。 高级威胁防护检测异常活动，指示尝试访问或利用数据库的行为异常且可能有害。  

    > &#128221; 在此阶段不会看到任何安全警报。 在下一步中，将运行会触发警报的测试，因此可以在高级威胁防护中查看结果。  

    高级威胁防护可用于识别威胁，并在怀疑发生以下任何事件时发出警报：  

    - SQL 注入
    - SQL 注入漏洞
    - 数据外泄
    - 不安全操作
    - 暴力攻击
    - 异常客户端登录

    在本部分中，你将了解如何通过 SSMS 触发 SQL 注入警报。 SQL 注入警报适用于自定义编写的应用程序，而不适用于标准工具（例如 SSMS）。 因此，要通过 SSMS 触发警报来进行 SQL 注入测试，你需要“设置”“应用程序名称”，这是连接到 SQL Server 或 Azure SQL 的客户端的连接属性。

1. 如果未提供实验室虚拟机或本地计算机，请打开 SQL Server Management Studio （SSMS）。 在“连接到服务器”对话框中，粘贴 Azure SQL 数据库服务器的名称，然后使用以下凭据登录：

    - 服务器名称：&lt;在此处粘贴你的 Azure SQL 数据库服务器名称&gt;
    - 身份验证：SQL Server 身份验证
    - **服务器管理员登录：** Azure SQL 数据库服务器管理员登录
    - **密码：** Azure SQL 数据库服务器管理员密码

1. 选择“连接” 。

1. 在 SSMS 中，选择“文件” > “新建” > “数据库引擎查询”，以使用新连接创建查询。  

1. 在主登录窗口中，像往常一样，使用 SQL 身份验证和 Azure SQL Server 名称和管理员凭据登录到 **AdventureWorksLT** 数据库。 在连接之前，请选择“**选项 >>**” > “**连接属性**”。 对于“**连接到数据库**”选项，键入“**AdventureWorksLT**”。  

1. 选择“其他连接参数”选项卡，然后在文本框中插入以下连接字符串：  

    ```sql
    Application Name=webappname
    ```

1. 选择“连接”。  

1. 在新查询窗口中粘贴以下代码，然后选择“执行”****：  

    ```sql
    SELECT * FROM sys.databases WHERE database_id like '' or 1 = 1 --' and family = 'test1';
    ```

1. 在 Azure 门户中，转到 **AdventureWorksLT** 数据库。 在左侧窗格的“安全性”下，选择“Microsoft Defender for Cloud”。

1. 在“**安全事件和警报**”下，选择“**在 Microsoft Defender for Cloud 中检查此资源的警报**”。  

1. 现在可以看到总体安全警报。  

1. 选择“潜在 SQL 注入”以显示更具体的警报并接收调查步骤。

1. 选择“**查看完整详细信息**”以显示警报的详细信息。

1. 在“**警报详细信息**”选项卡下，请注意，将显示“*易受攻击*”语句。 这是为了触发警报而执行的 SQL 语句。 这也是在 SSMS 中执行的 SQL 语句。 此外，请注意，**客户端应用程序**显示为 **webappname**。 这是你在 SSMS 中连接字符串中指定的名称。

1. 作为清理步骤，考虑在 SSMS 中关闭所有查询编辑器并移除所有连接，以免在下一练习中意外触发其他警报。

## 启用数据分类

1. 在 Azure SQL Server 的主边栏选项卡中，导航到“**设置**”部分，选择“**SQL 数据库**”，然后选择数据库名称 **AdventureWorksLT**。

1. 在 AdventureWorksLT 数据库的主边栏选项卡上，导航到“安全性”部分，然后选择“数据发现和分类”。

1. 在“数据发现和分类”屏幕上，你会看见一条信息性消息：“当前通过 SQL 信息保护策略，我们发现了 15 列分类建议”。 选择此链接。

1. 在下一个“数据发现和分类”屏幕中，选中“全选”旁的复选框，然后选择“接受所选建议”，最后选择“保存”以将分类保存到数据库中。

1. 返回到“数据发现和分类”屏幕，请注意，已将 15 列成功分类到在 5 个不同的表中。 查看每一列的“*信息类型*”和“*敏感度标签*”。

## 配置数据分类和掩码

1. 在 Azure 门户中，转到 **AdventureWorksLT** Azure SQL 数据库实例（而非逻辑服务器）。

1. 在左侧窗格的“安全性”**** 下，选择“数据发现和分类”****。  

1. 在“SalesLT Customer”表中，*数据发现和分类*标识了要分类的 `FirstName` 和 `LastName`，但没有标识 `MiddleName`。 使用下拉列表立即添加它。 为“*信息类型*”和“**机密- GDPR**”选择 *“敏感度”标签*的“**名称**”，然后选择“**添加分类**”。  

1. 选择“保存”。

1. 通过查看“概述”选项卡确认已成功添加分类，并确认 `MiddleName` 现已显示在 SalesLT 架构下的分类列列表中。

1. 在左侧窗格中选择“概述”，返回到数据库概述。  

   动态数据掩码 (DDM) 在 Azure SQL 和 SQL Server 中均可用。 DDM 通过在 SQL Server 级别（而不是在必须为这些类型的规则编写代码的应用程序级别）对非特权用户屏蔽敏感数据来限制数据泄露。 Azure SQL 将为你提供屏蔽项建议，你也可以手动添加掩码。

   在下一步中，你将屏蔽在上一步中查看的 `FirstName`、`MiddleName` 和 `LastName` 列。  

1. 在 Azure 门户中，转到 Azure SQL 数据库。 在左侧窗格的“**安全性**”下，选择“**动态数据掩码**”，然后选择“**添加掩码**”。  

1. 在下拉列表中，选择“**SalesLT**”架构、“**Customer**”表和“**FirstName**”列。 可以查看掩码选项，但默认选项即可满足此方案的要求。 选择“添加”以添加掩码规则。  

1. 对表中的“MiddleName”和“LastName”重复前面的步骤。  

    现在有三个掩码规则。  

1. 选择“保存”。

    > &#128221; 请注意，如果 Azure SQL Server 名称不由小写字母、数字和短划线组成，则此步骤将失败，并且无法继续执行数据掩码部分。

1. 在左侧窗格中选择“概述”，返回到数据库概述。

## 检索已分类和屏蔽的数据

接下来，你将模拟查询分类列，并通过实际操作探索动态数据掩码。

1. 转到 SQL Server Management Studio (SSMS)，连接到 Azure SQL Server 实例，打开新的查询窗口。

1. 右键单击数据库 **AdventureWorksLT** 并选择“**新建查询**”。  

1. 运行以下查询来返回分类数据（在某些情况下，标记进行屏蔽数据的列）。 选择“**执行**”以运行查询。

    ```sql
    SELECT TOP 10 FirstName, MiddleName, LastName
    FROM SalesLT.Customer;
    ```

    所得结果应显示前 10 个姓名，并且未应用任何掩码。 为什么？ 因为你是此 Azure SQL 数据库逻辑服务器的管理员。  

1. 在以下查询中，你将创建一个新用户并以该用户的身份运行之前的查询。 你还将使用 `EXECUTE AS` 来模拟 `Bob`。 运行 `EXECUTE AS` 语句时，会话的执行上下文会切换到登录名或用户。 这意味着将根据登录名或用户（而不是执行 `EXECUTE AS` 命令的人员，在本例中为你自己）检查权限。 然后使用 `REVERT` 停止模拟登录名或用户。  

    你可能认识后面命令的前几个部分，因为它们是上一练习的重复。 使用以下命令创建新查询，然后选择“**执行**”以运行查询并观察结果。

    ```sql
    -- Create a new SQL user and give them a password
    CREATE USER Bob WITH PASSWORD = 'c0mpl3xPassword!';

    -- Until you run the following two lines, Bob has no access to read or write data
    ALTER ROLE db_datareader ADD MEMBER Bob;
    ALTER ROLE db_datawriter ADD MEMBER Bob;

    -- Execute as our new, low-privilege user, Bob
    EXECUTE AS USER = 'Bob';
    SELECT TOP 10 FirstName, MiddleName, LastName
    FROM SalesLT.Customer;
    REVERT;
    ```

    结果现在应显示前 10 个姓名，并且未应用任何掩码。 没有授予 Bob 访问此数据的非掩码形式的权限。  

    如果出于某些原因，Bob 需要访问姓名并获取拥有此类访问权限的许可怎么办？  

    可以通过转到“安全性”下的“动态数据掩码”窗格，在 Azure 门户中更新从屏蔽中排除的用户，但也可以使用 T-SQL 执行此操作。

1. 右键单击 **AdventureWorksLT** 数据库并选择“**新建查询**”，然后输入以下查询以允许 Bob 在不屏蔽的情况下查询名称结果。 选择“**执行**”以运行查询。

    ```sql
    GRANT UNMASK TO Bob;  
    EXECUTE AS USER = 'Bob';
    SELECT TOP 10 FirstName, MiddleName, LastName
    FROM SalesLT.Customer;
    REVERT;  
    ```

    结果应包括全名。  

1. 你还可以删除用户的取消屏蔽特权，并通过在新查询中运行以下 T-SQL 命令来确认此操作：  

    ```sql
    -- Remove unmasking privilege
    REVOKE UNMASK TO Bob;  

    -- Execute as Bob
    EXECUTE AS USER = 'Bob';
    SELECT TOP 10 FirstName, MiddleName, LastName
    FROM SalesLT.Customer;
    REVERT;  
    ```

    结果应包括已屏蔽的姓名。  

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

在本练习中，你通过启用 Microsoft Defender for SQL 增强了 Azure SQL 数据库的安全性。 你基于 Azure 门户建议创建了分类列。
