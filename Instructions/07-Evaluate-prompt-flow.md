---
lab:
  title: 评估生成式 AI 模型的性能
  description: 了解如何评估模型和提示，以优化聊天应用的性能及响应的适当性。
---

# 评估生成式 AI 模型的性能

在本练习中，你将使用手动和自动化评估，以评估 Azure AI Foundry 门户中模型的性能。

该练习大约需要 **30** 分钟。

> **注意**：本练习中使用的一些技术处于预览版或积极开发阶段。 可能会遇到一些意想不到的行为、警告或错误。

## 创建 Azure AI Foundry 项目

让我们首先创建 Azure AI Foundry 项目。

1. 在 Web 浏览器中打开 [Azure AI Foundry 门户](https://ai.azure.com)，网址为：`https://ai.azure.com`，然后使用 Azure 凭据登录。 关闭首次登录时打开的任何使用技巧或快速入门窗格，如有必要，使用左上角的 **Azure AI Foundry** 徽标导航到主页，类似下图所示（若已打开**帮助**面板，请关闭）：

    ![Azure AI Foundry 门户的屏幕截图。](./media/ai-foundry-home.png)

1. 在主页中，选择“**+ 创建项目**”。
1. 在**创建项目**向导中，输入项目的有效名，如果出现建议使用现有中心的提示，请选择新建中心的选项。 然后查看将自动创建的 Azure 资源以支持中心和项目。
1. 选择“**自定义**”并为中心指定以下设置：
    - **中心名**：*有效的中心名*
    - **订阅**：Azure 订阅
    - **资源组**：*创建或选择资源组*
    - **位置**：选择以下任一区域\*：
        - 美国东部 2
        - 法国中部
        - 英国南部
        - 瑞典中部
    - **连接 Azure AI 服务或 Azure OpenAI**：*新建 AI 服务资源*
    - **连接 Azure AI 搜索**：跳过连接

    > \* 撰写本文时，这些区域支持评估 AI 安全指标。 模型可用性受区域配额的约束。 如果稍后在练习中达到配额限制，可能需要在不同的区域中创建另一个资源。

1. 选择“**下一步**”查看配置。 然后，选择“**创建**”并等待该进程完成。
1. 创建项目后，关闭显示的所有使用技巧，并查看 Azure AI Foundry 门户中的项目页面，如下图所示：

    ![Azure AI Foundry 门户中 Azure AI 项目详细信息的屏幕截图。](./media/ai-foundry-project.png)

## 部署模型

在本练习中，你将评估 gpt-4o-mini 模型的性能。 你还将使用 gpt-4o 模型生成 AI 辅助的评估指标。

1. 在项目左侧窗格的**我的资产**部分中，选择**模型 + 终结点**页面。
1. 在“**模型 + 终结点**”页的“**模型部署**”选项卡中，在“**+ 部署模型**”菜单中，选择“**部署基础模型**”。
1. 在列表中搜索 **gpt-4o** 模型，然后选择并确认。
1. 在部署详细信息中选择“**自定义**”，并使用以下设置部署模型：
    - **部署名**：*有效的模型部署名*
    - **部署类型**：全局标准
    - **自动版本更新**：启用
    - **模型版本**：*选择最新可用版本*
    - **连接的 AI 资源**：*选择 Azure OpenAI 资源连接*
    - **每分钟令牌限制（千令牌）**：50K *（或如果订阅的可用上限低于 50K，则以其为准）*
    - **内容筛选器**：DefaultV2

    > **注意**：减少 TPM 有助于避免过度使用正在使用的订阅中可用的配额。 50,000 TPM 足以应对本练习所需的数据处理量。 如果可用配额低于上述 50,000 TPM，你仍然可完成本练习，但如果超过速率限制，可能会出现错误。

1. 等待部署完成。
1. 返回到**模型 + 终结点**页面，重复前述步骤，部署具有相同设置的 **gpt-4o-mini** 模型。

## 手动评估模型

可以根据测试数据手动审阅模型响应。 手动审阅可以让你每次测试不同的输入，以评估模型是否按预期执行。

1. 在新的“浏览器”选项卡中，从`https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-studio/refs/heads/main/data/travel_evaluation_data.csv`中下载[travel_evaluation_data.csv](https://raw.githubusercontent.com/MicrosoftLearning/mslearn-ai-studio/refs/heads/main/data/travel_evaluation_data.csv)，并将其保存在本地文件夹中。
1. 返回到 Azure AI Foundry 门户选项卡，在导航窗格中的 **评估和改进**部分，选择**评估**。
1. 在**评估**页面中，查看**手动评估**选项卡，然后选择 **+ 新建手动评估**。
1. 将**系统消息**更改为以下 AI 旅行助理的说明：

   ```
   Objective: Assist users with travel-related inquiries, offering tips, advice, and recommendations as a knowledgeable travel agent.

   Capabilities:
   - Provide up-to-date travel information, including destinations, accommodations, transportation, and local attractions.
   - Offer personalized travel suggestions based on user preferences, budget, and travel dates.
   - Share tips on packing, safety, and navigating travel disruptions.
   - Help with itinerary planning, including optimal routes and must-see landmarks.
   - Answer common travel questions and provide solutions to potential travel issues.
    
   Instructions:
   1. Engage with the user in a friendly and professional manner, as a travel agent would.
   2. Use available resources to provide accurate and relevant travel information.
   3. Tailor responses to the user's specific travel needs and interests.
   4. Ensure recommendations are practical and consider the user's safety and comfort.
   5. Encourage the user to ask follow-up questions for further assistance.
   ```

1. 在**配置**部分的**模型**列表中，选择 **gpt-4o-mini** 模型部署。
1. 在**手动评估结果**部分中，选择**导入测试数据**，并上传之前下载的**travel_evaluation_data.csv** 文件；映射数据集字段，如下所示：
    - **输入**：问题
    - **预期响应**：ExpectedResponse
1. 审阅测试文件中的问题和预期答案 - 你将使用这些问题和答案来评估模型生成的响应。
1. 从顶部栏中选择“运行”，以便为添加为输入的所有问题生成输出。**** 几分钟后，模型的响应应显示在新的 **输出**列中，如下所示：

    ![Azure AI Foundry 门户中的手动评估页面屏幕截图。](./media/manual-evaluation.png)

1. 审阅每个问题的输出，将模型生成的输出与预期答案对比，并在每个响应右下角选择拇指向上或向下图标来“评分”结果。
1. 对响应评分后，请审阅列表上方的摘要卡片。 然后在工具栏中，选择**保存结果**并分配合适的名称。 保存结果后，你可以稍后检索，以便进一步评估或与其他模型比较。

## 使用自动评估

虽然手动比较模型输出与自己的预期响应可能是评估模型性能的有效方法，但在预期存在各种问题和响应的情况下，这种方法较为耗时；而且它提供的标准化指标的方式较少，难以用于比较不同的模型和提示组合。

自动评估是一种尝试计算指标并使用 AI 评估相应的一致性、相关性及其他因素，以解决这些不足的方法。

1. 使用**手动评估**页标题旁边的后退箭头（&larr;****），返回到**评估**页面。
1. 查看**自动评估**选项卡。
1. 选择**创建新评估**，在提示出现时，选择评估**模型和提示**的选项
1. 在**创建新评估**页的**基本信息**部分中，审阅默认自动生成的评估名（如需更改可修改），然后选择 **gpt-40-mini** 模型部署。
1. 将**系统消息**更改为之前使用的 AI 旅行助理的说明：

   ```
   Objective: Assist users with travel-related inquiries, offering tips, advice, and recommendations as a knowledgeable travel agent.

   Capabilities:
   - Provide up-to-date travel information, including destinations, accommodations, transportation, and local attractions.
   - Offer personalized travel suggestions based on user preferences, budget, and travel dates.
   - Share tips on packing, safety, and navigating travel disruptions.
   - Help with itinerary planning, including optimal routes and must-see landmarks.
   - Answer common travel questions and provide solutions to potential travel issues.
    
   Instructions:
   1. Engage with the user in a friendly and professional manner, as a travel agent would.
   2. Use available resources to provide accurate and relevant travel information.
   3. Tailor responses to the user's specific travel needs and interests.
   4. Ensure recommendations are practical and consider the user's safety and comfort.
   5. Encourage the user to ask follow-up questions for further assistance.
   ```

1. 在**配置测试数据**部分中，请注意，可以使用 GPT 模型生成测试数据（然后可以编辑和补充数据以符合你的期望）、使用现有数据集或上传文件。 在本练习中，选择**使用现有数据集**，然后选择**travel_evaluation_data_csv_*xxxx...*** 数据集（该数据集是你之前上传.csv文件时创建的）。
1. 审阅数据集中的示例行，然后在**选择数据列**部分中，选择以下列的映射：
    - **查询**：问题
    - **上下文**：*将此项留空。它用于在为模型关联上下文数据源时评估“基础性”。*
    - **基本事实**：ExpectedAnswer
1. 在**选择要评估的内容**部分中，选择以下<u>所有</u>评估类别：
    - AI 质量（AI 辅助）
    - 风险与安全（AI 辅助）
    - AI 质量 (NLP)
1. 在**选择模型部署作为判断**列表中，选择 **gpt-4o** 模型。 此模型将用于评估 ***gpt-4o-mini** 模型的响应，重点考察语言质量及标准的生成式 AI 比较指标。
1. 选择**创建**开始评估流程，等待评估完成。 可能需要几分钟时间。

    > **提示**：如果出现“正在设置项目权限”的错误提示，请稍等一分钟，然后再次选择**创建**。 新创建的项目在资源权限传播完成前可能需要一些时间。

1. 评估完成后，请向下滚动（如有必要），查看**指标仪表板**区域，并查看 **AI 质量（AI 辅助）** 指标：

    ![Azure AI Foundry 门户中的 AI 质量指标的屏幕截图。](./media/ai-quality-metrics.png)

    使用 **<sup>(i)</sup>** 图标查看指标定义。

1. 查看**风险与安全**选项卡，查看与潜在有害内容相关的各项指标。
1. 查看 **AI 质量 (NLP**) 选项卡，以查看生成式 AI 模型的标准指标。
1. 向上滚动到页面顶部（如有必要），然后选择**数据**选项卡，以查看评估中的原始数据。 数据包括每个输入的指标，以及 gpt-4o 模型在评估响应时所应用的推理说明。

    ![Azure AI Foundry 门户中评估数据的屏幕截图。](./media/evaluation-data.png)

## 清理

完成对 Azure AI Foundry 的探索后，应删除已创建的资源，以避免产生不必要的 Azure 成本。

- 导航到 Azure 门户，地址为 `https://portal.azure.com`[](https://portal.azure.com)。
- 在 Azure 门户的**主页**上，选择“资源组”。
- 选择你为本练习创建的资源组。
- 在资源组的“概述”页的顶部，选择“删除资源组”。
- 输入资源组名称以确认要删除该资源组，然后选择“删除”。
