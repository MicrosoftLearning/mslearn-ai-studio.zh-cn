---
lab:
  title: 创建一个使用你自己的数据的自定义 Copilot
---

# 创建一个使用你自己的数据的自定义 Copilot

检索增强生成 (RAG) 是一种用于生成应用程序的技术，这些应用程序将来自自定义数据源的数据集成到生成式 AI 模型的提示中。 RAG 是开发自定义 Copilot**（基于聊天的应用程序，使用语言模型来解释输入并生成适当的响应）的常用模式。

在本练习中，我们将使用 Azure AI Studio 将自定义数据集成到生成式 AI 提示流中。

本练习大约需要 **45** 分钟。

## 创建 Azure AI 搜索资源

Copilot 解决方案会将自定义数据集成到提示流中。 为了支持此集成，你需要一个 Azure AI 搜索资源来为数据建立索引。

1. 在 Web 浏览器中打开 [Azure 门户](https://portal.azure.com) (`https://portal.azure.com`)，然后使用 Azure 凭据登录。
1. 在主页上选择“+ 创建资源”，然后搜索 `Azure AI Search`。**** 然后使用以下设置创建新的 Azure AI 搜索资源：

    - 订阅****：*选择 Azure 订阅*
    - **资源组**：选择或创建资源组
    - **服务名称**：输入独一无二的服务名称**
    - 位置****：从以下任何区域中进行随机选择******\*
        - 澳大利亚东部
        - 加拿大东部
        - 美国东部
        - 美国东部 2
        - 法国中部
        - 日本东部
        - 美国中北部
        - 瑞典中部
        - 瑞士 
    - **定价层**：标准

    > \* 稍后，你将在 Azure AI 搜索资源所在的同一区域中创建一个 Azure AI 中心（其中包括 Azure OpenAI 服务）。 Azure OpenAI 资源受区域配额限制在租户级别。 列出的区域包括本练习中使用的模型类型的默认配额。 在与其他用户共享租户的情况下，随机选择一个区域可以降低单个区域达到配额限制的风险。 如果稍后在练习中达到配额限制，你可能需要在不同的区域中创建另一个 Azure AI 中心。

1. 等待 Azure AI 搜索资源部署完成。

## 创建 Azure AI 项目

现在，你已准备好创建 Azure AI Studio 项目和支持它的 Azure AI 资源。

1. 在 Web 浏览器中打开 [Azure AI Studio](https://ai.azure.com) (`https://ai.azure.com`)，然后使用 Azure 凭据登录。
1. 在 Azure AI Studio 的“主页”上，选择“+ 新建项目”********。 然后，在“创建项目”向导中，使用以下设置创建项目****：

    - 项目名称****：项目的唯一名称**
    - **中心**：使用以下设置创建新资源：**

        - **中心名称**：唯一的名称**
        - **Azure 订阅**：Azure 订阅**
        - 资源组****：*选择包含 Azure AI 搜索资源的资源组*
        - 位置****：*你的 Azure AI 搜索资源所在的同一位置*
        - **Azure OpenAI**：（新）*自动填充所选的中心名称*
        - Azure AI 搜索****：选择 Azure AI 搜索资源**

1. 等待创建项目。

## 部署模型

需要两个模型来实现解决方案：

- 一个嵌入** 模型，用于矢量化文本数据以实现高效索引和处理。
- 一个可以根据数据生成对问题的自然语言回答的模型。

1. 在 Azure AI Studio 的项目中，在左侧导航窗格中的“组件”下，选择“部署”页。********
1. 使用以下设置创建“text-embedding-ada-002”**** 模型的新部署：

    - **部署名称**：`text-embedding-ada-002`
    - **模型版本**：*默认*
    - **高级选项**：
        - **内容筛选器**：*默认*
        - **每分钟令牌的速率限制**：`5K`
1. 重复上述步骤，使用部署名称 `gpt-35-turbo-16k` 部署 gpt-35-turbo-16k**** 模型。

    > **注意**：减少每分钟令牌数 (TPM) 有助于避免正在使用的订阅中可用配额的过度使用。 对于本练习中使用的数据，5,000 TPM 已经足够。

## 将数据添加到项目

Copilot 的数据包括一组 PDF 格式的旅行手册，来自虚构的旅行社 Margie's Travel**。 让我们将它们添加到项目。

1. 从 `https://github.com/MicrosoftLearning/mslearn-ai-studio/raw/main/data/brochures.zip` 下载[手册的压缩存档](https://github.com/MicrosoftLearning/mslearn-ai-studio/raw/main/data/brochures.zip)，将其解压缩到本地文件系统上名为 brochures**** 的文件夹中。
1. 在 Azure AI Studio 的项目的左侧导航窗格中的“组件”**** 下，选择“数据”**** 页。
1. 选择“+ 新建数据”。****
1. 在“添加数据”向导中，展开下拉菜单以选择“上传文件/文件夹”。********
1. 选择“上传文件夹”，然后选择“手册”文件夹。********
1. 将数据名称设置为 `brochures`。
1. 等待文件夹上传，并注意它包含多个 .pdf 文件。

## 为数据创建索引

现在，你已将数据源添加到项目中，可以使用它在 Azure AI 搜索资源中创建索引了。

1. 在 Azure AI Studio 的项目的左侧导航窗格中的“组件”**** 下，选择“索引”**** 页。
1. 添加一个新的索引，设置如下:
    - 源数据****：
        - **数据源**：Azure AI Studio 中的数据
            - 选择 brochures**** 数据源**
    - 索引设置****：
        - 选择 Azure AI 搜索服务****：选择到 Azure AI 搜索资源的 AzureAISearch**** 连接**
        - **索引名称**：`brochures-index`
        - **虚拟机**：自动选择
    - 搜索设置****：
        - 矢量设置****：向此搜索资源添加矢量搜索
        - **选择嵌入模型**：*为你的中心选择默认的 Azure OpenAI 资源。*
        
1. 等待索引编制过程完成，这可能需要几分钟时间。 索引创建操作包含以下作业：

    - 将文本标记破解、分块，然后将其嵌入到“手册”数据中。
    - 创建 Azure AI 搜索索引。
    - 注册索引资产。

## 测试索引

在基于 RAG 的提示流中使用索引之前，我们先验证它是否可用于影响生成式 AI 响应。

1. 在左侧导航窗格的“项目操场”**** 下，选择“聊天”**** 页。
1. 在“聊天”页的“选项”面板中，确保选择了你的 gpt-35-turbo-16k **** 模型部署。 然后，在主聊天会话面板中提交提示“`Where can I stay in New York?`”
1. 查看响应，该响应应该是来自模型的通用答案，没有来自索引的任何数据。
1. 在“设置”面板中，选择“添加数据”选项卡，然后添加 brochures-index 项目索引，并选择“混合(矢量 + 关键字)”搜索类型。************

   > **备注**：某些用户会立即发现新建的索引不可用。 刷新浏览器通常很有帮助，但如果仍然遇到找不到索引的问题，可能需要等到索引被识别。
   
1. 添加索引并重启聊天会话后，重新提交提示“`Where can I stay in New York?`”
1. 查看响应，该响应应基于索引中的数据。

## 在提示流中使用索引

矢量索引已保存在 Azure AI Studio 项目中，使你能够在提示流中轻松使用它。

1. 在 Azure AI Studio 中项目的左侧导航窗格的“工具”**** 下，选择“提示流”**** 页。
1. 通过克隆库中的“基于数据的多轮问答”**** 示例来创建新的提示流。 将此示例的克隆保存在名为 `brochure-flow` 的文件夹中。
    <details>  
      <summary><b>故障排除提示</b>：权限错误</summary>
        <p>如果新建提示流时收到权限错误，请尝试执行以下操作进行故障排除：</p>
        <ul>
          <li>在 Azure 门户中，选择 AI 服务资源。</li>
          <li>在 IAM 页上的“标识”选项卡中，确认它是系统分配的托管标识。</li>
          <li>导航到关联的存储帐户。 在 IAM 页上，添加角色分配<em>存储 Blob 数据读取器</em>。</li>
          <li>在“<strong>分配访问权限</strong>”下，选择“<strong>托管标识</strong>”、“<strong>+ 选择成员</strong>”，然后选择“<strong>所有系统分配的托管标识</strong>”。</li>
          <li>查看并分配以保存新设置，然后重试上一步。</li>
        </ul>
    </details>

1. 当提示流设计器页面打开时，查看 brochure-flow****。 其图应如下图所示：

    ![提示流图的屏幕截图](./media/chat-flow.png)

    你正在使用的示例提示流实现了聊天应用程序的提示逻辑，用户可以在其中通过迭代方式将文本输入提交到聊天界面。 对话历史记录将保留，包含在每次迭代的上下文中。 提示流会协调一系列工具**，以实现以下目的：

    - 将历史记录追加到聊天输入，以问题的上下文化形式定义提示。
    - 使用你的索引和你自己根据问题选择的查询类型来检索上下文。
    - 通过使用从索引检索到的数据来生成提示上下文，从而扩充问题。
    - 通过添加系统消息和构建聊天历史记录来创建提示变体。
    - 将提示提交给语言模型以生成自然语言响应。

1. 使用“启动计算会话”按钮启动流的运行时计算。****

    等待运行时启动。 这为提示流提供计算上下文。 在等待期间，请在“流”**** 选项卡中查看流中的工具部分。

1. 在****“输入”部分，确保输入包括以下项：
    - chat_history****
    - chat_input****

    此示例中的默认聊天记录包括一些有关 AI 的对话。

1. 在****“输出”部分，确保输出包括以下项：

    - **** chat_output：值为 ${chat_with_context.output}

1. 在“modify_query_with_history”**** 部分选择以下设置（其他部分保持原样）：

    - 连接****：** AI 中心的默认 Azure OpenAI 资源
    - **** Api：chat
    - deployment_name****：gpt-35-turbo-16k
    - response_format****：{"type":"text"}

1. 在 lookup**** 部分，设置以下参数值：

    - mlindex_content****：选择空字段以打开“生成”窗格**
        - index_type****：已注册的索引
        - mlindex_asset_id****：brochures-index:1
    - **** queries：${modify_query_with_history.output}
    - **** query_type：Hybrid（矢量 + 关键字）
    - top_k****：2

1. 在 generate_prompt_context**** 部分，检查 Python 脚本并确保此工具的“inputs”**** 包含以下参数：

    - search_result（对象）：${lookup.output}******

1. 在 Prompt_variants**** 部分，检查 Python 脚本并确保此工具的“inputs”**** 包含以下参数：

    - contexts**** (string)**：${generate_prompt_context.output}
    - chat_history**** (string)**：${inputs.chat_history}
    - chat_input**** (string)**：${inputs.chat_input}

1. 在 chat_with_context**** 部分选择以下设置（其他部分保持原样）：

    - 连接****：Default_AzureOpenAI
    - Api****：聊天
    - deployment_name****：gpt-35-turbo-16k
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

> **注意**：部署有时可能需要耗费一定时间，具体取决于区域和数据中心负载。 在部署过程中，可随时转到下面的挑战部分，如果时间紧迫，可跳过部署测试。

1. 在工具栏中选择“部署”。****
1. 使用以下设置创建一个部署：
    - 基本设置：
        - **终结点**：新建
        - **终结点名称**：*使用默认唯一终结点名称*
        - **部署名称**：*使用默认部署终结点名称*
        - **虚拟机**：Standard_DS3_v2
        - **实例计数**：3
        - 推理数据收集****：选定
    - 高级设置****：
        - 使用默认设置**
1. 在 Azure AI Studio 的项目的左侧导航窗格中的“组件”**** 下，选择“部署”**** 页。
1. 不断刷新视图，直到 brochure-endpoint-1**** 部署在 brochure-endpoint**** 终结点下显示为已成功**（这可能需要较长时间）。
1. 部署成功后，请选择它。 然后，在其****“测试”页上输入提示“`What is there to do in San Francisco?`”并查看响应。
1. 输入提示“`Where else could I go?`”并查看响应。
1. 查看终结点的“使用”**** 页。请注意，它包含连接信息和示例代码，你可以使用它们为终结点生成客户端应用程序，这样就能够将提示流解决方案集成到一个用作自定义 Copilot 的应用程序中。

## 难题 

现在，你已了解如何将自己的数据集成到使用 Azure AI Studio 生成的 copilot 中，让我们进一步探讨一下！

尝试通过 Azure AI Studio 添加新数据源，为其编制索引，并在提示流中集成索引数据。 可以尝试的一些数据集包括：

- 计算机上有的一系列（研究）文章。
- 一组来自过去会议的演示文稿。
- [Azure 搜索示例数据](https://github.com/Azure-Samples/azure-search-sample-data)存储库中提供的任何数据集。

尽可能利用各种资源创建数据源，并将其整合到提示流中。 试用新的提示流并提交只能由所选数据集回答的提示！

## 清理

为了避免不必要的 Azure 成本和资源使用，应删除在本练习中部署的资源。

1. 如果你已探索完 Azure AI Studio，请返回到 [Azure 门户](https://portal.azure.com) (`https://portal.azure.com`)，并根据需要使用 Azure 凭据登录。 然后删除你在其中预配了 Azure AI 搜索和 Azure AI 资源的资源组中的资源。