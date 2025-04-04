---
lab:
  title: 评估生成式 AI 性能
  description: 了解如何评估模型和聊天流，以优化聊天应用的性能及其适时响应的能力。
---

# 评估生成式 AI 性能

在本练习中，将探索内置和自定义评估，以使用 Azure AI Foundry 门户评估和比较 AI 应用程序的性能。

该练习大约需要 **30** 分钟。

## 创建 Azure AI 中心和项目

Azure AI 中心会提供协作式工作区，可在其中定义一个或多个项目。** 让我们创建项目和 Azure AI 中心。

1. 在 Web 浏览器中打开 [Azure AI Foundry 门户](https://ai.azure.com)，网址为：`https://ai.azure.com`，然后使用 Azure 凭据登录。

1. 在主页中，选择“**+ 创建项目**”。
1. 在“**创建项目**”向导中，输入合适的项目名称（例如 `my-ai-project`），然后查看为支持项目而自动创建的 Azure 资源。
1. 选择“**自定义**”并为中心指定以下设置：
    - **中心名称**：*唯一名称 - 例如`my-ai-hub`*
    - **订阅**：Azure 订阅
    - **资源组**：*新建资源组并提供唯一名称（例如 `my-ai-resources`），或选择现有资源组*
    - **位置**：选择“**帮助我选择**”，然后在“位置帮助程序”窗口中选择“**gpt-4**”，并使用推荐的区域\*
    - **连接 Azure AI 服务或 Azure OpenAI**：*新建 AI 服务资源并提供适当的名称（例如 `my-ai-services`）或使用现有资源*
    - **连接 Azure AI 搜索**：跳过连接

    > \*模型配额在租户级别受区域配额的限制。 如果稍后在练习中达到配额限制，你可能需要在不同的区域中创建另一个资源。

1. 选择“**下一步**”查看配置。 然后，选择“**创建**”并等待该进程完成。
1. 创建项目后，关闭显示的所有使用技巧，并查看 Azure AI Foundry 门户中的项目页面，如下图所示：

    ![Azure AI Foundry 门户中 Azure AI 项目详细信息的屏幕截图。](./media/ai-foundry-project.png)

## 部署 GPT 模型

要在提示流中使用语言模型，首先需要部署模型。 Azure AI Foundry 门户允许部署可在流中使用的 OpenAI 模型。

1. 使用左侧菜单导航到“**我的资源**”部分下的“**模型 + 终结点**”页。
1. 选择“**+ 部署模型**”按钮，然后选择“**部署基础模型**”选项。
1. 在“**部署模型**”向导中选择“**自定义**”，并使用以下设置新建 **GPT-4** 模型部署：
    - 部署名称****：模型部署的唯一名称**
    - **部署类型**：标准
    - **模型版本**：*选择默认版本*
    - **AI 资源**：*选择先前创建的资源*
    - **每分钟令牌数速率限制（数千个）**：5K
    - **内容筛选器**：DefaultV2
    - **启用动态配额**：已禁用

    > **备注**：如果当前 AI 资源位置没有可用于要部署模型的配额，系统会要求你选择其他位置，以便新建 AI 资源并连接到项目。

1. 等待模型部署。 部署就绪后，选择“在操场中打开”****。
1. 在“**提供模型说明和上下文**”文本框中，将内容更改为以下内容：

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

1. 选择“**应用更改**”。
1. 在聊天（历史记录）窗口中，输入查询：`What can you do?`，以验证语言模型是否按预期运行。

现在，已部署的模型包含了更新后的系统消息，可以评估模型。

## 在 Azure AI Studio Foundry 门户中手动评估语言模型

可以根据测试数据手动审阅模型响应。 通过手动审阅，可以每次测试一个不同的输入，以评估模型是否按预期方式执行。

1. 在“**聊天操场**”中，从顶部栏中选择“**评估**”下拉列表，然后选择“**手动评估**”。
1. 将“**系统消息**”更改为上面所用的同一消息（此处再次包含）：

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

1. 在“手动评估结果”部分中，你将添加五个输入以查看其输出。**** 输入以下五个问题作为五个单独的 **输入**：

   `Can you provide a list of the top-rated budget hotels in Rome?`

   `I'm looking for a vegan-friendly restaurant in New York City. Can you help?`

   `Can you suggest a 7-day itinerary for a family vacation in Orlando, Florida?`

   `Can you help me plan a surprise honeymoon trip to the Maldives?`

   `Are there any guided tours available for the Great Wall of China?`

1. 从顶部栏中选择“运行”，以便为添加为输入的所有问题生成输出。****
1. 现在，可以通过选择响应右下角的拇指向上或向下图标来手动审阅每个问题的输出。 对每个响应进行评价，确保至少在评价中包含一个拇指向上和一个拇指向下回复。
1. 从顶部栏中选择“**保存结果**”。 输入 `manual_evaluation_results` 作为结果的名称。
1. 使用左侧菜单导航到“**评估**”。
1. 选择“手动评估”选项卡以查找刚刚保存的手动评估。**** 请注意，可以浏览之前创建的手动评估，从上次中断的地方继续，并保存更新后的评估。

## 使用内置指标评估聊天应用

使用提示流创建聊天应用程序后，可以通过执行批处理运行并使用内置指标评估流的性能来评估流。

![用于评估的输入数据集的构造图。](./media/diagram-dataset-evaluation.png)

为了评估聊天流，提供的用户查询和聊天响应将作为评估的输入。

为了节省时间，我们创建了批处理输出数据集，其中包含由提示流处理的多个输入的结果。 每个结果都存储在后续步骤将进行评估的数据集中。

1. 选择“**自动运行评估**”选项卡，并使用以下设置创建**新评估**：<details>  
      <summary><b>故障排除提示</b>：权限错误</summary>
        <p>如果新建提示流时收到权限错误，请尝试执行以下操作进行故障排除：</p>
        <ul>
          <li>在 Azure 门户中，选择 AI 服务资源。</li>
          <li>在“资源管理”下的“标识”选项卡中，确认它是系统分配的托管标识。</li>
          <li>导航到关联的存储帐户。 在 IAM 页上，添加角色分配<em>存储 Blob 数据读取器</em>。</li>
          <li>在“<strong>分配访问权限</strong>”下，选择“<strong>托管标识</strong>”、“<strong>+ 选择成员</strong>”，然后选择“<strong>所有系统分配的托管标识</strong>”。</li>
          <li>查看并分配以保存新设置，然后重试上一步。</li>
        </ul>
    </details>

    - **要评估什么内容？**：数据集
    - **评估名称**：输入唯一名称**
    - 选择“下一步”****
    - **选择要评估的数据**：添加数据集
        - 下载[验证数据集](https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-studio/main/data/travel-qa.jsonl) (`https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-studio/main/data/travel-qa.jsonl`)，将其保存为 JSONL 文件并上传到 UI。

    > **备注**：设备可能默认将文件另存为 .txt 文件。 选择所有文件并移除 .txt 后缀，以确保将文件另存为 JSONL。

    - 选择“下一步”****
    - **选择指标**：一致性、流畅性
    - 连接****：*你的 AI 服务连接*
    - **部署名称/模型**：*已部署的 GPT-4 模型*
    - **查询**：选择“**查询**”作为数据源
    - **响应**：选择“**响应**”作为数据源
      
1. 选择“**下一步**”，然后查看数据并“**提交**”新评估。
1. 等待评估完成，可能需要刷新。
1. 选择刚刚创建的评估运行。
1. 在“**报表**”选项卡中浏览“**指标仪表板**”，并在“**数据**”选项卡中浏览“**详细指标结果**”。

## 删除 Azure 资源

完成对 Azure AI Foundry 的探索后，应删除已创建的资源，以避免产生不必要的 Azure 成本。

- 导航到 Azure 门户，地址为 `https://portal.azure.com`[](https://portal.azure.com)。
- 在 Azure 门户的**主页**上，选择“资源组”。
- 选择你为本练习创建的资源组。
- 在资源组的“概述”页的顶部，选择“删除资源组”。
- 输入资源组名称以确认要删除该资源组，然后选择“删除”。
