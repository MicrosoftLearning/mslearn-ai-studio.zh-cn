---
lab:
  title: 基于自有数据构建生成式 AI 应用
  description: 了解如何使用检索增强生成 (RAG) 模型，构建基于自有数据优化提示的聊天应用。
---

# 基于自有数据构建生成式 AI 应用

检索增强生成 (RAG) 是一种用于生成应用程序的技术，这些应用程序将来自自定义数据源的数据集成到生成式 AI 模型的提示中。 RAG 是开发生成式 AI 应用（基于聊天的应用程序，使用语言模型来解释输入并生成适当的响应）的常用模式。

在本练习中，你将使用 Azure AI Foundry 将自定义数据集成到生成式 AI 解决方案中。

> **注意**：本练习中的代码基于预发布 SDK 软件，可能会有更改。 必要时，我们使用了特定版本的包；这可能没有反映最新的可用版本。 可能会遇到一些意想不到的行为、警告或错误。

尽管本练习基于 Azure OpenAI Python SDK，但你也可以使用多种语言特定的 SDK 开发 AI 聊天应用程序，包括：

- [适用于 Python 的 OpenAI](https://pypi.org/project/openai/)
- [适用于 Microsoft .NET 的 Azure Open AI](https://www.nuget.org/packages/Azure.AI.OpenAI)
- [适用于 TypeScript 的 Azure OpenAI](https://www.npmjs.com/package/@azure/openai)

本练习大约需要 **45** 分钟。

## Azure AI Foundry 中心和项目

我们将在本练习中使用的 Azure AI Foundry 功能，需要基于 Azure AI Foundry *中心*资源的项目。

1. 在 Web 浏览器中打开 [Azure AI Foundry 门户](https://ai.azure.com)，网址为：`https://ai.azure.com`，然后使用 Azure 凭据登录。 关闭首次登录时打开的任何使用技巧或快速入门窗格，如有必要，使用左上角的 **Azure AI Foundry** 徽标导航到主页，类似下图所示（若已打开**帮助**面板，请关闭）：

    ![Azure AI Foundry 门户的屏幕截图。](./media/ai-foundry-home.png)

1. 在浏览器中，浏览到 `https://ai.azure.com/managementCenter/allResources`，并选择“新建”****。 然后选择创建新的 **AI 中心资源**的选项。
1. 在**创建项目**向导中，输入有效的项目名称，并选择创建新中心。 然后，使用**重命名中心**链接为你的新中心指定一个有效名称，展开“**高级选项**”，并为项目配置以下设置：
    - **订阅**：Azure 订阅
    - **资源组**：*创建或选择资源组*
    - **区域**：美国东部 2 或瑞典中部（*如果稍后在练习中超出配额限制，可能需要在其他区域创建新的资源。*）

    > **注意**：如果你使用的 Azure 订阅启用了用于限制资源名称的策略，可能需要点击“**创建新项目**”对话框底部的链接，转到 Azure 门户以创建该中心。

    > **提示**：如果“**创建**”按钮仍然禁用，请确认你已将中心名称更改为一个唯一的字母和数字组合。

1. 等待项目创建，然后导航到项目。

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
1. 在 Azure AI Foundry 门户的项目中，在左侧导航窗格“**我的资产**”下，选择“**数据 + 索引**”页。
1. 选择“+ 新建数据”。****
1. 在“添加数据”向导中，展开下拉菜单以选择“上传文件/文件夹”。********
1. 选择“**上传文件夹**”，然后上传 **brochures** 文件夹。 等到列出文件夹中的所有文件。
1. 选择“**下一步**”并将数据名称设置为 `brochures`。
1. 等待文件夹上传完成，并注意它包含多个 .pdf 文件。

## 为数据创建索引

现在，你已将数据源添加到项目中，可以使用它在 Azure AI 搜索资源中创建索引了。

1. 在 Azure AI Foundry 门户的项目中，在左侧导航窗格“**我的资产**”下，选择“**数据 + 索引**”页。
1. 在“**索引**”选项卡中，使用以下设置添加新索引：
    - **源位置**：
        - **数据源**：Azure AI Foundry 中的数据
            - 选择 brochures**** 数据源**
    - **索引配置**：
        - **选择 Azure AI 搜索服务**：*使用以下设置创建新的 Azure AI 搜索资源*：
            - **订阅**：*Azure 订阅*
            - **资源组**：*与 AI 中心相同的资源组*
            - **服务名称**：*AI 搜索资源的有效名称*
            - **位置**：*与 AI 中心相同的位置*
            - 定价层：基本
            
            等待 AI 搜索资源创建完毕。 然后返回到 Azure AI Foundry，通过选择“**连接其他 Azure AI 搜索资源**”并将连接添加到刚刚创建的 AI 搜索资源来完成索引配置。
 
        - **矢量索引**：`brochures-index`
        - **虚拟机**：自动选择
    - 搜索设置****：
        - 矢量设置****：向此搜索资源添加矢量搜索
        - **Azure OpenAI 连接**：*为中心选择默认 Azure OpenAI 资源。*
        - **嵌入模型**：text-embedding-ada-002
        - **嵌入模型部署**：*部署* text-embedding-ada-002 *模型*

1. 创建向量索引并等待索引过程完成，这可能需要一段时间，具体取决于订阅中的可用计算资源。

    索引创建操作包含以下作业：

    - 将文本标记破解、分块，然后将其嵌入到“手册”数据中。
    - 创建 Azure AI 搜索索引。
    - 注册索引资产。

    > **提示**：等待创建索引时，不妨查看下载的手册以熟悉其内容。

## 在操场中测试索引

在基于 RAG 的提示流中使用索引之前，我们先验证它是否可用于影响生成式 AI 响应。

1. 在左侧导航窗格中，选择“**操场**”页，然后打开“**聊天**”操场。
1. 在“聊天操场”页面的“设置”窗格中，确认已选择**gpt-4o**模型部署。 然后，在主聊天会话面板中提交提示“`Where can I stay in New York?`”
1. 查看响应，该响应应该是来自模型的通用答案，没有来自索引的任何数据。
1. 在“设置”窗格中，展开“**添加数据**”字段，然后添加 **brochures-index** 项目索引，并选择“**混合（矢量 + 关键字）**”搜索类型。

   > **提示**：在某些情况下，新建的索引可能无法立即使用。 刷新浏览器通常很有帮助，但如果仍然遇到找不到索引的问题，可能需要等到索引被识别。

1. 添加索引并重启聊天会话后，重新提交提示“`Where can I stay in New York?`”
1. 查看响应，该响应应基于索引中的数据。

## 创建 RAG 客户端应用

拥有工作索引后，可以使用 Azure OpenAI SDK 在客户端应用程序中实现 RAG 模式。 我们将在一个简单的示例中探索代码实现。

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

    ```
   cd mslearn-ai-foundry/labfiles/rag-app/python
    ```

1. 在 Cloud Shell 命令行窗格中，输入以下命令以安装 OpenAI SDK 库：

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt openai
    ```

1. 输入以下命令以编辑已提供的配置文件：

    ```
   code .env
    ```

    该文件已在代码编辑器中打开。

1. 在配置文件中，替换以下占位符： 
    - your_openai_endpoint****：Azure AI Foundry 门户中项目“概述”页中的 Open AI 终结点（请务必选择“Azure OpenAI”功能选项卡）。********
    - your_openai_api_key****：Azure AI Foundry 门户中项目“概述”页的 Open AI API 键（请务必选择“Azure OpenAI”功能选项卡）********。
    - **your_chat_model**：从 Azure AI Foundry 门户中“**模型 + 终结点**”页（默认名称为`gpt-4o`）分配给 **GPT-4o** 模型部署的名称。
    - **your_embedding_model**：从 Azure AI Foundry 门户中“**模型 + 终结点**”页（默认名称为`text-embedding-ada-002`）分配给 **text-embedding-ada-002** 模型部署的名称。
    - **your_search_endpoint**：Azure AI 搜索资源的 URL。 可以在 Azure AI Foundry 门户的**管理中心**中查找此信息。
    - **your_search_api_key**：Azure AI 搜索资源的 API 密钥。 可以在 Azure AI Foundry 门户的**管理中心**中查找此信息。
    - **your_index**：替换为 Azure AI Foundry 门户中项目的“**数据 + 索引**”页中的索引名称（应该为`brochures-index`）。
1. 替换占位符后，在代码编辑器中使用 “CTRL+S”**** 命令或“ 右键单击 > 保存”**** 保存更改，然后使用 “CTRL+Q”**** 命令或 “右键单击 > 退出”**** 关闭代码编辑器，同时保持 Cloud Shell 命令行打开。

### 探索实现 RAG 模式的代码

1. 输入以下命令以编辑已提供的代码文件：

    ```
   code rag-app.py
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

    ```
   python rag-app.py
    ```

1. 出现提示时，输入问题，例如`Where should I go on vacation to see architecture?`并查看生成式 AI 模型的回复。

    请注意，响应包含源引用，用于指示找到答案的索引数据。

1. 尝试提出跟进问题，例如`Where can I stay there?`

1. 完成后，输入`quit`退出程序。 然后关闭 Cloud Shell 窗格。

## 清理

为了避免不必要的 Azure 成本和资源使用，应删除在本练习中部署的资源。

1. 如果已探索完 Azure AI Foundry，请返回到 [Azure 门户](https://portal.azure.com) (`https://portal.azure.com`)，并根据需要使用 Azure 凭据登录。 然后删除你在其中预配了 Azure AI 搜索和 Azure AI 资源的资源组中的资源。
