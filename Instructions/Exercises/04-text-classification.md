---
lab:
  title: Classification de texte personnalisée
  module: Module 3 - Getting Started with Natural Language Processing
---

# Classification de texte personnalisée

Azure AI Language fournit plusieurs fonctionnalités de traitement du langage naturel, notamment l’identification de phrases clés, le résumé de texte et l’analyse des sentiments. Le service Language fournit également des fonctionnalités personnalisées comme les réponses aux questions personnalisées et la classification de texte personnalisée.

Pour tester la classification de texte personnalisée du service Azure AI Language, nous allons configurer le modèle en utilisant Language Studio, puis utiliser une petite application en ligne de commande qui s’exécute dans Cloud Shell pour le tester. Le même modèle et les mêmes fonctionnalités que ceux utilisés ici peuvent être suivis pour les applications réelles.

## Configurer une ressource *Azure AI Language*

Si vous n’en avez pas encore dans votre abonnement, vous devez approvisionner une ressource **Service Azure AI Language**. En outre, utilisez la classification de texte personnalisée. Pour cela, vous devez activer la fonctionnalité **Classification et extraction de texte personnalisées**.

1. Dans un navigateur, ouvrez le Portail Azure (`https://portal.azure.com`) et connectez-vous avec votre compte Microsoft.
1. Sélectionnez le champ de recherche en haut du Portail, recherchez `Azure AI services` et créez une ressource **Language Service**.
1. Cochez la case qui inclut **Classification de texte personnalisée**. Sélectionnez ensuite **Continuer pour créer votre ressource**.
1. Créez une ressource avec les paramètres suivants :
    - **Abonnement** : *votre abonnement Azure*.
    - **Groupe de ressources** : *sélectionnez ou créez un groupe de ressources*.
    - **Région** : *choisissez n’importe quelle région disponible* :
    - **Nom** : *entrez un nom unique.*
    - **Niveau tarifaire** : sélectionnez **F0** (*gratuit*). Si cette option n’est pas disponible, sélectionnez **S** (*standard*).
    - **Compte de stockage** : nouveau compte de stockage
      - **Nom du compte de stockage** : *entrez un nom unique*.
      - **Type de compte de stockage** : Standard LRS
    - **Avis sur l’IA responsable** : sélectionné.

1. Sélectionnez **Vérifier + créer**, puis **Créer** pour provisionner la ressource.
1. Attendez la fin du déploiement, puis accédez à la ressource déployée.
1. Consultez la page **Clés et points de terminaison**. Plus loin dans l’exercice, vous aurez besoin des informations disponibles sur cette page.

## Charger des exemples d’articles

Une fois que vous avez créé le service et le compte de stockage Azure AI Language, vous devez charger des exemples d’articles pour entraîner votre modèle par la suite.

1. Dans un nouvel onglet de navigateur, téléchargez des exemples d’articles à partir de `https://aka.ms/classification-articles` et extrayez les fichiers dans un dossier de votre choix.

1. Dans le Portail Azure, accédez au compte de stockage que vous avez créé et sélectionnez-le.

1. Dans votre compte de stockage, sélectionnez **Configuration** sous les **Paramètres**. Dans l’écran de configuration, activez l’option **Autoriser l’accès anonyme aux objets blob**, puis sélectionnez **Enregistrer**.

