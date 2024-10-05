---
title: Exercices Azure AI Language
permalink: index.html
layout: home
---

# Exercices Azure AI Language

Les exercices suivants ont été conçus afin de soutenir les modules Microsoft Learn pour [développer des solutions de langage naturel](https://learn.microsoft.com/training/paths/develop-language-solutions-azure-ai/).


{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Exercises'" %} {% for activity in labs %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) {% endfor %}
