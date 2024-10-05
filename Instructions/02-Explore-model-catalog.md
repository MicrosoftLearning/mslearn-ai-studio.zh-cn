---
lab:
  title: 在 Azure AI Studio 中探索、部署语言模型并与其聊天。
---

# 在 Azure AI Studio 中探索、部署语言模型并与其聊天。

Azure AI Studio 的模型目录充当中央存储库，可在其中浏览和使用各种模型，从而协助创建生成式 AI 方案。

在本练习中，你将浏览 Azure AI Studio 中的模型目录。

该练习大约需要 25 分钟。

## 创建 Azure AI 中心

需要使用 Azure 订阅中的 Azure AI 中心来托管项目。 可以在创建项目时创建此资源，也可以提前预配它（这就是我们将在本练习中执行的操作）。

1. 在“管理”部分，选择“所有中心”，然后选择“+ 新建中心”。************ 使用以下设置创建新中心：
    - **中心名称**：唯一的名称**
    - **订阅**：Azure 订阅
    - 资源组****：创建具有唯一名称的新资源组，或选择现有资源组**
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
    - **连接 Azure AI 服务或 Azure OpenAI**：选择创建新的 AI 服务或使用现有 AI 服务**
    - **连接 Azure AI 搜索**：跳过连接

    > \* Azure OpenAI 资源受区域配额限制在租户级别。 列出的区域包括本练习中使用的模型类型的默认配额。 在与其他用户共享租户的情况下，随机选择一个区域可以降低单个区域达到配额限制的风险。 如果稍后在练习中达到配额限制，你可能需要在不同的区域中创建另一个资源。

    创建 Azure AI 中心后，它应如下图所示：

    ![Azure AI Studio 中 Azure AI 中心详细信息的屏幕截图。](./media/azure-ai-resource.png)

