---
lab:
  title: 开始使用 Azure AI Studio 中的提示流
---

# 开始使用 Azure AI Studio 中的提示流

从文本中提取有价值的信息称为命名实体识别 (NER)。 实体是给定文本中你感兴趣的关键字。

![实体提取](./media/get-started-prompt-flow-use-case.gif)

大型语言模型 (LLM) 可用于执行 NER。 若要创建将文本作为输入和输出实体的应用程序，可以创建一个流，该流使用具有提示流的 LLM 节点。

在本练习中，你将使用 Azure AI Studio 的提示流创建一个 LLM 应用程序，它需要实体类型和文本作为输入。 它通过 LLM 节点从 Azure OpenAI 调用 GPT 模型，以从给定文本中提取所需的实体，清理结果并输出提取的实体。

![练习概述](./media/get-started-lab.png)

首先需要在 Azure AI Studio 中创建项目，以创建必要的 Azure 资源。 然后，可以使用 Azure OpenAI 服务部署 GPT 模型。 获得所需的资源后，可以创建流。 最后，你将运行流来测试它并查看示例输出。

## 在 Azure AI Studio 中创建项目

首先创建一个 Azure AI Studio 项目和一个 Azure AI 中心来支持它。

1. 在 Web 浏览器中，使用 Azure 凭据打开 [https://ai.azure.com](https://ai.azure.com) 并登录。
1. 选择“生成”页，然后选择“+ 新建项目”。********
1. 在“创建新项目”向导中，使用以下设置创建项目****：
    - 项目名称****：项目的唯一名称**
    - Azure 中心****：使用以下设置创建新资源：**
        - AI 中心名称****：唯一的名称**
        - **订阅**：*Azure 订阅*
        - 资源组****：新资源组**
        - 位置****：从以下任何区域中进行随机选择******\*
        - 澳大利亚东部
        - 加拿大东部
        - 美国东部
        - 美国东部 2
        - 法国中部
        - 日本东部
        - 美国中北部
        - 瑞典中部
        - 瑞士北部
        - 英国南部

    > \* Azure OpenAI 资源受区域配额限制在租户级别。 列出的区域包括本练习中使用的模型类型的默认配额。 在与其他用户共享租户的情况下，随机选择一个区域可以降低单个区域达到配额限制的风险。 如果稍后在练习中达到配额限制，你可能需要在不同的区域中创建另一个资源。

1. 查看配置并创建项目。
1. 等待 5-10 分钟，让项目创建完成。

## 部署 GPT 模型

若要在提示流中使用 LLM 模型，首先需要部署模型。 Azure AI Studio 让你能够部署可在流中使用的 OpenAI 模型。

1. 在左侧导航窗格中的“组件”下，选择“部署”页。********
1. 在 Azure OpenAI Studio 中，导航到“部署”页。****
1. 创建一个具有以下设置的 gpt-35-turbo**** 模型的新部署：
    - **模型**：`gpt-35-turbo`
    - **模型版本**：** 保留默认值
    - **部署名称**：`gpt-35-turbo`
    - 设置“高级”**** 选项以使用默认内容筛选器并将每分钟标记数 (TPM) 限制为 5K****。

部署 LLM 模型后，可以在 Azure AI Studio 中创建调用已部署模型的流。

## 在 Azure AI Studio 中创建和运行流

预配所有必要的资源后，可以创建流。

### 创建新流

若要使用模板创建新流，可以选择要开发的流类型之一。

1. 在左侧导航窗格中的“工具”下，选择“提示流”。********
1. 选择“+ 创建”以创建新的流。****
1. 创建新的**标准流**，并输入 `entity-recognition` 作为文件夹名称。

将创建一个包含一个输入、两个节点和一个输出的标准流。 你将更新流以获取两个输入，提取实体，清理 LLM 节点的输出，然后将实体作为输出返回。

### 启动自动运行时

若要测试流，需要计算。 必要的计算通过运行时提供给你。

1. 创建你命名为 `entity-recognition` 的新流后，该流应在工作室中打开。
1. 从顶部栏中选择“选择运行时”字段。****
1. 在“自动运行时”列表中，选择“开始”以启动自动运行时。********
1. 等待运行时启动。

### 配置输入

你要创建的流将接收两个输入：一个文本和你要从文本中提取的实体类型。

1. 在“输入”下，配置了一个类型为 `string` 名为 `topic` 的输入。**** 使用以下设置更改现有输入并更新：
    - **名称**：`entity_type`
    - 类型：`string`
    - **值**：`job title`
1. 选择“添加输入”****。
1. 将第二个输入配置为具有以下设置：
    - **名称**：`text`
    - 类型：`string`
    - **值**：`The software engineer is working on a new update for the application.`

### 配置 LLM 节点

标准流已包含使用 LLM 工具的节点。 可以在流概述中找到节点。 默认提示要求开一个玩笑。 你将更新 LLM 节点，以基于上一部分中指定的两个输入提取实体。

1. 导航到名为 `joke` 的 **LLM 节点**。
1. 将名称替换为 `NER_LLM`
1. 对于“连接”，请选择 `Default_AzureOpenAI` 连接。****
1. 对于“deployment_name”，请选择已部署的 `gpt-35-turbo` 模型。****
1. 将提示字段替换为以下代码：

   ```yml
   {% raw %}
   system:

   Your task is to find entities of a certain type from the given text content.
   If there're multiple entities, please return them all with comma separated, e.g. "entity1, entity2, entity3".
   You should only return the entity list, nothing else.
   If there's no such entity, please return "None".

   user:
   
   Entity type: {{entity_type}}
   Text content: {{text}}
   Entities:
   {% endraw %}
   ```

1. 选择“验证和分析输入”。****
1. 在 LLM 节点的“输入”部分中，配置以下内容：****
    - 对于 `entity_type`，请选择值 `${inputs.entity_type}`。
    - 对于 `text`，请选择值 `${inputs.text}`。

LLM 节点现在会接收实体类型和文本作为输入，将其包含在你指定的提示中，并将请求发送到你部署的模型。

### 配置 Python 节点

若要仅从模型结果中提取关键信息，可以使用 Python 工具清理 LLM 节点的输出。

1. 导航到名为 `echo` 的 Python 节点。
1. 将名称替换为 `cleansing`。
1. 将  代码替换为以下内容：

   ```python
   from typing import List
   from promptflow import tool
    
    
   @tool
   def cleansing(entities_str: str) -> List[str]:
       # Split, remove leading and trailing spaces/tabs/dots
       parts = entities_str.split(",")
       cleaned_parts = [part.strip(" \t.\"") for part in parts]
       entities = [part for part in cleaned_parts if len(part) > 0]
       return entities
    
   ```

1. 选择“验证和分析输入”。****
1. 在 Python 节点的“输入”部分中，将值 `entities_str` 设置为 `${NER_LLM.output}`。****

### 配置输出

最后，可以配置整个流的输出。 只需将一个输出配置到流，该输出应为提取的实体。

1. 导航到流的“输出”。****
1. 对于“名称”，请输入 `entities`。
1. 对于“值”，请选择 `${cleansing.output}`。****

### 运行流

开发流后，可以运行它来进行测试。 由于你已向输入添加了默认值，因此可以轻松地在工作室中测试流。

1. 选择“运行”以测试流****。
1. 等待运行完成。
1. 选择“查看输出”。**** 此时会显示一个弹出窗口，显示默认输入的输出。 （可选）还可以检查日志。

## 删除 Azure 资源

当你完成对 Azure AI Studio 的探索时，应删除已创建的资源，以避免产生不必要的 Azure 成本。

- 导航到 Azure 门户，地址为 `https://portal.azure.com`[](https://portal.azure.com)。
- 在 Azure 门户的**主页**上，选择“资源组”。
- 选择你为本练习创建的资源组。
- 在资源组的“概述”页的顶部，选择“删除资源组”。
- 输入资源组名称以确认要删除该资源组，然后选择“删除”。
