---
lab:
  title: 在 Azure AI Foundry 中针对聊天补全微调语言模型
---

# 在 Azure AI Foundry 中针对聊天补全微调语言模型

如果希望语言模型以某种方式运行，可以使用提示工程定义所需的行为。 要提高所需行为的一致性，可以选择微调模型，将其与提示工程方法进行比较，以评估哪种方法最适合你的需求。

在本练习中，使用 Azure AI Foundry 微调要用于自定义聊天应用程序场景的语言模型。 将微调后的模型与基础模型进行比较，以评估微调后的模型是否更符合需求。

假设你在旅行社工作且正在开发一款聊天应用程序，以帮助人们规划假期。 目标是创建一款简单而吸引人的聊天工具，向他们推荐目的地和活动。 由于聊天未连接到任何数据源，因此它**不**应为酒店、航班或餐馆提供特定建议，以确保客户的信任。

该练习大约需要 **60** 分钟。

## 在 Azure AI Foundry 门户中创建 AI 中心和项目

首先，在 Azure AI 中心内创建 Azure AI Foundry 门户项目：

1. 在 Web 浏览器中，使用 Azure 凭据打开 [https://ai.azure.com](https://ai.azure.com) 并登录。
1. 在主页中，选择“**+ 创建资源**”。
1. 在“创建新项目”向导中，使用以下设置创建项目****：
    - 项目名称****：项目的唯一名称**
    - 选择“**自定义**”
        - **中心**：*使用默认名称自动填充*
        - **订阅**：*使用已登录帐户自动填充*
        - **资源组**：（新建）*使用项目名称自动填充*
        - **位置**：选择以下区域之一：**美国东部 2**、**美国中北部**、**瑞典中部**、**瑞士西部**\*
        - **连接 Azure AI 服务或 Azure OpenAI**：（新建）*使用所选中心名称自动填充*
        - **连接 Azure AI 搜索**：跳过连接

    > \* Azure OpenAI 资源受区域配额限制在租户级别。 位置帮助程序中列出的区域包括本练习中使用的模型类型的默认配额。 随机选择区域可降低单个区域达到其配额限制的风险。 如果稍后在练习中达到配额限制，你可能需要在不同的区域中创建另一个资源。 详细了解[如何微调模型区域](https://learn.microsoft.com/en-us/azure/ai-services/openai/concepts/models?tabs=python-secure%2Cglobal-standard%2Cstandard-chat-completions#fine-tuning-models)

1. 查看配置并创建项目。
1. 等待创建项目。

## 微调 GPT-3.5 模型

由于微调模型需要一些时间才能完成，因此首先启动微调作业。 微调模型之前，需要一个数据集。

1. 从 `https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-studio/refs/heads/main/data/travel-finetune-hotel.jsonl` 下载[训练数据集](https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-studio/refs/heads/main/data/travel-finetune-hotel.jsonl)，并将其保存为本地 JSONL 文件。

    > **备注**：设备可能默认将文件另存为 .txt 文件。 选择所有文件并移除 .txt 后缀，以确保将文件另存为 JSONL。

1. 使用左侧菜单导航到“**生成和自定义**”部分下的“**微调**”页。
1. 选择此按钮以添加新的微调模型，选择 `gpt-35-turbo` 模型，选择“**下一步**”，然后选择“**确认**”。
1. 使用以下配置**微调**该模型：
    - **模型版本**：*选择默认版本*
    - **模型后缀**：`ft-travel`
    - **连接的 AI 资源**：*选择创建中心时创建的连接。默认情况下应处于选中状态。*
    - **训练数据**：上传文件

    <details>  
    <summary><b>故障排除提示</b>：权限错误</summary>
    <p>如果收到权限错误，请尝试执行以下操作进行故障排除：</p>
    <ul>
        <li>在 Azure 门户中，选择 AI 服务资源。</li>
        <li>在 IAM 页上的“标识”选项卡中，确认它是系统分配的托管标识。</li>
        <li>导航到关联的存储帐户。 在 IAM 页上，添加角色分配<em>存储 Blob 数据读取器</em>。</li>
        <li>在“<strong>分配访问权限</strong>”下，选择“<strong>托管标识</strong>”、“<strong>++ 选择成员</strong>”，然后选择“<strong>所有系统分配的托管标识</strong>”。</li>
        <li>查看并分配以保存新设置，然后重试上一步。</li>
    </ul>
    </details>

    - **上传文件**：选择在上一步骤中下载的 JSONL 文件。
    - **验证数据**：无
    - **任务参数**：*保留默认设置*
1. 微调将启动，可能需要一些时间才能完成。

> **备注**：微调和部署可能需要一些时间，因此可能需要定期回查。 等待期间，可以继续执行下一步。

## 与基础模型聊天

等待微调作业完成期间，让我们与基础 GPT 3.5 模型聊天，以评估其性能。

1. 使用左侧菜单导航到“**我的资源**”部分下的“**模型 + 终结点**”页。
1. 选择“**+ 部署模型**”按钮，然后选择“**部署基础模型**”选项。
1. 部署 `gpt-35-turbo` 模型，该模型与微调时使用的模型类型相同。
1. 部署完成后，选择“**在操场中打开**”按钮。
1. 验证是否已在设置窗格中选择已部署的 `gpt-35-model` 基础模型。
1. 在聊天窗口中，输入查询 `What can you do?`，然后查看响应。
    答案非常笼统。 请记住，我们希望创建一款聊天应用程序，以激励人们旅行。
1. 使用以下提示更新设置窗格中的系统消息：

    ```md
    You are an AI assistant that helps people plan their holidays.
    ```

1. 选择“**应用更改**”，选择“**清除聊天**”，然后再次询问 `What can you do?`。助手可能会回复你，它可以帮助为你的旅行预订航班、酒店和租车。 你想要避免这种行为。
1. 使用新提示再次更新系统消息：

    ```md
    You are an AI travel assistant that helps people plan their trips. Your objective is to offer support for travel-related inquiries, such as visa requirements, weather forecasts, local attractions, and cultural norms.
    You should not provide any hotel, flight, rental car or restaurant recommendations.
    Ask engaging questions to help someone plan their trip and think about what they want to do on their holiday.
    ```

1. 选择“**应用更改**”和“**清除聊天**”。
1. 继续测试聊天应用程序，以验证它不会提供任何未在检索到的数据中记录的信息。 例如，提出以下问题并浏览模型的答案：
   
    `Where in Rome should I stay?`
    
    `I'm mostly there for the food. Where should I stay to be within walking distance of affordable restaurants?`

    `Give me a list of five bed and breakfasts in Trastevere.`

    即使指示模型不提供酒店建议，它也可以为你提供酒店列表。 这是行为不一致的示例。 让我们探讨微调后的模型性能在这些情况下是否会更好。

1. 导航到“**生成和自定义**”下的“**微调**”页，查找微调作业及其状态。 如果仍在运行，可以选择继续手动评估已部署的基础模型。 如果已完成，可以继续执行下一部分。

## 部署微调后模型

成功完成微调后，即可部署微调后的模型。

1. 选择微调后模型。 选择“**指标**”选项卡并浏览微调指标。
1. 使用以下配置部署微调后模型：
    - **部署名称**：*模型的唯一名称，可以使用默认值*
    - **部署类型**：标准
    - **每分钟令牌数速率限制（数千个）**：5K
    - **内容筛选器**：默认
1. 等待部署完成，然后才能对其进行测试，这可能需要一段时间。

## 测试微调后模型

部署微调后的模型后，可以像测试已部署的基础模型一样测试该模型。

1. 部署准备就绪后，导航到微调后模型并选择“**在操场中打开**”。
1. 使用以下说明更新系统消息：

    ```md
    You are an AI travel assistant that helps people plan their trips. Your objective is to offer support for travel-related inquiries, such as visa requirements, weather forecasts, local attractions, and cultural norms.
    You should not provide any hotel, flight, rental car or restaurant recommendations.
    Ask engaging questions to help someone plan their trip and think about what they want to do on their holiday.
    ```

1. 测试微调后的模型，以评估其行为现在是否更一致。 例如，再次提出以下问题并浏览模型的答案：
   
    `Where in Rome should I stay?`
    
    `I'm mostly there for the food. Where should I stay to be within walking distance of affordable restaurants?`

    `Give me a list of five bed and breakfasts in Trastevere.`

## 清理

如果已完成对 Azure AI Foundry 的探索，则应删除已创建的资源，以避免产生不必要的 Azure 成本。

- 导航到 Azure 门户，地址为 `https://portal.azure.com`[](https://portal.azure.com)。
- 在 Azure 门户的**主页**上，选择“资源组”。
- 选择你为本练习创建的资源组。
- 在资源组的“概述”页的顶部，选择“删除资源组”。
- 输入资源组名称以确认要删除该资源组，然后选择“删除”。
