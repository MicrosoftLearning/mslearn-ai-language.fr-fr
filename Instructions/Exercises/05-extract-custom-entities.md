---
lab:
  title: Extraire des entités personnalisées
  module: Module 3 - Getting Started with Natural Language Processing
---

# Extraire des entités personnalisées

En plus d’autres fonctionnalités de traitement du langage naturel, le service Azure AI Language vous permet de définir des entités personnalisées et d’en extraire des instances à partir d’un texte.

Pour tester l’extraction d’entités personnalisées, nous allons créer un modèle et l’entraîner via Azure AI Language Studio, puis utiliser une application de ligne de commande pour la tester.

## Configurer une ressource *Azure AI Language*

Si vous n’en avez pas encore dans votre abonnement, vous devez configurer une ressource pour le **service Azure AI Language**. En outre, pour utiliser la classification de texte personnalisée, vous devez activer la fonctionnalité **Classification et extraction de texte personnalisée**.

1. Dans un navigateur, ouvrez le Portail Azure (`https://portal.azure.com`) et connectez-vous avec votre compte Microsoft.
1. Sélectionnez le bouton **Créer une ressource**, recherchez *Langage*, puis créez une ressource de **service Azure AI Language**. Quand le système vous invite à ajouter des *Fonctionnalités supplémentaires*, sélectionnez **Classification et extraction de texte personnalisée**. Créez la ressource avec les paramètres suivants :
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *sélectionnez ou créez un groupe de ressources*.
    - **Région** : *choisissez n’importe quelle région disponible*
    - **Nom** : *Entrez un nom unique.*
    - **Niveau tarifaire** : sélectionnez **F0** (*gratuit*). Si cette option n’est pas disponible, sélectionnez **S** (*standard*).
    - **Compte de stockage** : nouveau compte de stockage.
      - **Nom du compte de stockage** : *entrez un nom unique*.
      - **Type de compte de stockage** : Standard LRS
    - **Avis sur l’IA responsable** : sélectionné.

1. Sélectionnez **Vérifier + créer**, puis **Créer** pour provisionner la ressource.
1. Attendez la fin du déploiement, puis accédez à la ressource déployée.
1. Consultez la page **Clés et points de terminaison**. Plus loin dans l’exercice, vous aurez besoin des informations disponibles sur cette page.

## Charger des exemples d’annonces

Une fois que vous avez créé le service Azure AI Language et le compte de stockage, vous devez charger des exemples d’annonces pour entraîner votre modèle par la suite.

1. Dans un nouvel onglet de navigateur, téléchargez des exemples d’annonces classifiées à partir de `https://aka.ms/entity-extraction-ads` et extrayez les fichiers dans un dossier de votre choix.

2. Dans le Portail Azure, accédez au compte de stockage que vous avez créé et sélectionnez-le.

3. Dans votre compte de stockage, sélectionnez **Configuration** sous les **Paramètres**, puis activez l’option **Autoriser l’accès anonyme aux objets blob** avant de sélectionner **Enregistrer**.

