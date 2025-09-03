---
lab:
  title: 创建生成式 AI 聊天应用
  description: 了解如何使用 Azure AI Foundry SDK 生成连接到项目并与语言模型聊天的应用。
---

# 创建生成式 AI 聊天应用

在本练习中，你将使用 Azure AI Foundry Python SDK 创建一个简单的聊天应用程序，该应用程序会连接到某个项目并与语言模型进行聊天。

> **注意**：本练习基于预发布版 SDK 软件，未来可能会有所变动。 必要时，我们使用了特定版本的包；这可能没有反映最新的可用版本。 可能会遇到一些意想不到的行为、警告或错误。

尽管本练习基于 Azure AI Foundry Python SDK，但你也可以使用多种语言特定的 SDK 开发 AI 聊天应用程序，包括：

- [适用于 Python 的 Azure AI 项目](https://pypi.org/project/azure-ai-projects)
- [适用于 Microsoft .NET 的 Azure AI 项目](https://www.nuget.org/packages/Azure.AI.Projects)
- [适用于 JavaScript 的 Azure AI 项目](https://www.npmjs.com/package/@azure/ai-projects)

此练习大约需要 **40** 分钟。

## 在 Azure AI Foundry 项目中部署模型

让我们首先在 Azure AI Foundry 项目中部署模型。

1. 在 Web 浏览器中打开 [Azure AI Foundry 门户](https://ai.azure.com)，网址为：`https://ai.azure.com`，然后使用 Azure 凭据登录。 关闭首次登录时打开的任何使用技巧或快速入门窗格，如有必要，使用左上角的 **Azure AI Foundry** 徽标导航到主页，类似下图所示（若已打开**帮助**面板，请关闭）：

    ![Azure AI Foundry 门户的屏幕截图。](./media/ai-foundry-home.png)

1. 在主页的“**浏览模型和功能**”部分中，搜索 `gpt-4o` 模型；我们将在项目中使用它。
1. 在搜索结果中，选择 **gpt-4o** 模型以查看其详细信息，然后在模型的页面顶部，选择“**使用此模型**”。
1. 当提示创建项目时，输入项目的有效名称并展开“**高级选项**”。
1. 选择“**自定义**”，并为项目指定以下设置：
    - **Azure AI Foundry 资源**：*Azure AI Foundry 资源的有效名称*
    - **订阅**：Azure 订阅
    - **资源组**：*创建或选择资源组*
    - **区域**：选择推荐的任何 AI Foundry******\*

    > \* 某些 Azure AI 资源受区域模型配额约束。 如果稍后在练习中达到配额限制，你可能需要在不同的区域中创建另一个资源。

1. 选择“**创建**”并等待创建项目。 如果出现提示，请使用“全局标准”部署类型部署 gpt-4o 模型，并自定义部署详细信息，将“每分钟标记数速率限制”设置为 50,000（如果小于 50,000，则设置为最大可用值）。********

    > **注意**：减少 TPM 有助于避免过度使用正在使用的订阅中可用的配额。 50,000 TPM 足以应对本练习所需的数据处理量。 如果可用配额低于上述 50,000 TPM，你仍然可完成本练习，但如果超过速率限制，可能会出现错误。

1. 创建项目后，聊天操场将自动打开，以便可以测试模型：
1. 在“**设置**”窗格中，记下模型部署的名称；应为 **gpt-4o**。 可以通过在“**模型和终结点**”页中查看部署来确认这一点（只需在左侧导航窗格中打开该页）。
1. 在左侧导航窗格中，选择“**概述**”以查看项目的主页；如下所示：

    ![Azure AI Foundry 项目概述页面的屏幕截图。](./media/ai-foundry-project.png)

## 创建客户端应用程序以与模型聊天

部署模型后，可以使用 Azure AI Foundry 和 Azure OpenAI SDK 开发与之聊天的应用程序。

### 准备应用程序配置

1. 在 Azure AI Foundry 门户中，查看项目的“**概述**”页。
1. 在“终结点和密钥”区域，确保已选择 Azure AI Foundry 库，并查看“Azure AI Foundry 项目终结点”************。 你将使用此终结点连接到客户端应用程序中的项目和模型。

    > **注意**：你也可以使用 Azure OpenAI 终结点！

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

1. 克隆存储库后，导航到包含聊天应用程序代码文件的文件夹进行查看：

    ```
   cd mslearn-ai-foundry/labfiles/chat-app/python
   ls -a -l
    ```

    该文件夹包含一个代码文件、一个用于应用程序设置的配置文件，以及一个定义项目运行时环境和所需依赖包的文件。

1. 在 Cloud Shell 命令行窗格中，输入以下命令以安装将要使用的库：

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-identity azure-ai-projects openai
    ```
1. 输入以下命令以编辑已提供的配置文件：

    ```
   code .env
    ```

    该文件已在代码编辑器中打开。

1. 在代码文件中，将“your_project_endpoint”占位符替换为项目的 Azure AI Foundry 项目终结点（从 Azure AI Foundry 门户中的项目“概述”页复制而来），并将“your_model_deployment”占位符替换为 gpt-4 模型部署的名称****************。
1. 替换占位符后，在代码编辑器中使用 **CTRL+S** 命令或 ** 右键单击 > 保存** 保存更改，然后使用 **CTRL+Q** 命令或 ** 右键单击 > 退出** 关闭代码编辑器，同时保持 Cloud Shell 命令行打开。

### 写入代码以连接到项目并与模型聊天

> **提示**：添加代码时，请务必保持正确的缩进。

1. 输入以下命令以编辑已提供的代码文件：

    ```
   code chat-app.py
    ```

1. 在代码文件中，请注意在文件顶部添加的现有语句，以导入必要的 SDK 命名空间。 然后，查找注释“**添加引用**”，添加以下代码以引用之前安装的库中的命名空间：

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
   from openai import AzureOpenAI
    ```

1. 在 **main** 函数的注释“**获取配置设置**”下，请注意，代码将加载配置文件中定义的项目连接字符串和模型部署名称值。
1. 找到注释“初始化项目客户端”，添加以下代码，以连接到 Azure AI Foundry 项目****：

    > **提示**：注意保持代码的正确缩进级别。

    ```python
   # Initialize the project client
   project_client = AIProjectClient(            
            credential=DefaultAzureCredential(
                exclude_environment_credential=True,
                exclude_managed_identity_credential=True
            ),
            endpoint=project_endpoint,
        )
    ```

1. 查找注释“**获取聊天客户端**”，添加以下代码，以创建与模型聊天的客户端对象：

    ```python
   # Get a chat client
   openai_client = project_client.get_openai_client(api_version="2024-10-21")
    ```

1. 查找注释“**使用系统消息初始化提示符**”，并添加以下代码以使用系统提示初始化消息集合。

    ```python
   # Initialize prompt with system message
   prompt = [
            {"role": "system", "content": "You are a helpful AI assistant that answers questions."}
        ]
    ```

1. 请注意，代码包含一个循环，允许用户输入提示，直到输入“退出”。 然后在循环部分中，查找注释“**获取聊天完成**”，并添加以下代码，将用户输入添加到提示，从模型检索完成，并将完成添加到提示（以便保留历史聊天记录以供将来迭代）：

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

1. 使用 **Ctrl+S** 命令保存对代码文件的更改。

### 登录到 Azure 并运行应用

1. 在 Cloud Shell 命令行窗格中，输入以下命令以登录到 Azure。

    ```
   az login
    ```

    **<font color="red">必须登录到 Azure - 即使 Cloud Shell 会话已经过身份验证。</font>**

    > **备注**：在大多数情况下，仅使用 *az login* 就足够了。 但是，如果在多个租户中有订阅，则可能需要使用 *--tenant* 参数指定租户。 有关详细信息，请参阅[使用 Azure CLI 以交互方式登录到 Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively)。
    
1. 出现提示时，请按照说明在新选项卡中打开登录页，并输入提供的验证码和 Azure 凭据。 然后在命令行中完成登录过程，并在出现提示时选择包含 Azure AI Foundry 中心的订阅。
1. 登录后，输入以下命令来运行应用程序：

    ```
   python chat-app.py
    ```

1. 出现提示时，输入问题，例如`What is the fastest animal on Earth?`并查看生成式 AI 模型的回复。
1. 尝试一些后续问题，例如 `Where can I see one?` 或 `Are they endangered?`。 对话应继续，使用历史聊天记录作为每次迭代的上下文。
1. 完成后，输入`quit`退出程序。

> **提示**：如果应用因超出速率限制而失败。 等待几秒钟，然后重试。 如果订阅配额不足，模型可能无法响应。

## 总结

在本练习中，你使用 Azure AI Foundry SDK 为在 Azure AI Foundry 项目中部署的生成式 AI 模型创建了客户端应用程序。

## 清理

如果已完成对 Azure AI Foundry 门户的探索，则应删除在本练习中创建的资源，以避免产生不必要的 Azure 成本。

1. 打开[Azure 门户](https://portal.azure.com)并查看在其中部署了本练习中使用的资源的资源组的内容。
1. 在工具栏中，选择“删除资源组”****。
1. 输入资源组名称，并确认要删除该资源组。
