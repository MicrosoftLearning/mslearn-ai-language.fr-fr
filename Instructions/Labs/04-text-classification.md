---
lab:
  title: Classification de texte personnalisée
  description: "Appliquez des classifications personnalisées à une entrée de texte à l’aide d’Azure\_AI\_Language."
---

# Classification de texte personnalisée

Azure AI Language fournit plusieurs fonctionnalités de traitement du langage naturel, notamment l’identification de phrases clés, le résumé de texte et l’analyse des sentiments. Le service Language fournit également des fonctionnalités personnalisées comme les réponses aux questions personnalisées et la classification de texte personnalisée.

Pour tester la classification de texte personnalisée du service Azure AI Language, vous devez configurer le modèle à l’aide de Language Studio, puis utiliser une application Python pour la tester.

Bien que cet exercice soit basé sur Python, vous pouvez développer des applications de classification de texte à l’aide de plusieurs kits SDK spécifiques à une langue, notamment :

- [Bibliothèque de client Azure AI Analyse de texte pour Python](https://pypi.org/project/azure-ai-textanalytics/)
- [Bibliothèque de client Azure AI Analyse de texte pour .NET](https://www.nuget.org/packages/Azure.AI.TextAnalytics)
- [Bibliothèque de client Azure AI Analyse de texte pour JavaScript](https://www.npmjs.com/package/@azure/ai-text-analytics)

Cet exercice prend environ **35** minutes.

## Configurer une ressource *Azure AI Language*

Si vous n’en avez pas encore dans votre abonnement, vous devez approvisionner une ressource **Service Azure AI Language**. En outre, utilisez la classification de texte personnalisée. Pour cela, vous devez activer la fonctionnalité **Classification et extraction de texte personnalisées**.

1. Ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.
1. Sélectionnez **Créer une ressource**.
1. Dans le champ de recherche, recherchez le **service de language**. Sélectionnez **Créer** sous **Service Language**.
1. Cochez la case qui inclut **Classification de texte personnalisée**. Sélectionnez ensuite **Continuer pour créer votre ressource**.
1. Créez une ressource avec les paramètres suivants :
    - **Abonnement** : *votre abonnement Azure*.
    - **Groupe de ressources** : *sélectionnez ou créez un groupe de ressources*.
    - **Région** : *choisissez parmi l’une des régions suivantes :*\*
        - Australie Est
        - Inde centrale
        - USA Est
        - USA Est 2
        - Europe Nord
        - États-Unis - partie centrale méridionale
        - Suisse Nord
        - Royaume-Uni Sud
        - Europe Ouest
        - USA Ouest 2
        - USA Ouest 3
    - **Nom** : *entrez un nom unique.*
    - **Niveau tarifaire** : sélectionnez **F0** (*gratuit*). Si cette option n’est pas disponible, sélectionnez **S** (*standard*).
    - **Compte de stockage** : nouveau compte de stockage
      - **Nom du compte de stockage** : *entrez un nom unique*.
      - **Type de compte de stockage** : Standard LRS
    - **Avis sur l’IA responsable** : sélectionné.

1. Sélectionnez **Vérifier + créer**, puis **Créer** pour provisionner la ressource.
1. Attendez la fin du déploiement, puis accédez au groupe de ressources.
1. Recherchez le compte de stockage que vous avez créé, sélectionnez-le et vérifiez que le _Type de compte_ est bien **StorageV2**. Si c’est v1, mettez à niveau le type de votre compte de stockage sur cette page de ressources.

## Configurer l’accès en fonction du rôle pour votre utilisateur

> **REMARQUE** : si vous ignorez cette étape, vous recevrez un message d’erreur 403 lorsque vous tenterez de vous connecter à votre projet personnalisé. Il est important que ce rôle soit attribué à votre utilisateur actuel pour accéder aux données blob du compte de stockage, même si vous êtes le propriétaire du compte de stockage.

1. Accédez à la page de votre compte de stockage dans le portail Azure.
2. Sélectionnez **Contrôle d’accès (IAM)** dans le menu de navigation de gauche.
3. Sélectionnez **Ajouter** pour ajouter des attributions de rôle, puis choisissez le rôle **Propriétaire de données Blob du stockage** sur le compte de stockage.
4. Dans le champ **Attribuer l’accès à**, sélectionnez **Utilisateur, groupe ou principal du service**.
5. Sélectionnez **Sélectionner des membres**.
6. Sélectionnez votre flux utilisateur. Vous pouvez rechercher des noms d’utilisateur dans le champ **Sélectionner**.

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

## Préparer le développement d’une application dans Cloud Shell

Pour tester les fonctionnalités de classification de texte personnalisées du service Azure AI Language, vous allez développer une application console simple dans Azure Cloud Shell.

1. Dans le portail Azure, cliquez sur le bouton **[\>_]** à droite de la barre de recherche, en haut de la page, pour créer un environnement Cloud Shell dans le portail Azure, puis sélectionnez un environnement ***PowerShell***. Cloud Shell fournit une interface de ligne de commande dans un volet situé en bas du portail Azure.

    > **Remarque** : si vous avez déjà créé un Cloud Shell qui utilise un environnement *Bash*, basculez-le vers ***PowerShell***.

1. Dans la barre d’outils Cloud Shell, dans le menu **Paramètres**, sélectionnez **Accéder à la version classique** (cela est nécessaire pour utiliser l’éditeur de code).

    **<font color="red">Assurez-vous d’avoir basculé vers la version classique du Cloud Shell avant de continuer.</font>**

1. Dans le volet PowerShell, entrez les commandes suivantes pour cloner le référentiel GitHub pour cet exercice :

    ```
   rm -r mslearn-ai-language -f
   git clone https://github.com/microsoftlearning/mslearn-ai-language
    ```

    > **Conseil** : lorsque vous collez des commandes dans cloudshell, la sortie peut prendre une grande quantité de mémoire tampon d’écran. Vous pouvez effacer le contenu de l’écran en saisissant la commande `cls` pour faciliter le focus sur chaque tâche.

1. Une fois le référentiel cloné, accédez au dossier contenant les fichiers de code de l’application :  

    ```
   cd mslearn-ai-language/Labfiles/04-text-classification/Python/classify-text
    ```

## Configuration de votre application

1. Dans le volet de ligne de commande, exécutez la commande suivante pour afficher les fichiers de code dans le dossier **classify-text** :

    ```
   ls -a -l
    ```

    Les fichiers incluent un fichier de configuration (**.env**) et un fichier de code (**classify-text.py**). Le texte que votre application analysera se trouve dans le sous-dossier **articles**.

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

1. Mettez à jour les valeurs de configuration pour inclure le **point de terminaison** et une **clé** à partir de la ressource de langue Azure que vous avez créée (disponible dans la page **Clés et point de terminaison** de votre ressource Azure AI Language dans le portail Azure). Le fichier doit déjà contenir les noms de projet et de déploiement de votre modèle de classification de texte.
1. Une fois que vous avez remplacé les espaces réservés, utilisez la commande **CTRL+S** ou **Clic droit > Enregistrer** dans l’éditeur de code pour enregistrer vos modifications, puis utilisez la commande **CTRL+Q** ou **Clic droit > Quitter** pour fermer l’éditeur tout en gardant la ligne de commande du Cloud Shell ouverte.

## Ajouter du code pour classifier des documents

1. Entrez la commande suivante pour modifier le fichier de code de l’application :

    ```
    code classify-text.py
    ```

1. Passez en revue le code existant. Vous allez ajouter du code pour travailler avec le kit SDK d’analyse de texte du langage à technologie IA.

    > **Conseil** : lorsque vous ajoutez du code au fichier de code, veillez à conserver la bonne mise en retrait.

1. En haut du fichier de code, sous les références d’espace de noms existantes, recherchez le commentaire **Importer les espaces de noms** et ajoutez le code suivant pour importer les espaces de noms dont vous aurez besoin pour utiliser le kit SDK d’analyse de texte :

    ```python
   # import namespaces
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.textanalytics import TextAnalyticsClient
    ```

1. Dans la fonction **Main**, notez que le code permettant de charger le point de terminaison et la clé du service Azure AI Language et les noms de projet et de déploiement à partir du fichier de configuration ont déjà été fournis. Recherchez ensuite le commentaire **Créer un client à l’aide du point de terminaison et de la clé**, puis ajoutez le code suivant pour créer un client d’analyse de texte :

    ```Python
   # Create client using endpoint and key
   credential = AzureKeyCredential(ai_key)
   ai_client = TextAnalyticsClient(endpoint=ai_endpoint, credential=credential)
    ```

1. Notez que le code existant lit tous les fichiers du dossier **articles** et crée une liste contenant leur contenu. Recherchez ensuite le commentaire **Get Classifications** (Obtenir les classifications) et ajoutez le code suivant :

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

1. Enregistrez vos modifications (CTRL+S), puis entrez la commande suivante pour exécuter le programme (vous pouvez agrandir le volet Cloud Shell et redimensionner les volets pour afficher davantage de texte dans le volet de ligne de commande) :

    ```
   python classify-text.py
    ```

1. Observez la sortie. L’application doit indiquer une classification et un score de confiance pour chaque fichier texte.

## Nettoyage

Quand vous n’avez plus besoin de votre projet, vous pouvez le supprimer de la page **Projets** dans Language Studio. Vous pouvez aussi supprimer le service Azure AI Language et le compte de stockage associé dans le [portail Azure](https://portal.azure.com).
