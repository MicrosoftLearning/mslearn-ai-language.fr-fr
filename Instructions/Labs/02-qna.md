---
lab:
  title: Créez une solution de réponses aux questions
  description: "Utilisez Azure\_AI\_Language pour créer une solution de réponses aux questions personnalisée."
---

# Créer une solution de réponses aux questions

L’un des scénarios conversationnels les plus courants consiste à fournir une prise en charge par le biais d’un base de connaissances de questions fréquentes (FAQ). De nombreuses organisations publient les FAQ sous forme de documents ou de pages web, ce qui fonctionne bien pour un petit ensemble de paires de questions et de réponses, mais les documents volumineux peuvent être difficiles à consulter et prendre du temps.

**Azure AI Language** comprend une fonction de *réponse aux questions* qui vous permet de créer une base de connaissances de paires de questions-réponses qui peuvent être interrogées à l’aide d’une entrée en langage naturel. La plupart du temps, cette base sert de ressource à un robot qui recherche des réponses aux questions posées par les utilisateurs. Dans cet exercice, vous allez utiliser le kit de développement logiciel (SDK) Python Azure AI Language pour l’analyse de texte afin de mettre en œuvre une simple application de réponses aux questions.

Bien que cet exercice soit basé sur Python, vous pouvez développer des applications de réponses aux questions en utilisant plusieurs kits de développement logiciel (SDK) spécifiques à une langue, notamment :

