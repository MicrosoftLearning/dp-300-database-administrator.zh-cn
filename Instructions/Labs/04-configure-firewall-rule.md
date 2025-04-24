---
lab:
  title: 实验室 4 - 配置 Azure SQL 数据库防火墙规则
  module: Implement a Secure Environment for a Database Service
---

# 实现安全环境

预计时间：30 分钟

学生将运用在课程中学到的信息，在 Azure 门户和 *AdventureWorksLT* 数据库中进行配置，再随后实现安全性。

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

## 配置 Azure SQL 数据库防火墙规则

1. 如果未提供实验室虚拟机或本地计算机，请启动浏览器会话并导航到 [https://portal.azure.com](https://portal.azure.com/)。 使用 Azure 凭据登录到门户。

1. 在 Azure 门户顶部的搜索框中搜索“*SQL servers*”，然后从选项列表中选择“**SQL Server**”。

1. 选择 SQL Server **dp300-lab-xxxxxxxx**，其中 *xxxxxxxx* 是随机数值字符串。

    > &#128221; 请注意，如果使用此实验室未创建自己的 Azure SQL Server，请选择该 SQL Server 的名称。

1. 在SQL Server 的“*概述*”屏幕中，在服务器名称的右侧，选择“**复制到剪贴板**”按钮。

1. 选择“显示网络设置”。

1. 在“**网络**”页上的“**防火墙规则**”下，查看列表并确保列出客户端 IP 地址。 如果未列出，请选择 **“+ 添加客户端 IPv4 地址”（IP 地址）**，然后选择“**保存**”。

    > 请注意，已为你自动输入客户端 IP 地址。 通过将客户端 IP 地址添加到列表，可使用 SQL Server Management Studio (SSMS) 或其他任何客户端工具连接到 Azure SQL 数据库。 **记下你的客户端 IP 地址，稍后将会用到它。**

1. 打开 SQL Server Management Studio。 在“连接到服务器”对话框中，粘贴 Azure SQL 数据库服务器的名称，然后使用以下凭据登录：

    - 服务器名称：&lt;在此处粘贴你的 Azure SQL 数据库服务器名称&gt;
    - 身份验证：SQL Server 身份验证
    - **服务器管理员登录：** Azure SQL 数据库服务器管理员登录
    - **密码：** Azure SQL 数据库服务器管理员密码

1. 选择“连接” 。

1. 在对象资源管理器中，展开服务器节点，然后右键单击“数据库”。 选择“**导入数据层应用程序**”。

1. 在“导入数据层应用程序”对话框中，单击第一个屏幕上的“下一步”。

1. 在“**导入设置**”屏幕中，单击“**浏览**”并导航至 **C:\LabFiles\dp-300-database-administrator\Allfiles\Labs\04** 文件夹，单击 **AdventureWorksLT.bacpac** 文件，然后单击“**打开**”。 返回到“**导入数据层应用程序**”屏幕，单击“**下一步**”。

1. 在“数据库设置”屏幕上，进行如下更改：

    - 数据库名称：AdventureWorksFromBacpac
    - Microsoft Azure SQL 数据库的版本：基本

1. 选择**下一步**。

1. 在“摘要”屏幕上选择“完成”。******** 这可能需要几分钟的时间。 导入完成后，你将看到以下结果。 然后选择“**关闭**”。

1. 返回到 SQL Server Management Studio，在对象资源管理器中，展开“数据库”文件夹。 然后右键单击 AdventureWorksFromBacpac 数据库，再单击“新建查询”。

1. 通过将文本粘贴到查询窗口中来执行以下 T-SQL 查询。
    1. **重要提示：** 请将 **000.000.000.000** 替换为客户端 IP 地址。 选择“执行”。

    ```sql
    EXECUTE sp_set_database_firewall_rule 
            @name = N'AWFirewallRule',
            @start_ip_address = '000.000.000.000', 
            @end_ip_address = '000.000.000.000'
    ```

1. 接下来，将在 AdventureWorksFromBacpac 数据库中创建一个包含的用户。 选择“新建查询”并执行以下 T-SQL。****

    ```sql
    USE [AdventureWorksFromBacpac]
    GO
    CREATE USER ContainedDemo WITH PASSWORD = 'P@ssw0rd01'
    ```

    > &#128221; 此命令会在 **AdventureWorksFromBacpac** 数据库中创建一个包含的用户。 我们将在下一步中测试此凭据。

1. 导航到对象资源管理器。 依次单击“连接”和“数据库引擎”。

1. 使用你在之前步骤中创建的凭据尝试连接。 你将需要使用以下信息：

    - 登录名：ContainedDemo
    - 密码：P@ssw0rd01

     单击“连接” 。

     你会收到以下错误。

    <span style="color:red">用户“ContainedDemo”登录失败。（Microsoft SQL Server，错误：18456）</span>

    > &#128221; 生成此错误是因为该连接尝试登录 *master* 数据库，而不是创建用户的 **AdventureWorksFromBacpac**。 选择“**确定**”以退出错误消息，然后在“**连接到服务器**”中选择 **“选项”>>**，以更改连接上下文。

1. 在“**连接属性**”选项卡上，键入数据库名称 **AdventureWorksFromBacpac**，然后选择“**连接**”。

1. 请注意，你能够使用 ContainedDemo 用户成功进行身份验证。 这次，你直接登录到 AdventureWorksFromBacpac，它是新创建的用户有权访问的唯一数据库。

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

在本练习中，你已将服务器和数据库防火墙规则配置为访问在 Azure SQL 数据库上托管的数据库。 你还使用 T-SQL 语句创建了一个包含的用户，还使用 SQL Server Management Studio 检查了访问权限。
