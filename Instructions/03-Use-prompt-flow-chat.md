---
lab:
  title: 使用提示流管理聊天应用中的对话
  description: 了解如何使用提示流管理对话，并确保构建和编排提示以获得最佳结果。
---

# 使用提示流管理聊天应用中的对话

在本练习中，将使用 Azure AI Foundry 门户的提示流创建自定义聊天应用，使用用户提示和聊天历史记录作为输入，并使用 Azure OpenAI 中的 GPT 模型生成输出。

该练习大约需要 **30** 分钟。

## 创建 Azure AI Foundry 项目

让我们首先创建 Azure AI Foundry 项目。

1. 在 Web 浏览器中打开 [Azure AI Foundry 门户](https://ai.azure.com)，网址为：`https://ai.azure.com`，然后使用 Azure 凭据登录。 关闭首次登录时打开的任何使用技巧或快速入门窗格，如有必要，请使用左上角的 **Azure AI Foundry** 徽标导航到主页，如下图所示：

    ![Azure AI Foundry 门户的屏幕截图。](./media/ai-foundry-home.png)

1. 在主页中，选择“**+ 创建项目**”。
1. 在“**创建项目**”向导中，输入合适的项目名称（例如 `my-ai-project`），然后查看为支持项目而自动创建的 Azure 资源。
1. 选择“**自定义**”并为中心指定以下设置：
    - **中心名称**：*唯一名称 - 例如`my-ai-hub`*
    - **订阅**：Azure 订阅
    - **资源组**：*新建资源组并提供唯一名称（例如 `my-ai-resources`），或选择现有资源组*
    - **位置**：选择“**帮助我选择**”，然后在“位置帮助程序”窗口中选择“**gpt-4**”，并使用推荐的区域\*
    - **连接 Azure AI 服务或 Azure OpenAI**：*新建 AI 服务资源并提供适当的名称（例如 `my-ai-services`）或使用现有资源*
    - **连接 Azure AI 搜索**：跳过连接

    > \* Azure OpenAI 资源受区域配额限制在租户级别。 如果稍后在练习中达到配额限制，你可能需要在不同的区域中创建另一个资源。

1. 选择“**下一步**”查看配置。 然后，选择“**创建**”并等待该进程完成。
1. 创建项目后，关闭显示的所有使用技巧，并查看 Azure AI Foundry 门户中的项目页面，如下图所示：

    ![Azure AI Foundry 门户中 Azure AI 项目详细信息的屏幕截图。](./media/ai-foundry-project.png)

## 部署 GPT 模型

要在提示流中使用语言模型，首先需要部署模型。 Azure AI Foundry 门户允许部署可在流中使用的 OpenAI 模型。

1. 在左侧导航窗格中的“**我的资产**”下，选择“**模型 + 终结点**”页。
1. 选择“**+ 部署模型**”和“**部署基础模型**”。 
1. 在“部署详细信息”中选择“**自定义**”，并使用以下设置新建 **GPT-4** 模型部署：
    - 部署名称****：模型部署的唯一名称**
    - **部署类型**：标准
    - **模型版本**：*选择默认版本*
    - **AI 资源**：*选择先前创建的资源*
    - **每分钟令牌数速率限制（数千个）**：5K
    - **内容筛选器**：DefaultV2
    - **启用动态配额**：已禁用

    > **备注**：如果当前 AI 资源位置没有可用于要部署模型的配额，系统会要求你选择其他位置，以便新建 AI 资源并连接到项目。

1. 等待模型部署。 部署就绪后，选择“在操场中打开”****。
1. 在聊天窗口中，输入查询 `What can you do?`。

    请注意，答案是通用的，因为没有给助手特定的说明。 若要使其专注于某任务，可以更改系统提示。

1. 将“**为模型提供说明和上下文**”消息更改为以下内容：

   ```md
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
1. 在聊天窗口中，输入与之前相同的查询：`What can you do?` 请注意响应中的更改。

现在，你已熟悉了已部署的 GPT 模型的系统消息，接下来可以通过使用提示流进一步自定义应用程序。

## 在 Azure AI Foundry 门户中创建和运行聊天流

可以从模板创建新流，也可以基于操场中的配置创建流。 由于已在操场中试验，因此你将使用此选项创建新的流。

<details>  
    <summary><b>故障排除提示</b>：权限错误</summary>
    <p>如果新建提示流时收到权限错误，请尝试执行以下操作进行故障排除：</p>
    <ul>
        <li>在 Azure 门户中，选择 AI 服务资源。</li>
        <li>在“资源管理”下的“标识”选项卡中，确认它是系统分配的托管标识。</li>
        <li>导航到关联的存储帐户。 在 IAM 页上，添加角色分配<em>存储 Blob 数据读取器</em>。</li>
        <li>在“<strong>分配访问权限</strong>”下，选择“<strong>托管标识</strong>”、“<strong>+ 选择成员</strong>”，选择“<strong>所有系统分配的托管标识</strong>”，然后选择 Azure AI 服务资源。</li>
        <li>查看并分配以保存新设置，然后重试上一步。</li>
    </ul>
</details>

1. 在“聊天操场”中，从顶部栏中选择“提示流”。********
1. 输入 `Travel-Chat` 作为文件夹名称。

    会为你创建一个简单的聊天流。 请注意，有两个输入（聊天历史记录和用户的问题）、一个将与已部署的语言模型连接的 LLM 节点，以及一个输出来反映聊天中的响应。

    为了能够测试流，需要计算。

1. 从顶部栏中选择**启动计算会话**。
1. 计算会话需要 1-3 分钟才能启动。
1. 查找名为**聊天**的 LLM 节点。 请注意，提示已包含你在聊天操场中指定的系统提示。

    仍需将 LLM 节点连接到已部署的模型。

1. 在 LLM 节点部分中，对于**连接**，请选择创建 AI 中心时为你创建的连接。
1. 对于 **Api**，请选择**聊天**。
1. 对于 **deployment_name**，请选择已部署的“**GPT-4**”模型。
1. 对于 **response_format**，请选择 **{"type":"text"}**。
1. 查看提示字段并确保其如下所示：

   ```yml
   system:
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

   {% for item in chat_history %}
   user:
   {{item.inputs.question}}
   assistant:
   {{item.outputs.answer}}
   {% endfor %}

   user:
   {{question}}
   ```

### 测试和部署流

开发流后，可以使用聊天窗口测试流。

1. 确保计算会话正在运行。
1. 选择“保存”。
1. 选择**聊天**以测试流。
1. 输入查询：`I have one day in London, what should I do?` 并查看输出。

    当对创建的流的行为感到满意时，即可部署流。

1. 选择“**部署**”以使用以下设置部署流：
    - 基本设置：
        - **终结点**：新建
        - **终结点名称**：输入唯一名称**
        - **部署名称**：输入唯一名称**
        - **虚拟机**：Standard_DS3_v2
        - **实例计数**：3
        - 推理数据收集****：已启用
    - 高级设置****：
        - 使用默认设置**
1. 在 Azure AI Foundry 门户的项目中，在左侧导航窗格中“**我的资产**”下，选择“**模型 + 终结点**”页。
1. 请注意，默认情况下会列出**模型部署**，包括已部署的语言模型和已部署的流。 在列出并成功创建部署之前，可能需要一些时间。
1. 部署成功后，请选择它。 然后，在其****“测试”页上输入提示“`What is there to do in San Francisco?`”并查看响应。
1. 输入提示“`Where else could I go?`”并查看响应。
1. 查看终结点的“使用”**** 页。请注意，它包含连接信息和示例代码，你可以使用它们为终结点生成客户端应用程序，这样就能够将提示流解决方案集成到一个用作自定义 Copilot 的应用程序中。

## 删除 Azure 资源

浏览完 Azure AI Foundry 门户后，应删除已创建的资源，以避免产生不必要的 Azure 成本。

- 导航到 Azure 门户，地址为 `https://portal.azure.com`[](https://portal.azure.com)。
- 在 Azure 门户的**主页**上，选择“资源组”。
- 选择你为本练习创建的资源组。
- 在资源组的“概述”页的顶部，选择“删除资源组”。
- 输入资源组名称以确认要删除该资源组，然后选择“删除”。
