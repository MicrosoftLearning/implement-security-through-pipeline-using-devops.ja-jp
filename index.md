---
title: Azure DevOps を使用してパイプラインを介してセキュリティを実装する
permalink: index.html
layout: home
---

# Azure DevOps を使用してパイプラインを介してセキュリティを実装する

次の演習は、DevOps を使用してパイプラインを介してセキュリティを実装するモジュール [をサポートするように設計されています](https://learn.microsoft.com/training/paths/implement-security-through-pipeline-using-devops/)。

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %} {% for activity in labs  %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
