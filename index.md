---
title: Ejercicios de lenguaje de Azure AI
permalink: index.html
layout: home
---

# Ejercicios de lenguaje de Azure AI

Los siguientes ejercicios han sido diseñados para brindarte una experiencia práctica de aprendizaje en la que explorarás las tareas comunes que realizan los desarrolladores al crear soluciones de lenguaje natural en Azure. 

> **Nota**: Para completar los ejercicios, necesitarás una suscripción de Azure. Si aún no tienes una, puedes registrarte para obtener una [cuenta de Azure](https://azure.microsoft.com/free). Hay una opción de evaluación gratuita para los nuevos usuarios que incluye créditos durante los primeros 30 días.

## Ejercicios

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Exercises'" %} {% for activity in labs  %}
<hr>
### [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }})

{{activity.lab.description}}

{% endfor %}

<hr>

> **Nota**: aunque puedes completar estos ejercicios por tu cuenta, han sido diseñados para complementar módulos en [Microsoft Learn](https://learn.microsoft.com/training/paths/develop-language-solutions-azure-ai/) en los que encontrarás un análisis más profundo de algunos de los conceptos subyacentes en los que se basan estos ejercicios. 
