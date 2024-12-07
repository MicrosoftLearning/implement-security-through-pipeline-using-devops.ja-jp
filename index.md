---
title: Azure DevOps 演習を使用しパイプラインを介してセキュリティを実装する
permalink: index.html
layout: home
---

# Azure DevOps 演習を使用しパイプラインを介してセキュリティを実装する

次の演習は、「[DevOps を使用しパイプラインを介してセキュリティを実装する](https://learn.microsoft.com/training/paths/implement-security-through-pipeline-using-devops/)」モジュールをサポートするように設計されています。

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %} {% for activity in labs  %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
