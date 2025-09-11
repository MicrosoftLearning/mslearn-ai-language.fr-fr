---
lab:
  title: Analyser le texte
  description: "Utilisez Azure\_AI\_Language pour analyser du texte, notamment la détection de langue, l’analyse des sentiments, l’extraction de phrases clés et la reconnaissance d’entité."
---

# Analyser le texte

**Azure AI Language** prend en charge l’analyse du texte, notamment la détection de langue, l’analyse des sentiments, l’extraction de phrases clés et la reconnaissance d’entité.

Par exemple, supposons qu’une agence de voyages souhaite traiter les avis d’hôtel soumis au site web de l’entreprise. En utilisant Azure AI Language, il est possible de déterminer la langue dans laquelle chaque avis est écrit, le sentiment (positif, neutre ou négatif) des avis, les expressions clés qui peuvent indiquer les principaux sujets abordés dans l’avis et les entités nommées, comme les lieux, les monuments ou les personnes mentionnées dans les avis. Dans cet exercice, vous utiliserez le kit de développement logiciel (SDK) Python Azure AI Language pour l’analyse de texte afin de mettre en œuvre une application simple d’évaluation d’hôtels basée sur cet exemple.

Bien que cet exercice soit basé sur Python, vous pouvez développer des applications d’analyse de texte à l’aide de plusieurs kits SDK spécifiques à une langue, notamment :

