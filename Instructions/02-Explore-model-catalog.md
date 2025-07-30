---
lab:
  title: 选择和部署语言模型
  description: 生成式 AI 应用程序基于一个或多个语言模型构建。 了解如何为生成式 AI 项目查找和选择相应的模型。
---

# 选择和部署语言模型

Azure AI Foundry 模型目录充当中央存储库，在其中可浏览和使用各种模型，从而为创建生成式 AI 方案提供便利。

在本练习中，你将探索 Azure AI Foundry 门户中的模型目录，并比较有助于解决问题的生成式 AI 应用程序的潜在模型。

该练习大约需要 **25** 分钟。

> **注意**：本练习中使用的一些技术处于预览版或积极开发阶段。 可能会遇到一些意想不到的行为、警告或错误。

## 浏览模型

首先，登录到 Azure AI Foundry 门户并探索一些可用的模型。

1. 在 Web 浏览器中打开 [Azure AI Foundry 门户](https://ai.azure.com)，网址为：`https://ai.azure.com`，然后使用 Azure 凭据登录。 关闭首次登录时打开的任何使用技巧或快速入门窗格，如有必要，使用左上角的 **Azure AI Foundry** 徽标导航到主页，类似下图所示（若已打开**帮助**面板，请关闭）：

    ![Azure AI Foundry 门户的屏幕截图。](./media/ai-foundry-home.png)

1. 查看主页上的信息。
1. 在主页的“**浏览模型和功能**”部分中，搜索 `gpt-4o` 模型；我们将在项目中使用它。
1. 在搜索结果中，选择 **gpt-4o** 模型以查看其详细信息。
1. 阅读说明并查看“**详细信息**”选项卡上提供的其他信息。

    ![gpt-4o 模型详细信息页的屏幕截图。](./media/gpt4-details.png)

1. 在 **gpt-4o** 页面上，查看**基准**选项卡，了解模型在一些标准性能基准中与其他在类似场景中使用的模型的比较。

    ![gpt-4o 模型基准页的屏幕截图。](./media/gpt4-benchmarks.png)

1. 使用 **gpt-4o** 页标题旁边的后退箭头（**&larr;**），返回到模型目录。
1. 搜索 `Phi-4-mini-instruct` 并查看 Phi-4-mini-instruct**** 模型的详细信息和基准。

## 比较模型

你已查看了两个不同的模型，这两种模型都可用于实现生成式 AI 聊天应用程序。 现在，让我们直观地比较这两个模型的指标。

1. 使用后退箭头 (**&larr;**) 返回到模型目录。
1. 选择“**比较模型**”。 模型比较的可视化图表与一系列常见模型一起显示。

    ![模型比较页面的屏幕截图。](./media/compare-models.png)

1. 在“**模型比较**”窗格中，请注意，可以选择常用任务，例如，*问题解答*，以便自动为特定任务选择常用模型。
1. 使用“**清除所有模型**” (&#128465;) 图标，移除所有预先选择的模型。
1. 使用 **+ 模型比较**按钮，将 **gpt-4o** 模型添加到列表中。 然后使用同一按钮将 Phi-4-mini-instruct**** 模型添加到列表中。
1. 查看图表，该图表根据“**质量索引**”（指示模型质量的标准化分数）和“**成本**”比较模型。 可以通过将鼠标悬停在图表中表示它的点上来查看模型的特定值。

    ![gpt-4o 和 Phi-4-mini-instruct 的模型比较图表的屏幕截图。](./media/comparison-chart.png)

1. 在“**X 轴**”下拉菜单的“**质量**”下，选择以下指标并观察每个生成的图表，然后再切换到下一个图表：
    - 准确性
    - 质量指数

    根据基准测试，gpt-4o 模型看起来提供最佳的整体性能，但成本更高。

1. 在要比较的模型列表中，选择要重新打开其基准页的 **gpt-4o** 模型。
1. 在 **gpt-4o** 模型页的页面中，选择“**概述**”选项卡以查看模型详细信息。

## 创建 Azure AI Foundry 项目

要使用模型，需要创建 Azure AI Foundry *项目*。

1. 在 **gpt-4o** 模型概述页面顶部，选择“**使用此模型**”。
1. 当提示创建项目时，输入项目的有效名称并展开“**高级选项**”。
1. 在“**高级选项**”部分中，为你的项目指定以下设置：
    - **Azure AI Foundry 资源**：*Azure AI Foundry 资源的有效名称*
    - **订阅**：Azure 订阅
    - **资源组**：*创建或选择资源组*
    - **区域**：*选择任何**支持 AI 服务的位置***\*

    > \* 某些 Azure AI 资源受区域模型配额约束。 如果稍后在练习中达到配额限制，你可能需要在不同的区域中创建另一个资源。

1. 选择“**创建**”并等待项目（包括所选的 gpt-4 模型部署）创建。
1. 创建项目后，聊天操场将自动打开，以便可以测试模型：

    ![Azure AI Foundry 项目聊天操场的屏幕截图。](./media/ai-foundry-chat-playground.png)

## 与 *gpt-4o* 模型聊天

有了模型部署后，即可使用操场对其进行测试。

1. 在聊天操场的“**设置**”窗格中，确保已选择 **gpt-4o** 模型，并在“**提供模型说明和上下文**”字段中，将系统提示设置为 `You are an AI assistant that helps solve problems.`
1. 选择“**应用更改**”以更新系统提示。

1. 在查询窗口中输入以下查询

    ```
   I have a fox, a chicken, and a bag of grain that I need to take over a river in a boat. I can only take one thing at a time. If I leave the chicken and the grain unattended, the chicken will eat the grain. If I leave the fox and the chicken unattended, the fox will eat the chicken. How can I get all three things across the river without anything being eaten?
    ```

1. 查看响应 然后输入以下跟进查询：

    ```
   Explain your reasoning.
    ```

## 部署其他模型

创建项目时，选中的 **gpt-4o** 模型已自动部署。 现在来部署你同样考虑过的 *Phi-4-mini-instruct**** 模型。

1. 在左侧导航栏中的“**我的资产**”部分，选择“**模型 + 终结点**”页。
1. 在“**模型部署**”选项卡中的“**+ 部署模型**”下拉列表中，选择“**部署基础模型**”。 然后搜索 `Phi-4-mini-instruct` 并确认选择。
1. 同意模型许可证。
1. 使用以下设置部署 Phi-4-mini-instruct**** 模型：
    - **部署名**：*有效的模型部署名*
    - **部署类型**：全局标准
    - **部署详细信息**：*使用默认设置*

1. 等待部署完成。

## 与 Phi-4** 模型聊天

现在，让我们在操场中与新模型聊天。

1. 在导航栏中，选择“**操场**”。 然后选择“**聊天操场**”。
1. 在“聊天”操场的“设置”窗格中，确保已选中“Phi-4-mini-instruct”模型，并在聊天框中提供第一行作为 `System message: You are an AI assistant that helps solve problems.`（用于测试 gpt-4o 模型的同一系统提示，但由于没有系统消息设置，我们在第一个聊天中提供它以便模型理解上下文）。********
1. 在聊天窗口中新起一行（系统消息下方），输入以下查询

    ```
   I have a fox, a chicken, and a bag of grain that I need to take over a river in a boat. I can only take one thing at a time. If I leave the chicken and the grain unattended, the chicken will eat the grain. If I leave the fox and the chicken unattended, the fox will eat the chicken. How can I get all three things across the river without anything being eaten?
    ```

1. 查看响应 然后输入以下跟进查询：

    ```
   Explain your reasoning.
    ```

## 执行进一步比较

1. 使用“**设置**”窗格中的下拉列表在模型之间切换，并使用以下谜题测试这两个模型（正确答案为 40！）：

    ```
   I have 53 socks in my drawer: 21 identical blue, 15 identical black and 17 identical red. The lights are out, and it is completely dark. How many socks must I take out to make 100 percent certain I have at least one pair of black socks?
    ```

## 反思模型

已比较了两种模型，这两种模型在生成适当的响应和成本方面可能有所不同。 在任何生成式场景中，都需要找到一种模型，既要适合你需要它执行的任务，又要满足你期望它处理的请求数量所需的使用成本。

模型目录中提供的详细信息和基准，以及直观比较模型的能力，为识别生成式 AI 解决方案的候选模型提供了一个有用的起点。 然后，可以在聊天操场中使用各种系统和用户提示测试候选模型。

## 清理

如果已完成对 Azure AI Foundry 门户的探索，则应删除在本练习中创建的资源，以避免产生不必要的 Azure 成本。

1. 打开[Azure 门户](https://portal.azure.com)并查看在其中部署了本练习中使用的资源的资源组的内容。
1. 在工具栏中，选择“删除资源组”****。
1. 输入资源组名称，并确认要删除该资源组。
