---
title: Exercices Azure AI Language
permalink: index.html
layout: home
---

# Exercices Azure AI Language

Les exercices suivants sont conçus pour prendre en charge les modules sur Microsoft Learn pour [développer des solutions de traitement du langage naturel](https://learn.microsoft.com/training/paths/develop-language-solutions-azure-ai/).


{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Labs'" %} {% for activity in labs  %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
