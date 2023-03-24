---
title: 联机托管说明
permalink: index.html
layout: home
---

# <a name="database-administration-exercises"></a>数据库管理练习

这些练习支持 Microsoft 课程 [DP-300：管理 Microsoft Azure SQL 解决方案](https://docs.microsoft.com/training/courses/dp-300t00)。

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %}
| 模块 | 练习 |
| --- | --- | 
{% for activity in labs %}| {{ activity.lab.module }} | [{{ activity.lab.title }}{% if activity.lab.type %} - {{ activity.lab.type }}{% endif %}]({{ site.github.url }}{{ activity.url }}) |
{% endfor %}

