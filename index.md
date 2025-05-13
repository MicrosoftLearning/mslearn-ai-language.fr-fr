---
title: Exercices Azure AI Language
permalink: index.html
layout: home
---

# Exercices Azure AI Language

Les exercices suivants sont conçus pour vous fournir une expérience d’apprentissage pratique qui va vous permettre d’explorer les tâches courantes que les développeurs effectuent lors de la création de solutions de langage naturel sur Microsoft Azure. 

> **Remarque** : pour effectuer les exercices, vous devez disposer d’un abonnement Azure. Si vous n’avez pas de compte Azure, vous pouvez en créer un [ici](https://azure.microsoft.com/free). Les nouveaux utilisateurs peuvent profiter d’un essai gratuit qui inclut des crédits pour les 30 premiers jours.

## Exercices

{% assign labs = site.pages | where_exp:"page", "page.url contains '/Instructions/Exercises'" %} {% for activity in labs %}
<hr>
### [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }})

{{activity.lab.description}}

{% endfor %}

<hr>

> **Note** : bien que vous puissiez effectuer ces exercices de façon indépendante, ils sont conçus pour accompagner des modules [Microsoft Learn](https://learn.microsoft.com/training/paths/develop-language-solutions-azure-ai/), dans lesquels vous trouverez une présentation plus approfondie de certains des concepts sous-jacents sur lesquels se basent ces exercices. 
