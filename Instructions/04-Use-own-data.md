---
lab:
  title: 基于自有数据构建生成式 AI 应用
  description: 了解如何使用检索增强生成 (RAG) 模型，构建基于自有数据优化提示的聊天应用。
---

# 基于自有数据构建生成式 AI 应用

检索增强生成 (RAG) 是一种用于生成应用程序的技术，这些应用程序将来自自定义数据源的数据集成到生成式 AI 模型的提示中。 RAG 是开发生成式 AI 应用（基于聊天的应用程序，使用语言模型来解释输入并生成适当的响应）的常用模式。

在本练习中，你将使用 Azure AI Foundry 门户以及 Azure AI Foundry 和 Azure OpenAI SDK 将自定义数据集成到生成式 AI 应用中。

本练习大约需要 **45** 分钟。

> **备注**：本练习基于预发布 SDK，可能会有更改。 必要时，我们使用了特定版本的包；这可能没有反映最新的可用版本。

## 创建 Azure AI Foundry 项目

首先，创建一个 Azure AI Foundry 项目及其所需的服务资源，以便使用自己的数据（包括 Azure AI 搜索资源）提供支持。

1. 在 Web 浏览器中打开 [Azure AI Foundry 门户](https://ai.azure.com)，网址为：`https://ai.azure.com`，然后使用 Azure 凭据登录。 关闭首次登录时打开的任何使用技巧或快速入门窗格，如有必要，请使用左上角的 **Azure AI Foundry** 徽标导航到主页，如下图所示：

    ![Azure AI Foundry 门户的屏幕截图。](./media/ai-foundry-home.png)

1. 在主页中，选择“**+ 创建项目**”。
1. 在“**创建项目**”向导中，输入合适的项目名称（例如，`my-ai-project`），如果建议使用现有中心，请选择创建新中心的选项。 然后查看将自动创建的 Azure 资源以支持中心和项目。
1. 选择“**自定义**”并为中心指定以下设置：
    - **中心名称**：*唯一名称 - 例如`my-ai-hub`*
    - **订阅**：Azure 订阅
    - **资源组**：*新建资源组并提供唯一名称（例如 `my-ai-resources`），或选择现有资源组*
    - **位置**：选择“**帮助我选择**”，然后在“位置帮助程序”窗口中选择“**gpt-4**”和“**text-embedding-ada-002**”，并使用推荐的区域\*
    - **连接 Azure AI 服务或 Azure OpenAI**：*新建 AI 服务资源并提供适当的名称（例如 `my-ai-services`）或使用现有资源*
    - **连接 Azure AI 搜索**：*使用唯一名称创建新的 Azure AI 搜索资源*

    > \* Azure OpenAI 资源受区域配额限制在租户级别。 如果达到配额限制，并且针对这两个模型都不建议使用任何区域，请仅选择其中一个区域并使用建议的区域。 在稍后的练习中，你将在不同区域为第二个模型创建另一个资源。

1. 选择“**下一步**”查看配置。 然后，选择“**创建**”并等待该进程完成。
1. 创建项目后，关闭显示的所有使用技巧，并查看 Azure AI Foundry 门户中的“**概览**”页，与下图类似：

    ![Azure AI Foundry 门户中 Azure AI 项目详细信息的屏幕截图。](./media/ai-foundry-project.png)
   
## 部署模型

需要两个模型来实现解决方案：

- 一个嵌入** 模型，用于矢量化文本数据以实现高效索引和处理。
- 一个可以根据数据生成对问题的自然语言回答的模型。

1. 在 Azure AI Foundry 门户的项目中，在左侧导航窗格“**我的资产**”下，选择“**模型 + 终结点**”页。
1. 在“部署模型”向导中选择“**自定义**”，并使用以下设置新建 **text-embedding-ada-002** 模型部署：

    - **部署名称**：`text-embedding-ada-002`
    - **部署类型**：标准
    - **模型版本**：*选择默认版本*
    - **AI 资源**：*选择先前创建的资源*
    - **每分钟令牌数速率限制（数千个）**：5K
    - **内容筛选器**：DefaultV2
    - **启用动态配额**：已禁用

    > **备注**：如果当前 AI 资源位置没有可用于要部署模型的配额，系统会要求你选择其他位置，以便新建 AI 资源并连接到项目。

1. 重复上述步骤，使用部署名称`gpt-4`部署 **GPT-4** 模型。

    > **注意**：减少每分钟令牌数 (TPM) 有助于避免正在使用的订阅中可用配额的过度使用。 对于本练习中使用的数据，5,000 TPM 已经足够。

## 将数据添加到项目

Copilot 的数据包括一组 PDF 格式的旅行手册，来自虚构的旅行社 Margie's Travel**。 让我们将它们添加到项目。

1. 从 `https://github.com/MicrosoftLearning/mslearn-ai-studio/raw/main/data/brochures.zip` 下载[手册的压缩存档](https://github.com/MicrosoftLearning/mslearn-ai-studio/raw/main/data/brochures.zip)，将其解压缩到本地文件系统上名为 brochures**** 的文件夹中。
1. 在 Azure AI Foundry 门户的项目中，在左侧导航窗格“**我的资产**”下，选择“**数据 + 索引**”页。
1. 选择“+ 新建数据”。****
1. 在“添加数据”向导中，展开下拉菜单以选择“上传文件/文件夹”。********
1. 选择“上传文件夹”，然后选择“手册”文件夹。********
1. 选择“**下一步**”并将数据名称设置为 `brochures`。
1. 等待文件夹上传完成，并注意它包含多个 .pdf 文件。

## 为数据创建索引

现在，你已将数据源添加到项目中，可以使用它在 Azure AI 搜索资源中创建索引了。

1. 在 Azure AI Foundry 门户的项目中，在左侧导航窗格“**我的资产**”下，选择“**数据 + 索引**”页。
1. 在“**索引**”选项卡中，使用以下设置添加新索引：
    - **源位置**：
        - **数据源**：Azure AI Foundry 门户中的数据
            - 选择 brochures**** 数据源**
    - **索引配置**：
        - 选择 Azure AI 搜索服务****：选择到 Azure AI 搜索资源的 AzureAISearch**** 连接**
        - **矢量索引**：`brochures-index`
        - **虚拟机**：自动选择
    - 搜索设置****：
        - 矢量设置****：向此搜索资源添加矢量搜索
        - **Azure OpenAI 连接**：*为中心选择默认 Azure OpenAI 资源。*

1. 等待索引过程完成，这可能需要一段时间，具体取决于订阅中的可用计算资源。 索引创建操作包含以下作业：

    - 将文本标记破解、分块，然后将其嵌入到“手册”数据中。
    - 创建 Azure AI 搜索索引。
    - 注册索引资产。

## 在操场中测试索引

在基于 RAG 的提示流中使用索引之前，我们先验证它是否可用于影响生成式 AI 响应。

1. 在左侧导航窗格中，选择“**操场**”页，然后打开“**聊天**”操场。
1. 在“聊天操场”页的“设置”窗格中，确保已选择“**GPT-4**”模型部署。 然后，在主聊天会话面板中提交提示“`Where can I stay in New York?`”
1. 查看响应，该响应应该是来自模型的通用答案，没有来自索引的任何数据。
1. 在“设置”窗格中，展开“**添加数据**”字段，然后添加 **brochures-index** 项目索引，并选择“**混合（矢量 + 关键字）**”搜索类型。

   > **提示**：在某些情况下，新建的索引可能无法立即使用。 刷新浏览器通常很有帮助，但如果仍然遇到找不到索引的问题，可能需要等到索引被识别。

1. 添加索引并重启聊天会话后，重新提交提示“`Where can I stay in New York?`”
1. 查看响应，该响应应基于索引中的数据。

## 使用 Azure AI Foundry 和 Azure OpenAI SDK 创建 RAG 客户端应用程序

拥有工作索引后，可以使用 Azure AI Foundry 和 Azure OpenAI SDK 在客户端应用程序中实现 RAG 模式。 我们将在一个简单的示例中探索代码实现。

> **提示**：你可以选择使用 Python 或 Microsoft C# 开发 RAG 解决方案。 按照所选语言的相应部分中的说明进行操作。

### 准备应用程序配置

1. 在 Azure AI Foundry 门户中，查看项目的“**概述**”页。
1. 在“**项目详细信息**”区域中，记下**项目连接字符串**。 你将使用此连接字符串连接到客户端应用程序中的项目。
1. 打开新的浏览器选项卡（使 Azure AI Foundry 门户在现有选项卡中保持打开状态）。 然后在新选项卡中，浏览到 [Azure 门户](https://portal.azure.com)，网址为：`https://portal.azure.com`；如果出现提示，请使用 Azure 凭据登录。
1. 使用页面顶部搜索栏右侧的 **[\>_]** 按钮在 Azure 门户中创建新的 Cloud Shell，选择 ***PowerShell*** 环境。 Cloud Shell 在 Azure 门户底部的窗格中提供命令行接口。

    > **备注**：如果以前创建了使用 *Bash* 环境的 Cloud Shell，请将其切换到 ***PowerShell***。

1. 在 Cloud Shell 工具栏的“**设置**”菜单中，选择“**转到经典版本**”（这是使用代码编辑器所必需的）。

    > **提示**：将命令粘贴到 cloudshell 中时，输出可能会占用大量屏幕缓冲区。 可以通过输入 `cls` 命令来清除屏幕，以便更轻松地专注于每项任务。

1. 在 PowerShell 窗格中，输入以下命令以克隆包含此练习的 GitHub 存储库：

    ```
    rm -r mslearn-ai-foundry -f
    git clone https://github.com/microsoftlearning/mslearn-ai-studio mslearn-ai-foundry
    ```

> **备注**：按照所选编程语言的步骤操作。

1. 克隆存储库后，导航到包含聊天应用程序代码文件的文件夹：  

    **Python**

    ```
   cd mslearn-ai-foundry/labfiles/rag-app/python
    ```

    **C#**

    ```
   cd mslearn-ai-foundry/labfiles/rag-app/c-sharp
    ```

1. 在 Cloud Shell 命令行窗格中，输入以下命令安装将使用的库：

    **Python**

    ```
   pip install python-dotenv azure-ai-projects azure-identity openai
    ```

    **C#**

    ```
   dotnet add package Azure.Identity
   dotnet add package Azure.AI.Projects --prerelease
   dotnet add package Azure.AI.OpenAI --prerelease
    ```
    

1. 输入以下命令以编辑已提供的配置文件：

    **Python**

    ```
   code .env
    ```

    **C#**

    ```
   code appsettings.json
    ```

    该文件已在代码编辑器中打开。

1. 在代码文件中，替换以下占位符： 
    - **your_project_endpoint**：替换为项目的连接字符串（从 Azure AI Foundry 门户中的项目**概述**页面复制）
    - **your_model_deployment** 替换为分配给模型部署的名称（应为`gpt-4`）
    - **your_index**：替换为你的索引名称（应为`brochures-index`）
1. 替换占位符后，在代码编辑器中使用 **CTRL+S** 命令或 ** 右键单击 > 保存** 保存更改，然后使用 **CTRL+Q** 命令或 ** 右键单击 > 退出** 关闭代码编辑器，同时保持 Cloud Shell 命令行打开。

### 探索实现 RAG 模式的代码

1. 输入以下命令以编辑已提供的代码文件：

    **Python**

    ```
   code rag-app.py
    ```

    **C#**

    ```
   code Program.cs
    ```

1. 查看文件中的代码，并注意以下几点：
    - 使用 Azure AI Foundry SDK 连接到项目（使用项目连接字符串）
    - 从项目检索默认的 Azure AI 搜索连接，帮助确定 Azure AI 搜索服务的终结点和密钥。
    - 根据项目中的默认 Azure OpenAI 服务连接创建经过身份验证的 Azure OpenAI 客户端。
    - 将提示（包括系统消息和用户消息）提交给 Azure OpenAI 客户端，并添加有关 Azure AI 搜索索引的附加信息，为提示提供支持。
    - 显示依据提示生成的响应。
1. 使用 **Ctrl+Q** 命令关闭代码编辑器，同时保持 cloud shell 命令行打开。

### 运行聊天应用程序

1. 在 Cloud Shell 命令行窗格中，输入以下命令以运行应用：

    **Python**

    ```
   python rag-app.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. 出现提示时，输入问题，例如`Where can I travel to?`并查看生成式 AI 模型的回复。

    请注意，响应包含源引用，用于指示找到答案的索引数据。

1. 尝试其他问题，例如`Where should I stay in London?`

    > **注意**：此简单示例应用程序未包含保留会话历史的逻辑，因此每个提示都被视为一次新对话。

1. 完成后，输入`quit`退出程序。 然后关闭 Cloud Shell 窗格。

## 难题

既然你已经体验了如何在使用 Azure AI Foundry 门户构建的生成式 AI 应用中集成自己的数据，就让我们进一步探索吧！

尝试通过 Azure AI Foundry 门户添加新数据源，为其编制索引，并将索引数据集成到客户端应用中。 可以尝试的一些数据集包括：

- 计算机上有的一系列（研究）文章。
- 一组来自过去会议的演示文稿。
- [Azure 搜索示例数据](https://github.com/Azure-Samples/azure-search-sample-data)存储库中提供的任何数据集。

提交只能依据所选数据集回答的提示，测试解决方案！

## 清理

为了避免不必要的 Azure 成本和资源使用，应删除在本练习中部署的资源。

1. 如果已探索完 Azure AI Foundry，请返回到 [Azure 门户](https://portal.azure.com) (`https://portal.azure.com`)，并根据需要使用 Azure 凭据登录。 然后删除你在其中预配了 Azure AI 搜索和 Azure AI 资源的资源组中的资源。
