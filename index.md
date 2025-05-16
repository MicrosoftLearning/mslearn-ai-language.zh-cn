---
title: Azure AI 语言练习
permalink: index.html
layout: home
---

# Azure AI 语言练习

以下练习旨在为你提供动手学习体验，你将在其中探索开发人员在 Azure 上生成自然语言解决方案时执行的常见任务。 

> **备注**：你需要一个 Azure 订阅来完成练习。 如果还没有 Azure 订阅，请注册 [Azure 帐户](https://azure.microsoft.com/free)。 针对新用户提供免费试用选项，其中包括前 30 天的积分。

## 练习

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %} {% for activity in labs  %}
<hr>
### [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }})

{{activity.lab.description}}

{% endfor %}

<hr>

> **备注**：虽然可以自行完成这些练习，但它们旨在补充有关 [Microsoft Learn](https://learn.microsoft.com/training/paths/develop-language-solutions-azure-ai/) 的模块；在其中可更深入地了解这些练习所基于的一些基础概念。 
