---
lab:
  title: Créer un modèle de compréhension du langage avec le service Azure AI Language
  module: Module 5 - Create language understanding solutions
---

# Créer un modèle de compréhension du langage avec le service Langage

> **REMARQUE** : la fonctionnalité de compréhension du langage courant du service Azure AI Language est actuellement en version préliminaire et peut être modifiée. Dans certains cas, l’entraînement du modèle peut échouer. Si cela se produit, réessayez.  

Le service Azure AI Language vous permet de définir un *modèle de compréhension du langage courant* que les applications peuvent utiliser pour interpréter l’entrée en langage naturel des utilisateurs, prédire l’*intention* des utilisateurs (les résultats souhaités) et identifier toutes les *entités* auxquelles l’intention doit être appliquée.

Par exemple, on peut s’attendre à ce qu’un modèle de langage courant pour une application d’horloge traite des entrées telles que :

*Quelle heure est-il à Londres ?*

Ce type d’entrée est un exemple d’*énoncé* (quelque chose qu’un utilisateur peut dire ou type), pour lequel l’*intention* souhaitée est d’obtenir l’heure pour un emplacement spécifique (une *entité*) ; dans ce cas, Londres.

> **REMARQUE** : la tâche d’un modèle de langage courant consiste à prédire l’intention de l’utilisateur et à identifier toutes les entités auxquelles l’intention s’applique. La tâche d’un modèle de langage courant n’est <u>pas</u> d’effectuer réellement les actions requises pour satisfaire l’intention. Par exemple, une application d’horloge peut utiliser un modèle de langage courant pour discerner que l’utilisateur souhaite connaître l’heure à Londres ; mais l’application cliente elle-même doit ensuite implémenter la logique pour déterminer la bonne heure et la présenter à l’utilisateur.

## Configurer une ressource *Azure AI Language*

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
1. Consultez la page **Clés et points de terminaison**. Plus loin dans l’exercice, vous aurez besoin des informations disponibles sur cette page.

## Créer un projet de compréhension du langage courant

Maintenant que vous avez créé une ressource de création, vous pouvez l’utiliser pour créer un projet de compréhension du langage courant.

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

1. En haut du portail, dans le menu **Créer**, sélectionnez **Conversational Language Understanding**.

1. Dans la boîte de dialogue **Créer un projet**, sur la page **Entrer les informations de base**, entrez les détails suivants, puis sélectionnez **Suivant** :
    - **Nom :** `Clock`
    - **Langue principale des énoncés** : anglais
    - **Activer plusieurs langages dans le projet ?** : *Non sélectionné*
    - **Description** : `Natural language clock`

1. Sur la page **Vérifier et terminer**, sélectionnez **Créer**.

### Créer des intentions

La première chose que nous allons faire dans le nouveau projet consiste à définir certaines intentions. L’objectif du modèle est de prédire quelles sont les intentions spécifiques d’un utilisateur lorsqu’il envoie un énoncé en langage naturel.

> **Conseil** : lorsque vous travaillez sur votre projet, si des conseils sont affichés, lisez-les et sélectionnez **J’ai compris** ou **Ignorer tout** pour les ignorer.

1. Sur la page **Définition du schéma**, sous l’onglet **Intentions**, sélectionnez **&#65291; Ajouter** pour ajouter une nouvelle intention nommée `GetTime`.
1. Vérifiez que l’intention **GetTime** est répertoriée (ainsi que l’intention **None** par défaut). Ajoutez ensuite les intentions supplémentaires suivantes :
    - `GetDay`
    - `GetDate`

### Étiqueter chaque intention avec des exemples d’énoncés

Pour aider le modèle à prédire l’intention demandée par un utilisateur, vous devez étiqueter chaque option avec des exemples d’énoncés.

1. Dans le volet de gauche, sélectionnez la page **Étiquetage des données**.

> **Conseil** : vous pouvez développer le volet avec l’icône **>>** pour afficher les noms des pages. Pour masquer les détails, utilisez l’icône **<<**.

1. Sélectionnez la nouvelle intention **GetTime** et entrez l’énoncé `what is the time?`. Cet énoncé devient un exemple d’entrée pour l’intention.
1. Ajoutez les énoncés supplémentaires suivants pour l’intention **GetTime** :
    - `what's the time?`
    - `what time is it?`
    - `tell me the time`

