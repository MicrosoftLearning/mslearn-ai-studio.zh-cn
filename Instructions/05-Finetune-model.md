---
lab:
  title: 微调语言模型
  description: 了解如何使用自有训练数据微调模型，并自定义其行为。
---

# 微调语言模型

如果希望语言模型以某种方式运行，可以使用提示工程定义所需的行为。 要提高所需行为的一致性，可以选择微调模型，将其与提示工程方法进行比较，以评估哪种方法最适合你的需求。

在本练习中，使用 Azure AI Foundry 微调要用于自定义聊天应用程序场景的语言模型。 将微调后的模型与基础模型进行比较，以评估微调后的模型是否更符合需求。

假设你在旅行社工作且正在开发一款聊天应用程序，以帮助人们规划假期。 目标是创建一款简单而吸引人的聊天工具，以一致、亲切的语气推荐目的地和活动。

此练习大约需要 **60** 分钟\*。

> \***备注**：此计时是基于平均体验的估计值。 微调操作依赖于云基础结构资源，根据数据中心容量和并发需求，预配所需时间长短可能不尽相同。 本练习中的一些活动可能需要<u>很长时间</u>才能完成，并且需要耐心。 如果需要一段时间，请考虑查看 [Azure AI Foundry 微调文档](https://learn.microsoft.com/azure/ai-studio/concepts/fine-tuning-overview)或休息一段时间。 某些进程可能会超时或似乎无限期运行。 本练习中使用的一些技术是预览版，或正在积极开发中。 可能会遇到一些意想不到的行为、警告或错误。

## 在 Azure AI Foundry 项目中部署模型

让我们首先在 Azure AI Foundry 项目中部署模型。

1. 在 Web 浏览器中打开 [Azure AI Foundry 门户](https://ai.azure.com)，网址为：`https://ai.azure.com`，然后使用 Azure 凭据登录。 关闭首次登录时打开的任何使用技巧或快速入门窗格，如有必要，使用左上角的 **Azure AI Foundry** 徽标导航到主页，类似下图所示（若已打开**帮助**面板，请关闭）：

    ![Azure AI Foundry 门户的屏幕截图。](./media/ai-foundry-home.png)

1. 在主页的“**浏览模型和功能**”部分中，搜索 `gpt-4o` 模型；我们将在项目中使用它。
1. 在搜索结果中，选择 **gpt-4o** 模型以查看其详细信息，然后在模型的页面顶部，选择“**使用此模型**”。
1. 当提示创建项目时，输入项目的有效名称并展开“**高级选项**”。
1. 选择“**自定义**”并为中心指定以下设置：
    - **Azure AI Foundry 资源**：*Azure AI Foundry 资源的有效名称*
    - **订阅**：Azure 订阅
    - **资源组**：*创建或选择资源组*
    - **区域**：*选择下列区域之一*：\*
        - 美国东部 2
        - 美国中北部
        - 瑞典中部

    > \* 编写时，这些区域支持对 gpt-4o 模型进行微调。

1. 选择“**创建**”并等待项目（包括所选的 gpt-4 模型部署）创建。
1. 创建项目后，将自动打开聊天操场。
1. 在“**设置**”窗格中，记下模型部署的名称；应为 **gpt-4o**。 可以通过在“**模型和终结点**”页中查看部署来确认这一点（只需在左侧导航窗格中打开该页）。
1. 在左侧导航窗格中，选择“**概述**”以查看项目的主页；如下所示：

    > **备注**：如果显示“*权限不足*”*错误，请使用“**修复我**”按钮解决此问题。

    ![Azure AI Foundry 项目概述页面的屏幕截图。](./media/ai-foundry-project.png)

## 微调模型

由于微调模型需要一些时间才能完成，因此现在将启动微调作业，并在探索已部署的基础 gpt-4o 模型后返回该作业。

1. 从 `https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-studio/refs/heads/main/data/travel-finetune-hotel.jsonl` 下载[训练数据集](https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-studio/refs/heads/main/data/travel-finetune-hotel.jsonl)，并将其保存为本地 JSONL 文件。

    > **备注**：设备可能默认将文件另存为 .txt 文件。 选择所有文件并移除 .txt 后缀，以确保将文件另存为 JSONL。

1. 使用左侧菜单导航到“**生成和自定义**”部分下的“**微调**”页。
1. 选择按钮以添加新的微调模型，选择 **gpt-4o** 模型，然后选择“下一步”****。
1. 使用以下配置**微调**该模型：
    - **自定义方法**：监督式
    - **基础模型**：*选择 **gpt-4o** 的默认版本*
    - **训练数据**：*选择“**添加训练数据**”选项，并上传和应用之前下载的 .jsonl 文件*。
    - **模型后缀**：`ft-travel`
    - **种子**：*随机
1. 提交微调详细信息，作业将启动。 完成此操作可能需要一段时间。 在等待期间，可以继续学习练习的下一部分。

> **备注**：微调和部署可能需要一些时间（30 分钟或更长），因此可能需要定期回查。 截至目前，你可以选择微调模型作业并查看其**日志**选项卡，以查看更多详细进展信息。

## 与基础模型聊天

等待微调作业完成期间，让我们与基础 GPT 4o 模型聊天，以评估其性能。

1. 在左侧导航窗格中，选择“**操场**”，并打开“**聊天操场**”。
1. 验证是否已在“设置”窗格中选择已部署的 **gpt-4o** 基础模型。
1. 在聊天窗口中，输入查询 `What can you do?`，然后查看响应。

    答案可能较为泛泛。 请记住，我们希望创建一款聊天应用程序，以激励人们旅行。

1. 使用以下提示更新设置窗格中的系统消息：

    ```
    You are an AI assistant that helps people plan their travel.
    ```

1. 选择“**应用更改**”以更新系统消息。
1. 在聊天窗口中，再次输入查询 `What can you do?`，然后查看响应。
1 作为响应，助手可能会回复你，它可以帮助为你的旅行预订航班、酒店和出租车。 你想要避免这种行为。

1. 使用新提示再次更新系统消息：

    ```
    You are an AI travel assistant that helps people plan their trips. Your objective is to offer support for travel-related inquiries, such as visa requirements, weather forecasts, local attractions, and cultural norms.
    You should not provide any hotel, flight, rental car or restaurant recommendations.
    Ask engaging questions to help someone plan their trip and think about what they want to do on their holiday.
    ```
。
1. 继续测试聊天应用程序，以验证它不会提供任何未在检索到的数据中记录的信息。 例如，提出以下问题并查看模型的答案，特别注意模型用于响应的语气和写作风格：
   
    `Where in Rome should I stay?`
    
    `I'm mostly there for the food. Where should I stay to be within walking distance of affordable restaurants?`

    `What are some local delicacies I should try?`

    `When is the best time of year to visit in terms of the weather?`

    `What's the best way to get around the city?`

## 查看训练文件

基础模型似乎效果足够好，但你可能正在寻找生成式 AI 应用中的特定对话风格。 用于微调的训练数据提供了创建所需响应类型的显式示例的机会。

1. 打开之前下载的 JSONL 文件（可以在任何文本编辑器中打开该文件）
1. 检查训练数据文件中的 JSON 文档列表。 第一个应与此类似（格式化为可读性）：

    ```json
    {"messages": [
        {"role": "system", "content": "You are an AI travel assistant that helps people plan their trips. Your objective is to offer support for travel-related inquiries, such as visa requirements, weather forecasts, local attractions, and cultural norms. You should not provide any hotel, flight, rental car or restaurant recommendations. Ask engaging questions to help someone plan their trip and think about what they want to do on their holiday."},
        {"role": "user", "content": "What's a must-see in Paris?"},
        {"role": "assistant", "content": "Oh la la! You simply must twirl around the Eiffel Tower and snap a chic selfie! After that, consider visiting the Louvre Museum to see the Mona Lisa and other masterpieces. What type of attractions are you most interested in?"}
        ]}
    ```

    列表中的每个示例交互都包含使用基础模型测试的相同系统消息、与旅行查询相关的用户提示和响应。 训练数据中响应的样式将有助于微调模型了解响应方式。

## 部署微调后模型

成功完成微调后，即可部署微调后的模型。

1. 导航到“**生成和自定义**”下的“**微调**”页，查找微调作业及其状态。 如果仍在运行，可以选择继续与已部署的基础模型聊天或休息一下。 如果已完成，可以继续操作。

    > **提示**：使用“微调”页面中的“刷新“**** 按钮刷新视图。 如果微调作业完全消失，请刷新浏览器页面。

1. 选择微调作业链接以打开其详细信息页面。 然后，选择**指标**选项卡，并浏览微调指标。
1. 使用以下配置部署微调后模型：
    - **部署名**：*有效的模型部署名*
    - **部署类型**：标准
    - **每分钟令牌限制（千令牌）**：50K *（或如果订阅的可用上限低于 50K，则以其为准）*
    - **内容筛选器**：默认
1. 等待部署完成，才能对其进行测试，这可能需要一段时间。 检查“**预配状态**”，直到成功（可能需要刷新浏览器才能看到更新的状态）。

## 测试微调后模型

部署微调后的模型后，可以像测试已部署的基础模型一样测试该模型。

1. 部署准备就绪后，导航到微调后模型并选择“**在操场中打开**”。
1. 确保系统消息包含以下指令：

    ```
    You are an AI travel assistant that helps people plan their trips. Your objective is to offer support for travel-related inquiries, such as visa requirements, weather forecasts, local attractions, and cultural norms.
    You should not provide any hotel, flight, rental car or restaurant recommendations.
    Ask engaging questions to help someone plan their trip and think about what they want to do on their holiday.
    ```

1. 测试微调后的模型，以评估其行为现在是否更一致。 例如，再次提出以下问题并浏览模型的答案：

    `Where in Rome should I stay?`

    `I'm mostly there for the food. Where should I stay to be within walking distance of affordable restaurants?`

    `What are some local delicacies I should try?`

    `When is the best time of year to visit in terms of the weather?`

    `What's the best way to get around the city?`

1. 查看这些回复后，它们与基础模型的回复相比如何？

## 清理

如果已完成对 Azure AI Foundry 的探索，则应删除已创建的资源，以避免产生不必要的 Azure 成本。

- 导航到 Azure 门户，地址为 `https://portal.azure.com`[](https://portal.azure.com)。
- 在 Azure 门户的**主页**上，选择“资源组”。
- 选择你为本练习创建的资源组。
- 在资源组的“概述”页的顶部，选择“删除资源组”。
- 输入资源组名称以确认要删除该资源组，然后选择“删除”。
