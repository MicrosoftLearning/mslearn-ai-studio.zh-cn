---
lab:
  title: 创建生成式 AI 聊天应用
  description: 了解如何使用 Azure AI Foundry SDK 生成连接到项目并与语言模型聊天的应用。
---

# 创建生成式 AI 聊天应用

在本练习中，将使用 Azure AI Foundry SDK 创建连接到项目并与语言模型聊天的简单聊天应用。

此练习大约需要 **40** 分钟。

> **备注**：本练习基于预发布 SDK，可能会有更改。 必要时，我们使用了特定版本的包；这可能没有反映最新的可用版本。 可能会遇到一些意想不到的行为、警告或错误。

## 创建 Azure AI Foundry 项目

让我们首先创建 Azure AI Foundry 项目。

1. 在 Web 浏览器中打开 [Azure AI Foundry 门户](https://ai.azure.com)，网址为：`https://ai.azure.com`，然后使用 Azure 凭据登录。 关闭首次登录时打开的任何使用技巧或快速入门窗格，如有必要，使用左上角的 **Azure AI Foundry** 徽标导航到主页，类似下图所示（若已打开**帮助**面板，请关闭）：

    ![Azure AI Foundry 门户的屏幕截图。](./media/ai-foundry-home.png)

1. 在主页中，选择“**+ 创建项目**”。
1. 在**创建项目**向导中，输入项目的有效名，如果出现建议使用现有中心的提示，请选择新建中心的选项。 然后查看将自动创建的 Azure 资源以支持中心和项目。
1. 选择“**自定义**”并为中心指定以下设置：
    - **中心名称**：*中心的有效名称*
    - **订阅**：Azure 订阅
    - **资源组**：*创建或选择资源组*
    - **位置**：选择**帮助我选择**，然后在“位置帮助程序”窗口中选择**gpt-4o**，并使用推荐的区域\*
    - **连接 Azure AI 服务或 Azure OpenAI**：*新建 AI 服务资源*
    - **连接 Azure AI 搜索**：跳过连接

    > \* Azure OpenAI 资源受区域模型配额约束。 如果稍后在练习中达到配额限制，你可能需要在不同的区域中创建另一个资源。

1. 选择“**下一步**”查看配置。 然后，选择“**创建**”并等待该进程完成。
1. 创建项目后，关闭显示的所有使用技巧，并查看 Azure AI Foundry 门户中的项目页面，如下图所示：

    ![Azure AI Foundry 门户中 Azure AI 项目详细信息的屏幕截图。](./media/ai-foundry-project.png)

## 部署生成式 AI 模型

现在，可随时部署生成式 AI 语言模型以支持聊天应用程序。 在此示例中，你将使用 OpenAI gpt-4o 模型；不过，无论哪种模型，原理都是一样的。

1. 在 Azure AI Foundry 项目页右上角的工具栏中，使用**预览功能** (**&#9215;**) 图标启用**将模型部署到 Azure AI 模型推理服务**功能。 此功能可确保模型部署可供 Azure AI 推理服务使用，你可在应用程序代码中使用该服务。
1. 在项目左侧窗格的“**我的资产**”部分中，选择“**模型 + 终结点**”页。
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

## 创建客户端应用程序以与模型聊天

部署模型后，可以使用 Azure AI Foundry 和 Azure AI 模型推理 SDK 开发与之聊天的应用程序。

> **提示**：可以选择使用 Python 或 Microsoft C# 开发解决方案。 按照所选语言的相应部分中的说明进行操作。

### 准备应用程序配置

1. 在 Azure AI Foundry 门户中，查看项目的“**概述**”页。
1. 在“**项目详细信息**”区域中，记下**项目连接字符串**。 你将使用此连接字符串连接到客户端应用程序中的项目。
1. 打开新的浏览器选项卡（使 Azure AI Foundry 门户在现有选项卡中保持打开状态）。 然后在新选项卡中，浏览到 [Azure 门户](https://portal.azure.com)，网址为：`https://portal.azure.com`；如果出现提示，请使用 Azure 凭据登录。

    关闭任何欢迎通知以查看 Azure 门户主页。

1. 使用页面顶部搜索栏右侧的 **[\>_]** 按钮在 Azure 门户中新建 Cloud Shell，选择订阅中不含存储的 ***PowerShell*** 环境。

    在 Azure 门户底部的窗格中，Cloud Shell 提供命令行接口。 可以调整此窗格的大小或最大化此窗格，以便更易于使用。

    > **备注**：如果以前创建了使用 *Bash* 环境的 Cloud Shell，请将其切换到 ***PowerShell***。

1. 在 Cloud Shell 工具栏的“**设置**”菜单中，选择“**转到经典版本**”（这是使用代码编辑器所必需的）。

    **<font color="red">在继续作之前，请确保已切换到 Cloud Shell 的经典版本。</font>**

1. 在 Cloud Shell 窗格中，输入以下命令以克隆包含此练习代码文件的 GitHub 存储库（键入命令，或将其复制到剪贴板后，在命令行中右键单击并粘贴为纯文本）：

    ```
    rm -r mslearn-ai-foundry -f
    git clone https://github.com/microsoftlearning/mslearn-ai-studio mslearn-ai-foundry
    ```

    > **提示**：在 Cloudshell 中时输入命令时，输出可能会占用大量屏幕缓冲区。 可以通过输入 `cls` 命令来清除屏幕，以便更轻松地专注于每项任务。

1. 克隆存储库后，导航到包含聊天应用程序代码文件的文件夹：

    根据所选编程语言，使用以下命令。

    **Python**

    ```
   cd mslearn-ai-foundry/labfiles/chat-app/python
    ```

    **C#**

    ```
   cd mslearn-ai-foundry/labfiles/chat-app/c-sharp
    ```

1. 在 Cloud Shell 命令行窗格中，输入以下命令以安装将要使用的库：

    **Python**

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-identity azure-ai-projects azure-ai-inference
    ```

    **C#**

    ```
   dotnet add package Azure.Identity
   dotnet add package Azure.AI.Projects --version 1.0.0-beta.3
   dotnet add package Azure.AI.Inference --version 1.0.0-beta.3
    ```
    

1. 输入以下命令以编辑已提供的配置文件：

    **Python**

    ```
   code .env
    ```

    **C#**

    ```
   code appsettings.json
    ```

    该文件已在代码编辑器中打开。

1. 在代码文件中，将 **your_project_connection_string** 占位符替换为项目的连接字符串（从 Azure AI Foundry 门户中的项目“**概述**”页复制），并将 **your_model_deployment** 占位符替换为分配给 GPT-4 模型部署的名称。
1. 替换占位符后，在代码编辑器中使用 **CTRL+S** 命令或 ** 右键单击 > 保存** 保存更改，然后使用 **CTRL+Q** 命令或 ** 右键单击 > 退出** 关闭代码编辑器，同时保持 Cloud Shell 命令行打开。

### 写入代码以连接到项目并与模型聊天

> **提示**：添加代码时，请务必保持正确的缩进。

1. 输入以下命令以编辑已提供的代码文件：

    **Python**

    ```
   code chat-app.py
    ```

    **C#**

    ```
   code Program.cs
    ```

1. 在代码文件中，请注意在文件顶部添加的现有语句，以导入必要的 SDK 命名空间。 然后，查找注释“**添加引用**”，添加以下代码以引用之前安装的库中的命名空间：

    **Python**

    ```python
   # Add references
   from dotenv import load_dotenv
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   from azure.ai.inference.models import SystemMessage, UserMessage, AssistantMessage
    ```

    **C#**

    ```csharp
   // Add references
   using Azure.Identity;
   using Azure.AI.Projects;
   using Azure.AI.Inference;
    ```

1. 在 **main** 函数的注释“**获取配置设置**”下，请注意，代码将加载配置文件中定义的项目连接字符串和模型部署名称值。
1. 查找注释**初始化项目客户端**，然后添加以下代码，使用当前登录所使用的 Azure 凭据连接到 Azure AI Foundry 项目：

    > **提示**：注意保持代码的正确缩进级别。

    **Python**

    ```python
   # Initialize the project client
   projectClient = AIProjectClient.from_connection_string(
        conn_str=project_connection,
        credential=DefaultAzureCredential())
    ```

    **C#**

    ```csharp
   // Initialize the project client
   var projectClient = new AIProjectClient(project_connection,
                        new DefaultAzureCredential());
    ```

1. 查找注释“**获取聊天客户端**”，添加以下代码，以创建与模型聊天的客户端对象：

    **Python**

    ```python
   # Get a chat client
   chat = projectClient.inference.get_chat_completions_client()
    ```

    **C#**

    ```csharp
   // Get a chat client
   ChatCompletionsClient chat = projectClient.GetChatCompletionsClient();
    ```

    > **备注**：此代码使用 Azure AI Foundry 项目客户端创建与项目关联的默认 Azure AI 模型推理服务终结点的安全连接。 还可以使用 Azure AI 模型推理 SDK *直接*连接到终结点，方法是：对于 Azure AI Foundry 门户中的或是 Azure 门户的相应 Azure AI 服务资源页面中的服务连接，指定显示的终结点 URI，并使用身份验证密钥或 Entra 凭据令牌。 有关连接到 Azure AI 模型推理服务的详细信息，请参阅 [Azure AI 模型推理 API](https://learn.microsoft.com/azure/machine-learning/reference-model-inference-api)。

1. 查找注释“**使用系统消息初始化提示符**”，并添加以下代码以使用系统提示初始化消息集合。

    **Python**

    ```python
   # Initialize prompt with system message
   prompt=[
            SystemMessage("You are a helpful AI assistant that answers questions.")
        ]
    ```

    **C#**

    ```csharp
   // Initialize prompt with system message
   var prompt = new List<ChatRequestMessage>(){
                    new ChatRequestSystemMessage("You are a helpful AI assistant that answers questions.")
                };
    ```

1. 请注意，代码包含一个循环，允许用户输入提示，直到输入“退出”。 然后在循环部分中，查找注释“**获取聊天完成**”，并添加以下代码，将用户输入添加到提示，从模型检索完成，并将完成添加到提示（以便保留历史聊天记录以供将来迭代）：

    **Python**

    ```python
   # Get a chat completion
   prompt.append(UserMessage(input_text))
   response = chat.complete(
        model=model_deployment,
        messages=prompt)
   completion = response.choices[0].message.content
   print(completion)
   prompt.append(AssistantMessage(completion))
    ```

    **C#**

    ```csharp
   // Get a chat completion
   prompt.Add(new ChatRequestUserMessage(input_text));
   var requestOptions = new ChatCompletionsOptions()
   {
       Model = model_deployment,
       Messages = prompt
   };

   Response<ChatCompletions> response = chat.Complete(requestOptions);
   var completion = response.Value.Content;
   Console.WriteLine(completion);
   prompt.Add(new ChatRequestAssistantMessage(completion));
    ```

1. 使用 **Ctrl+S** 命令保存对代码文件的更改。

### 运行聊天应用程序

1. 在代码编辑器下方的 Cloud Shell 命令行窗格中，输入以下命令以运行应用：

    **Python**

    ```
   python chat-app.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. 出现提示时，输入问题，例如`What is the fastest animal on Earth?`并查看生成式 AI 模型的回复。
1. 尝试一些后续问题，例如 `Where can I see one?` 或 `Are they endangered?`。 对话应继续，使用历史聊天记录作为每次迭代的上下文。
1. 完成后，输入`quit`退出程序。

> **提示**：如果应用因超出速率限制而失败。 等待几秒钟，然后重试。 如果订阅配额不足，模型可能无法响应。

## 使用 Azure OpenAI SDK

客户端应用是使用 Azure AI 模型推理 SDK 生成的，这意味着它可以与部署到 Azure AI 模型推理服务的任何模型一起使用。 部署的模型是 OpenAI GPT 模型，也可以使用 OpenAI SDK 来使用此模型。

让我们进行一些代码修改，了解如何使用 OpenAI SDK 实现聊天应用程序。

1. 在代码文件夹（*python* 或 *c-sharp*）的 Cloud Shell 命令行中，输入以下命令以安装所需的包：

    **Python**

    ```
   pip install openai
    ```

    **C#**

    ```
   dotnet add package Azure.AI.Projects --version 1.0.0-beta.6
   dotnet add package Azure.AI.OpenAI --prerelease
    ```

> **备注**：需要不同的 Azure.AI.Projects 包预发行版本，作为与 Azure AI 模型推理 SDK 不兼容的临时工作区。

1. 如果代码文件（*chat-app.py* 或 *Program.cs*）尚未打开，请输入以下命令在代码编辑器中打开：

    **Python**

    ```
   code chat-app.py
    ```

    **C#**

    ```
   code Program.cs
    ```

1. 在代码文件顶部添加以下引用：

    **Python**

    ```python
   import openai
    ```

    **C#**

    ```csharp
   using OpenAI.Chat;
   using Azure.AI.OpenAI;
    ```

1. 查找注释“**获取聊天客户端**”，并修改用于创建客户端对象的代码，如下所示：

    **Python**

    ```python
   # Get a chat client 
   openai_client = projectClient.inference.get_azure_openai_client(api_version="2024-10-21")
    ```

    **C#**

    ```csharp
   // Get a chat client
   ChatClient openaiClient = projectClient.GetAzureOpenAIChatClient(model_deployment);
    ```

    > **备注**：此代码使用 Azure AI Foundry 项目客户端，创建与项目关联的默认 Azure OpenAI 服务终结点的安全连接。 还可以使用 Azure OpenAI SDK *直接*连接到终结点，方法是：对于 Azure AI Foundry 门户中的或 Azure 门户的相应 Azure OpenAI 或 AI 服务资源页面中的服务连接，指定显示的终结点 URI，并使用身份验证密钥或 Entra 凭据令牌。 有关连接到 Azure OpenAI 服务的详细信息，请参阅 [Azure OpenAI 支持的编程语言](https://learn.microsoft.com/azure/ai-services/openai/supported-languages)。

1. 查找注释“**使用系统消息初始化提示符**”，并修改代码以使用系统提示初始化消息集合，如下所示：

    **Python**

    ```python
   # Initialize prompt with system message
   prompt=[
        {"role": "system", "content": "You are a helpful AI assistant that answers questions."}
    ]
    ```

    **C#**

    ```csharp
   // Initialize prompt with system message
    var prompt = new List<ChatMessage>(){
        new SystemChatMessage("You are a helpful AI assistant that answers questions.")
    };
    ```

1. 查找注释**获取聊天完成**并修改代码，将用户输入添加到提示，从模型检索获取结果，并将结果添加到提示中，如下所示：

    **Python**

    ```python
   # Get a chat completion
   prompt.append({"role": "user", "content": input_text})
   response = openai_client.chat.completions.create(
        model=model_deployment,
        messages=prompt)
   completion = response.choices[0].message.content
   print(completion)
   prompt.append({"role": "assistant", "content": completion})
    ```

    **C#**

    ```csharp
   // Get a chat completion
   prompt.Add(new UserChatMessage(input_text));
   ChatCompletion completion = openaiClient.CompleteChat(prompt);
   var completionText = completion.Content[0].Text;
   Console.WriteLine(completionText);
   prompt.Add(new AssistantChatMessage(completionText));
    ```

1. 使用 **Ctrl+S** 命令保存对代码文件的更改。

1. 在代码编辑器下方的 Cloud Shell 命令行窗格中，输入以下命令以运行应用：

    **Python**

    ```
   python chat-app.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. 通过像以前一样提交问题来测试应用。 完成后，输入`quit`退出程序。

    > **备注**：Azure AI 模型推理 SDK 和 OpenAI SDK 使用类似的类和代码构造，因此代码需要最少的更改。 可以将 Azure AI 模型推理 SDK 与部署到 Azure AI 模型推理服务终结点的*任何*模型配合使用。 OpenAI SDK 仅适用于 OpenAI 模型，但可以将其用于部署到 Azure AI 模型推理服务终结点或 Azure OpenAI 终结点的模型。  

## 总结

在本练习中，你使用 Azure AI Foundry、Azure AI 模型推理和 Azure OpenAI SDK，为在 Azure AI Foundry 项目中部署的生成式 AI 模型创建了客户端应用程序。

## 清理

如果已完成对 Azure AI Foundry 门户的探索，则应删除在本练习中创建的资源，以避免产生不必要的 Azure 成本。

1. 返回到包含 Azure 门户的浏览器选项卡（或在新的浏览器选项卡中重新打开 [Azure 门户](https://portal.azure.com)，网址为：`https://portal.azure.com`），查看已在其中部署本练习中使用的资源的资源组内容。
1. 在工具栏中，选择“删除资源组”****。
1. 输入资源组名称，并确认要删除该资源组。