1. 打开新的浏览器标签页（Azure AI Studio 标签页保持打开状态），浏览到 Azure 门户 ([https://portal.azure.com](https://portal.azure.com?azure-portal=true))，如果系统提示，请使用 Azure 凭据登录。
1. 浏览到在其中创建了 Azure AI 中心的资源组，并查看已创建的 Azure 资源。

    ![Azure 门户中 Azure AI 中心和相关资源的屏幕截图。](./media/azure-portal.png)

1. 返回到 Azure AI Studio 浏览器标签页。
1. 在 Azure AI 中心页面左侧的窗格中查看每个页面，并记下可以创建和管理的项目。 在“连接”页上，可以看到已创建与 Azure OpenAI 和 AI 服务的连接。****

## 创建项目

Azure AI 中心会提供协作式工作区，可在其中定义一个或多个项目。** 让我们在 Azure AI 中心中创建一个项目。

1. 在 Azure AI Studio 中，确保你位于刚刚创建的中心（可以通过检查屏幕顶部的路径来验证位置）。
1. 使用左侧菜单导航到“所有项目”。****
1. 选择“+ New project”。
1. 在“创建新项目”向导中，使用以下设置创建项目****：
    - 当前中心：**** 你的 AI 中心**
    - 项目名称****：项目的唯一名称**
1. 等待创建项目。 准备就绪后，它应如下图所示：

    ![Azure AI Studio 中项目详细信息页的屏幕截图。](./media/azure-ai-project.png)

1. 查看左侧窗格中的页面，展开每个部分，并记下可执行的任务以及可在项目中管理的资源。

## 使用模型基准选择模型

部署模型之前，可以探索模型基准，以确定哪种模型最适合你的需求。

假设要创建一个充当旅行助理的自定义 Copilot。 具体而言，你希望 Copilot 为旅行相关的查询提供支持，如签证要求、天气预报、当地景点和文化规范。

Copilot 需要提供事实准确的信息，因此基础性很重要。 除此之外，你希望 Copilot 提供的答案易于阅读和理解。 因此，还需要选择在流畅性和连贯性方面得分较高的模型。

1. 在 Azure AI Studio 中，使用左侧菜单导航到“**入门**”部分下的“**模型基准**”。
    在“**质量基准**” 选项卡中，查找已可视化的一些图表，并比较不同的模型。
1. 筛选显示的模型：
    - **任务**：问题解答
    - **集合**：Azure OpenAI
    - **指标**：连贯性、流畅性、基础性
1. 浏览生成的图表和比较表。 浏览时，可以尝试回答以下问题：
    - 是否注意到 GPT-3.5 和 GPT-4 模型之间的性能差异？
    - 同一模型的版本之间是否存在差异？
    - 32k 变体与基本模型有何不同？

在 Azure OpenAI 集合中，可以在 GPT-3.5 和 GPT-4 模型之间进行选择。 让我们部署这两个模型，并探索它们如何针对你的用例进行比较。

## 部署 Azure OpenAI 模型

通过模型基准探索完选项后，现在可以部署语言模型。 可以浏览模型目录并从中进行部署，也可以通过“**部署**”页部署模型。 让我们探讨这两个选项。

### 从模型目录部署模型

首先从模型目录部署模型。 如果要筛选所有可用模型，则可以首选此选项。

1. 使用左侧菜单导航到“**入门**”部分下的“**模型目录**”页。
1. 使用以下设置搜索并部署由 Azure AI 策划的 `gpt-35-turbo` 模型：
    - **部署名称**：*模型部署的唯一名称，指示其为 GPT-3.5 模型*
    - **模型版本**：*选择默认版本*
    - **部署类型**：标准
    - 连接的 Azure OpenAI 资源****：选择创建中心时创建的默认连接**
    - **每分钟令牌数速率限制（数千个）**：5K
    - **内容筛选器**：默认

### 通过部署部署模型

如果已确切知道要部署的模型，则可能需要通过部署执行此操作。

1. 使用左侧菜单导航到“**组件**”部分下的“**部署**”页。
1. 在“**模型部署**”选项卡中，使用以下设置新建部署：
    - **模型**：GPT 4
    - **部署名称**：*模型部署的唯一名称，指示其为 GPT-4 模型*
    - **模型版本**：*选择默认版本*
    - **部署类型**：标准
    - 连接的 Azure OpenAI 资源****：选择创建中心时创建的默认连接**
    - **每分钟令牌数速率限制（数千个）**：5K
    - **内容筛选器**：默认

    > **备注**：你可能已经注意到一些显示模型基准的模型，但不作为模型目录中的选项。 模型可用性因位置而异。 可在 AI 中心级别指定位置，在其中可使用**位置帮助程序**指定要部署的模型，以获取可部署该模型的位置列表。

## 在聊天操场中测试模型

现在我们有两个模型要比较，让我们看看这些模型在对话交互中的行为。

1. 使用左侧菜单导航到“**项目操场 **”部分下的“** 聊天**”页。
1. 在“**聊天操场**”中，选择你的 GPT-3.5 部署。
1. 在聊天窗口中，输入查询 `What can you do?`，然后查看响应。
    答案非常笼统。 请记住，我们希望创建可用作旅行助理的自定义 Copilot。 可以在提出的问题中指定所需的帮助类型。
1. 在聊天窗口中，输入查询 `Imagine you're a travel assistant, what can you help me with?` 答案已更具体。 你可能不希望最终用户每次与 Copilot 交互时都提供必要的上下文。 要添加全局说明，可以编辑系统消息。
1. 使用以下提示更新系统消息：

   ```
   You are an AI travel assistant that helps people plan their trips. Your objective is to offer support for travel-related inquiries, such as visa requirements, weather forecasts, local attractions, and cultural norms.
   ```

1. 选择“**应用更改**”和“**清除聊天**”。
1. 在聊天窗口中，输入查询 `What can you do?` 并查看新响应。 观察它与之前收到的答案有何区别。 答案现在特定于旅行。
1. 通过提出以下问题继续对话：`I'm planning a trip to London, what can I do there?` Copilot 提供大量与旅行相关的信息。 你可能仍希望改进输出。 例如，你可能希望答案更简洁。
1. 通过在消息末尾添加 `Answer with a maximum of two sentences.` 来更新系统消息。 应用更改，清除聊天，并提出以下问题再次测试聊天： `I'm planning a trip to London, what can I do there?` 你可能还希望 Copilot 继续对话，而不是简单地回答问题。
1. 通过在消息末尾添加 `End your answer with a follow-up question.` 来更新系统消息。 应用更改、清除聊天并通过提出以下问题再次测试聊天：`I'm planning a trip to London, what can I do there?`
1. 将**部署**更改为 GPT-4 模型，并重复本部分中的所有步骤。 请注意模型在其输出中有何不同。
1. 最后，在查询 `Who is the prime minister of the UK?` 上测试这两个模型。 此问题的性能与模型的基础性（响应是否真实准确）相关。 性能是否与模型基准得出的结论相关？

探索这两个模型后，请考虑现在为用例选择哪种模型。 起初，模型输出可能有所不同，你可能更喜欢一个模型而不是另一个模型。 但是，更新系统消息后，你可能会注意到差异很小。 从成本优化的角度来看，你可以选择 GPT-3.5 模型而不是 GPT-4 模型，因其性能非常相似。

## 清理

如果已完成 Azure AI Studio 的探索，则应删除在本练习中创建的资源，以避免产生不必要的 Azure 成本。

1. 返回到包含 Azure 门户的浏览器标签页（或在新的浏览器标签页中重新打开 [Azure 门户](https://portal.azure.com?azure-portal=true) ），查看在其中部署了本练习中使用的资源的资源组的内容。
1. 在工具栏中，选择“删除资源组”****。
1. 输入资源组名称，并确认要删除该资源组。