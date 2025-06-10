---
lab:
  title: 基于自有数据构建生成式 AI 应用
  description: 了解如何使用检索增强生成 (RAG) 模型，构建基于自有数据优化提示的聊天应用。
---

# 基于自有数据构建生成式 AI 应用

检索增强生成 (RAG) 是一种用于生成应用程序的技术，这些应用程序将来自自定义数据源的数据集成到生成式 AI 模型的提示中。 RAG 是开发生成式 AI 应用（基于聊天的应用程序，使用语言模型来解释输入并生成适当的响应）的常用模式。

在本练习中，你将使用 Azure AI Foundry 将自定义数据集成到生成式 AI 解决方案中。

本练习大约需要 **45** 分钟。

> **备注**：本练习基于预发布的服务，可能会有更改。

## 创建 Azure AI Foundry 资源

我们首先创建一个 Azure AI Foundry 资源。

1. 在 Web 浏览器中打开 [Azure 门户](https://portal.azure.com) (`https://portal.azure`)，然后使用 Azure 凭据登录。 关闭首次登录时弹出的所有提示或快速入门窗格。
1. 使用以下设置，创建新的 `Azure AI Foundry` 资源：
    - **订阅**：Azure 订阅
    - **资源组**：*创建或选择资源组*
    - **名称**：*Azure AI Foundry 资源的有效名称*
    - **区域**：选择以下区域之一：
        - 美国东部 2
        - 瑞典中部
    - **默认项目名称**：*项目的有效名称*

1. 等待资源创建完成后，进入 Azure 门户中的该资源页。
1. 在你的 Azure AI Foundry 资源页，选择“**转到 Azure AI Foundry 门户**”。

## 部署模型

需要两个模型来实现解决方案：

- 一个嵌入** 模型，用于矢量化文本数据以实现高效索引和处理。
- 一个可以根据数据生成对问题的自然语言回答的模型。

1. 在 Azure AI Foundry 门户的项目中，在左侧导航窗格“**我的资产**”下，选择“**模型 + 终结点**”页。
1. 在“部署模型”向导中选择“**自定义**”，并使用以下设置新建 **text-embedding-ada-002** 模型部署：

    - **部署名称**：*模型部署的有效名称*
    - **部署类型**：全局标准
    - **模型版本**：*选择默认版本*
    - **关联的 AI 资源**：*选择先前创建的资源*
    - **每分钟令牌限制（千令牌）**：50K *（或若订阅的可用上限低于 50K，则以其为准）*
    - **内容筛选器**：DefaultV2

    > **备注**：如果当前 AI 资源位置没有可用于要部署模型的配额，系统会要求你选择其他位置，以便新建 AI 资源并连接到项目。

1. 返回到**模型 + 终结点**页面，重复前述步骤，使用**全局标准**部署最新版本的 **gpt-4o** 模型，并将 TPM 速率限制设置为 **50K**（或订阅中可用的最大值，如果低于 50K）。

    > **注意**：减少每分钟令牌数 (TPM) 有助于避免正在使用的订阅中可用配额的过度使用。 50,000 TPM 对本练习中使用的数据是足够的。

## 将数据添加到项目

你的应用数据包括一组来自虚构旅行社 *Margie's Travel*PDF 格式旅行手册。 让我们将它们添加到项目。

1. 在新的浏览器标签页中，从`https://github.com/MicrosoftLearning/mslearn-ai-studio/raw/main/data/brochures.zip`下载[手册的压缩存档](https://github.com/MicrosoftLearning/mslearn-ai-studio/raw/main/data/brochures.zip)，将其解压缩到本地文件系统上名为 **brochures** 的文件夹中。
1. 在 Azure AI Foundry 门户的项目中，在左侧导航窗格中，选择“**操场**”，然后选择“**试用聊天操场**”。
1. 在操场的“**设置**”窗格中，展开“**添加数据**”部分，接着选择“**添加数据源**”。
1. 在“**添加数据**”向导中，展开下拉菜单，选择“**上传文件**”。
1. 使用以下设置，创建新的 Azure Blob 存储资源：
    - **订阅**：Azure 订阅
    - **资源组**：*与你的 Azure AI Foundry 资源相同的资源组*
    - **存储帐户名称**：*存储帐户资源的有效名称*
    - **区域**：*与你的 Azure AI Foundry 资源相同的区域*
    - **性能**：标准
    - **冗余**：LRS
1. 创建资源，并等待部署完成。
1. 返回到 Azure AI Foundry 选项卡，刷新 Azure Blob 存储资源列表，然后选择新创建帐户。

    > **注意**：如果收到警告提示 Azure OpenAI 需要你授权才能访问资源，请选择“**开启 CORS**”。

1. 使用以下设置，创建新的 Azure AI 搜索资源：
    - **订阅**：Azure 订阅
    - **资源组**：*与你的 Azure AI Foundry 资源相同的资源组*
    - **服务名称**：*你的 Azure AI 搜索资源的有效名称*
    - **区域**：*与你的 Azure AI Foundry 资源相同的区域*
    - 定价层：基本

1. 创建资源，并等待部署完成。
1. 返回 Azure AI Foundry 选项卡，刷新 Azure AI 搜索资源列表，然后选择新创建的帐户。
1. 给索引命名为 `brochures-index`。
1. 启用“**向此搜索资源添加矢量搜索”**”选项，然后选择之前部署的嵌入模型。 选择**下一步**。

   >**注意**：**添加数据**向导识别已部署的嵌入模型可能需要一些时间，因此如果无法启用矢量搜索选项，请取消向导，等待几分钟后再重试。

1. 从之前解压的**小册子**文件夹中上传所有 .pdf 文件，然后选择“**下一步**”。
1. 在**数据管理**步骤中，选择搜索类型为“**混合（矢量+关键字）**”，并将区块大小设置为 **1024**。 选择**下一步**。
1. 在**数据连接**步骤中，选择“**API 密钥**”作为身份验证类型。 选择**下一步**。
1. 查看所有配置步骤，然后选择“**保存并关闭**”。
1. 等待索引过程完成，这可能需要一段时间，具体取决于订阅中的可用计算资源。

    > **提示**：等待创建索引时，不妨查看下载的手册以熟悉其内容。

## 在操场中测试索引

在基于 RAG 的提示流中使用索引之前，我们先验证它是否可用于影响生成式 AI 响应。

1. 在“聊天操场”页的“设置”窗格中，确认已选择**gpt-4o**模型部署。 然后，在主聊天会话面板中提交提示“`Where can I stay in New York?`”
1. 查看响应，该响应应基于索引中的数据。

## 创建 RAG 客户端应用

拥有工作索引后，可以使用 Azure OpenAI SDK 在客户端应用程序中实现 RAG 模式。 我们将在一个简单的示例中探索代码实现。

> **提示**：你可以选择使用 Python 或 Microsoft C# 开发 RAG 解决方案。 按照所选语言的相应部分中的说明进行操作。

### 准备应用程序配置

1. 返回到包含 Azure 门户的浏览器标签页（使 Azure AI Foundry 门户在现有选项卡中保持打开状态）。
1. 使用页面顶部搜索栏右侧的 **[\>_]** 按钮在 Azure 门户中新建 Cloud Shell，选择订阅中不含存储的 ***PowerShell*** 环境。

    在 Azure 门户底部的窗格中，Cloud Shell 提供命令行接口。 可以调整此窗格的大小或最大化此窗格，以便更易于使用。

    > **备注**：如果以前创建了使用 *Bash* 环境的 Cloud Shell，请将其切换到 ***PowerShell***。

1. 在 Cloud Shell 工具栏的“**设置**”菜单中，选择“**转到经典版本**”（这是使用代码编辑器所必需的）。

    **<font color="red">在继续作之前，请确保已切换到 Cloud Shell 的经典版本。</font>**

1. 在 Cloud Shell 窗格中，输入以下命令以克隆包含此练习代码文件的 GitHub 存储库（键入命令，或将其复制到剪贴板后，在命令行中右键单击并粘贴为纯文本）：

    ```
    rm -r mslearn-ai-foundry -f
    git clone https://github.com/microsoftlearning/mslearn-ai-studio mslearn-ai-foundry
    ```

    > **提示**：将命令粘贴到 Cloudshell 中时，输出可能会占用大量屏幕缓冲区。 可以通过输入 `cls` 命令来清除屏幕，以便更轻松地专注于每项任务。

1. 克隆存储库后，导航到包含聊天应用程序代码文件的文件夹：

    > **备注**：按照所选编程语言的步骤操作。

    **Python**

    ```
   cd mslearn-ai-foundry/labfiles/rag-app/python
    ```

    **C#**

    ```
   cd mslearn-ai-foundry/labfiles/rag-app/c-sharp
    ```

1. 在 Cloud Shell 命令行窗格中，输入以下命令以安装 OpenAI SDK 库：

    **Python**

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt openai
    ```

    **C#**

    ```
   dotnet add package Azure.AI.OpenAI
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
    - **your_openai_endpoint**：Azure AI Foundry 门户中项目“**概述**”页的“Open AI 终结点”（请务必选择“**Azure OpenAI**”功能选项卡，而不是 Azure AI 推理或 Azure AI 服务功能）。
    - ** your_openai_api_key** Azure AI Foundry 门户中项目“**概述**”页的“打开 AI API 密钥”（请务必选择“**Azure OpenAI**”功能选项卡，而不是 Azure AI 推理或 Azure AI 服务功能）。
    - **your_chat_model**：从 Azure AI Foundry 门户中“**模型 + 终结点**”页（默认名称为`gpt-4o`）分配给 **GPT-4o** 模型部署的名称。
    - **your_embedding_model**：从 Azure AI Foundry 门户中“**模型 + 终结点**”页（默认名称为`text-embedding-ada-002`）分配给 **text-embedding-ada-002** 模型部署的名称。
    - **your_search_endpoint**：Azure AI 搜索资源的 URL。 可以在 Azure AI Foundry 门户的**管理中心**中查找此信息。
    - **your_search_api_key**：Azure AI 搜索资源的 API 密钥。 可以在 Azure AI Foundry 门户的**管理中心**中查找此信息。
    - **your_index**：替换为 Azure AI Foundry 门户中项目的“**数据 + 索引**”页中的索引名称（应该为`brochures-index`）。
1. 替换占位符后，在代码编辑器中使用 “CTRL+S”**** 命令或“ 右键单击 > 保存”**** 保存更改，然后使用 “CTRL+Q”**** 命令或 “右键单击 > 退出”**** 关闭代码编辑器，同时保持 Cloud Shell 命令行打开。

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
    - 使用终结点、密钥和聊天模型创建 Azure OpenAI 客户端。
    - 为旅行相关的聊天解决方案创建合适的系统消息。
    - 将提示（包括系统消息和基于用户输入的用户消息）提交给 Azure OpenAI 客户端，并添加：
        - 要查询的 Azure AI 搜索索引的连接详细信息。
        - 要用于矢量化查询\*的嵌入模型的详细信息。
    - 显示依据提示生成的响应。
    - 将响应添加到聊天历史记录。

    \**搜索索引的查询基于提示，用于查找索引文档中的相关文本。可以使用以文本形式提交查询的基于关键字的搜索，但使用基于矢量的搜索可以更高效 - 因此，使用嵌入模型在提交查询文本之前对查询文本进行矢量化。*

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

1. 出现提示时，输入问题，例如`Where should I go on vacation to see architecture?`并查看生成式 AI 模型的回复。

    请注意，响应包含源引用，用于指示找到答案的索引数据。

1. 尝试提出跟进问题，例如`Where can I stay there?`

1. 完成后，输入`quit`退出程序。 然后关闭 Cloud Shell 窗格。

## 清理

为了避免不必要的 Azure 成本和资源使用，应删除在本练习中部署的资源。

1. 如果已探索完 Azure AI Foundry，请返回到 [Azure 门户](https://portal.azure.com) (`https://portal.azure.com`)，并根据需要使用 Azure 凭据登录。 然后删除你在其中预配了 Azure AI 搜索和 Azure AI 资源的资源组中的资源。
