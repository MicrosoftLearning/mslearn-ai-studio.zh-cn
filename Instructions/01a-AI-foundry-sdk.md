---
lab:
  title: 使用 Azure AI Foundry SDK 创建聊天应用
---

# 使用 Azure AI Foundry SDK 创建聊天应用

在本练习中，你将使用 Azure AI Foundry SDK 创建一个简单的聊天应用。

此练习大约需要 **20** 分钟。

## 创建 Azure AI Foundry 项目

首先创建 Azure AI Foundry 项目。

1. 在 Web 浏览器中打开 [Azure AI Foundry 门户](https://ai.azure.com)，网址为：`https://ai.azure.com`，然后使用 Azure 凭据登录。 关闭首次登录时打开的任何使用技巧或快速入门窗格，如有必要，请使用左上角的 **Azure AI Foundry** 徽标导航到主页，如下图所示：

    ![Azure AI Foundry 门户的屏幕截图。](./media/ai-foundry-home.png)

1. 在主页中，选择“**+ 创建项目**”。
1. 在“**创建项目**”向导中，输入合适的项目名称（例如 `my-ai-project`），然后查看将自动创建用来支持项目的 Azure 资源。
1. 选择“**自定义**”并为中心指定以下设置：
    - **中心名称**：*唯一名称 - 例如 `my-ai-hub`*
    - **订阅**：Azure 订阅
    - **资源组**：*创建具有唯一名称的新资源组（例如 `my-ai-resources`），或选择现有资源组*
    - **位置**：从以下列表\*中选择一个随机区域：
        - 美国东部
        - 美国东部 2
        - 美国中北部
        - 美国中南部
        - 瑞典中部
        - 美国西部
        - 美国西部 3
    - **连接 Azure AI 服务或 Azure OpenAI**：*新建 AI 服务资源并提供适当的名称（例如 `my-ai-services`）或使用现有资源*
    - **连接 Azure AI 搜索**：跳过连接

    > \*模型配额在租户级别受区域配额的限制。 选择随机区域有助于在多个用户使用同一租户时分配配额可用性。 如果稍后在练习中达到配额限制，你可能需要在不同的区域中创建另一个资源。

1. 选择“**下一步**”查看配置。 然后，选择“**创建**”并等待该进程完成。
1. 创建项目后，关闭显示的所有使用技巧，并查看 Azure AI Foundry 门户中的项目页面，如下图所示：

    ![Azure AI Foundry 门户中 Azure AI 项目详细信息的屏幕截图。](./media/ai-foundry-project.png)

## 部署生成式 AI 模型

现在，可随时部署生成式 AI 语言模型以支持聊天应用程序。 在此示例中，你将使用 Microsoft Phi-4 模型；但任何模型的原理都是相同的。

1. 在项目左侧窗格的“**我的资产**”部分中，选择“**模型 + 终结点**”页。
1. 在“**模型 + 终结点**”页的“**模型部署**”选项卡中，在“**+ 部署模型**”菜单中，选择“**部署基础模型**”。
1. 在列表中搜索 **Phi-4** 模型，然后选择并确认该模型。
1. 选择“**具有 Azure AI 内容安全的无服务器 API**”的部署选项。
1. 在部署详细信息中选择“**自定义**”，并使用以下设置部署模型：
    - **部署名称**：*模型部署的唯一名称 - 例如 `phi-4-model`（请记住你分配的名称，稍后将需要它*）
    - **内容筛选器**：已启用

## 创建客户端应用程序以与模型聊天

部署模型后，可以使用 Azure AI Foundry SDK 开发与之聊天的应用程序。

### 准备应用程序配置

1. 在 Azure AI Foundry 门户中，查看项目的“**概述**”页。
1. 在“**项目详细信息**”区域中，记下**项目连接字符串**。 你将使用此连接字符串连接到客户端应用程序中的项目。
1. 打开新的浏览器选项卡（使 Azure AI Foundry 门户在现有选项卡中保持打开状态）。 然后在新选项卡中，浏览到 [Azure 门户](https://portal.azure.com)，网址为：`https://portal.azure.com`；如果出现提示，请使用 Azure 凭据登录。
1. 使用页面顶部搜索栏右侧的 **[\>_]** 按钮在 Azure 门户中创建新的 Cloud Shell，选择 ***PowerShell*** 环境。 Cloud Shell 在 Azure 门户底部的窗格中提供命令行接口。

    > **备注**：如果以前创建了使用 *Bash* 环境的 Cloud Shell，请将其切换到 ***PowerShell***。

1. 在 Cloud Shell 工具栏的“**设置**”菜单中，选择“**转到经典版本**”（这是使用代码编辑器所必需的）。

1. 在 PowerShell 窗格中，输入以下命令以克隆包含此练习的 GitHub 存储库：

    ```
    rm -r mslearn-ai-foundry -f
    git clone https://github.com/microsoftlearning/mslearn-ai-studio mslearn-ai-foundry
    ```

1. 克隆存储库后，导航到 **mslearn-ai-foundry/labfiles/01a-azure-foundry-sdk/python** 文件夹：

    ```
    cd mslearn-ai-foundry/labfiles/01a-azure-foundry-sdk/python
    ```

1. 在 Cloud Shell 命令行窗格中，输入以下命令，以安装将使用的 Python 库，包括：
    - **python-dotenv**：用于从应用程序配置文件加载设置。
    - **azure-identity**：用于使用 Entra ID 凭据进行验证。
    - **azure-ai-projects**：用于处理 Azure AI Foundry 项目。
    - **azure-ai-inference**：用于与生成式 AI 模型聊天。

    ```
    pip install python-dotenv azure-identity azure-ai-projects azure-ai-inference
    ```

1. 输入以下命令，以编辑已提供的 **.env** Python 配置文件：

    ```
    code .env
    ```

    在代码编辑器中，打开文件。

1. 在代码文件中，将 **your_project_endpoint** 占位符替换为项目的连接字符串（从 Azure AI Foundry 门户中的项目“**概述**”页复制），并将 **your_model_deployment** 占位符替换为分配给 Phi-4 模型部署的名称。
1. 替换占位符后，使用 **Ctrl+S** 命令保存更改，然后使用 **Ctrl+Q** 命令关闭代码编辑器，同时使 Cloud Shell 命令行保持打开状态。

### 写入代码以连接到项目并与模型聊天

> **提示**：向 Python 代码文件添加代码时，请务必保持正确的缩进。

1. 输入以下命令，以编辑已提供的 **chat-app.py** Python 代码文件：

    ```
    code chat-app.py
    ```

1. 在代码文件中，请注意文件顶部已添加的现有**导入**语句。 然后，在注释“**# 添加 AI 项目引用**”下，添加以下代码以引用 Azure AI 项目库：

    ```python
    from azure.ai.projects import AIProjectClient
    ```

1. 在 **main** 函数的注释“**# 获取配置设置**”下，请注意，代码将加载 **.env** 文件中定义的项目连接字符串和模型部署名称值。
1. 在注释“**# 初始化项目客户端**”下，添加以下代码，以使用当前登录所使用的 Azure 凭据连接到 Azure AI Foundry 项目：

    ```python
    project = AIProjectClient.from_connection_string(
        conn_str=project_connection,
        credential=DefaultAzureCredential()
        )
    ```
    
1. 在注释“**# 获取聊天客户端**”下，添加以下代码，以创建与模型聊天的客户端对象：

    ```python
    chat = project.inference.get_chat_completions_client()
    ```

1. 请注意，代码包含一个循环，允许用户输入提示，直到输入“退出”。 然后，在循环部分的注释“**# 获取聊天补全**”下，添加以下代码，以提交提示并从模型检索补全：

    ```python
    response = chat.complete(
        model=model_deployment,
        messages=[
            {"role": "system", "content": "You are a helpful AI assistant that answers questions."},
            {"role": "user", "content": input_text},
            ],
        )
    print(response.choices[0].message.content)
    ```

1. 使用 **Ctrl+S** 命令保存对代码文件的更改，然后使用 **Ctrl+Q** 命令关闭代码编辑器，同时保持 Cloud Shell 命令行处于打开状态。

### 运行聊天应用程序

1. 在 Cloud Shell 命令行窗格中，输入以下命令以运行 Python 代码：

    ```
    python chat-app.py
    ```

1. 出现提示时，输入问题，例如 `What is the fastest animal on Earth?`，并查看生成式 AI 模型的回复。
1. 再尝试输入一些问题。 完成后，输入 `quit` 退出程序。

## 总结

在本练习中，使用 Azure AI Foundry SDK，为在 Azure AI Foundry 项目中部署的生成式 AI 模型创建客户端应用程序。

## 清理

如果已完成对 Azure AI Foundry 门户的探索，则应删除在本练习中创建的资源，以避免产生不必要的 Azure 成本。

1. 返回包含 Azure 门户的浏览器标签页（或在新浏览器标签页中重新打开 `https://portal.azure.com` 中的[Azure 门户](https://portal.azure.com)），并查看部署了本练习所用资源的资源组的内容。
1. 在工具栏中，选择“删除资源组”****。
1. 输入资源组名称，并确认要删除该资源组。