1. Sélectionnez l’intention **GetDay** et ajoutez les énoncés suivants comme exemples d’entrée utilisateur :
    - `what day is it?`
    - `what's the day?`
    - `what is the day today?`
    - `what day of the week is it?`

1. Sélectionnez l’intention **GetDate** et ajoutez-lui les énoncés suivants :
    - `what date is it?`
    - `what's the date?`
    - `what is the date today?`
    - `what's today's date?`

1. Une fois que vous avez ajouté des énoncés pour chacune de vos intentions, sélectionnez **Enregistrer les modifications**.

### Entraîner et tester le modèle

Maintenant que vous avez ajouté certaines intentions, nous allons entraîner le modèle de langage et voir s’il peut correctement les prédire à partir de l’entrée utilisateur.

1. Dans le volet de gauche, sélectionnez **Travaux d’entraînement**. Ensuite, sélectionnez **+ Démarrer un travail d’entraînement**.

1. Dans la boîte de dialogue **Démarrer un travail de formation**, sélectionnez l’option pour former un nouveau modèle et nommez-le `Clock`. Sélectionnez le mode **Entraînement standard** et utilisez les options par défaut pour la section **Fractionnement des données**.

1. Pour commencer le processus d’entraînement de votre modèle, sélectionnez **Entraîner**.

1. Une fois l’entraînement terminé (ce qui peut prendre plusieurs minutes), l’**État** du travail passe à **Apprentissage réussi**.

