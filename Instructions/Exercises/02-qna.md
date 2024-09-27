---
lab:
  title: Créer une solution de réponses aux questions
  module: Module 6 - Create question answering solutions with Azure AI Language
---

# Créer une solution de réponses aux questions

L’un des scénarios conversationnels les plus courants consiste à fournir une prise en charge par le biais d’un base de connaissances de questions fréquentes (FAQ). De nombreuses organisations publient les FAQ sous forme de documents ou de pages web, ce qui fonctionne bien pour un petit ensemble de paires de questions et de réponses, mais les documents volumineux peuvent être difficiles à consulter et prendre du temps.

**Azure AI Language** comprend une fonction de *réponse aux questions* qui vous permet de créer une base de connaissances de paires de questions-réponses qui peuvent être interrogées à l’aide d’une entrée en langage naturel. La plupart du temps, cette base sert de ressource à un robot qui recherche des réponses aux questions posées par les utilisateurs.

## Configurer une ressource *Azure AI Language*

Si vous n’en avez pas encore dans votre abonnement, vous devez configurer une ressource pour le **service Azure AI Language**. En outre, pour créer et héberger une base de connaissances pour répondre à des questions, vous devez activer la fonctionnalité **Réponses aux questions**.

1. Ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.
1. Sélectionnez **Créer une ressource**.
1. Dans le champ de recherche, recherchez le **service de language**. Sélectionnez **Créer** sous **Service Language**.
1. Sélectionnez le bloc **Réponses aux questions personnalisées**. Sélectionnez ensuite **Continuer pour créer votre ressource**. Vous devez entrer les paramètres suivants :

    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *sélectionnez ou créez un groupe de ressources*.
    - **Région** : *choisissez n’importe quelle région disponible*
    - **Nom** : *Entrez un nom unique.*
    - **Niveau tarifaire** : sélectionnez **F0** (*gratuit*). Si cette option n’est pas disponible, sélectionnez **S** (*standard*).
    - **Région de la Recherche Azure** : *choisissez un emplacement dans la région du monde qui correspond à votre ressource Language*.
    - **Niveau tarifaire Recherche Azure** : Gratuit (F) (*Si ce niveau n'est pas disponible, sélectionnez Basique (B)* )
    - **Mention sur l’IA responsable** : *J’accepte*

1. Sélectionnez **Créer + vérifier**, puis sélectionnez **Créer**.

    > **Remarque :** les réponses aux questions personnalisées utilisent la Recherche Azure pour indexer et interroger la base de connaissances des questions et réponses.

1. Attendez la fin du déploiement, puis accédez à la ressource déployée.
1. Consultez la page **Clés et points de terminaison**. Plus loin dans l’exercice, vous aurez besoin des informations disponibles sur cette page.

## Créer un projet de réponses aux questions

Pour créer une base de connaissances permettant de répondre aux questions dans votre ressource Azure AI Language, vous pouvez utiliser le portail Language Studio pour créer un projet de réponse aux questions. Dans ce cas, vous allez créer une base de connaissances contenant des questions et des réponses sur [Microsoft Learn](https://docs.microsoft.com/learn).

1. Dans un nouvel onglet de navigateur, accédez au portail Language Studio à l’adresse [https://language.cognitive.azure.com/](https://language.cognitive.azure.com/), puis connectez-vous en utilisant le compte Microsoft associé à votre abonnement Azure.
1. Si le système vous invite à choisir une ressource Language, sélectionnez les paramètres suivants :
    - **Azure Directory** : Annuaire Azure contenant votre abonnement.
    - **Abonnement Azure** : Votre abonnement Azure.
    - **Type de ressource** : Language.
    - **Nom de la ressource** : nom de la ressource Azure AI Language que vous avez créée précédemment.

    Si vous n’êtes <u>pas</u> invité à choisir une ressource de langue, c’est peut-être parce que vous avez plusieurs ressources de langue dans votre abonnement, auquel cas :

    1. Dans la barre en haut de la page, sélectionnez le bouton **Paramètres (&#9881;)**.
    2. Dans la page **Paramètres**, affichez l’onglet **Ressources**.
    3. Sélectionnez la ressource de langue que vous venez de créer, puis cliquez sur **Changer de ressource**.
    4. En haut de la page, cliquez sur **Language Studio** pour revenir à la page d’accueil de Language Studio.

1. En haut du portail, dans le menu **Créer**, sélectionnez **Réponses aux questions personnalisées**.
1. Dans l’assistant ***Créer un projet**, sur la page **Choisir un paramètre de langue**, sélectionnez l’option **Sélectionner la langue de tous les projets de cette ressource**, puis sélectionnez **Anglais** comme langue. Sélectionnez ensuite **Suivant**.
1. Sur la page **Entrer les informations de base**, entrez les informations suivantes :
    - **Nom** `LearnFAQ`
    - **Description** : `FAQ for Microsoft Learn`
    - **Réponse par défaut quand aucune réponse n’est retournée** : `Sorry, I don't understand the question`
1. Cliquez sur **Suivant**.
1. Sur la page **Vérifier et terminer**, sélectionnez **Créer un projet**.

## Ajouter des sources à la base de connaissances

Vous pouvez créer une base de connaissances à partir de zéro, mais il est courant de commencer par importer les questions et les réponses d'une page ou d'un document de FAQ existant. Dans ce cas, vous allez importer des données à partir d’une page web de FAQ existante pour Microsoft Learn, et vous importerez également certaines questions et réponses prédéfinies de type « chit-chat » pour prendre en charge les échanges conversationnels courants.

1. Dans la page **Gérer les sources** de votre projet de réponse aux questions, dans la liste **&#9547; Ajouter une source**, sélectionnez **URL**. Ensuite, dans la boîte de dialogue **Ajouter des URL**, cliquez sur **&#9547; Ajouter une URL** et définissez le nom et l’URL suivants, puis sélectionnez **Ajouter tout** pour l’ajouter à la base de connaissances :
    - **Nom :** `Learn FAQ Page`
    - **URL** : `https://docs.microsoft.com/en-us/learn/support/faq`
1. Dans la page **Gérer les sources** de votre projet de réponse aux questions, dans la liste **&#9547; Ajouter une source**, sélectionnez **Chitchat**. Dans la boîte de dialogue **Ajouter une conversation de chit-chat**, sélectionnez **Convivial**, puis sélectionnez **Ajouter une conversation de chit-chat**.

## Modifier la base de connaissances

Votre base de connaissances a été rempli avec des paires de questions et réponses de la FAQ Microsoft Learn, complétée par un ensemble de paires de questions et de réponses de type *chit-chat*. Vous pouvez étendre le base de connaissances en ajoutant des paires de questions et réponses supplémentaires.

1. Dans votre projet **LearnFAQ** dans Language Studio, sélectionnez la page **Modifier base de connaissances** pour afficher les paires de questions-réponses existantes (si certains conseils sont affichés, lisez-les et sélectionnez **J’ai compris** pour les ignorer ou sélectionnez **Ignorer tout**).
1. Dans la base de connaissances, sous l’onglet **Paires question-réponse**, sélectionnez **&#65291;**, puis créez une paire question-réponse avec les paramètres suivants :
    - **Source** : `https://docs.microsoft.com/en-us/learn/support/faq`
    - **Question** : `What are Microsoft credentials?`
    - **Réponse** : `Microsoft credentials enable you to validate and prove your skills with Microsoft technologies.`
1. Cliquez sur **Terminé**.
1. Dans la page de la question **What are Microsoft credentials?** (Que sont les titres de compétences Microsoft ?) créée, développez **Questions alternatives**. Ajoutez ensuite la question alternative `How can I demonstrate my Microsoft technology skills?`.

    Dans certains cas, il est judicieux d’autoriser l’utilisateur à rebondir sur une réponse en créant une invite *multitour* qui lui permet d’affiner de manière itérative la question pour obtenir la réponse dont il a besoin.

1. Sous la réponse que vous avez entrée pour la question sur la certification, développez **Invites de suivi**, ajoutez l’invite de suivi suivante :
    - **Texte affiché dans l’invite à l’utilisateur** : `Learn more about credentials`.
    - Sélectionnez l’onglet **Créer un lien vers une nouvelle paire**, puis entrez ce texte : `You can learn more about credentials on the [Microsoft credentials page](https://docs.microsoft.com/learn/credentials/).`
    - Sélectionnez **Afficher dans le flux contextuel uniquement**. Cette option garantit que la réponse n’est jamais retournée dans le contexte d’une question de suivi de la question de certification d’origine.
1. Sélectionnez **Ajouter une invite**.

## Entraîner et tester la base de connaissances

Maintenant que vous avez une base de connaissances, vous pouvez la tester dans le portail Language Studio.

1. Enregistrez les modifications apportées à votre base de connaissances en sélectionnant le bouton **Enregistrer** sous l’onglet **Paires question-réponse** sur la gauche.
1. Une fois les modifications enregistrées, sélectionnez le bouton **Tester** pour ouvrir le volet de test.
1. Dans le volet de test, dans la partie supérieure, décochez **Inclure une réponse courte** (si l’option n’est pas déjà décochée). Dans la partie inférieure, entrez le message `Hello`. Une réponse appropriée doit être retournée.
1. Dans la partie inférieure du volet de test, entrez le message `What is Microsoft Learn?`. Une réponse appropriée du FAQ devrait être retournée.
1. Entrez le message `Thanks!` Une réponse de conversation appropriée doit être retournée.
1. Entrez le message `Tell me about Microsoft credentials`. La réponse que vous avez créée doit être retournée avec un lien d’invite de suivi.
1. Sélectionnez le lien de suivi **En savoir plus sur les certifications**. La réponse de suivi avec un lien vers la page de certification doit être retournée.
1. Lorsque vous avez fini de tester la base de connaissances, fermez le volet de test.

## Déployer la base de connaissances

La base de connaissances fournit un service de bout en bout que les applications clientes peuvent utiliser pour répondre aux questions. Vous êtes maintenant prêt à publier votre base de connaissances et à accéder à son interface REST à partir d’un client.

1. Dans le projet **LearnFAQ** dans Language Studio, sélectionnez la page **Déployer la base de connaissances** dans le menu de navigation situé à gauche.
1. En haut de la page, sélectionnez **Déployer**. Sélectionnez ensuite **Déployer** pour confirmer que vous souhaitez déployer la base de connaissances.
1. Une fois le déploiement terminé, sélectionnez **Obtenir l’URL de prédiction** pour afficher le point de terminaison REST de votre base de connaissances et notez que l’exemple de requête inclut des paramètres pour les éléments suivants :
    - **projectName** : nom de votre projet (qui doit être *LearnFAQ*)
    - **deploymentName** : nom de votre déploiement (qui doit être *production*)
1. Fermez la boîte de dialogue de l’URL de prédiction.

## Préparer le développement d’une application dans Visual Studio Code

Vous allez développer votre application de réponse à des questions à l’aide de Visual Studio Code. Les fichiers de code de votre application ont été fournis dans un référentiel GitHub.

> **Conseil** : si vous avez déjà cloné le référentiel **mslearn-ai-language**, ouvrez-le dans Visual Studio Code. Dans le cas contraire, procédez comme suit pour le cloner dans votre environnement de développement.

1. Démarrez Visual Studio Code.
2. Ouvrez la palette (Maj+CTRL+P) et exécutez une commande **Git : Cloner** pour cloner le référentiel `https://github.com/MicrosoftLearning/mslearn-ai-language` vers un dossier local (peu importe quel dossier).
3. Lorsque le référentiel a été cloné, ouvrez le dossier dans Visual Studio Code.

    > **Remarque** : Si Visual Studio Code affiche un message contextuel qui vous invite à approuver le code que vous ouvrez, cliquez sur l’option **Oui, je fais confiance aux auteurs** dans la fenêtre contextuelle.

4. Attendez que des fichiers supplémentaires soient installés pour prendre en charge les projets de code C# dans le référentiel.

    > **Remarque** : si vous êtes invité à ajouter des ressources requises pour générer et déboguer, sélectionnez **Not Now** (Pas maintenant).

## Configuration de votre application

Des applications pour C# et Python sont fournies, ainsi qu’un exemple de fichier texte que vous utiliserez pour tester le résumé. Les deux applications présentent les mêmes fonctionnalités. Premièrement, vous allez terminer certaines parties clés de l’application pour lui permettre d’utiliser votre ressource Azure AI Language.

1. Dans Visual Studio Code, dans le volet **Explorateur**, accédez au dossier **Labfiles/02-qna** et développez le dossier **CSharp** ou **Python** en fonction de votre préférence de langage, ainsi que le dossier **qna-app** qu’il contient. Chaque dossier contient les fichiers propres au langage d’une application dans laquelle vous allez intégrer des fonctionnalités de réponse aux questions d’Azure AI Language.
2. Cliquez avec le bouton droit sur le dossier **qna-app**, qui contient vos fichiers de code, et ouvrez un terminal intégré. Installez ensuite le package du kit de développement logiciel (SDK) de réponse aux questions d’Azure AI Language en exécutant la commande appropriée en fonction de votre préférence de langage :

    **C# :**

    ```
    dotnet add package Azure.AI.Language.QuestionAnswering
    ```

    **Python** :

    ```
    pip install azure-ai-language-questionanswering
    ```

3. Dans le volet **Explorateur**, dans le dossier **qna-app**, ouvrez le fichier de configuration correspondant à votre langage préféré.

    - **C#** : appsettings.json
    - **Python** : .env
    
4. Mettez à jour les valeurs de configuration de sorte à inclure un **point de terminaison** et une **clé** de la ressource Azure Language que vous avez créée (disponible sur la page **Clés et point de terminaison** de votre ressource Azure AI Language dans le Portail Azure). Le nom du projet et du déploiement de votre base de connaissances déployée doivent également figurer dans ce fichier.
5. Enregistrez le fichier de configuration.

## Ajouter du code à l’application

Vous pouvez maintenant ajouter le code nécessaire pour importer les bibliothèques de SDK requises, établir une connexion authentifiée à votre projet déployé et poser des questions.

1. Notez que le dossier **qna-app** contient un fichier de code pour l’application cliente :

    - **C#** : Program.cs
    - **Python** : qna-app.py

    Ouvrez le fichier de code et, en haut, sous les références d’espace de noms existantes, recherchez le commentaire **Importer des espaces de noms**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour importer les espaces de noms dont vous aurez besoin pour utiliser le kit de développement logiciel (SDK) d’analyse de texte :

    **C#**  : Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.Language.QuestionAnswering;
    ```

    **Python** : qna-app.py

    ```python
    # import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.language.questionanswering import QuestionAnsweringClient
    ```

1. Dans la fonction **Main**, notez que le code permettant de charger le point de terminaison et la clé du service Azure AI Language à partir du fichier de configuration a déjà été fourni. Recherchez ensuite le commentaire **Créer un client à l’aide du point de terminaison et de la clé**, puis ajoutez le code suivant pour créer un client pour l’API Analyse de texte :

    **C#**  : Programs.cs

    ```C#
    // Create client using endpoint and key
    AzureKeyCredential credentials = new AzureKeyCredential(aiSvcKey);
    Uri endpoint = new Uri(aiSvcEndpoint);
    QuestionAnsweringClient aiClient = new QuestionAnsweringClient(endpoint, credentials);
    ```

    **Python** : qna-app.py

    ```Python
    # Create client using endpoint and key
    credential = AzureKeyCredential(ai_key)
    ai_client = QuestionAnsweringClient(endpoint=ai_endpoint, credential=credential)
    ```

1. Dans la fonction **Main**, recherchez le commentaire **Submit a question and display the answer** (Poser une question et afficher la réponse), puis ajoutez le code suivant pour lire les questions à partir de la ligne de commande, les envoyer au service et afficher les détails des réponses :

    **C#**  : Programs.cs

    ```C#
    // Submit a question and display the answer
    string user_question = "";
    while (user_question.ToLower() != "quit")
        {
            Console.Write("Question: ");
            user_question = Console.ReadLine();
            QuestionAnsweringProject project = new QuestionAnsweringProject(projectName, deploymentName);
            Response<AnswersResult> response = aiClient.GetAnswers(user_question, project);
            foreach (KnowledgeBaseAnswer answer in response.Value.Answers)
            {
                Console.WriteLine(answer.Answer);
                Console.WriteLine($"Confidence: {answer.Confidence:P2}");
                Console.WriteLine($"Source: {answer.Source}");
                Console.WriteLine();
            }
        }
    ```

    **Python** : qna-app.py

    ```Python
    # Submit a question and display the answer
    user_question = ''
    while user_question.lower() != 'quit':
        user_question = input('\nQuestion:\n')
        response = ai_client.get_answers(question=user_question,
                                        project_name=ai_project_name,
                                        deployment_name=ai_deployment_name)
        for candidate in response.answers:
            print(candidate.answer)
            print("Confidence: {}".format(candidate.confidence))
            print("Source: {}".format(candidate.source))
    ```

1. Enregistrez vos modifications et revenez au terminal intégré du dossier **qna-app**, puis entrez la commande suivante pour exécuter le programme :

    - **C#**  : `dotnet run`
    - **Python** : `python qna-app.py`

    > **Conseil** : vous pouvez utiliser l’icône **Agrandir le volet** (**^**) dans la barre d’outils du terminal pour mieux voir le texte de la console.

1. Lorsque le système vous y invite, entrez une question à soumettre à votre projet de réponse aux questions. Par exemple : `What is a learning path?`.
1. Consultez la réponse reçue.
1. Posez d’autres questions. Lorsque vous avez terminé, entrez `quit`.

## Nettoyer les ressources

Si vous avez fini d’explorer le service Azure AI Language, vous pouvez supprimer les ressources que vous avez créées dans cet exercice. Voici comment procéder :

1. Ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.
2. Accédez à la ressource Azure AI Language que vous avez créée dans ce labo.
3. Dans la page de la ressource, sélectionnez **Supprimer** et suivez les instructions pour supprimer la ressource.

## Plus d’informations

Pour en savoir plus sur la réponse aux questions avec Azure AI Language, consultez la [documentation d’Azure AI Language](https://learn.microsoft.com/azure/ai-services/language-service/question-answering/overview).
