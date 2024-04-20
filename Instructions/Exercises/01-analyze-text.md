---
lab:
  title: Analyser le texte
  module: Module 3 - Develop natural language processing solutions
---

# Analyser le texte

**Azure Language** prend en charge l’analyse du texte, notamment la détection de langue, l’analyse des sentiments, l’extraction de phrases clés et la reconnaissance d’entité.

Par exemple, supposons qu’une agence de voyages souhaite traiter les avis d’hôtel soumis au site web de l’entreprise. En utilisant Azure AI Language, il est possible de déterminer la langue dans laquelle chaque avis est écrit, le sentiment (positif, neutre ou négatif) des avis, les expressions clés qui peuvent indiquer les principaux sujets abordés dans l’avis et les entités nommées, comme les lieux, les monuments ou les personnes mentionnées dans les avis.

## Configurer une ressource *Azure AI Language*

Si vous n’avez pas encore de ressource dans votre abonnement, vous devez configurer une ressource du **service Azure AI Language** dans votre abonnement Azure.

1. Ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.
1. Recherchez **Services Azure AI** à l’aide du champ disponible dans la partie supérieure. Sélectionnez **Créer** sous **Service Language**.
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
1. Consultez la page **Clés et points de terminaison**. Vous aurez besoin des informations de cette page plus loin dans l’exercice.

## Préparer le développement d’une application dans Visual Studio Code

Vous allez développer votre application d’analyse de texte à l’aide de Visual Studio Code. Les fichiers de code de votre application ont été fournis dans un référentiel GitHub.

> **Conseil** : si vous avez déjà cloné le référentiel **mslearn-ai-language**, ouvrez-le dans Visual Studio Code. Dans le cas contraire, procédez comme suit pour le cloner dans votre environnement de développement.

1. Démarrez Visual Studio Code.
2. Ouvrez la palette (Maj+CTRL+P) et exécutez une commande **Git : Cloner** pour cloner le référentiel `https://github.com/MicrosoftLearning/mslearn-ai-language` vers un dossier local (peu importe quel dossier).
3. Lorsque le référentiel a été cloné, ouvrez le dossier dans Visual Studio Code.
4. Attendez que des fichiers supplémentaires soient installés pour prendre en charge les projets de code C# dans le référentiel.

    > **Remarque** : si vous êtes invité à ajouter des ressources requises pour générer et déboguer, sélectionnez **Not Now** (Pas maintenant).

## Configuration de votre application

Des applications pour C# et Python sont fournies, ainsi qu’un exemple de fichier texte que vous utiliserez pour tester le résumé. Les deux applications présentent les mêmes fonctionnalités. Pour commencer, vous allez définir certaines parties clés de l’application pour lui permettre d’utiliser votre ressource Azure AI Language.

