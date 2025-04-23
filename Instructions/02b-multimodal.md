---
lab:
  title: 开发多模式生成式 AI 应用
  description: 了解如何使用 Azure AI Foundry 生成支持文本、图像和音频输入的生成式 AI 应用。
---

# 开发多模式生成式 AI 应用

在本练习中，你将使用 *Phi-4-multimodal-instruct* 生成式 AI 模型来生成对包含文本、图像和音频的提示的响应。 你将开发一款应用，该应用使用 Azure AI Foundry 和 Azure AI 模型推理服务为杂货店中的新鲜农产品提供 AI 帮助。

此练习大约需要 **30** 分钟。

> **备注**：本练习基于预发布 SDK，可能会有更改。 必要时，我们使用了特定版本的包；这可能没有反映最新的可用版本。 可能会遇到一些意想不到的行为、警告或错误。

## 创建 Azure AI Foundry 项目

让我们首先创建 Azure AI Foundry 项目。

1. 在 Web 浏览器中打开 [Azure AI Foundry 门户](https://ai.azure.com)，网址为：`https://ai.azure.com`，然后使用 Azure 凭据登录。 关闭首次登录时打开的任何使用技巧或快速入门窗格，如有必要，请使用左上角的 **Azure AI Foundry** 徽标导航到主页，如下图所示：

    ![Azure AI Foundry 门户的屏幕截图。](./media/ai-foundry-home.png)

2. 在主页中，选择“**+ 创建项目**”。
3. 在**创建项目**向导中，输入项目的有效名，如果出现建议使用现有中心的提示，请选择新建中心的选项。 然后查看将自动创建的 Azure 资源以支持中心和项目。
4. 选择“**自定义**”并为中心指定以下设置：
    - **中心名称**：*中心的有效名称*
    - **订阅**：Azure 订阅
    - **资源组**：*创建或选择资源组*
    - **位置**：选择以下任一区域\*：
        - 美国东部
        - 美国东部 2
        - 美国中北部
        - 美国中南部
        - 瑞典中部
        - 美国西部
        - 美国西部 3
    - **连接 Azure AI 服务或 Azure OpenAI**：*新建 AI 服务资源*
    - **连接 Azure AI 搜索**：跳过连接

    > \*撰写本文时，我们将在本练习中使用的 *Phi-4-multimodal-instruct* 模型在这些区域中可用。 可以在 [Azure AI Foundry 文档](https://learn.microsoft.com/azure/ai-foundry/how-to/deploy-models-serverless-availability#region-availability)中查看特定模型的最新区域可用性。 如果在稍后的练习中达到区域配额限制，则可能需要在其他区域中创建另一个资源。

5. 选择“**下一步**”查看配置。 然后，选择“**创建**”并等待该进程完成。
6. 创建项目后，关闭显示的所有使用技巧，并查看 Azure AI Foundry 门户中的项目页面，如下图所示：

    ![Azure AI Foundry 门户中 Azure AI 项目详细信息的屏幕截图。](./media/ai-foundry-project.png)

## 部署模型

现在，你已准备好部署 *Phi-4-multimodal-instruct* 模型以支持多模式提示。

1. 在 Azure AI Foundry 项目页右上角的工具栏中，使用**预览功能** (**&#9215;**) 图标启用**将模型部署到 Azure AI 模型推理服务**功能。 此功能可确保模型部署可供 Azure AI 推理服务使用，你可在应用程序代码中使用该服务。
2. 在项目左侧窗格的“**我的资产**”部分中，选择“**模型 + 终结点**”页。
3. 在“**模型 + 终结点**”页的“**模型部署**”选项卡中，在“**+ 部署模型**”菜单中，选择“**部署基础模型**”。
4. 在列表中搜索“**Phi-4-multimodal-instruct**”模型，然后选择并确认此模型。
5. 如果出现提示，请同意许可协议，然后在部署详细信息中选择“**自定义**”并使用以下设置部署模型：
    - **部署名**：*有效的模型部署名*
    - **部署类型**：全局标准
    - **部署详细信息**：*使用默认设置*
6. 等待部署预配状态为“**完成**”。

## 创建客户端应用程序

部署模型后，可以在客户端应用程序中使用部署。

> **提示**：可以选择使用 Python 或 Microsoft C# 开发解决方案。 按照所选语言的相应部分中的说明进行操作。

### 准备应用程序配置

1. 在 Azure AI Foundry 门户中，查看项目的“**概述**”页。
2. 在“**项目详细信息**”区域中，记下**项目连接字符串**。 你将使用此连接字符串连接到客户端应用程序中的项目。
3. 打开新的浏览器选项卡（使 Azure AI Foundry 门户在现有选项卡中保持打开状态）。 然后在新选项卡中，浏览到 [Azure 门户](https://portal.azure.com)，网址为：`https://portal.azure.com`；如果出现提示，请使用 Azure 凭据登录。

    关闭任何欢迎通知以查看 Azure 门户主页。

1. 使用页面顶部搜索栏右侧的 **[\>_]** 按钮在 Azure 门户中新建 Cloud Shell，选择订阅中不含存储的 ***PowerShell*** 环境。

    在 Azure 门户底部的窗格中，Cloud Shell 提供命令行接口。 可以调整此窗格的大小或最大化此窗格，以便更易于使用。

    > **备注**：如果以前创建了使用 *Bash* 环境的 Cloud Shell，请将其切换到 ***PowerShell***。

5. 在 Cloud Shell 工具栏的“**设置**”菜单中，选择“**转到经典版本**”（这是使用代码编辑器所必需的）。

    **<font color="red">在继续作之前，请确保已切换到 Cloud Shell 的经典版本。</font>**

1. 在 Cloud Shell 窗格中，输入以下命令以克隆包含此练习代码文件的 GitHub 存储库（键入命令，或将其复制到剪贴板后，在命令行中右键单击并粘贴为纯文本）：

    ```
    rm -r mslearn-ai-foundry -f
    git clone https://github.com/microsoftlearning/mslearn-ai-studio mslearn-ai-foundry
    ```

    > **提示**：将命令粘贴到 Cloudshell 中时，输出可能会占用大量屏幕缓冲区。 可以通过输入 `cls` 命令来清除屏幕，以便更轻松地专注于每项任务。

7. 克隆存储库后，导航到包含应用程序代码文件的文件夹：  

    **Python**

    ```
   cd mslearn-ai-foundry/labfiles/multimodal/python
    ```

    **C#**

    ```
   cd mslearn-ai-foundry/labfiles/multimodal/c-sharp
    ```

8. 在 Cloud Shell 命令行窗格中，输入以下命令以安装将要使用的库：

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

9. 输入以下命令以编辑已提供的配置文件：

    **Python**

    ```
   code .env
    ```

    **C#**

    ```
   code appsettings.json
    ```

    该文件已在代码编辑器中打开。

10. 在代码文件中，将 **your_project_connection_string** 占位符替换为项目的连接字符串（从 Azure AI Foundry 门户中的项目“**概述**”页复制），并将 **your_model_deployment** 占位符替换为分配给 Phi-4-multimodal-instruct 模型部署的名称。
11. 替换占位符后，在代码编辑器中使用 “CTRL+S”**** 命令或“ 右键单击 > 保存”**** 保存更改，然后使用 “CTRL+Q”**** 命令或 “右键单击 > 退出”**** 关闭代码编辑器，同时保持 Cloud Shell 命令行打开。

### 写入代码以连接到项目并获取模型的聊天客户端

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

2. 在代码文件中，请注意在文件顶部添加的现有语句，以导入必要的 SDK 命名空间。 然后，在注释“**添加引用**”下，添加以下代码以引用之前安装的库中的命名空间：

    **Python**

    ```python
   # Add references
   from dotenv import load_dotenv
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   from azure.ai.inference.models import (
       SystemMessage,
       UserMessage,
       TextContentItem,
       ImageContentItem,
       ImageUrl,
       AudioContentItem,
       InputAudio,
       AudioContentFormat,
   )
    ```

    **C#**

    ```csharp
   // Add references
   using Azure.Identity;
   using Azure.AI.Projects;
   using Azure.AI.Inference;
    ```

3. 在 **main** 函数的注释“**获取配置设置**”下，请注意，代码将加载配置文件中定义的项目连接字符串和模型部署名称值。
4. 在注释**初始化项目客户端**下，添加以下代码，使用当前登录所使用的 Azure 凭据连接到 Azure AI Foundry 项目：

    **Python**

    ```python
   # Get configuration settings
   project_client = AIProjectClient.from_connection_string(
        conn_str=project_connection,
        credential=DefaultAzureCredential())
    ```

    **C#**

    ```csharp
   // Get configuration settings
   var projectClient = new AIProjectClient(project_connection,
                        new DefaultAzureCredential());
    ```

5. 在注释“**获取聊天客户端**”下，添加以下代码以创建与模型聊天的客户端对象：

    **Python**

    ```python
   # Get a chat client
   chat_client = project_client.inference.get_chat_completions_client(model=model_deployment)
    ```

    **C#**

    ```csharp
   // Get a chat client
   ChatCompletionsClient chat = projectClient.GetChatCompletionsClient();
    ```


### 写入代码以使用基于文本的提示

1. 请注意，代码包含一个循环，允许用户输入提示，直到输入“退出”。 然后，在循环部分的注释“**获取对文本输入的响应**”下，添加以下代码以提交基于文本的提示并从模型中检索响应：

    **Python**

    ```python
   # Get a response to text input
   response = chat_client.complete(
       messages=[
           SystemMessage(system_message),
           UserMessage(content=[TextContentItem(text= prompt)])
       ])
   print(response.choices[0].message.content)
    ```

    **C#**

    ```csharp
   // Get a response to text input
   var requestOptions = new ChatCompletionsOptions()
   {
   Model = model_deployment,
   Messages =
       {
           new ChatRequestSystemMessage(system_message),
           new ChatRequestUserMessage(prompt),
       }
   };

   Response<ChatCompletions> response = chat.Complete(requestOptions);
   Console.WriteLine(response.Value.Content);
    ```

2. 使用 **Ctrl+S** 命令保存对代码文件的更改 - 请勿先关闭。

3. 在代码编辑器下方的 Cloud Shell 命令行窗格中，输入以下命令以运行应用：

    **Python**

    ```
   python chat-app.py
    ```

    **C#**

    ```
   dotnet run
    ```

4. 出现提示时，输入`1`以使用基于文本的提示，然后输入提示`I want to make an apple pie. What kind of apple should I use?`
5. 查看回应。 然后输入`quit`以退出程序。

### 写入代码以使用基于图像的提示

1. 在 **chat-app.py** 文件的代码编辑器中，在循环部分的注释“**获取对图像输入的响应**”下，添加以下代码以提交包含以下图像的提示：

    ![桔子的照片。](../labfiles/multimodal/orange.jpg)

    **Python**

    ```python
   # Get a response to image input
   image_url = "https://github.com/microsoftlearning/mslearn-ai-studio/raw/refs/heads/main/labfiles/multimodal/orange.jpg"
   image_format = "jpeg"
   request = Request(image_url, headers={"User-Agent": "Mozilla/5.0"})
   image_data = base64.b64encode(urlopen(request).read()).decode("utf-8")
   data_url = f"data:image/{image_format};base64,{image_data}"

   response = chat_client.complete(
       messages=[
           SystemMessage(system_message),
           UserMessage(content=[
               TextContentItem(text=prompt),
               ImageContentItem(image_url=ImageUrl(url=data_url))
           ]),
       ]
   )
   print(response.choices[0].message.content)
    ```

    **C#**

    ```csharp
  // Get a response to image input
   string imageUrl = "https://github.com/microsoftlearning/mslearn-ai-studio/raw/refs/heads/main/labfiles/multimodal/orange.jpg";
   ChatCompletionsOptions requestOptions = new ChatCompletionsOptions()
   {
       Messages = {
           new ChatRequestSystemMessage(system_message),
           new ChatRequestUserMessage([
               new ChatMessageTextContentItem(prompt),
               new ChatMessageImageContentItem(new Uri(imageUrl))
           ]),
       },
       Model = model_deployment
   };
   var response = chat.Complete(requestOptions);
   Console.WriteLine(response.Value.Content);
    ```

2. 使用 **Ctrl+S** 命令保存对代码文件的更改 - 请勿先关闭。

3. 在代码编辑器下方的 Cloud Shell 命令行窗格中，输入以下命令以运行应用：

    **Python**

    ```
   python chat-app.py
    ```

    **C#**

    ```
   dotnet run
    ```

4. 出现提示时，输入`2`以使用基于图像的提示，然后输入提示`I don't know what kind of fruit this is. Can you identify it, and tell me what kinds of food I could make with it?`
5. 查看回应。 然后输入`quit`以退出程序。

### 写入代码以使用基于音频的提示

1. 在 **chat-app.py** 文件的代码编辑器中，在循环部分的注释“**获取对音频输入的响应**”下，添加以下代码以提交包含以下音频的提示：

    <video controls src="./media/manzanas.mp4" title="时间是 2:15" width="150"></video>

    **Python**

    ```python
   # Get a response to audio input
   file_path="https://github.com/microsoftlearning/mslearn-ai-studio/raw/refs/heads/main/labfiles/multimodal/manzanas.mp3"
   response = chat_client.complete(
           messages=[
               SystemMessage(system_message),
               UserMessage(
                   [
                       TextContentItem(text=prompt),
                       {
                           "type": "audio_url",
                           "audio_url": {"url": file_path}
                       }
                   ]
               )
           ]
       )
   print(response.choices[0].message.content)
    ```

    **C#**

    ```csharp
   // Get a response to audio input
   string audioUrl="https://github.com/microsoftlearning/mslearn-ai-studio/raw/refs/heads/main/labfiles/multimodal/manzanas.mp3";
   var requestOptions = new ChatCompletionsOptions()
   {
       Messages =
       {
           new ChatRequestSystemMessage(system_message),
           new ChatRequestUserMessage(
               new ChatMessageTextContentItem(prompt),
               new ChatMessageAudioContentItem(new Uri(audioUrl))),
       },
       Model = model_deployment
   };
   var response = chat.Complete(requestOptions);
   Console.WriteLine(response.Value.Content);
    ```

2. 使用 **Ctrl+S** 命令保存对代码文件的更改。 还可以根据需要关闭代码编辑器 (**CTRL+Q**)。

3. 在代码编辑器下方的 Cloud Shell 命令行窗格中，输入以下命令以运行应用：

    **Python**

    ```
   python chat-app.py
    ```

    **C#**

    ```
   dotnet run
    ```

4. 出现提示时，输入`3`以使用基于音频的提示，然后输入提示`What is this customer saying in English?`
5. 查看回应。
6. 可以继续运行应用，选择不同的提示类型并尝试不同的提示。 完成后，输入`quit`退出程序。

    如果有时间，可以修改代码以使用不同的系统提示以及自己的可通过 Internet 访问的图像和音频文件。

    > **备注**：在此简单应用中，我们尚未实现用于保留对话历史记录的逻辑；因此模型会将每个提示视为一个新请求，且没有上一提示的上下文。

## 总结

在本练习中，你使用 Azure AI Foundry 和 Azure AI 推理 SDK 创建客户端应用程序，该应用程序使用多模式模型来生成对文本、图像和音频的响应。

## 清理

如果已完成对 Azure AI Foundry 的探索，则应删除在本练习中创建的资源，以避免产生不必要的 Azure 成本。

1. 返回到包含 Azure 门户的浏览器选项卡（或在新的浏览器选项卡中重新打开 [Azure 门户](https://portal.azure.com)，网址为：`https://portal.azure.com`），查看已在其中部署本练习中使用的资源的资源组内容。
1. 在工具栏中，选择“删除资源组”****。
1. 输入资源组名称，并确认要删除该资源组。
