---
title: Azure OpenAI 练习
permalink: index.html
layout: home
---

# 使用 Azure AI Studio 开发生成式 AI 应用程序

以下练习旨在为你提供动手学习体验，在其中可浏览开发人员用来生成基于聊天的“Copilot”等生成式 AI 应用程序的常见模式和技术，并了解如何使用 Azure AI 服务（特别是 Azure OpenAI 服务和 Azure AI Studio）实现这些模式。

虽然可以自行完成这些练习，但它们旨在补充有关 [Microsoft Learn](https://learn.microsoft.com/training/paths/create-custom-copilots-ai-studio/) 的模块；在其中可更深入地了解这些练习所基于的一些基础概念。

> **备注**：要完成练习，需要一个 Azure 订阅，其中包含足够的权限和配额以预配 Azure AI Studio 使用的 Azure 资源，以及部署和使用 Azure OpenAI GPT 模型。 如果还没有 Azure 订阅，请注册 [Azure 帐户](https://azure.microsoft.com/free)。 针对新用户提供免费试用选项，其中包括前 30 天的积分。

## 练习

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions'" %} {% for activity in labs  %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}