1. Dans Visual Studio Code, dans le volet **Explorateur**, accédez au dossier **Labfiles/01-analyze-text** et développez le dossier **CSharp** ou **Python** selon vos préférencesde language, ainsi que le dossier **text-analytics** qu’il contient. Chaque dossier contient les fichiers spécifiques au langage d’application où vous allez intégrer la fonctionnalité d’analyse de texte d’Azure AI Language.
2. Cliquez avec le bouton droit sur le dossier **text-analytics** contenant vos fichiers de code et ouvrez un terminal intégré. Ensuite, installez le package SDK d’analyse de texte d’Azure AI Language en exécutant la commande correspondante du langage choisi. Si vous avez choisi Python, installez également le package `dotenv` :

    **C# :**

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.3.0
    ```

    **Python** :

    ```
    pip install azure-ai-textanalytics==5.3.0
    pip install python-dotenv
    ```

3. Dans le volet **Explorateur**, dans le dossier **text-analytics**, ouvrez le fichier config correspondant au langage choisi.

    - **C#** : appsettings.json
    - **Python** : .env
    
4. Mettez à jour les valeurs de configuration pour inclure le **point de terminaison** et une **clé** de la ressource Azure AI Language que vous avez créée (disponible sur la page **Clés et point de terminaison** de votre ressource Azure AI Language, dans le portail Azure).
5. Enregistrez le fichier de configuration.

6. Notez que le dossier **text-analysis** contient un fichier de code pour l’application cliente :

    - **C#** : Program.cs
    - **Python** : text-analysis.py

    Ouvrez le fichier de code et, en haut, sous les références d’espace de noms existantes, recherchez le commentaire **Importer des espaces de noms**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour importer les espaces de noms dont vous aurez besoin pour utiliser le kit de développement logiciel (SDK) d’analyse de texte :

    **C#**  : Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.TextAnalytics;
    ```

    **Python** : text-analysis.py

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.textanalytics import TextAnalyticsClient
    ```

7. Dans la fonction **Main**, notez que le code permettant de charger le point de terminaison et la clé du service Azure AI Language à partir du fichier de configuration a déjà été fourni. Recherchez ensuite le commentaire **Créer un client à l’aide du point de terminaison et de la clé**, puis ajoutez le code suivant pour créer un client pour l’API Analyse de texte :

    **C#**  : Programs.cs

    ```C#
    // Create client using endpoint and key
    AzureKeyCredential credentials = new AzureKeyCredential(aiSvcKey);
    Uri endpoint = new Uri(aiSvcEndpoint);
    TextAnalyticsClient aiClient = new TextAnalyticsClient(endpoint, credentials);
    ```

    **Python** : text-analysis.py

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(ai_key)
    ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

8. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **text-analysis** et entrez la commande suivante pour exécuter le programme :

    - **C#**  : `dotnet run`
    - **Python** : `python text-analysis.py`

    > **Conseil** : vous pouvez utiliser l’icône **Agrandir la taille du volet** (**^**) dans la barre d’outils du terminal pour afficher plus de texte sur la console.

9. Observez le résultat pour vérifier si le code s'exécute sans erreur, en affichant le contenu de chaque fichier texte d'avis dans le dossier **Avis**. L’application crée correctement un client pour l’API Analyse de texte, mais ne l’utilise pas. Nous le corrigerons dans la procédure suivante.

## Ajouter du code pour détecter le langage

Maintenant que vous avez créé un client pour l’API, nous allons l’utiliser pour détecter la langue dans laquelle chaque avis est écrit.

1. Dans la fonction **Main** de votre programme, recherchez le commentaire **Obtenir la langue**. Ensuite, sous ce commentaire, ajoutez le code nécessaire pour détecter la langue dans chaque document d’avis :

    **C#**  : Programs.cs

    ```csharp
    // Get language
    DetectedLanguage detectedLanguage = aiClient.DetectLanguage(text);
    Console.WriteLine($"\nLanguage: {detectedLanguage.Name}");
    ```

    **Python** : text-analysis.py

    ```python
    # Get language
    detectedLanguage = ai_client.detect_language(documents=[text])[0]
    print('\nLanguage: {}'.format(detectedLanguage.primary_language.name))
    ```

     > **Remarque** : *Dans cet exemple, chaque avis est analysé individuellement, ce qui entraîne un appel distinct au service pour chaque fichier. Une autre approche consiste à créer une collection de documents et à les transmettre au service en un seul appel. Dans les deux approches, la réponse du service consiste en une collection de documents ; c'est pourquoi dans le code Python ci-dessus, l'index du premier (et seul) document de la réponse ([0]) est spécifié.*

1. Enregistrez vos modifications. Revenez ensuite au terminal intégré du dossier **text-analysis**, puis ré-exécutez le programme.
1. Observez la sortie, notant que cette fois la langue de chaque avis est identifiée.

## Ajouter du code pour évaluer le sentiment

*L’analyse des sentiments* est une technique couramment utilisée pour classer le texte comme *positif* ou *négatif* (ou *éventuellement neutre* ou *mixte*). Elle est couramment utilisée pour analyser les publications de médias sociaux, les avis sur des produits et d’autres éléments où le sentiment du texte peut fournir des informations utiles.

1. Dans la fonction **Main** de votre programme, recherchez le commentaire **Obtenir le sentiment**. Ensuite, sous ce commentaire, ajoutez le code nécessaire pour détecter le sentiment dans chaque document d’avis :

    **C#** : Program.cs

    ```csharp
    // Get sentiment
    DocumentSentiment sentimentAnalysis = aiClient.AnalyzeSentiment(text);
    Console.WriteLine($"\nSentiment: {sentimentAnalysis.Sentiment}");
    ```

    **Python** : text-analysis.py

    ```python
    # Get sentiment
    sentimentAnalysis = ai_client.analyze_sentiment(documents=[text])[0]
    print("\nSentiment: {}".format(sentimentAnalysis.sentiment))
    ```

1. Enregistrez vos modifications. Revenez ensuite au terminal intégré du dossier **text-analysis**, puis ré-exécutez le programme.
1. Observez le résultat pour vérifier la détection d'un sentiment dans les avis.

## Ajouter du code pour identifier les expressions clés

Il peut être utile d’identifier les expressions clés dans un corps de texte pour vous aider à déterminer les sujets principaux qu’il traite.

1. Dans la fonction **Main** de votre programme, recherchez le commentaire **Obtenir les expressions clés**. Ensuite, sous ce commentaire, ajoutez le code nécessaire pour détecter les expressions clés dans chaque document d’avis :

    **C#** : Program.cs

    ```csharp
    // Get key phrases
    KeyPhraseCollection phrases = aiClient.ExtractKeyPhrases(text);
    if (phrases.Count > 0)
    {
        Console.WriteLine("\nKey Phrases:");
        foreach(string phrase in phrases)
        {
            Console.WriteLine($"\t{phrase}");
        }
    }
    ```

    **Python** : text-analysis.py

    ```python
    # Get key phrases
    phrases = ai_client.extract_key_phrases(documents=[text])[0].key_phrases
    if len(phrases) > 0:
        print("\nKey Phrases:")
        for phrase in phrases:
            print('\t{}'.format(phrase))
    ```

1. Enregistrez vos modifications. Revenez ensuite au terminal intégré du dossier **text-analysis**, puis ré-exécutez le programme.
1. Observez la sortie, notant que chaque document contient des expressions clés qui donnent des informations sur l’avis.

## Ajouter du code pour extraire des entités

Souvent, les documents ou d’autres corps de texte mentionnent des personnes, des lieux, des périodes ou d’autres entités. L’API d’analyse de texte peut détecter plusieurs catégories (et sous-catégories) d’entités dans votre texte.

1. Dans la fonction **Main** de votre programme, recherchez le commentaire **Obtenir les entités**. Ensuite, sous ce commentaire, ajoutez le code nécessaire pour identifier les entités mentionnées dans chaque avis :

    **C#** : Program.cs

    ```csharp
    // Get entities
    CategorizedEntityCollection entities = aiClient.RecognizeEntities(text);
    if (entities.Count > 0)
    {
        Console.WriteLine("\nEntities:");
        foreach(CategorizedEntity entity in entities)
        {
            Console.WriteLine($"\t{entity.Text} ({entity.Category})");
        }
    }
    ```

    **Python** : text-analysis.py

    ```python
    # Get entities
    entities = ai_client.recognize_entities(documents=[text])[0].entities
    if len(entities) > 0:
        print("\nEntities")
        for entity in entities:
            print('\t{} ({})'.format(entity.text, entity.category))
    ```

1. Enregistrez vos modifications. Revenez ensuite au terminal intégré du dossier **text-analysis**, puis ré-exécutez le programme.
1. Observez la sortie, notant les entités détectées dans le texte.

## Ajouter du code pour extraire des entités liées

Outre les entités classées, l’API d’analyse de texte peut détecter les entités pour lesquelles il existe des liens connus vers des sources de données, telles que Wikipédia.

1. Dans la fonction **Main** de votre programme, recherchez le commentaire **Obtenir les entités liées**. Ensuite, sous ce commentaire, ajoutez le code nécessaire pour identifier les entités liées mentionnées dans chaque avis :

    **C#** : Program.cs

    ```csharp
    // Get linked entities
    LinkedEntityCollection linkedEntities = aiClient.RecognizeLinkedEntities(text);
    if (linkedEntities.Count > 0)
    {
        Console.WriteLine("\nLinks:");
        foreach(LinkedEntity linkedEntity in linkedEntities)
        {
            Console.WriteLine($"\t{linkedEntity.Name} ({linkedEntity.Url})");
        }
    }
    ```

    **Python** : text-analysis.py

    ```python
    # Get linked entities
    entities = ai_client.recognize_linked_entities(documents=[text])[0].entities
    if len(entities) > 0:
        print("\nLinks")
        for linked_entity in entities:
            print('\t{} ({})'.format(linked_entity.name, linked_entity.url))
    ```

1. Enregistrez vos modifications. Revenez ensuite au terminal intégré du dossier **text-analysis**, puis ré-exécutez le programme.
1. Observez la sortie, en notant les entités liées identifiées.

## Nettoyer les ressources

Si vous avez fini d’explorer le service Azure AI Language, vous pouvez supprimer les ressources que vous avez créées dans cet exercice. Voici comment procéder :

1. Ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.

2. Accédez à la ressource Azure AI Language que vous avez créée dans ce labo.

3. Dans la page de la ressource, sélectionnez **Supprimer** et suivez les instructions pour supprimer la ressource.

## Plus d’informations

Pour en savoir plus sur l’utilisation d’**Azure AI Language**, consultez la [documentation](https://learn.microsoft.com/azure/ai-services/language-service/).
