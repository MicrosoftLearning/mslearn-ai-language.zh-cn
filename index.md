---
title: Azure AI 语言练习
permalink: index.html
layout: home
---

# Azure AI 语言练习

以下练习旨在支持开发自然语言解决方案的 Microsoft Learn[](https://learn.microsoft.com/training/paths/develop-language-solutions-azure-ai/) 上的模块。


{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Exercises'" %} {% for activity in labs  %} {% unless activity.url contains 'ai-foundry' %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endunless %} {% endfor %}