- [Bibliothèque de client Azure AI Analyse de texte pour Python](https://pypi.org/project/azure-ai-textanalytics/)
- [Bibliothèque de client Azure AI Analyse de texte pour .NET](https://www.nuget.org/packages/Azure.AI.TextAnalytics)
- [Bibliothèque de client Azure AI Analyse de texte pour JavaScript](https://www.npmjs.com/package/@azure/ai-text-analytics)

Cet exercice prend environ **30** minutes.

## Configurer une ressource *Azure AI Language*

Si vous n’avez pas encore de ressource dans votre abonnement, vous devez configurer une ressource du **service Azure AI Language** dans votre abonnement Azure.

1. Ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.
1. Sélectionnez **Créer une ressource**.
1. Dans le champ de recherche, recherchez le **service de language**. Sélectionnez **Créer** sous **Service Language**.
1. Sélectionnez **Continuer pour créer votre ressource**.
1. Configurez la ressource avec les paramètres suivants :
    - **Abonnement** : *votre abonnement Azure*.
    - **Groupe de ressources** : *créez ou sélectionnez un groupe de ressources*.
    - **Région** : *choisissez n’importe quelle région disponible*.
    - **Nom** : *entrez un nom unique.*
    - **Niveau tarifaire** : sélectionnez **F0** (*gratuit*) ou **S** (*standard*) si F n’est pas disponible.
    - **Mention sur l’IA responsable** : J’accepte.
1. Sélectionnez **Vérifier + créer**, puis **Créer** pour provisionner la ressource.
1. Attendez la fin du déploiement, puis accédez à la ressource déployée.
1. Affichez la page **Point de terminaison et clés** dans la section **Gestion des ressources**. Plus loin dans l’exercice, vous aurez besoin des informations disponibles sur cette page.

## Cloner le référentiel pour ce cours

Vous allez développer votre code à l’aide de Cloud Shell à partir du portail Azure. Les fichiers de code de votre application ont été fournis dans un référentiel GitHub.

1. Dans le portail Azure, cliquez sur le bouton **[\>_]** à droite de la barre de recherche, en haut de la page, pour créer un environnement Cloud Shell dans le portail Azure, puis sélectionnez un environnement ***PowerShell***. Cloud Shell fournit une interface de ligne de commande dans un volet situé en bas du portail Azure.

    > **Remarque** : si vous avez déjà créé un Cloud Shell qui utilise un environnement *Bash*, basculez-le vers ***PowerShell***.

1. Dans la barre d’outils Cloud Shell, dans le menu **Paramètres**, sélectionnez **Accéder à la version classique** (cela est nécessaire pour utiliser l’éditeur de code).

    **<font color="red">Assurez-vous d’avoir basculé vers la version classique du Cloud Shell avant de continuer.</font>**

1. Dans le volet PowerShell, entrez les commandes suivantes pour cloner le référentiel GitHub pour cet exercice :

    ```
    rm -r mslearn-ai-language -f
    git clone https://github.com/microsoftlearning/mslearn-ai-language
    ```

    > **Conseil** : lorsque vous entrez des commandes dans le Cloud Shell, la sortie peut occuper une grande partie de la mémoire tampon d’écran. Vous pouvez effacer le contenu de l’écran en saisissant la commande `cls` pour faciliter le focus sur chaque tâche.

1. Une fois le référentiel cloné, accédez au dossier contenant les fichiers de code de l’application :  

    ```
    cd mslearn-ai-language/Labfiles/01-analyze-text/Python/text-analysis
    ```

## Configuration de votre application

1. Dans le volet de ligne de commande, exécutez la commande suivante pour afficher les fichiers de code dans le dossier **text-analysis** :

    ```
   ls -a -l
    ```

    Les fichiers incluent un fichier de configuration (**.env**) et un fichier de code (**text-analysis.py**). Le texte que votre application analysera se trouve dans le sous-dossier **revues**.

1. Créez un environnement virtuel Python et installez le package du kit de développement logiciel (SDK) Azure AI Language Analyse de texte, ainsi que les autres packages requis en exécutant la commande suivante :

    ```
    python -m venv labenv
    ./labenv/bin/Activate.ps1
    pip install -r requirements.txt azure-ai-textanalytics==5.3.0
    ```

1. Entrez la commande suivante pour modifier le fichier de configuration de l’application :

    ```
   code .env
    ```

    Le fichier s’ouvre dans un éditeur de code.

1. Mettez à jour les valeurs de configuration pour inclure le **point de terminaison** et une **clé** de la ressource Azure AI Language que vous avez créée (disponible sur la page **Clés et point de terminaison** de votre ressource Azure AI Language, dans le portail Azure).
1. Une fois que vous avez remplacé les espaces réservés, utilisez la commande **CTRL+S** ou **Clic droit > Enregistrer** dans l’éditeur de code pour enregistrer vos modifications, puis utilisez la commande **CTRL+Q** ou **Clic droit > Quitter** pour fermer l’éditeur tout en gardant la ligne de commande du Cloud Shell ouverte.

## Ajouter du code pour vous connecter à votre ressource Azure AI Language

1. Entrez la commande suivante pour modifier le fichier de code de l’application :

    ```
    code text-analysis.py
    ```

1. Passez en revue le code existant. Vous allez ajouter du code pour travailler avec le kit SDK d’analyse de texte du langage à technologie IA.

    > **Conseil** : lorsque vous ajoutez du code au fichier de code, veillez à conserver la bonne mise en retrait.

1. En haut du fichier de code, sous les références d’espace de noms existantes, recherchez le commentaire **Importer les espaces de noms** et ajoutez le code suivant pour importer les espaces de noms dont vous aurez besoin pour utiliser le kit SDK d’analyse de texte :

    ```python
   # import namespaces
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. Dans la fonction **Main**, notez que le code permettant de charger le point de terminaison et la clé du service Azure AI Language depuis le fichier de configuration est déjà fourni. Recherchez ensuite le commentaire **Créer un client à l’aide du point de terminaison et de la clé**, puis ajoutez le code suivant pour créer un client pour l’API Analyse de texte :

    ```Python
   # Create client using endpoint and key
   credential = AzureKeyCredential(ai_key)
   ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

1. Enregistrez vos modifications (CTRL+S), puis entrez la commande suivante pour exécuter le programme (vous pouvez agrandir le volet Cloud Shell et redimensionner les volets pour afficher davantage de texte dans le volet de ligne de commande) :

    ```
   python text-analysis.py
    ```

1. Observez le résultat pour vérifier si le code s'exécute sans erreur, en affichant le contenu de chaque fichier texte d'avis dans le dossier **Avis**. L’application crée correctement un client pour l’API Analyse de texte, mais ne l’utilise pas. Nous le corrigerons dans la section suivante.

## Ajouter du code pour détecter le langage

Maintenant que vous avez créé un client pour l’API, nous allons l’utiliser pour détecter la langue dans laquelle chaque avis est écrit.

1. Dans l’éditeur de code, recherchez le commentaire **Obtenir la langue**. Ajoutez ensuite le code nécessaire pour détecter la langue dans chaque document de révision :

    ```python
   # Get language
   detectedLanguage = ai_client.detect_language(documents=[text])[0]
   print('\nLanguage: {}'.format(detectedLanguage.primary_language.name))
    ```

     > **Remarque** : *Dans cet exemple, chaque avis est analysé individuellement, ce qui entraîne un appel distinct au service pour chaque fichier. Une autre approche consiste à créer une collection de documents et à les transmettre au service en un seul appel. Dans les deux approches, la réponse du service consiste en une collection de documents ; c'est pourquoi dans le code Python ci-dessus, l'index du premier (et seul) document de la réponse ([0]) est spécifié.*

1. Enregistrez les changements apportés. Puis, exécutez à nouveau le programme.
1. Observez la sortie, notant que cette fois la langue de chaque avis est identifiée.

## Ajouter du code pour évaluer le sentiment

*L’analyse des sentiments* est une technique couramment utilisée pour classer le texte comme *positif* ou *négatif* (ou *éventuellement neutre* ou *mixte*). Elle est couramment utilisée pour analyser les publications de médias sociaux, les avis sur des produits et d’autres éléments où le sentiment du texte peut fournir des informations utiles.

1. Dans l’éditeur de code, recherchez le commentaire **Obtenir le sentiment**. Ajoutez ensuite le code nécessaire pour détecter le sentiment de chaque document de révision :

    ```python
   # Get sentiment
   sentimentAnalysis = ai_client.analyze_sentiment(documents=[text])[0]
   print("\nSentiment: {}".format(sentimentAnalysis.sentiment))
    ```

1. Enregistrez les changements apportés. Fermez ensuite l’éditeur de code et exécutez à nouveau le programme.
1. Observez le résultat pour vérifier la détection d'un sentiment dans les avis.

## Ajouter du code pour identifier les expressions clés

Il peut être utile d’identifier les expressions clés dans un corps de texte pour vous aider à déterminer les sujets principaux qu’il traite.

1. Dans l’éditeur de code, recherchez le commentaire **Obtenir les expressions clés**. Ajoutez ensuite le code nécessaire pour détecter les expressions clés dans chaque document de révision :

    ```python
   # Get key phrases
   phrases = ai_client.extract_key_phrases(documents=[text])[0].key_phrases
   if len(phrases) > 0:
        print("\nKey Phrases:")
        for phrase in phrases:
            print('\t{}'.format(phrase))
    ```

1. Enregistrez vos modifications et réexécutez le programme.
1. Observez la sortie, notant que chaque document contient des expressions clés qui donnent des informations sur l’avis.

## Ajouter du code pour extraire des entités

Souvent, les documents ou d’autres corps de texte mentionnent des personnes, des lieux, des périodes ou d’autres entités. L’API d’analyse de texte peut détecter plusieurs catégories (et sous-catégories) d’entités dans votre texte.

1. Dans l’éditeur de code, recherchez le commentaire **Obtenir les entités**. Ensuite, ajoutez le code nécessaire pour identifier les entités mentionnées dans chaque révision :

    ```python
   # Get entities
   entities = ai_client.recognize_entities(documents=[text])[0].entities
   if len(entities) > 0:
        print("\nEntities")
        for entity in entities:
            print('\t{} ({})'.format(entity.text, entity.category))
    ```

1. Enregistrez vos modifications et réexécutez le programme.
1. Observez la sortie, notant les entités détectées dans le texte.

## Ajouter du code pour extraire des entités liées

Outre les entités classées, l’API d’analyse de texte peut détecter les entités pour lesquelles il existe des liens connus vers des sources de données, telles que Wikipédia.

1. Dans l’éditeur de code, recherchez le commentaire **Obtenir les entités liées**. Ensuite, ajoutez le code nécessaire pour identifier les entités liées mentionnées dans chaque révision :

    ```python
   # Get linked entities
   entities = ai_client.recognize_linked_entities(documents=[text])[0].entities
   if len(entities) > 0:
        print("\nLinks")
        for linked_entity in entities:
            print('\t{} ({})'.format(linked_entity.name, linked_entity.url))
    ```

1. Enregistrez vos modifications et réexécutez le programme.
1. Observez la sortie, en notant les entités liées identifiées.

## Nettoyer les ressources

Si vous avez fini d’explorer le service Azure AI Language, vous pouvez supprimer les ressources que vous avez créées dans cet exercice. Voici comment procéder :

1. Fermez le volet Azure Cloud Shell.
1. Dans le portail Azure, accédez à la ressource Azure AI Language que vous avez créée dans cette activité.
1. Dans la page de la ressource, sélectionnez **Supprimer** et suivez les instructions pour supprimer la ressource.

## Plus d’informations

Pour en savoir plus sur l’utilisation d’**Azure AI Language**, consultez la [documentation](https://learn.microsoft.com/azure/ai-services/language-service/).
