---
lab:
  title: 创建一个使用你自己的数据的自定义 Copilot
---

# 创建一个使用你自己的数据的自定义 Copilot

检索增强生成 (RAG) 是一种用于生成应用程序的技术，这些应用程序将来自自定义数据源的数据集成到生成式 AI 模型的提示中。 RAG 是开发自定义 Copilot**（基于聊天的应用程序，使用语言模型来解释输入并生成适当的响应）的常用模式。

在本练习中，我们将使用 Azure AI Studio 将自定义数据集成到生成式 AI 提示流中。

> **注意**：在撰写本文时，Azure AI Studio 为预览版，目前正在积极开发中。 该服务的某些元素可能与本文所述不完全相同，并且一些功能可能无法按预期工作。

本练习大约需要 **45** 分钟。

## 创建 Azure AI 搜索资源

Copilot 解决方案会将自定义数据集成到提示流中。 为了支持此集成，你需要一个 Azure AI 搜索资源来为数据建立索引。

1. 在 Web 浏览器中打开 [Azure 门户](https://portal.azure.com) (`https://portal.azure.com`)，然后使用 Azure 凭据登录。
1. 在主页上选择“+ 创建资源”，然后搜索 `Azure AI Search`。**** 然后使用以下设置创建新的 Azure AI 搜索资源：
    - 订阅****：*选择 Azure 订阅*
    - **资源组**：选择或创建资源组
    - **服务名称**：输入独一无二的服务名称**
    - 位置****：选择任何可用位置**
    - **定价层**：标准
1. 等待 Azure AI 搜索资源部署完成。

## 创建 Azure AI 项目

现在，你已准备好创建 Azure AI Studio 项目和支持它的 Azure AI 资源。

1. 在 Web 浏览器中打开 [Azure AI Studio](https://ai.azure.com) (`https://ai.azure.com`)，然后使用 Azure 凭据登录。
1. 在“生成”页上选择“+ 新建 AI 项目”。******** 然后，在“入门指南”向导中，使用下面的设置创建一个项目：****
    - 项目名称****：项目的唯一名称**
    - AI 中心****：使用以下设置创建新资源：**
        - AI 中心名称****：唯一的名称**
        - **Azure 订阅**：Azure 订阅**
        - 资源组****：*选择包含 Azure AI 搜索资源的资源组*
        - 位置****：与 Azure AI 搜索资源相同的位置（或在地理上靠近它的位置）**
        - **Azure OpenAI**：（新）*自动填充所选的中心名称*
        - Azure AI 搜索****：选择 Azure AI 搜索资源**

1. 等待创建项目。

## 部署模型

需要两个模型来实现解决方案：

- 一个嵌入** 模型，用于矢量化文本数据以实现高效索引和处理。
- 一个可以根据数据生成对问题的自然语言回答的模型。

1. 在 Azure AI Studio 的项目中，在左侧导航窗格中的“组件”下，选择“部署”页。********
1. 使用以下设置创建一个“text-embedding-ada-002”模型的新部署（使用“实时终结点”）：********

    - **部署名称**：`text-embedding-ada-002`
    - **模型版本**：*默认*
    - **高级选项**：
        - **内容筛选器**：*默认*
        - **每分钟令牌的速率限制**：`5K`

> **注意**：减少每分钟令牌数 (TPM) 有助于避免正在使用的订阅中可用配额的过度使用。 对于本练习中使用的数据，5,000 TPM 已经足够。

## 将数据添加到项目

Copilot 的数据包括一组 PDF 格式的旅行手册，来自虚构的旅行社 Margie's Travel**。 让我们将它们添加到项目。

1. 从 `https://github.com/MicrosoftLearning/mslearn-ai-studio/raw/main/data/brochures.zip` 下载[手册的压缩存档](https://github.com/MicrosoftLearning/mslearn-ai-studio/raw/main/data/brochures.zip)，将其解压缩到本地文件系统上名为 brochures**** 的文件夹中。
1. 在 Azure AI Studio 的项目的左侧导航窗格中的“组件”**** 下，选择“数据”**** 页。
1. 选择“+ 新建数据”。****
1. 在“添加数据”向导中，展开下拉菜单以选择“上传文件/文件夹”。********
1. 选择“上传文件夹”，然后选择“手册”文件夹。********
1. 将数据名称设置为“手册”。****

## 为数据创建索引

现在，你已将数据源添加到项目中，可以使用它在 Azure AI 搜索资源中创建索引了。

1. 在 Azure AI Studio 的项目的左侧导航窗格中的“组件”**** 下，选择“索引”**** 页。
1. 添加一个新的索引，设置如下:
    - 源数据****：
        - **数据源**：使用现有项目数据
            - 选择 brochures**** 数据源**
    - 索引存储****：
        - 选择到 Azure AI 搜索资源的 AzureAISearch**** 连接**
    - 搜索设置****：
        - 矢量设置****：向此搜索资源添加矢量搜索
        - Azure OpenAI 资源****：Default_AzureOpenAI
        - 确认会部署嵌入模型**
    - 索引设置****：
        - 索引名称****：brochures-index
        - **虚拟机**：自动选择
1. 等待索引准备就绪，这可能需要几分钟的时间。 索引创建操作包含以下作业：

    - 将文本标记破解、分块，然后将其嵌入到“手册”数据中。
    - 更新索引。
    - 注册索引资产。

## 测试索引

在基于 RAG 的提示流中使用索引之前，我们先验证它是否可用于影响生成式 AI 响应。

1. 在左侧导航窗格的“工具”**** 下，选择“操场”**** 页。
1. 在“操场”页的“配置”**** 窗格中，确保选择了你的 gpt-35-turbo**** 模型部署。 然后，在****“聊天会话”窗格中提交提示“`Where can I stay in New York?`”
1. 查看响应，该响应应该是来自模型的通用答案，没有来自索引的任何数据。
1. 在****“助手设置”窗格中，选择“添加数据****”，然后使用以下设置添加数据源：
    - **数据源**：
        - 选择数据源****：Azure AI 搜索
        - **订阅**：*Azure 订阅*
        - Azure AI 搜索服务****：你的 Azure AI 搜索资源**
        - Azure AI 搜索索引****：brochures-index
        - 添加矢量搜索****：未选中<u></u>
    - 数据字段映射****：
        - 内容数据****：content
        - 文件名****：filepath
        - 标题****：标题
        - URL****：url
    - **数据管理**：
        - 搜索类型****：关键字
1. 添加数据源并重启聊天会话后，重新提交提示“`Where can I stay in New York?`”
1. 查看响应，该响应应基于索引中的数据。

## 在提示流中使用索引

矢量索引已保存在 Azure AI Studio 项目中，使你能够在提示流中轻松使用它。

1. 在 Azure AI Studio 的项目的左侧导航窗格中的“组件”**** 下，选择“数据”****。
1. 选择 brochures-index**** 文件夹，其中包含之前创建的索引的数据。
1. 在索引的“数据链接”**** 部分，将“存储 URI”**** 值复制到剪贴板（它应该类似于 `https://xxx.blob.core.windows.net/xxx/azureml/xxx/index/`）。 需要此 URI 来连接到提示流中的索引数据。
1. 在项目中的左侧导航窗格的“工具”**** 下，选择“提示流”**** 页。
1. 通过克隆库中的“基于数据的多轮问答”**** 示例来创建新的提示流。 将此示例的克隆保存在名为 `brochure-flow` 的文件夹中。
1. 当提示流设计器页面打开时，查看 brochure-flow****。 其图应如下图所示：

    ![提示流图的屏幕截图](./media/brochure-flow.png)

    你正在使用的示例提示流实现了聊天应用程序的提示逻辑，用户可以在其中通过迭代方式将文本输入提交到聊天界面。 对话历史记录将保留，包含在每次迭代的上下文中。 提示流会协调一系列工具**，以实现以下目的：

    1. 将历史记录追加到聊天输入，以问题的上下文化形式定义提示。
    1. 为问题创建嵌入**（使用嵌入模型将文本转换为矢量）。
    1. 根据问题搜索矢量索引以获取相关信息。
    1. 通过使用从索引检索到的数据来生成提示上下文，从而扩充问题。
    1. 通过添加系统消息和构建聊天历史记录来创建提示变体。
    1. 将提示提交给语言模型以生成自然语言响应。

1. 在“运行时”**** 列表中，选择“启动”**** 以启动自动运行时。 然后等待它启动。 这为提示流提供计算上下文。 在等待期间，请在“流”**** 选项卡中查看流中的工具部分。
1. 在“输入”**** 部分，确保输入包括 chat_history**** 和 chat_input****。 此示例中的默认聊天记录包括一些有关 AI 的对话。
1. 在“输出”**** 部分，确保 chat_output**** 值为 ${chat_with_context.output}**。
1. 在“modify_query_with_history”**** 部分选择以下设置（其他部分保持原样）：
    - 连接****：Default_AzureOpenAI
    - Api****：聊天
    - deployment_name****：gpt-35-turbo
    - response_format****：{"type":"text"}
1. 在 embed_the_question**** 部分，设置以下参数值：
    - 连接****(Azure OpenAI、OpenAI)**：Default_AzureOpenAI
    - deployment_name**** (字符串)**：text-embedding-ada-00
    - 输入****(字符串)**：${modify_query_with_history.output}
1. 在 search_question_from_indexed_docs**** 部分，设置以下参数值：
    - path**** (string)**：删除现有 URI 并粘贴矢量索引的 URI**
    - query**** (object)**：${embed_the_question.output}
    - top_k**** *(int)*：2
1. 在 generate_prompt_context**** 部分，检查 Python 脚本并确保此工具的“inputs”**** 包含以下参数：
    - search_result**** (object)**：${search_question_from_indexed_docs.output}
1. 在 Prompt_variants**** 部分，检查 Python 脚本并确保此工具的“inputs”**** 包含以下参数：
    - contexts**** (string)**：${generate_prompt_context.output}
    - chat_history**** (string)**：${inputs.chat_history}
    - chat_input**** (string)**：${inputs.chat_input}
1. 在 chat_with_context**** 部分选择以下设置（其他部分保持原样）：
    - 连接****：Default_AzureOpenAI
    - Api****：聊天
    - deployment_name****：gpt-35-turbo
    - response_format****：{"type":"text"}

    然后确保该工具的 inputs**** 包含以下参数：
    - prompt_text**** (string)**：${Prompt_variants.output}

1. 在工具栏上，使用“保存”**** 按钮保存对提示流中的工具所做的更改。
1. 在工具栏中选择“聊天”。**** 此时会打开一个聊天窗格，其中包含示例对话历史记录以及已根据示例值填写的输入。 可以忽略这些警告。
1. 在聊天窗格中，将默认输入替换为问题“`Where can I stay in London?`”，然后提交它。
1. 查看响应，该响应应基于索引中的数据。
1. 查看流中每个工具的输出。
1. 在聊天窗格中，输入问题“`What can I do there?`”
1. 查看响应，该响应应基于索引中的数据并考虑聊天历史记录（因此，“那里”被理解为“在伦敦”）。
1. 查看流中每个工具的输出，注意流中的每个工具如何对其输入进行操作以准备上下文化提示并获得相应的响应。

## 部署流

有了一个使用索引数据的工作流后，就可以将其部署为服务以供某个 Copilot 应用程序使用了。

1. 在工具栏中选择“部署”。****
1. 使用以下设置创建一个部署：
    - 基本设置：
        - **终结点**：新建
        - 终结点名称****：brochure-endpoint
        - 部署名称****：brochure-endpoint-1
        - **虚拟机**：Standard_DS3_v2
        - **实例计数**：3
        - 推理数据收集****：选定
        - Application Insights 诊断****：选定
    - 高级设置****：
        - 使用默认设置**
1. 在 Azure AI Studio 的项目的左侧导航窗格中的“组件”**** 下，选择“部署”**** 页。
1. 不断刷新视图，直到 brochure-endpoint-1**** 部署在 brochure-endpoint**** 终结点下显示为已成功**（这可能需要一些时间）。
1. 部署成功后，请选择它。 然后，在其****“测试”页上输入提示“`What is there to do in San Francisco?`”并查看响应。
1. 输入提示“`Where else could I go?`”并查看响应。
1. 查看终结点的“使用”**** 页。请注意，它包含连接信息和示例代码，你可以使用它们为终结点生成客户端应用程序，这样就能够将提示流解决方案集成到一个用作自定义 Copilot 的应用程序中。

## 清理

为了避免不必要的 Azure 成本和资源使用，应删除在本练习中部署的资源。

1. 在 Azure AI Studio 中，查看“生成****”页。 然后选择在本练习中创建的项目，并使用“删除项目”**** 按钮将其删除。 删除所有组件可能需要几分钟的时间。
1. 如果你已探索完 Azure AI Studio，请返回到 [Azure 门户](https://portal.azure.com) (`https://portal.azure.com`)，并根据需要使用 Azure 凭据登录。 然后删除为 Azure AI 搜索和 Azure AI 资源创建的资源组。
