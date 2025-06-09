---
lab:
  title: 基于自有数据构建生成式 AI 应用
  description: 了解如何使用检索增强生成 (RAG) 模型，构建基于自有数据优化提示的聊天应用。
---

# 基于自有数据构建生成式 AI 应用

检索增强生成 (RAG) 是一种用于生成应用程序的技术，这些应用程序将来自自定义数据源的数据集成到生成式 AI 模型的提示中。 RAG 是开发生成式 AI 应用（基于聊天的应用程序，使用语言模型来解释输入并生成适当的响应）的常用模式。

在本练习中，你将使用 Azure AI Foundry 将自定义数据集成到生成式 AI 解决方案中。

本练习大约需要 **45** 分钟。

> **备注**：本练习基于预发布的服务，可能会有更改。

## Azure AI Foundry 中心和项目

我们将在本练习中使用的 Azure AI Foundry 功能，需要基于 Azure AI Foundry *中心*资源的项目。

1. 在 Web 浏览器中打开 [Azure AI Foundry 门户](https://ai.azure.com)，网址为：`https://ai.azure.com`，然后使用 Azure 凭据登录。 关闭首次登录时打开的任何使用技巧或快速入门窗格，如有必要，使用左上角的 **Azure AI Foundry** 徽标导航到主页，类似下图所示（若已打开**帮助**面板，请关闭）：

    ![Azure AI Foundry 门户的屏幕截图。](./media/ai-foundry-home.png)

1. 在浏览器中，导航到 `https://ai.azure.com/managementCenter/allResources` 并选择“**创建**”。 然后选择创建新的 **AI 中心资源**的选项。
1. 在“**创建项目**”向导中，输入项目的有效名称，如果建议使用现有中心，请选择用于创建新中心的选项，然后展开“**高级选项**”，为项目指定以下设置：
    - **订阅**：Azure 订阅
    - **资源组**：*创建或选择资源组*
    - **中心名称**：中心的有效名称
    - **位置**：美国东部 2 或瑞典中部\*

    > \* 某些 Azure AI 资源受区域模型配额约束。 如果稍后在练习中达到配额限制，你可能需要在不同的区域中创建另一个资源。

1. 等待创建项目。

## 部署模型

需要两个模型来实现解决方案：

- 一个嵌入** 模型，用于矢量化文本数据以实现高效索引和处理。
- 一个可以根据数据生成对问题的自然语言回答的模型。

1. 在 Azure AI Foundry 门户的项目中，在左侧导航窗格“**我的资产**”下，选择“**模型 + 终结点**”页。
1. 在“部署模型”向导中选择“**自定义**”，并使用以下设置新建 **text-embedding-ada-002** 模型部署：

    - **部署名称**：*模型部署的有效名称*
    - **部署类型**：全局标准
    - **模型版本**：*选择默认版本*
    - **关联的 AI 资源**：*选择先前创建的资源*
    - **每分钟令牌限制（千令牌）**：50K *（或若订阅的可用上限低于 50K，则以其为准）*
    - **内容筛选器**：DefaultV2

    > **备注**：如果当前 AI 资源位置没有可用于要部署模型的配额，系统会要求你选择其他位置，以便新建 AI 资源并连接到项目。

1. 返回到**模型 + 终结点**页面，重复前述步骤，使用**全局标准**部署最新版本的 **gpt-4o** 模型，并将 TPM 速率限制设置为 **50K**（或订阅中可用的最大值，如果低于 50K）。

    > **注意**：减少每分钟令牌数 (TPM) 有助于避免正在使用的订阅中可用配额的过度使用。 50,000 TPM 对本练习中使用的数据是足够的。

## 将数据添加到项目

你的应用数据包括一组来自虚构旅行社 *Margie's Travel*PDF 格式旅行手册。 让我们将它们添加到项目。

1. 在新的浏览器标签页中，从`https://github.com/MicrosoftLearning/mslearn-ai-studio/raw/main/data/brochures.zip`下载[手册的压缩存档](https://github.com/MicrosoftLearning/mslearn-ai-studio/raw/main/data/brochures.zip)，将其解压缩到本地文件系统上名为 **brochures** 的文件夹中。
1. 在 Azure AI Foundry 门户的项目中，在左侧导航窗格“**我的资产**”下，选择“**数据 + 索引**”页。
1. 选择“+ 新建数据”。****
1. 在“添加数据”向导中，展开下拉菜单以选择“上传文件/文件夹”。********
1. 选择“**上传文件夹**”，然后上传 **brochures** 文件夹。 等到列出文件夹中的所有文件。
1. 选择“**下一步**”并将数据名称设置为 `brochures`。
1. 等待文件夹上传完成，并注意它包含多个 .pdf 文件。

## 为数据创建索引

现在，你已将数据源添加到项目中，可以使用它在 Azure AI 搜索资源中创建索引了。

1. 在 Azure AI Foundry 门户的项目中，在左侧导航窗格“**我的资产**”下，选择“**数据 + 索引**”页。
1. 在“**索引**”选项卡中，使用以下设置添加新索引：
    - **源位置**：
        - **数据源**：Azure AI Foundry 中的数据
            - 选择 brochures**** 数据源**
    - **索引配置**：
        - **选择 Azure AI 搜索服务**：*使用以下设置创建新的 Azure AI 搜索资源*：
            - **订阅**：*Azure 订阅*
            - **资源组**：*与 AI 中心相同的资源组*
            - **服务名称**：*AI 搜索资源的有效名称*
            - **位置**：*与 AI 中心相同的位置*
            - 定价层：基本
            
            等待 AI 搜索资源创建完毕。 然后返回到 Azure AI Foundry，通过选择“**连接其他 Azure AI 搜索资源**”并将连接添加到刚刚创建的 AI 搜索资源来完成索引配置。
 
        - **矢量索引**：`brochures-index`
        - **虚拟机**：自动选择
    - 搜索设置****：
        - 矢量设置****：向此搜索资源添加矢量搜索
        - **Azure OpenAI 连接**：*为中心选择默认 Azure OpenAI 资源。*
        - **嵌入模型**：text-embedding-ada-002
        - **嵌入模型部署**：*部署* text-embedding-ada-002 *模型*

1. 创建向量索引并等待索引过程完成，这可能需要一段时间，具体取决于订阅中的可用计算资源。

    索引创建操作包含以下作业：

    - 将文本标记破解、分块，然后将其嵌入到“手册”数据中。
    - 创建 Azure AI 搜索索引。
    - 注册索引资产。

    > **提示**：等待创建索引时，不妨查看下载的手册以熟悉其内容。

## 在操场中测试索引

在基于 RAG 的提示流中使用索引之前，我们先验证它是否可用于影响生成式 AI 响应。

1. 在左侧导航窗格中，选择“**操场**”页，然后打开“**聊天**”操场。
1. 在“聊天操场”页面的“设置”窗格中，确认已选择**gpt-4o**模型部署。 然后，在主聊天会话面板中提交提示“`Where can I stay in New York?`”
1. 查看响应，该响应应该是来自模型的通用答案，没有来自索引的任何数据。
1. 在“设置”窗格中，展开“**添加数据**”字段，然后添加 **brochures-index** 项目索引，并选择“**混合（矢量 + 关键字）**”搜索类型。

   > **提示**：在某些情况下，新建的索引可能无法立即使用。 刷新浏览器通常很有帮助，但如果仍然遇到找不到索引的问题，可能需要等到索引被识别。

1. 添加索引并重启聊天会话后，重新提交提示“`Where can I stay in New York?`”
1. 查看响应，该响应应基于索引中的数据。

<!-- DEPRECATED STEPS

## Create a RAG client app with the Azure AI Foundry and Azure OpenAI SDKs

Now that you have a working index, you can use the Azure AI Foundry and Azure OpenAI SDKs to implement the RAG pattern in a client application. Let's explore the code to accomplish this in a simple example.

> **Tip**: You can choose to develop your RAG solution using Python or Microsoft C#. Follow the instructions in the appropriate section for your chosen language.

### Prepare the application configuration

1. In the Azure AI Foundry portal, view the **Overview** page for your project.
1. In the **Project details** area, note the **Project connection string**. You'll use this connection string to connect to your project in a client application.
1. Return to the browser tab containing the Azure portal (keeping the Azure AI Foundry portal open in the existing tab).
1. Use the **[\>_]** button to the right of the search bar at the top of the page to create a new Cloud Shell in the Azure portal, selecting a ***PowerShell*** environment with no storage in your subscription.

    The cloud shell provides a command-line interface in a pane at the bottom of the Azure portal. You can resize or maximize this pane to make it easier to work in.

    > **Note**: If you have previously created a cloud shell that uses a *Bash* environment, switch it to ***PowerShell***.

1. In the cloud shell toolbar, in the **Settings** menu, select **Go to Classic version** (this is required to use the code editor).

    **<font color="red">Ensure you've switched to the classic version of the cloud shell before continuing.</font>**

1. In the cloud shell pane, enter the following commands to clone the GitHub repo containing the code files for this exercise (type the command, or copy it to the clipboard and then right-click in the command line and paste as plain text):

    ```
    rm -r mslearn-ai-foundry -f
    git clone https://github.com/microsoftlearning/mslearn-ai-studio mslearn-ai-foundry
    ```

    > **Tip**: As you paste commands into the cloudshell, the output may take up a large amount of the screen buffer. You can clear the screen by entering the `cls` command to make it easier to focus on each task.

1. After the repo has been cloned, navigate to the folder containing the chat application code files:

    > **Note**: Follow the steps for your chosen programming language.

    **Python**

    ```
   cd mslearn-ai-foundry/labfiles/rag-app/python
    ```

    **C#**

    ```
   cd mslearn-ai-foundry/labfiles/rag-app/c-sharp
    ```

1. In the cloud shell command-line pane, enter the following command to install the libraries you'll use:

    **Python**

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install python-dotenv azure-ai-projects azure-identity openai
    ```

    **C#**

    ```
   dotnet add package Azure.Identity
   dotnet add package Azure.AI.Projects --prerelease
   dotnet add package Azure.AI.OpenAI --prerelease
    ```
    

1. Enter the following command to edit the configuration file that has been provided:

    **Python**

    ```
   code .env
    ```

    **C#**

    ```
   code appsettings.json
    ```

    The file is opened in a code editor.

1. In the code file, replace the following placeholders: 
    - **your_project_connection_string**: Replace with the connection string for your project (copied from the project **Overview** page in the Azure AI Foundry portal).
    - **your_gpt_model_deployment** Replace with the name you assigned to your **gpt-4o** model deployment.
    - **your_embedding_model_deployment**: Replace with the name you assigned to your **text-embedding-ada-002** model deployment.
    - **your_index**: Replace with your index name (which should be `brochures-index`).
1. After you've replaced the placeholders, in the code editor, use the **CTRL+S** command or **Right-click > Save** to save your changes and then use the **CTRL+Q** command or **Right-click > Quit** to close the code editor while keeping the cloud shell command line open.

### Explore code to implement the RAG pattern

1. Enter the following command to edit the code file that has been provided:

    **Python**

    ```
   code rag-app.py
    ```

    **C#**

    ```
   code Program.cs
    ```

1. Review the code in the file, noting that it:
    - Uses the Azure AI Foundry SDK to connect to your project (using the project connection string)
    - Creates an authenticated Azure OpenAI client from your project connection.
    - Retrieves the default Azure AI Search connection from your project so it can determine the endpoint and key for your Azure AI Search service.
    - Creates a suitable system message.
    - Submits a prompt (including the system and a user message based on the user input) to the Azure OpenAI client, adding:
        - Connection details for the Azure AI Search index to be queried.
        - Details of the embedding model to be used to vectorize the query\*.
    - Displays the response from the grounded prompt.
    - Adds the response to the chat history.

    \* *The query for the search index is based on the prompt, and is used to find relevant text in the indexed documents. You can use a keyword-based search that submits the query as text, but using a vector-based search can be more efficient - hence the use of an embedding model to vectorize the query text before submitting it.*

1. Use the **CTRL+Q** command to close the code editor without saving any changes, while keeping the cloud shell command line open.

### Run the chat application

1. In the cloud shell command-line pane, enter the following command to run the app:

    **Python**

    ```
   python rag-app.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. When prompted, enter a question, such as `Where should I go on vacation to see architecture?` and review the response from your generative AI model.

    Note that the response includes source references to indicate the indexed data in which the answer was found.

1. Try a follow-up question, for example `Where can I stay there?`

1. When you're finished, enter `quit` to exit the program. Then close the cloud shell pane.

-->

## 清理

为了避免不必要的 Azure 成本和资源使用，应删除在本练习中部署的资源。

1. 如果已探索完 Azure AI Foundry，请返回到 [Azure 门户](https://portal.azure.com) (`https://portal.azure.com`)，并根据需要使用 Azure 凭据登录。 然后删除你在其中预配了 Azure AI 搜索和 Azure AI 资源的资源组中的资源。