- [Bibliothèque de client réponses aux questions du service Azure AI Language pour Python](https://pypi.org/project/azure-ai-language-questionanswering/)
- [Bibliothèque de client réponses aux questions du service Azure AI Language pour .NET](https://www.nuget.org/packages/Azure.AI.Language.QuestionAnswering)

Cet exercice prend environ **20** minutes.

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
1. Affichez la page **Point de terminaison et clés** dans la section **Gestion des ressources**. Plus loin dans l’exercice, vous aurez besoin des informations disponibles sur cette page.

## Créer un projet de réponses aux questions

Pour créer une base de connaissances permettant de répondre aux questions dans votre ressource Azure AI Language, vous pouvez utiliser le portail Language Studio pour créer un projet de réponse aux questions. Dans ce cas, vous allez créer une base de connaissances contenant des questions et des réponses sur [Microsoft Learn](https://learn.microsoft.com/training/).

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
    - **URL** : `https://learn.microsoft.com/en-us/training/support/faq?pivots=general`
1. Dans la page **Gérer les sources** de votre projet de réponse aux questions, dans la liste **&#9547; Ajouter une source**, sélectionnez **Chitchat**. Dans la boîte de dialogue **Ajouter une conversation de chit-chat**, sélectionnez **Convivial**, puis sélectionnez **Ajouter une conversation de chit-chat**.

## Modifier la base de connaissances

Votre base de connaissances a été rempli avec des paires de questions et réponses de la FAQ Microsoft Learn, complétée par un ensemble de paires de questions et de réponses de type *chit-chat*. Vous pouvez étendre le base de connaissances en ajoutant des paires de questions et réponses supplémentaires.

1. Dans votre projet **LearnFAQ** dans Language Studio, sélectionnez la page **Modifier base de connaissances** pour afficher les paires de questions-réponses existantes (si certains conseils sont affichés, lisez-les et sélectionnez **J’ai compris** pour les ignorer ou sélectionnez **Ignorer tout**).
1. Dans la base de connaissances, sous l’onglet **Paires question-réponse**, sélectionnez **&#65291;**, puis créez une paire question-réponse avec les paramètres suivants :
    - **Source** : `https://learn.microsoft.com/en-us/training/support/faq?pivots=general`
    - **Question** : `What are the different types of modules on Microsoft Learn?`
    - **Réponse** : `Microsoft Learn offers various types of training modules, including role-based learning paths, product-specific modules, and hands-on labs. Each module contains units with lessons and knowledge checks to help you learn at your own pace.`
1. Cliquez sur **Terminé**.
1. Dans la page de la question **Quels sont les différents types de modules disponibles sur Microsoft Learn ?** créée, développez **Questions alternatives**. Ajoutez ensuite la question alternative `How are training modules organized?`.

    Dans certains cas, il est judicieux d’autoriser l’utilisateur à rebondir sur une réponse en créant une invite *multitour* qui lui permet d’affiner de manière itérative la question pour obtenir la réponse dont il a besoin.

1. Sous la réponse que vous avez entrée pour la question sur la certification, développez **Invites de suivi**, ajoutez l’invite de suivi suivante :
    - **Texte affiché dans l’invite à l’utilisateur** : `Learn more about training`.
    - Sélectionnez l’onglet **Créer un lien vers une nouvelle paire**, puis entrez ce texte : `You can explore modules and learning paths on the [Microsoft Learn training page](https://learn.microsoft.com/training/).`
    - Sélectionnez **Afficher dans le flux contextuel uniquement**. Cette option garantit que la réponse n'est jamais renvoyée que dans le contexte d'une question complémentaire issue de la question initiale du module.
1. Sélectionnez **Ajouter une invite**.

## Entraîner et tester la base de connaissances

Maintenant que vous avez une base de connaissances, vous pouvez la tester dans le portail Language Studio.

1. Enregistrez les modifications apportées à votre base de connaissances en sélectionnant le bouton **Enregistrer** sous l’onglet **Paires question-réponse** sur la gauche.
1. Une fois les modifications enregistrées, sélectionnez le bouton **Tester** pour ouvrir le volet de test.
1. Dans le volet de test, dans la partie supérieure, décochez **Inclure une réponse courte** (si l’option n’est pas déjà décochée). Dans la partie inférieure, entrez le message `Hello`. Une réponse appropriée doit être retournée.
1. Dans la partie inférieure du volet de test, entrez le message `What is Microsoft Learn?`. Une réponse appropriée du FAQ devrait être retournée.
1. Entrez le message `Thanks!` Une réponse de conversation appropriée doit être retournée.
1. Entrez le message `What are the different types of modules on Microsoft Learn?`. La réponse que vous avez créée doit être retournée avec un lien d’invite de suivi.
1. Sélectionnez le lien de suivi **En savoir plus sur les formations**. La réponse suivante, accompagnée d'un lien vers la page de formation, doit être renvoyée.
1. Lorsque vous avez fini de tester la base de connaissances, fermez le volet de test.

## Déployer la base de connaissances

La base de connaissances fournit un service de bout en bout que les applications clientes peuvent utiliser pour répondre aux questions. Vous êtes maintenant prêt à publier votre base de connaissances et à accéder à son interface REST à partir d’un client.

1. Dans le projet **LearnFAQ** dans Language Studio, sélectionnez la page **Déployer la base de connaissances** dans le menu de navigation situé à gauche.
1. En haut de la page, sélectionnez **Déployer**. Sélectionnez ensuite **Déployer** pour confirmer que vous souhaitez déployer la base de connaissances.
1. Une fois le déploiement terminé, sélectionnez **Obtenir l’URL de prédiction** pour afficher le point de terminaison REST de votre base de connaissances et notez que l’exemple de requête inclut des paramètres pour les éléments suivants :
    - **projectName** : nom de votre projet (qui doit être *LearnFAQ*)
    - **deploymentName** : nom de votre déploiement (qui doit être *production*)
1. Fermez la boîte de dialogue de l’URL de prédiction.

## Préparer le développement d’une application dans Cloud Shell

Vous allez développer votre application de réponses aux questions à l’aide de Cloud Shell dans le portail Azure. Les fichiers de code de votre application ont été fournis dans un référentiel GitHub.

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
    cd mslearn-ai-language/Labfiles/02-qna/Python/qna-app
    ```

## Configuration de votre application

1. Dans le volet de ligne de commande, exécutez la commande suivante pour afficher les fichiers de code dans le dossier **qna-app** :

    ```
   ls -a -l
    ```

    Les fichiers incluent un fichier de configuration (**.env**) et un fichier de code (**qna-app.py**).

1. Créez un environnement virtuel Python et installez le package kit de développement logiciel (SDK) Azure AI Language de réponses aux questions ainsi que les autres packages requis en exécutant la commande suivante :

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-language-questionanswering
    ```

1. Saisissez la commande suivante pour modifier le fichier de configuration :

    ```
    code .env
    ```

    Le fichier s’ouvre dans un éditeur de code.

1. Dans le fichier de code, mettez à jour les valeurs de configuration qu’il contient pour qu’elles correspondent au **point de terminaison** et à une **clé** d’authentification de la ressource Azure Language que vous avez créée (disponibles sur la page **Clés et point de terminaison** de votre ressource Azure AI Language dans le portail Azure). Le nom du projet et du déploiement de votre base de connaissances déployée doivent également figurer dans ce fichier.
1. Une fois que vous avez remplacé les espaces réservés, utilisez la commande **CTRL+S** ou **Clic droit > Enregistrer** dans l’éditeur de code pour enregistrer vos modifications, puis utilisez la commande **CTRL+Q** ou **Clic droit > Quitter** pour fermer l’éditeur tout en gardant la ligne de commande du Cloud Shell ouverte.

## Ajouter du code à l’utilisateur de votre base de connaissances

1. Saisissez la commande suivante pour modifier le fichier de code de l’application :

    ```
    code qna-app.py
    ```

1. Passez en revue le code existant. Vous allez ajouter du code pour travailler avec votre base de connaissances.

    > **Conseil** : lorsque vous ajoutez du code au fichier de code, veillez à conserver la bonne mise en retrait.

1. Dans le fichier de code, repérez le commentaire **Importer des espaces de noms**. Puis, sous ce commentaire, ajoutez le code spécifique à une langue afin d’importer les espaces de noms nécessaires à l’utilisation du kit de développement logiciel (SDK) réponses aux questions :

    ```python
   # import namespaces
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.language.questionanswering import QuestionAnsweringClient
    ```

1. Dans la fonction **main**, notez que le code permettant de charger le point de terminaison et la clé du service Azure AI Language depuis le fichier de configuration est déjà fourni. Repérez ensuite le commentaire **Créer un client à l’aide du point de terminaison et de la clé**, et ajoutez le code suivant afin de créer un client de réponses aux questions :

    ```Python
   # Create client using endpoint and key
   credential = AzureKeyCredential(ai_key)
   ai_client = QuestionAnsweringClient(endpoint=ai_endpoint, credential=credential)
    ```

1. Dans le fichier de code, trouvez le commentaire **Envoyer une question et afficher la réponse**, puis ajoutez le code suivant afin de lire de manière répétée les questions depuis la ligne de commande, de les soumettre au service et d’afficher les détails des réponses :

    ```Python
   # Submit a question and display the answer
   user_question = ''
   while True:
        user_question = input('\nQuestion:\n')
        if user_question.lower() == "quit":                
            break
        response = ai_client.get_answers(question=user_question,
                                        project_name=ai_project_name,
                                        deployment_name=ai_deployment_name)
        for candidate in response.answers:
            print(candidate.answer)
            print("Confidence: {}".format(candidate.confidence))
            print("Source: {}".format(candidate.source))
    ```

1. Enregistrez vos modifications (CTRL+S), puis saisissez la commande suivante pour exécuter le programme (vous pouvez agrandir le volet Cloud Shell et redimensionner les volets pour voir davantage de texte dans le volet de ligne de commande) :

    ```
   python qna-app.py
    ```

1. Lorsque le système vous y invite, entrez une question à soumettre à votre projet de réponse aux questions. Par exemple : `What is a learning path?`.
1. Consultez la réponse reçue.
1. Posez d’autres questions. Lorsque vous avez terminé, entrez `quit`.

## Nettoyer les ressources

Si vous avez fini d’explorer le service Azure AI Language, vous pouvez supprimer les ressources que vous avez créées dans cet exercice. Voici comment procéder :

1. Fermez le volet Azure Cloud Shell.
1. Dans le portail Azure, accédez à la ressource Azure AI Language que vous avez créée dans cette activité.
1. Dans la page de la ressource, sélectionnez **Supprimer** et suivez les instructions pour supprimer la ressource.

## Plus d’informations

Pour en savoir plus sur la réponse aux questions avec Azure AI Language, consultez la [documentation d’Azure AI Language](https://learn.microsoft.com/azure/ai-services/language-service/question-answering/overview).