4. Sélectionnez **Conteneurs** dans le menu de gauche, situé sous **Stockage de données**. Dans l’écran qui s’affiche, sélectionnez **+ Conteneur**. Donnez au conteneur le nom `classifieds`, et définissez le **Niveau d’accès anonyme** sur **Conteneur (accès en lecture anonyme pour les conteneurs et les objets blob)**.

    > **REMARQUE** : lorsque vous configurez un compte de stockage pour une solution réelle, veillez à attribuer le niveau d’accès approprié. Pour en savoir plus sur chaque niveau d’accès, consultez la [documentation de Stockage Azure](https://learn.microsoft.com/azure/storage/blobs/anonymous-read-access-configure).

5. Après avoir créé le conteneur, sélectionnez-le et cliquez sur le bouton **Charger** pour charger les exemples d’annonces que vous avez téléchargés.

## Créer un projet de reconnaissance d’entité nommée personnalisée

Vous pouvez maintenant créer un projet de reconnaissance d’entités nommées personnalisées. Ce projet fournit un lieu de travail pour générer, entraîner et déployer votre modèle.

> **REMARQUE** : vous pouvez aussi créer, générer, entraîner et déployer votre modèle via l’API REST

1. Dans un nouvel onglet de navigateur, ouvrez le portail Azure AI Language Studio sur `https://language.cognitive.azure.com/` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.
1. Si vous êtes invité à choisir une ressource de langue, sélectionnez les paramètres suivants :

    - **Azure Directory** : Annuaire Azure contenant votre abonnement.
    - **Abonnement Azure** : Votre abonnement Azure.
    - **Type de ressource** : Language.
    - **Ressource Language** : nom de la ressource Azure AI Language que vous avez créée précédemment.

    Si vous n’êtes <u>pas</u> invité à choisir une ressource de langue, c’est peut-être parce que vous avez plusieurs ressources de langue dans votre abonnement, auquel cas :

    1. Dans la barre en haut de la page, sélectionnez le bouton **Paramètres (&#9881;)**.
    2. Dans la page **Paramètres**, affichez l’onglet **Ressources**.
    3. Sélectionnez la ressource de langue que vous venez de créer, puis cliquez sur **Changer de ressource**.
    4. En haut de la page, cliquez sur **Language Studio** pour revenir à la page d’accueil de Language Studio.

1. En haut du portail, dans le menu **Créer**, sélectionnez **Reconnaissance d’entités nommées personnalisées**.

1. Créez un projet avec les paramètres suivants :
    - **Connecter un stockage** : *cette valeur est probablement déjà remplie. Si besoin, remplacez-la pour qu’elle indique votre compte de stockage.*
    - **Informations de base :**
    - **Nom :** `CustomEntityLab`
        - **Langue principale du texte** : anglais (US)
        - **Votre jeu de données inclut-il des documents qui ne sont pas dans la même langue ?**  : *Non*.
        - **Description** : `Custom entities in classified ads`
    - **Conteneur** :
        - **Conteneur de magasin d’objets blob** : classifieds.
        - **Vos fichiers sont-ils étiquetés avec des classes ?**  : Non, je dois étiqueter mes fichiers dans le cadre de ce projet

## Étiqueter vos données

Maintenant que votre projet est créé, vous devez étiqueter vos données pour entraîner votre modèle à identifier des entités.

1. Si la page **Étiquetage des données** n’est pas déjà ouverte, dans le volet de gauche, sélectionnez **Étiquetage des données**. Vous voyez la liste des fichiers que vous avez chargés sur votre compte de stockage.
1. Sur le côté droit, dans le volet **Activité**, sélectionnez **+ Ajouter une entité** et ajoutez une entité appelée `ItemForSale`.
1.  Répétez l’étape précédente pour créer les entités suivantes :
    - `Price`
    - `Location`
1. Une fois que vous avez créé vos trois entités, sélectionnez **Ad 1.txt** pour lire le fichier.
1. Dans *Ad 1.txt* : 
    1. Mettez en surbrillance le texte *face cord of firewood* et sélectionnez l’entité **ItemForSale**.
    1. Mettez en surbrillance le texte *Denver, CO* et sélectionnez l’entité **Location**.
    1. Mettez en surbrillance le texte *$90* et sélectionnez l’entité **Price**.
Dans le volet **Activité**, notez que ce document sera ajouté au jeu de données d’entraînement du modèle.
1. Utilisez le bouton **Document suivant** pour passer au document suivant et continuer à affecter du texte aux entités appropriées pour tous les documents, puis tous les ajouter au jeu de données d’entraînement.
1. Une fois le dernier document (*Ad 9.txt*), étiqueté, enregistrez les étiquettes.

## Entraîner votre modèle

Après avoir étiqueté vos données, vous devez entraîner votre modèle.

1. Sur la gauche, sélectionnez le volet **Travaux d’entraînement**.
2. Sélectionnez **Démarrer un travail d’entraînement**.
3. Sélectionnez l’option permettant d’entraîner le nouveau modèle nommé `ExtractAds`.
4. Choisissez **Séparer automatiquement le jeu de test des données d’entraînement**

    > **CONSEIL** : dans vos propres projets d’extraction, utilisez la division des tests qui convient le mieux à vos données. Pour des données plus cohérentes et des jeux de données plus grands, le service Azure AI Language va fractionner automatiquement le jeu de test selon un pourcentage. Avec des jeux de données plus petits, il est important d’effectuer l’entraînement avec la diversité pertinente des documents d’entrée possibles.

5. Cliquez sur **Entraîner**

    > **IMPORTANT** : l’entraînement de votre modèle peut parfois prendre plusieurs minutes. Vous recevez une notification une fois l’opération terminée.

## Évaluer votre modèle

Dans les applications réelles, il est important d’évaluer votre modèle et de l’améliorer pour vérifier qu’il fonctionne comme attendu. Deux pages à gauche vous montrent les détails de votre modèle entraîné et les tests qui ont échoué.

Sélectionnez **Performances du modèle** dans le menu de gauche, puis sélectionnez votre modèle `ExtractAds`. Ici, vous pouvez voir le scoring de votre modèle, ses métriques de performances et quand il a été entraîné. Vous serez en mesure de voir si des documents de test ont échoué, et ces échecs vous aideront à comprendre où le modèle doit être amélioré.

## Déployer votre modèle

Quand vous êtes satisfait de l’entraînement de votre modèle, vous pouvez le déployer, ce qui vous permet de commencer à extraire des entités via l’API.

1. Dans le volet de gauche, sélectionnez **Déploiement d’un modèle**.
2. Sélectionnez **Ajouter un déploiement**, puis entrez le nom `AdEntities` et sélectionnez le modèle **ExtractAds**.
3. Cliquez sur **Déployer** pour déployer votre modèle.

## Préparer le développement d’une application dans Visual Studio Code

Pour tester les fonctionnalités d’extraction des entités personnalisées du service Azure AI Language, vous allez développer une application console simple dans Visual Studio Code.

> **Conseil** : si vous avez déjà cloné le référentiel **mslearn-ai-language**, ouvrez-le dans Visual Studio Code. Dans le cas contraire, procédez comme suit pour le cloner dans votre environnement de développement.

1. Démarrez Visual Studio Code.
2. Ouvrez la palette (Maj+CTRL+P) et exécutez une commande **Git : Cloner** pour cloner le référentiel `https://github.com/MicrosoftLearning/mslearn-ai-language` vers un dossier local (peu importe quel dossier).
3. Lorsque le référentiel a été cloné, ouvrez le dossier dans Visual Studio Code.
4. Attendez que des fichiers supplémentaires soient installés pour prendre en charge les projets de code C# dans le référentiel.

    > **Remarque** : Si vous êtes invité à ajouter des ressources requises pour générer et déboguer, sélectionnez **Not Now** (Pas maintenant).

## Configuration de votre application

Des applications pour C# et Python sont fournies, ainsi qu’un exemple de fichier texte que vous utiliserez pour tester le résumé. Les deux applications présentent les mêmes fonctionnalités. Premièrement, vous allez terminer certaines parties clés de l’application pour lui permettre d’utiliser votre ressource Azure AI Language.

1. Dans Visual Studio Code, dans le volet **Explorateur**, accédez au dossier **Labfiles/05-custom-entity-recognition** et développez le dossier **CSharp** ou **Python** en fonction de votre préférence de langage, ainsi que le dossier **custom-entities** qu’il contient. Chaque dossier contient les fichiers propres au langage d’une application dans laquelle vous allez intégrer des fonctionnalités de classification de texte d’Azure AI Language.
1. Cliquez avec le bouton droit sur le dossier **custom-entities**, qui contient vos fichiers de code, et ouvrez un terminal intégré. Installez ensuite le package du kit de développement logiciel (SDK) d’analyse de texte d’Azure AI Language en exécutant la commande appropriée en fonction de votre préférence de langage :

    **C# :**

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.3.0
    ```

    **Python** :

    ```
    pip install azure-ai-textanalytics==5.3.0
    ```

1. Dans le volet **Explorateur**, dans le dossier **custom-entities**, ouvrez le fichier de configuration correspondant à votre langage préféré.

    - **C#** : appsettings.json
    - **Python** : .env
    
1. Mettez à jour les valeurs de configuration de sorte à inclure un **point de terminaison** et une **clé** de la ressource Azure Language que vous avez créée (disponible sur la page **Clés et point de terminaison** de votre ressource Azure AI Language dans le Portail Azure). Le fichier doit déjà contenir les noms du projet et du déploiement de votre modèle d’extraction des entités personnalisées.
1. Enregistrez le fichier de configuration.

## Ajouter du code pour extraire les entités

Vous pouvez maintenant utiliser le service Azure AI Language pour extraire des entités personnalisées à partir de texte.

1. Développez le dossier **ads** dans le dossier **custom-entities** pour afficher les annonces classifiées que votre application va analyser.
1. Dans le dossier **custom-entities**, ouvrez le fichier de code de l’application cliente :

    - **C#** : Program.cs
    - **Python** : custom-entities.py

1. Recherchez le commentaire **Import namespaces** (Importer des espaces de noms). Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour importer les espaces de noms dont vous aurez besoin pour utiliser le kit de développement logiciel (SDK) d’analyse de texte :

    **C#**  : Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.TextAnalytics;
    ```

    **Python** : custom-entities.py

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. Dans la fonction **Main**, notez que le code permettant de charger le point de terminaison et la clé du service Azure AI Language ainsi que les noms du projet et du déploiement à partir du fichier de configuration a déjà été fourni. Recherchez ensuite le commentaire **Créer un client à l’aide du point de terminaison et de la clé**, puis ajoutez le code suivant pour créer un client pour l’API Analyse de texte :

    **C#**  : Programs.cs

    ```csharp
    // Create client using endpoint and key
    AzureKeyCredential credentials = new(aiSvcKey);
    Uri endpoint = new(aiSvcEndpoint);
    TextAnalyticsClient aiClient = new(endpoint, credentials);
    ```

    **Python** : custom-entities.py

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(ai_key)
    ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

1. Dans la fonction **Main**, notez que le code existant lit tous les fichiers du dossier **articles** et crée une liste répertoriant leur contenu. Dans le cas du code C#, une liste d’objets **TextDocumentInput** est utilisée pour que le nom de fichier corresponde à l’ID et à la langue. En Python, une simple liste du contenu du texte est utilisée.
1. Recherchez le commentaire **Extract entities** (Extraction des entités) et ajoutez le code suivant :

    **C#** : Program.cs

    ```csharp
    // Extract entities
    RecognizeCustomEntitiesOperation operation = await aiClient.RecognizeCustomEntitiesAsync(WaitUntil.Completed, batchedDocuments, projectName, deploymentName);

    await foreach (RecognizeCustomEntitiesResultCollection documentsInPage in operation.Value)
    {
        foreach (RecognizeEntitiesResult documentResult in documentsInPage)
        {
            Console.WriteLine($"Result for \"{documentResult.Id}\":");

            if (documentResult.HasError)
            {
                Console.WriteLine($"  Error!");
                Console.WriteLine($"  Document error code: {documentResult.Error.ErrorCode}");
                Console.WriteLine($"  Message: {documentResult.Error.Message}");
                Console.WriteLine();
                continue;
            }

            Console.WriteLine($"  Recognized {documentResult.Entities.Count} entities:");

            foreach (CategorizedEntity entity in documentResult.Entities)
            {
                Console.WriteLine($"  Entity: {entity.Text}");
                Console.WriteLine($"  Category: {entity.Category}");
                Console.WriteLine($"  Offset: {entity.Offset}");
                Console.WriteLine($"  Length: {entity.Length}");
                Console.WriteLine($"  ConfidenceScore: {entity.ConfidenceScore}");
                Console.WriteLine($"  SubCategory: {entity.SubCategory}");
                Console.WriteLine();
            }

            Console.WriteLine();
        }
    }
    ```

    **Python** : custom-entities.py

    ```Python
    # Extract entities
    operation = ai_client.begin_recognize_custom_entities(
        batchedDocuments,
        project_name=project_name,
        deployment_name=deployment_name
    )

    document_results = operation.result()

    for doc, custom_entities_result in zip(files, document_results):
        print(doc)
        if custom_entities_result.kind == "CustomEntityRecognition":
            for entity in custom_entities_result.entities:
                print(
                    "\tEntity '{}' has category '{}' with confidence score of '{}'".format(
                        entity.text, entity.category, entity.confidence_score
                    )
                )
        elif custom_entities_result.is_error is True:
            print("\tError with code '{}' and message '{}'".format(
                custom_entities_result.error.code, custom_entities_result.error.message
                )
            )
    ```

1. Enregistrez les modifications apportées à votre fichier de code.

## Tester votre application

Votre application est maintenant prête à être testée.

1. Revenez au terminal intégré du dossier **classify-text**, puis entrez la commande suivante pour exécuter le programme :

    - **C#**  : `dotnet run`
    - **Python** : `python custom-entities.py`

    > **Conseil** : vous pouvez utiliser l’icône **Agrandir le volet** (**^**) dans la barre d’outils du terminal pour mieux voir le texte de la console.

1. Observez la sortie. L’application doit répertorier les détails des entités trouvées dans chaque fichier texte.

## Nettoyage

Quand vous n’avez plus besoin de votre projet, vous pouvez le supprimer de la page **Projets** dans Language Studio. Vous pouvez aussi supprimer le service Azure AI Language et le compte de stockage associé dans le [portail Azure](https://portal.azure.com).
