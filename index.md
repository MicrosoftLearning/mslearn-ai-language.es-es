---
title: Ejercicios de lenguaje de Azure AI
permalink: index.html
layout: home
---

# Ejercicios de lenguaje de Azure AI

Los siguientes ejercicios están diseñados como apoyo para los módulos de Microsoft Learn para [desarrollar soluciones de lenguaje natural](https://learn.microsoft.com/training/paths/develop-language-solutions-azure-ai/).


{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %} {% for activity in labs  %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