1. Sélectionnez **Conteneurs** dans le menu de gauche, situé sous **Stockage de données**. Dans l’écran qui s’affiche, sélectionnez **+ Conteneur**. Donnez au conteneur le nom `articles`, puis définissez le **niveau d’accès anonyme** sur **Conteneur (accès en lecture anonyme pour les conteneurs et les objets blob)**.

    > **REMARQUE** : lorsque vous configurez un compte de stockage pour une solution réelle, veillez à attribuer le niveau d’accès approprié. Pour en savoir plus sur chaque niveau d’accès, consultez la [documentation de Stockage Azure](https://learn.microsoft.com/azure/storage/blobs/anonymous-read-access-configure).

1. Une fois que vous avez créé le conteneur, sélectionnez-le, puis sélectionnez le bouton **Charger**. Sélectionnez **Parcourir les fichiers** pour rechercher les exemples d’articles que vous avez téléchargés. Sélectionnez **Charger**.

## Créer un projet de classification de texte personnalisée

Une fois la configuration terminée, créez un projet de classification de texte personnalisée. Ce projet fournit un lieu de travail pour générer, entraîner et déployer votre modèle.

> **REMARQUE** : ce labo utilise **Language Studio**, mais vous pouvez également créer, générer, entraîner et déployer votre modèle avec l’API REST.

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

1. En haut du portail, dans le menu **Créer**, sélectionnez **Classification de texte personnalisée**.
1. La page **Connecter le stockage** apparaît. Toutes les valeurs ont déjà été renseignées. Sélectionnez **Suivant**.
1. Dans la page **Sélectionner le type de projet**, sélectionnez **Classification en une seule étiquette**. Sélectionnez ensuite **Suivant**.
1. Dans le volet **Entrer les informations de base**, définissez les éléments suivants :
    - **Nom :** `ClassifyLab`  
    - **Langue principale du texte** : anglais (US)
    - **Description** : `Custom text lab`

1. Cliquez sur **Suivant**.
1. Sur la page **Choisir un conteneur**, définissez la liste déroulante **Conteneur de magasin d’objets blob** sur votre conteneur d’*articles*.
1. Sélectionnez l’option **Non, je dois étiqueter mes fichiers dans le cadre de ce projet**. Sélectionnez ensuite **Suivant**.
1. Sélectionnez **Créer un projet**.

> **Conseil** : si vous obtenez un message d’erreur indiquant que vous n’êtes pas autorisé à effectuer cette opération, vous devez ajouter une attribution de rôle. Pour résoudre ce problème, le rôle « Contributeur aux données Blob du stockage » est ajouté au compte de stockage de l’utilisateur qui exécute le labo. Pour davantage de précisions, référez-vous à la [page de documentation.](https://learn.microsoft.com/azure/ai-services/language-service/custom-named-entity-recognition/how-to/create-project?tabs=portal%2Clanguage-studio#enable-identity-management-for-your-resource).

## Étiqueter vos données

Maintenant que votre projet est créé, vous devez étiqueter vos données pour entraîner votre modèle à classifier du texte.

1. Sur la gauche, sélectionnez **Étiquetage des données**, si ce n’est pas déjà fait. Vous voyez la liste des fichiers que vous avez chargés sur votre compte de stockage.
1. Sur le côté droit, dans le volet **Activité**, sélectionnez **+ Ajouter une classe**.  Les articles de ce labo appartiennent à quatre classes que vous devez créer : `Classifieds`, `Sports`, `News` et `Entertainment`.

    ![Capture d’écran montrant la page de données d’étiquette et le bouton Ajouter une classe.](../media/tag-data-add-class-new.png#lightbox)

1. Une fois que vous avez créé vos quatre classes, sélectionnez **Article 1** pour commencer. Ici, vous pouvez lire l’article, définir la classe du fichier et le jeu de données (entraînement ou test) auquel l’attribuer.
1. Attribuez à chaque article la classe et le jeu de données appropriés (entraînement ou test) à l’aide du volet **Activité** à droite.  Vous pouvez sélectionner une étiquette dans la liste des étiquettes à droite et définir chaque article sur **Entraînement** ou **Test** à l’aide des options en bas du volet Activité. Sélectionnez **Document suivant** pour passer au document suivant. Pour les besoins de ce labo, nous allons définir ceux à utiliser pour l’entraînement du modèle et ceux pour le test du modèle :

    | Article  | Classe  | Dataset  |
    |---------|---------|---------|
    | Article 1 | Sports | Entrainement |
    | Article 10 | Actualités | Entrainement |
    | Article 11 | Divertissement | Test |
    | Article 12 | Actualités | Test |
    | Article 13 | Sports | Test |
    | Article 2 | Sports | Entrainement |
    | Article 3 | Classifieds | Entrainement |
    | Article 4 | Classifieds | Entrainement |
    | Article 5 | Divertissement | Entrainement |
    | Article 6 | Divertissement | Entrainement |
    | Article 7 | Actualités | Entrainement |
    | Article 8 | Actualités | Entrainement |
    | Article 9 | Divertissement | Entrainement |

    > **REMARQUE** : les fichiers dans Language Studio sont triés par ordre alphabétique, c’est pourquoi la liste ci-dessus n’est pas dans l’ordre séquentiel. Veillez à consulter les deux pages de documents quand vous étiquetez vos articles.

1. Sélectionnez **Enregistrer les étiquettes** pour enregistrer vos étiquettes.

## Entraîner votre modèle

Après avoir étiqueté vos données, vous devez entraîner votre modèle.

1. Dans le menu de gauche, sélectionnez **Travaux d’entraînement**.
1. Sélectionnez **Start a training job** (Lancer une exécution d’entraînement).
1. Entraînez un nouveau modèle appelé `ClassifyArticles`.
1. Sélectionnez **Utiliser une division manuelle des données d’entraînement et de test**.

    > **CONSEIL** : dans vos propres projets de classification, le service Azure AI Language divise automatiquement le jeu de tests selon le pourcentage, ce qui est utile pour les grands jeux de données. Pour les jeux de données plus petits, vous devez effectuer l’entraînement avec la distribution de classes appropriée.

1. Sélectionnez **Train**.

> **IMPORTANT** : l’entraînement de votre modèle peut parfois prendre plusieurs minutes. Vous recevez une notification une fois l’opération terminée.

## Évaluer votre modèle

Dans les applications réelles de classification de texte, il est important d’évaluer votre modèle et de l’améliorer pour vérifier qu’il fonctionne comme prévu.

1. Sélectionnez **Performances du modèle**, puis votre modèle **ClassifierArticles**. Ici, vous pouvez voir le scoring de votre modèle, ses métriques de performances et quand il a été entraîné. Si le score de votre modèle n’est pas de 100 %, cela signifie que l’un des documents utilisés pour les tests n’a pas été évalué correctement par rapport à son étiquette initiale. Ces échecs vous aident à comprendre ce qu’il faut améliorer.
1. Sélectionnez l’onglet **Détails du jeu de test**. En cas d’erreurs, cet onglet vous permet de voir les articles que vous avez indiqués pour les tests, sous quelle étiquette le modèle les a prédits et si cela entre en conflit avec leur étiquette de test. Par défaut, l’onglet affiche uniquement les prédictions incorrectes. Vous pouvez activer/désactiver l’option **Afficher les non-correspondances uniquement** pour voir tous les articles que vous avez indiqués pour les tests et sous quelle étiquette chacun d’eux a été prédit.

## Déployer votre modèle

Une fois que vous êtes satisfait de l’entraînement de votre modèle, vous pouvez le déployer, pour ensuite commencer à classifier du texte avec l’API.

1. Dans le panneau de gauche, sélectionnez **Déploiement du modèle**.
1. Sélectionnez **Ajouter un déploiement**, entrez `articles` dans le champ **Créer un nom de déploiement**, puis sélectionnez **ClassifierArticles** dans le champ **Modèle**.
1. Sélectionnez **Déployer** pour déployer votre modèle.
1. Une fois votre modèle déployé, laissez cette page ouverte. Vous aurez besoin du nom de votre projet et de votre déploiement à l’étape suivante.

## Préparer le développement d’une application dans Visual Studio Code

Pour tester les fonctionnalités de classification de texte personnalisée du service Azure AI Language, vous allez développer une application console simple dans Visual Studio Code.

> **Conseil** : si vous avez déjà cloné le référentiel **mslearn-ai-language**, ouvrez-le dans Visual Studio Code. Dans le cas contraire, procédez comme suit pour le cloner dans votre environnement de développement.

1. Démarrez Visual Studio Code.
2. Ouvrez la palette (Maj+CTRL+P) et exécutez une commande **Git : Cloner** pour cloner le référentiel `https://github.com/MicrosoftLearning/mslearn-ai-language` vers un dossier local (peu importe quel dossier).
3. Lorsque le référentiel a été cloné, ouvrez le dossier dans Visual Studio Code.

    > **Remarque** : Si Visual Studio Code affiche un message contextuel qui vous invite à approuver le code que vous ouvrez, cliquez sur l’option **Oui, je fais confiance aux auteurs** dans la fenêtre contextuelle.

4. Attendez que des fichiers supplémentaires soient installés pour prendre en charge les projets de code C# dans le référentiel.

    > **Remarque** : si vous êtes invité à ajouter des ressources requises pour générer et déboguer, sélectionnez **Not Now** (Pas maintenant).

## Configuration de votre application

Des applications pour C# et Python sont fournies, ainsi qu’un exemple de fichier texte que vous utiliserez pour tester le résumé. Les deux applications présentent les mêmes fonctionnalités. Premièrement, vous allez terminer certaines parties clés de l’application pour lui permettre d’utiliser votre ressource Azure AI Language.

1. Dans Visual Studio Code, dans le volet **Explorateur**, accédez au dossier **Labfiles/04-text-classification** et développez le dossier **CSharp** ou **Python** en fonction de votre préférence de langage, ainsi que le dossier **classify-text** qu’il contient. Chaque dossier contient les fichiers propres au langage d’une application dans laquelle vous allez intégrer des fonctionnalités de classification de texte d’Azure AI Language.
1. Cliquez avec le bouton droit sur le dossier **classify-text**, qui contient vos fichiers de code, et ouvrez un terminal intégré. Installez ensuite le package du kit de développement logiciel (SDK) d’analyse de texte d’Azure AI Language en exécutant la commande appropriée en fonction de votre préférence de langage :

    **C# :**

    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.3.0
    ```

    **Python** :

    ```
    pip install azure-ai-textanalytics==5.3.0
    ```

1. Dans le volet **Explorateur**, dans le dossier **classify-text**, ouvrez le fichier de configuration correspondant à votre langage préféré.

    - **C#** : appsettings.json
    - **Python** : .env
    
1. Mettez à jour les valeurs de configuration de sorte à inclure un **point de terminaison** et une **clé** de la ressource Azure Language que vous avez créée (disponible sur la page **Clés et point de terminaison** de votre ressource Azure AI Language dans le Portail Azure). Le fichier doit déjà contenir les noms du projet et du déploiement de votre modèle de classification de texte.
1. Enregistrez le fichier de configuration.

## Ajouter du code pour classifier des documents

Vous pouvez maintenant utiliser le service Azure AI Language pour classifier des documents.

1. Développez le dossier **articles** dans le dossier **classify-text** pour afficher les articles de texte que votre application va classifier.
1. Dans le dossier **classify-text**, ouvrez le fichier de code de l’application cliente :

    - **C#** : Program.cs
    - **Python** : classify-text.py

1. Recherchez le commentaire **Import namespaces** (Importer des espaces de noms). Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour importer les espaces de noms dont vous aurez besoin pour utiliser le kit de développement logiciel (SDK) d’analyse de texte :

    **C#**  : Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.TextAnalytics;
    ```

    **Python** : classify-text.py

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. Dans la fonction **Main**, notez que le code permettant de charger le point de terminaison et la clé du service Azure AI Language ainsi que les noms du projet et du déploiement à partir du fichier de configuration a déjà été fourni. Recherchez ensuite le commentaire **Créer un client à l’aide du point de terminaison et de la clé**, puis ajoutez le code suivant pour créer un client pour l’API Analyse de texte :

    **C#**  : Programs.cs

    ```csharp
    // Create client using endpoint and key
    AzureKeyCredential credentials = new AzureKeyCredential(aiSvcKey);
    Uri endpoint = new Uri(aiSvcEndpoint);
    TextAnalyticsClient aiClient = new TextAnalyticsClient(endpoint, credentials);
    ```

    **Python** : classify-text.py

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(ai_key)
    ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

1. Dans la fonction **Main**, notez que le code existant lit tous les fichiers du dossier **articles** et crée une liste répertoriant leur contenu. Recherchez ensuite le commentaire **Get Classifications** (Obtenir les classifications) et ajoutez le code suivant :

    **C#** : Program.cs

    ```csharp
    // Get Classifications
    ClassifyDocumentOperation operation = await aiClient.SingleLabelClassifyAsync(WaitUntil.Completed, batchedDocuments, projectName, deploymentName);

    int fileNo = 0;
    await foreach (ClassifyDocumentResultCollection documentsInPage in operation.Value)
    {
        
        foreach (ClassifyDocumentResult documentResult in documentsInPage)
        {
            Console.WriteLine(files[fileNo].Name);
            if (documentResult.HasError)
            {
                Console.WriteLine($"  Error!");
                Console.WriteLine($"  Document error code: {documentResult.Error.ErrorCode}");
                Console.WriteLine($"  Message: {documentResult.Error.Message}");
                continue;
            }

            Console.WriteLine($"  Predicted the following class:");
            Console.WriteLine();

            foreach (ClassificationCategory classification in documentResult.ClassificationCategories)
            {
                Console.WriteLine($"  Category: {classification.Category}");
                Console.WriteLine($"  Confidence score: {classification.ConfidenceScore}");
                Console.WriteLine();
            }
            fileNo++;
        }
    }
    ```
    
    **Python** : classify-text.py

    ```Python
    # Get Classifications
    operation = ai_client.begin_single_label_classify(
        batchedDocuments,
        project_name=project_name,
        deployment_name=deployment_name
    )

    document_results = operation.result()

    for doc, classification_result in zip(files, document_results):
        if classification_result.kind == "CustomDocumentClassification":
            classification = classification_result.classifications[0]
            print("{} was classified as '{}' with confidence score {}.".format(
                doc, classification.category, classification.confidence_score)
            )
        elif classification_result.is_error is True:
            print("{} has an error with code '{}' and message '{}'".format(
                doc, classification_result.error.code, classification_result.error.message)
            )
    ```

1. Enregistrez les modifications apportées à votre fichier de code.

## Tester votre application

Votre application est à présent prête à être testée.

1. Dans le terminal intégré pour le dossier **classify-text**, entrez la commande suivante pour exécuter le programme :

    - **C#**  : `dotnet run`
    - **Python** : `python classify-text.py`

    > **Conseil** : vous pouvez utiliser l’icône **Agrandir la taille du volet** (**^**) dans la barre d’outils du terminal pour afficher plus de texte de la console.

1. Observez la sortie. L’application doit indiquer une classification et un score de confiance pour chaque fichier texte.


## Nettoyage

Quand vous n’avez plus besoin de votre projet, vous pouvez le supprimer de la page **Projets** dans Language Studio. Vous pouvez aussi supprimer le service Azure AI Language et le compte de stockage associé dans le [portail Azure](https://portal.azure.com).
