---
lab:
  title: 在 Azure AI Studio 中评估语言应用
---

# 在 Azure AI Studio 中评估语言应用

在本练习中，你将探索内置和自定义评估，以使用 Azure AI Studio 评估和比较 AI 应用程序的性能。

> 要完成此练习，你的 Azure 订阅必须获得批准才能访问 Azure OpenAI 服务。 填写[注册表单](https://learn.microsoft.com/legal/cognitive-services/openai/limited-access)以请求访问 Azure OpenAI 模型。

要使用提示流生成助手，需要：

- 在 Azure AI Studio 中创建 AI 中心和项目。
- 部署 GPT 模型。
- 使用内置指标评估测试数据集。
- 定义自定义评估指标，并针对测试数据集运行它。

## 在 Azure AI Studio 中创建 AI 中心和项目

首先，在 Azure AI 中心内创建 Azure AI Studio 项目：

1. 在 Web 浏览器中，使用 Azure 凭据打开 [https://ai.azure.com](https://ai.azure.com) 并登录。
1. 选择“开始”页，然后选择“+ 新建项目”。********
1. 在“创建新项目”向导中，使用以下设置创建项目****：
    - 项目名称****：项目的唯一名称**
    - **中心**：*使用以下设置创建新中心：*
        - **中心名称**：唯一的名称**
        - **订阅**：Azure 订阅
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
    - **连接 Azure AI 服务或 Azure OpenAI**：*创建新连接*
    - **连接 Azure AI 搜索**：跳过连接

    > \* Azure OpenAI 资源受区域配额限制在租户级别。 列出的区域包括本练习中使用的模型类型的默认配额。 随机选择区域可降低单个区域达到其配额限制的风险。 如果稍后在练习中达到配额限制，你可能需要在不同的区域中创建另一个资源。 详细了解 [每个区域的模型可用性](https://learn.microsoft.com/azure/ai-services/openai/concepts/models#gpt-35-turbo-model-availability)

1. 查看配置并创建项目。
1. 等待 5-10 分钟，让项目创建完成。

## 部署 GPT 模型

要在提示流中使用语言模型，首先需要部署模型。 Azure AI Studio 让你能够部署可在流中使用的 OpenAI 模型。

1. 在左侧导航窗格中的“组件”下，选择“部署”页。********
1. 创建一个具有以下设置的 gpt-35-turbo**** 模型的新部署：
    - 部署名称****：模型部署的唯一名称**
    - **模型版本**：*选择默认版本*
    - **部署类型**：标准
    - 连接的 Azure OpenAI 资源****：*选择默认连接*
    - **每分钟令牌数速率限制（数千个）**：5K
    - **内容筛选器**：默认
1. 等待模型部署。 部署就绪后，选择“在操场中打开”****。
1. 将“系统消息”更改为以下内容：****

   ```
   **Objective**: Assist users with travel-related inquiries, offering tips, advice, and recommendations as a knowledgeable travel agent.

   **Capabilities**:
   - Provide up-to-date travel information, including destinations, accommodations, transportation, and local attractions.
   - Offer personalized travel suggestions based on user preferences, budget, and travel dates.
   - Share tips on packing, safety, and navigating travel disruptions.
   - Help with itinerary planning, including optimal routes and must-see landmarks.
   - Answer common travel questions and provide solutions to potential travel issues.
    
   **Instructions**:
   1. Engage with the user in a friendly and professional manner, as a travel agent would.
   2. Use available resources to provide accurate and relevant travel information.
   3. Tailor responses to the user's specific travel needs and interests.
   4. Ensure recommendations are practical and consider the user's safety and comfort.
   5. Encourage the user to ask follow-up questions for further assistance.
   ```

1. 选择“应用更改”。
1. 在聊天窗口中，输入查询：`What can you do?`，以验证语言模型是否按预期方式运行。

现在，已部署的模型包含了更新后的系统消息，可以评估模型。

## 在 Azure AI Studio 中评估语言模型

可以根据测试数据手动审阅模型响应。 通过手动审阅，可以每次测试一个不同的输入，以评估模型是否按预期方式执行。

1. 在“聊天操场”中，从顶部栏中选择“评估”。********
1. 此时会打开一个新窗口，其中已填充了以前的系统消息，并且选择了所部署的模型。
1. 在“手动评估结果”部分中，你将添加五个输入以查看其输出。**** 输入以下五个问题作为五个单独的 **输入**：

   `Can you provide a list of the top-rated budget hotels in Rome?`

   `I'm looking for a vegan-friendly restaurant in New York City. Can you help?`

   `Can you suggest a 7-day itinerary for a family vacation in Orlando, Florida?`

   `Can you help me plan a surprise honeymoon trip to the Maldives?`

   `Are there any guided tours available for the Great Wall of China?`

1. 从顶部栏中选择“运行”，以便为添加为输入的所有问题生成输出。****
1. 现在，可以通过选择响应右下角的拇指向上或向下图标来手动审阅每个问题的输出。 对每个响应进行评价，确保至少在评价中包含一个拇指向上和一个拇指向下回复。
1. 从顶部栏中选择“**保存结果**”。 输入 `manual_evaluation_results` 作为结果的名称。
1. 使用左侧的菜单导航到“评估”。****
1. 选择“手动评估”选项卡以查找刚刚保存的手动评估。**** 请注意，可以浏览以前创建的手动评估、从离开的位置继续，以及保存更新后的评估。
1. 选择“指标评估”选项卡，并使用以下设置创建新的评估：****
    - **评估名称**：输入唯一名称**
    - **你正在评估哪种方案？**：没有上下文的问答
    - **选择要评估的数据**：添加数据集
        - 下载 https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-studio/main/data/travel-qa.jsonl JSONL 文件并将其上传到 UI。
    - **选择指标**：一致性、流畅性
    - 连接****：*你的 AI 服务连接*
    - **部署名称/模型**：*已部署的 GPT-3.5 模型*
1. 等待评估完成，可能需要刷新。
1. 选择刚刚创建的评估运行。
1. 浏览“指标仪表板”和“详细指标结果”********。

## 删除 Azure 资源

当你完成对 Azure AI Studio 的探索时，应删除已创建的资源，以避免产生不必要的 Azure 成本。

- 导航到 Azure 门户，地址为 `https://portal.azure.com`[](https://portal.azure.com)。
- 在 Azure 门户的**主页**上，选择“资源组”。
- 选择你为本练习创建的资源组。
- 在资源组的“概述”页的顶部，选择“删除资源组”。
- 输入资源组名称以确认要删除该资源组，然后选择“删除”。