1. Sélectionnez la page **Performances du modèle**, puis sélectionnez le modèle **Clock**. Passez en revue les métriques d’évaluation globales et par intention (*précision*, *rappel* et *score F1*) et la *matrice de confusion* générée par l’évaluation effectuée lors de l’entraînement (notez qu’en raison du petit nombre d’exemples d’énoncés, toutes les intentions ne peuvent pas être incluses dans les résultats).

    > **REMARQUE** : pour en savoir plus sur les métriques d’évaluation, consultez la [documentation](https://learn.microsoft.com/azure/ai-services/language-service/conversational-language-understanding/concepts/evaluation-metrics).

1. Accédez à la page **Déploiement d’un modèle**, puis sélectionnez **Ajouter un déploiement**.

1. Dans la boîte de dialogue **Ajouter un déploiement**, sélectionnez **Créer un nom de déploiement**, puis entrez `production`.

1. Sélectionnez le modèle **Horloge** dans le champ **Modèle**, puis sélectionnez **Déployer**. Le déploiement peut prendre un certain temps.

1. Une fois le modèle déployé, sélectionnez la page **Tests des déploiements**, puis sélectionnez le déploiement **production** dans le champ **Nom du déploiement**.

1. Dans le champ de texte vide, entrez ce qui suit, puis sélectionnez **Exécuter le test** :

    `what's the time now?`

    Passez en revue le résultat retourné, en notant qu’il inclut l’intention prédite (qui doit être **GetTime**) et un score de confiance qui indique la probabilité calculée par le modèle pour l’intention prédite. L’onglet JSON montre la confiance comparative pour chaque intention potentielle (celle qui a le score de confiance le plus élevé est l’intention prédite)

1. Effacez la zone de texte, puis exécutez un autre test avec le texte suivant :

    `tell me the time`

    Là encore, passez en revue l’intention prédite et le score de confiance.

1. Essayez le texte suivant :

    `what's the day today?`

    Espérons que le modèle prédit l’intention **GetDay**.

## Ajouter des entités

Jusqu’à présent, vous avez défini des énoncés simples qui mappent aux intentions. La plupart des applications réelles incluent des énoncés plus complexes à partir desquels des entités de données spécifiques doivent être extraites pour obtenir plus de contexte pour l’intention.

### Ajouter une entité apprise

Le type d’entité le plus courant est une entité *apprise*, dans laquelle le modèle apprend à identifier les valeurs d’entité en fonction d’exemples.

1. Dans Language Studio, revenez à la page **Définition du schéma** puis, sous l’onglet **Entités**, sélectionnez **&#65291; Ajouter** pour ajouter une nouvelle entité.

1. Dans la boîte de dialogue **Ajouter une entité**, entrez le nom d’entité `Location` et vérifiez que l’onglet **Apprise** est sélectionné. Ensuite, sélectionnez **Ajouter une entité**.

1. Une fois l’entité **Emplacement** créée, revenez à la page **Étiquetage des données**.
1. Sélectionnez l’intention **GetTime** et entrez le nouvel exemple d’énoncé suivant :

    `what time is it in London?`

1. Une fois l’énoncé ajouté, sélectionnez le mot **Londres**, puis dans la liste déroulante qui s’affiche, sélectionnez **Emplacement** pour indiquer que « Londres » est un exemple d’emplacement.

1. Ajoutez un autre exemple d’énoncé pour l’intention **GetTime** :

    `Tell me the time in Paris?`

1. Une fois l’énoncé ajouté, sélectionnez le mot **Paris**, puis mappez-le à l’entité **Emplacement**.

1. Ajoutez un autre exemple d’énoncé pour l’intention **GetTime** :

    `what's the time in New York?`

1. Une fois l’énoncé ajouté, sélectionnez le mot **New York**, puis mappez-les à l’entité **Emplacement**.

1. Sélectionnez **Enregistrer les modifications** pour enregistrer les nouveaux énoncés.

### Ajouter une entité de *liste*

Dans certains cas, les valeurs valides pour une entité peuvent être limitées à une liste de termes et de synonymes spécifiques, qui peut aider l’application à identifier les instances de l’entité dans les énoncés.

1. Dans Language Studio, revenez à la page **Définition du schéma** puis, sous l’onglet **Entités**, sélectionnez **&#65291; Ajouter** pour ajouter une nouvelle entité.

1. Dans la boîte de dialogue **Ajouter une entité**, entrez le nom d’entité `Weekday` et ouvrez l’onglet d’entité **Liste**, puis sélectionnez **Ajouter une entité**.

1. Sur la page de l’entité **Jour de la semaine**, dans la section **Apprise** , vérifiez que l’option **Non obligatoire** est sélectionnée. Ensuite, dans la section **Liste** , sélectionnez **&#65291; Ajouter une nouvelle liste**. Entrez ensuite la valeur et le synonyme suivants, puis sélectionnez **Enregistrer** :

    | Clé de liste | Synonymes|
    |-------------------|---------|
    | `Sunday` | `Sun` |

1. Répétez l’étape précédente pour ajouter les composants de liste suivants :

    | Valeur | Synonymes|
    |-------------------|---------|
    | `Monday` | `Mon` |
    | `Tuesday` | `Tue, Tues` |
    | `Wednesday` | `Wed, Weds` |
    | `Thursday` | `Thur, Thurs` |
    | `Friday` | `Fri` |
    | `Saturday` | `Sat` |

1. Après avoir ajouté et enregistré les valeurs de la liste, revenez à la page **Étiquetage des données**.
1. Sélectionnez l’intention **GetDate** et entrez le nouvel exemple d’énoncé suivant :

    `what date was it on Saturday?`

1. Une fois l’énoncé ajouté, sélectionnez le mot ***Samedi***, puis dans la liste déroulante qui s’affiche, sélectionnez **Jour de la semaine**.

1. Ajoutez un autre exemple d’énoncé pour l’intention **GetDate** :

    `what date will it be on Friday?`

1. Une fois l’énoncé ajouté, mappez **Vendredi** à l’entité **Jour de la semaine**.

1. Ajoutez un autre exemple d’énoncé pour l’intention **GetDate** :

    `what will the date be on Thurs?`

1. Une fois l’énoncé ajouté, mappez **Jeudi** à l’entité **Jour de la semaine**.

1. Sélectionnez **Enregistrer les modifications** pour enregistrer les nouveaux énoncés.

### Ajouter une entité *prédéfinie*

Le service Azure AI Language fournit un ensemble d’entités *prédéfinies* couramment utilisées dans les applications conversationnelles.

1. Dans Language Studio, revenez à la page **Définition du schéma** puis, sous l’onglet **Entités**, sélectionnez **&#65291; Ajouter** pour ajouter une nouvelle entité.

1. Dans la boîte de dialogue **Ajouter une entité**, entrez le nom d’entité `Date` et ouvrez l’onglet d’entité **Prédéfinie**, puis sélectionnez **Ajouter une entité**.

1. Sur la page de l’entité **Date**, dans la section **Apprise** , vérifiez que l’option **Non obligatoire** est sélectionnée. Ensuite, dans la section **Prédéfinie** , sélectionnez **&#65291; Ajouter une nouvelle entité prédéfinie**.

1. Dans la liste **Sélectionner une entité prédéfinie**, sélectionnez **DateTime**, puis **Enregistrer**.
1. Après avoir ajouté l’entité prédéfinie, retournez à la page **d’étiquetage des données**
1. Sélectionnez l’intention **GetDay** et entrez le nouvel exemple d’énoncé suivant :

    `what day was 01/01/1901?`

1. Une fois l’énoncé ajouté, sélectionnez ***01/01/1901***, puis dans la liste déroulante qui s’affiche, sélectionnez **Date**.

1. Ajoutez un autre exemple d’énoncé pour l’intention **GetDay** :

    `what day will it be on Dec 31st 2099?`

1. Une fois l’énoncé ajouté, mappez **31 déc 2099** à l’entité **Date**.

1. Sélectionnez **Enregistrer les modifications** pour enregistrer les nouveaux énoncés.

### Recycler le modèle

Maintenant que vous avez modifié le schéma, vous devez réentraîner et retester le modèle.

1. Dans la page **Travaux d’entraînement**, sélectionnez **Démarrer un travail d’entraînement**.

1. Dans la boîte de dialogue **Démarrer un travail d’entraînement**, sélectionnez **l’option permettant de remplacer un modèle existant** et spécifiez le modèle **Horloge**. Pour entraîner le modèle, sélectionnez **Entraîner**. Si le système vous y invite, confirmez que vous souhaitez remplacer le modèle existant.

1. Une l’entraînement terminé, l’**État** du travail est mis à jour vers **Entraînement réussi**.

1. Sélectionnez la page **Performances du modèle**, puis sélectionnez le modèle **Clock**. Passez en revue les métriques d’évaluation (*précision*, *rappel* et *score F1*) et la *matrice de confusion* générée par l’évaluation effectuée lors de l’entraînement (notez qu’en raison du petit nombre d’exemples d’énoncés, toutes les intentions ne peuvent pas être incluses dans les résultats).

1. Dans la page **Déploiement d’un modèle**, sélectionnez **Ajouter un déploiement**.

1. Dans la boîte de dialogue **Ajouter un déploiement**, sélectionnez **Remplacer un nom de déploiement existant**, puis sélectionnez **production**.

1. Sélectionnez le modèle **Horloge** dans le champ **Modèle**, puis sélectionnez **Déployer**. Cette opération peut prendre un certain temps.

1. Une fois le modèle déployé, sur la page **Test des déploiements**, sélectionnez le déploiement **production** sous le champ **Nom du déploiement**, puis testez-le avec le texte suivant :

    `what's the time in Edinburgh?`

1. Passez en revue le résultat retourné, qui devrait prédire l’intention **GetTime** et une entité **Emplacement** avec la valeur de texte « Édimbourg ».

1. Essayez de tester les énoncés suivants :

    `what time is it in Tokyo?`

    `what date is it on Friday?`

    `what's the date on Weds?`

    `what day was 01/01/2020?`

    `what day will Mar 7th 2030 be?`

## Utiliser le modèle à partir d’une application cliente

Dans un projet réel, vous affineriez de manière itérative les intentions et les entités, entraîneriez à nouveau et retesteriez jusqu’à ce que vous soyez satisfait des performances prédictives. Ensuite, une fois que vous auriez testé le modèle et que vous seriez satisfait de ses performances prédictives, vous pourriez l’utiliser dans une application cliente en appelant son interface REST ou un SDK spécifique au runtime.

### Préparer le développement d’une application dans Visual Studio Code

Vous allez développer votre application de compréhension du langage à l’aide de Visual Studio Code. Les fichiers de code de votre application ont été fournis dans un référentiel GitHub.

> **Conseil** : si vous avez déjà cloné le référentiel **mslearn-ai-language**, ouvrez-le dans Visual Studio Code. Dans le cas contraire, procédez comme suit pour le cloner dans votre environnement de développement.

1. Démarrez Visual Studio Code.
2. Ouvrez la palette (Maj+CTRL+P) et exécutez une commande **Git : Cloner** pour cloner le référentiel `https://github.com/MicrosoftLearning/mslearn-ai-language` vers un dossier local (peu importe quel dossier).
3. Lorsque le référentiel a été cloné, ouvrez le dossier dans Visual Studio Code.

    > **Remarque** : Si Visual Studio Code affiche un message contextuel qui vous invite à approuver le code que vous ouvrez, cliquez sur l’option **Oui, je fais confiance aux auteurs** dans la fenêtre contextuelle.

4. Attendez que des fichiers supplémentaires soient installés pour prendre en charge les projets de code C# dans le référentiel.

    > **Remarque** : si vous êtes invité à ajouter des ressources requises pour générer et déboguer, sélectionnez **Not Now** (Pas maintenant).

### Configuration de votre application

Des applications pour C# et Python sont fournies, ainsi qu’un exemple de fichier texte que vous utiliserez pour tester le résumé. Les deux applications présentent les mêmes fonctionnalités. Premièrement, vous allez terminer certaines parties clés de l’application pour lui permettre d’utiliser votre ressource Azure AI Language.

1. Dans Visual Studio Code, dans le volet **Explorateur**, accédez au dossier **Labfiles/03-language** et développez le dossier **CSharp** ou **Python** en fonction de votre préférence de langage, ainsi que le dossier **clock-client** qu’il contient. Chaque dossier contient les fichiers propres au langage d’une application dans laquelle vous allez intégrer des fonctionnalités de réponse aux questions d’Azure AI Language.
2. Cliquez avec le bouton droit sur le dossier **clock-client**, qui contient vos fichiers de code, et ouvrez un terminal intégré. Installez ensuite le package du kit de développement logiciel (SDK) de compréhension du langage courant d’Azure AI Language en exécutant la commande appropriée en fonction de votre préférence de langage :

    **C# :**

    ```
    dotnet add package Azure.AI.Language.Conversations --version 1.1.0
    ```

    **Python** :

    ```
    pip install azure-ai-language-conversations
    ```

3. Dans le volet **Explorateur**, dans le dossier **clock-client**, ouvrez le fichier de configuration correspondant à votre langage préféré.

    - **C#** : appsettings.json
    - **Python** : .env
    
4. Mettez à jour les valeurs de configuration de sorte à inclure un **point de terminaison** et une **clé** de la ressource Azure Language que vous avez créée (disponible sur la page **Clés et point de terminaison** de votre ressource Azure AI Language dans le Portail Azure).
5. Enregistrez le fichier de configuration.

### Ajouter du code à l’application

Vous pouvez maintenant ajouter le code nécessaire pour importer les bibliothèques de SDK requises, établir une connexion authentifiée à votre projet déployé et poser des questions.

1. Notez que le dossier **clock-client** contient un fichier de code pour l’application cliente :

    - **C#** : Program.cs
    - **Python** : clock-client.py

    Ouvrez le fichier de code et, en haut, sous les références d’espace de noms existantes, recherchez le commentaire **Importer des espaces de noms**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour importer les espaces de noms dont vous aurez besoin pour utiliser le kit de développement logiciel (SDK) d’analyse de texte :

    **C#**  : Programs.cs

    ```c#
    // import namespaces
    using Azure;
    using Azure.AI.Language.Conversations;
    ```

    **Python** : clock-client.py

    ```python
    # Import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.language.conversations import ConversationAnalysisClient
    ```

1. Dans la fonction **Principal**, notez que le code pour charger le point de terminaison et la clé de la prédiction à partir du fichier de configuration a déjà été fourni. Recherchez ensuite le commentaire **Créer un client pour le modèle de service de langage** et ajoutez le code suivant pour créer un client de prédiction pour votre application Language Service :

    **C#**  : Programs.cs

    ```c#
    // Create a client for the Language service model
    Uri endpoint = new Uri(predictionEndpoint);
    AzureKeyCredential credential = new AzureKeyCredential(predictionKey);

    ConversationAnalysisClient client = new ConversationAnalysisClient(endpoint, credential);
    ```

    **Python** : clock-client.py

    ```python
    # Create a client for the Language service model
    client = ConversationAnalysisClient(
        ls_prediction_endpoint, AzureKeyCredential(ls_prediction_key))
    ```

1. Notez que le code de la fonction **Main** invite l’utilisateur à entrer une entrée utilisateur jusqu’à ce que l’utilisateur entre « quit ». Dans cette boucle, recherchez le commentaire **Appeler le modèle de service de langage pour obtenir l’intention et les entités** et ajouter le code suivant :

    **C#**  : Programs.cs

    ```c#
    // Call the Language service model to get intent and entities
    var projectName = "Clock";
    var deploymentName = "production";
    var data = new
    {
        analysisInput = new
        {
            conversationItem = new
            {
                text = userText,
                id = "1",
                participantId = "1",
            }
        },
        parameters = new
        {
            projectName,
            deploymentName,
            // Use Utf16CodeUnit for strings in .NET.
            stringIndexType = "Utf16CodeUnit",
        },
        kind = "Conversation",
    };
    // Send request
    Response response = await client.AnalyzeConversationAsync(RequestContent.Create(data));
    dynamic conversationalTaskResult = response.Content.ToDynamicFromJson(JsonPropertyNames.CamelCase);
    dynamic conversationPrediction = conversationalTaskResult.Result.Prediction;   
    var options = new JsonSerializerOptions { WriteIndented = true };
    Console.WriteLine(JsonSerializer.Serialize(conversationalTaskResult, options));
    Console.WriteLine("--------------------\n");
    Console.WriteLine(userText);
    var topIntent = "";
    if (conversationPrediction.Intents[0].ConfidenceScore > 0.5)
    {
        topIntent = conversationPrediction.TopIntent;
    }
    ```

    **Python** : clock-client.py

    ```python
    # Call the Language service model to get intent and entities
    cls_project = 'Clock'
    deployment_slot = 'production'

    with client:
        query = userText
        result = client.analyze_conversation(
            task={
                "kind": "Conversation",
                "analysisInput": {
                    "conversationItem": {
                        "participantId": "1",
                        "id": "1",
                        "modality": "text",
                        "language": "en",
                        "text": query
                    },
                    "isLoggingEnabled": False
                },
                "parameters": {
                    "projectName": cls_project,
                    "deploymentName": deployment_slot,
                    "verbose": True
                }
            }
        )

    top_intent = result["result"]["prediction"]["topIntent"]
    entities = result["result"]["prediction"]["entities"]

    print("view top intent:")
    print("\ttop intent: {}".format(result["result"]["prediction"]["topIntent"]))
    print("\tcategory: {}".format(result["result"]["prediction"]["intents"][0]["category"]))
    print("\tconfidence score: {}\n".format(result["result"]["prediction"]["intents"][0]["confidenceScore"]))

    print("view entities:")
    for entity in entities:
        print("\tcategory: {}".format(entity["category"]))
        print("\ttext: {}".format(entity["text"]))
        print("\tconfidence score: {}".format(entity["confidenceScore"]))

    print("query: {}".format(result["result"]["query"]))
    ```

    L’appel au modèle de service de langage retourne une prédiction/un résultat, qui inclut l’intention supérieure (la plus probable) ainsi que toutes les entités détectées dans l’énoncé d’entrée. Votre application cliente doit maintenant utiliser cette prédiction pour déterminer et effectuer l’action appropriée.

1. Recherchez le commentaire **Appliquer l’action appropriée** et ajoutez le code suivant, qui recherche les intentions prises en charge par l’application (**GetTime**, **GetDate** et **GetDay**) et détermine si des entités pertinentes ont été détectées, avant d’appeler une fonction existante pour produire une réponse appropriée.

    **C#**  : Programs.cs

    ```c#
    // Apply the appropriate action
    switch (topIntent)
    {
        case "GetTime":
            var location = "local";           
            // Check for a location entity
            foreach (dynamic entity in conversationPrediction.Entities)
            {
                if (entity.Category == "Location")
                {
                    //Console.WriteLine($"Location Confidence: {entity.ConfidenceScore}");
                    location = entity.Text;
                }
            }
            // Get the time for the specified location
            string timeResponse = GetTime(location);
            Console.WriteLine(timeResponse);
            break;
        case "GetDay":
            var date = DateTime.Today.ToShortDateString();            
            // Check for a Date entity
            foreach (dynamic entity in conversationPrediction.Entities)
            {
                if (entity.Category == "Date")
                {
                    //Console.WriteLine($"Location Confidence: {entity.ConfidenceScore}");
                    date = entity.Text;
                }
            }            
            // Get the day for the specified date
            string dayResponse = GetDay(date);
            Console.WriteLine(dayResponse);
            break;
        case "GetDate":
            var day = DateTime.Today.DayOfWeek.ToString();
            // Check for entities            
            // Check for a Weekday entity
            foreach (dynamic entity in conversationPrediction.Entities)
            {
                if (entity.Category == "Weekday")
                {
                    //Console.WriteLine($"Location Confidence: {entity.ConfidenceScore}");
                    day = entity.Text;
                }
            }          
            // Get the date for the specified day
            string dateResponse = GetDate(day);
            Console.WriteLine(dateResponse);
            break;
        default:
            // Some other intent (for example, "None") was predicted
            Console.WriteLine("Try asking me for the time, the day, or the date.");
            break;
    }
    ```

    **Python** : clock-client.py

    ```python
    # Apply the appropriate action
    if top_intent == 'GetTime':
        location = 'local'
        # Check for entities
        if len(entities) > 0:
            # Check for a location entity
            for entity in entities:
                if 'Location' == entity["category"]:
                    # ML entities are strings, get the first one
                    location = entity["text"]
        # Get the time for the specified location
        print(GetTime(location))

    elif top_intent == 'GetDay':
        date_string = date.today().strftime("%m/%d/%Y")
        # Check for entities
        if len(entities) > 0:
            # Check for a Date entity
            for entity in entities:
                if 'Date' == entity["category"]:
                    # Regex entities are strings, get the first one
                    date_string = entity["text"]
        # Get the day for the specified date
        print(GetDay(date_string))

    elif top_intent == 'GetDate':
        day = 'today'
        # Check for entities
        if len(entities) > 0:
            # Check for a Weekday entity
            for entity in entities:
                if 'Weekday' == entity["category"]:
                # List entities are lists
                    day = entity["text"]
        # Get the date for the specified day
        print(GetDate(day))

    else:
        # Some other intent (for example, "None") was predicted
        print('Try asking me for the time, the day, or the date.')
    ```

1. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **clock-client** et entrez la commande suivante pour exécuter le programme :

    - **C#**  : `dotnet run`
    - **Python** : `python clock-client.py`

    > **Conseil** : vous pouvez utiliser l’icône **Agrandir le volet** (**^**) dans la barre d’outils du terminal pour mieux voir le texte de la console.

1. Lorsque vous y êtes invité, entrez des énoncés pour tester l’application. Par exemple, essayez :

    *Bonjour*

    *Quelle heure est-il ?*

    *Quelle heure est-il à Londres ?*

    *Quelle est la date ?*

    *Quelle est la date du dimanche ?*

    *Quel jour sommes-nous ?*

    *Quel jour sera le 01/01/2025 ?*

    > **Remarque** : La logique de l’application est délibérément simple et présente un certain nombre de limitations. Par exemple, lorsque vous obtenez l’heure, seul un ensemble restreint de villes est pris en charge et l’heure d’été est ignorée. L’objectif est de voir un exemple de modèle classique pour l’utilisation du service de langage dans lequel votre application doit :
    >   1. Se connecter à un point de terminaison de prédiction.
    >   2. Envoyer un énoncé pour obtenir une prédiction.
    >   3. Implémenter une logique pour répondre de manière appropriée à l’intention et aux entités prédites.

1. Une fois que vous avez terminé le test, saisissez *quit*.

## Nettoyer les ressources

Si vous avez fini d’explorer le service Azure AI Language, vous pouvez supprimer les ressources que vous avez créées dans cet exercice. Voici comment procéder :

1. Ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.
2. Accédez à la ressource Azure AI Language que vous avez créée dans ce labo.
3. Dans la page de la ressource, sélectionnez **Supprimer** et suivez les instructions pour supprimer la ressource.

## Plus d’informations

Pour en savoir plus sur la compréhension du langage courant avec Azure AI Language, consultez la [documentation d’Azure AI Language](https://learn.microsoft.com/azure/ai-services/language-service/conversational-language-understanding/overview).
