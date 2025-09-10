---
lab:
  title: Extraire des entités personnalisées
  description: "Entraîner un modèle pour extraire des entités personnalisées à partir d’une entrée de texte à l’aide d’Azure\_AI\_Language."
---

# Extraire des entités personnalisées

En plus d’autres fonctionnalités de traitement du langage naturel, le service Azure AI Language vous permet de définir des entités personnalisées et d’extraire leurs instances à partir de texte.

Pour tester l’extraction d’entités personnalisées, nous allons créer un modèle et l’entraîner via Azure AI Language Studio, puis utiliser une application Python pour la tester.

Bien que cet exercice soit basé sur Python, vous pouvez développer des applications de classification de texte à l’aide de plusieurs kits SDK spécifiques à une langue, notamment :

- [Bibliothèque de client Azure AI Analyse de texte pour Python](https://pypi.org/project/azure-ai-textanalytics/)
- [Bibliothèque de client Azure AI Analyse de texte pour .NET](https://www.nuget.org/packages/Azure.AI.TextAnalytics)
- [Bibliothèque de client Azure AI Analyse de texte pour JavaScript](https://www.npmjs.com/package/@azure/ai-text-analytics)

Cet exercice prend environ **35** minutes.

## Configurer une ressource *Azure AI Language*

Si vous n’en avez pas encore dans votre abonnement, vous devez approvisionner une ressource **Service Azure AI Language**. En outre, utilisez la classification de texte personnalisée. Pour cela, vous devez activer la fonctionnalité **Classification et extraction de texte personnalisées**.

1. Dans un navigateur, ouvrez le portail Azure à l’adresse `https://portal.azure.com`, puis connectez-vous avec votre compte Microsoft.
1. Sélectionnez le bouton **Créer une ressource**, recherchez *Langage*, puis créez une ressource **Service de langage**. Dans la page pour *Sélectionner des fonctionnalités supplémentaires*, sélectionnez la fonctionnalité personnalisée contenant l’**extraction de la reconnaissance d’entités nommées personnalisées**. Créez la ressource avec les paramètres suivants :
    - **Abonnement** : *votre abonnement Azure*
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
    - **Nom** : *Entrez un nom unique.*
    - **Niveau tarifaire** : sélectionnez **F0** (*gratuit*) ou **S** (*standard*) si F n’est pas disponible.
    - **Compte de stockage** : nouveau compte de stockage :
      - **Nom du compte de stockage** : *entrez un nom unique*.
      - **Type de compte de stockage** : Standard LRS
    - **Avis sur l’IA responsable** : sélectionné.

1. Sélectionnez **Vérifier + créer**, puis **Créer** pour provisionner la ressource.
1. Attendez la fin du déploiement, puis accédez à la ressource déployée.
1. Consultez la page **Clés et points de terminaison**. Plus loin dans l’exercice, vous aurez besoin des informations disponibles sur cette page.

## Configurer l’accès en fonction du rôle pour votre utilisateur

> **REMARQUE** : si vous ignorez cette étape, vous recevrez une erreur 403 lorsque vous tenterez de vous connecter à votre projet personnalisé. Il est important que ce rôle soit attribué à votre utilisateur actuel pour accéder aux données blob du compte de stockage, même si vous êtes le propriétaire du compte de stockage.

1. Accédez à la page de votre compte de stockage dans le portail Azure.
2. Sélectionnez **Contrôle d’accès (IAM)** dans le menu de navigation de gauche.
3. Sélectionnez **Ajouter** pour Ajouter des attributions de rôle, puis choisissez le rôle **Contributeur aux données Blob du stockage** sur le compte de stockage.
4. Dans le champ **Attribuer l’accès à**, sélectionnez **Utilisateur, groupe ou principal du service**.
5. Sélectionnez **Sélectionner des membres**.
6. Sélectionnez votre flux utilisateur. Vous pouvez rechercher des noms d’utilisateur dans le champ **Sélectionner**.

## Charger des exemples d’annonces

Une fois que vous avez créé le service Azure AI Language et le compte de stockage, vous devez charger des exemples d’annonces pour entraîner votre modèle par la suite.

1. Dans un nouvel onglet de navigateur, téléchargez des exemples d’annonces classifiées à partir de `https://aka.ms/entity-extraction-ads` et extrayez les fichiers dans un dossier de votre choix.

2. Dans le Portail Azure, accédez au compte de stockage que vous avez créé et sélectionnez-le.

3. Dans votre compte de stockage, sous **Paramètres**, sélectionnez **Configuration**, activez l’option **Autoriser l’accès anonyme aux objets blob**, puis sélectionnez **Enregistrer**.

4. Dans le menu de gauche, sous **Stockage de données**, sélectionnez **Conteneurs**. Dans l’écran qui s’affiche, sélectionnez **+ Conteneur**. Donnez au conteneur le nom `classifieds`, puis définissez le **niveau d’accès anonyme** sur **Conteneur (accès en lecture anonyme pour les conteneurs et les objets blob)**.

    > **REMARQUE** : lorsque vous configurez un compte de stockage pour une solution réelle, veillez à attribuer le niveau d’accès approprié. Pour en savoir plus sur chaque niveau d’accès, consultez la [documentation sur Stockage Azure](https://learn.microsoft.com/azure/storage/blobs/anonymous-read-access-configure).

5. Après avoir créé le conteneur, sélectionnez-le, puis cliquez sur le bouton **Charger** pour charger les exemples d’annonces que vous avez téléchargés.

## Créer un projet de reconnaissance d’entité nommée personnalisée

Vous êtes à présent prêt à créer un projet de reconnaissance d’entités nommées personnalisées. Ce projet fournit un lieu de travail pour générer, entraîner et déployer votre modèle.

> **REMARQUE** : vous pouvez aussi créer, générer, entraîner et déployer votre modèle via l’API REST.

1. Dans un nouvel onglet de navigateur, ouvrez le portail Azure AI Language Studio à l’adresse `https://language.cognitive.azure.com/`, puis connectez-vous avec le compte Microsoft associé à votre abonnement Azure.
1. Si vous êtes invité à choisir une ressource de langue, sélectionnez les paramètres suivants :

    - **Azure Directory** : Annuaire Azure contenant votre abonnement.
    - **Abonnement Azure** : Votre abonnement Azure.
    - **Type de ressource** : Language.
    - **Ressource Language** : nom de la ressource Azure AI Language que vous avez créée précédemment.

    Si vous n’êtes <u>pas</u> invité à choisir une ressource de langue, c’est peut-être parce que vous avez plusieurs ressources de langue dans votre abonnement, auquel cas :

    1. Dans la barre supérieure de la page, sélectionnez le bouton **Paramètres (&#9881;)**.
    2. Dans la page **Paramètres**, affichez l’onglet **Ressources**.
    3. Sélectionnez la ressource de langue que vous venez de créer, puis cliquez sur **Changer de ressource**.
    4. En haut de la page, cliquez sur **Language Studio** pour revenir à la page d’accueil de Language Studio.

1. En haut du portail, dans le menu **Créer**, sélectionnez **Reconnaissance d’entités nommées personnalisées**.

1. Créez un nouveau projet avec les paramètres suivants :
    - **Connecter un stockage** : *cette valeur est probablement déjà remplie. Remplacez la ressource par votre compte de stockage si ce n’est pas le cas*
    - **Informations de base :**
    - **Nom :** `CustomEntityLab`
        - **Langue principale du texte** : anglais (US)
        - **Votre jeu de données inclut-il des documents qui ne sont pas dans la même langue ?** : *Non*
        - **Description** : `Custom entities in classified ads`
    - **Conteneur** :
        - **Conteneur de magasin d’objets blob** : petites annonces
        - **Vos fichiers sont-ils étiquetés avec des classes ?**  : Non, je dois étiqueter mes fichiers dans le cadre de ce projet

> **Conseil** : si vous obtenez un message d’erreur indiquant que vous n’êtes pas autorisé à effectuer cette opération, vous devez ajouter une attribution de rôle. Pour résoudre ce problème, le rôle « Contributeur aux données Blob du stockage » est ajouté au compte de stockage de l’utilisateur qui exécute le labo. Pour davantage de précisions, référez-vous à la [page de documentation.](https://learn.microsoft.com/azure/ai-services/language-service/custom-named-entity-recognition/how-to/create-project?tabs=portal%2Clanguage-studio#enable-identity-management-for-your-resource).

## Étiqueter vos données

Maintenant que votre projet est créé, vous devez étiqueter vos données pour entraîner votre modèle à identifier des entités.

1. Si la page **Étiquetage des données** n’est pas déjà ouverte, dans le volet de gauche, sélectionnez **Étiquetage des données**. Vous voyez la liste des fichiers que vous avez chargés sur votre compte de stockage.
1. Sur le côté droit, dans le volet **Activité**, sélectionnez **Ajouter une entité**, puis ajoutez une entité nommée `ItemForSale`.
1.  Répétez l’étape précédente pour créer les entités suivantes :
    - `Price`
    - `Location`
1. Une fois que vous avez créé vos trois entités, sélectionnez **Ad 1.txt** pour pouvoir lire ce fichier.
1. Dans *Ad 1.txt* : 
    1. Mettez en surbrillance le texte *face cord of firewood*, puis sélectionnez l’entité **ItemForSale**.
    1. Mettez en surbrillance le texte *Denver, CO*, puis sélectionnez l’entité **Location**.
    1. Mettez en surbrillance le texte *$90*, puis sélectionnez l’entité **Price**.
1. Dans le volet **Activité**, notez que ce document sera ajouté au jeu de données pour l’apprentissage du modèle.
1. Utilisez le bouton **Document suivant** pour passer au document suivant et continuer à attribuer du texte aux entités appropriées pour l’ensemble des documents, en les ajoutant tous au jeu de données de formation.
1. Lorsque vous avez étiqueté le dernier document (*Ad 9.txt*), enregistrez les étiquettes.

## Entraîner votre modèle

Après avoir étiqueté vos données, vous devez entraîner votre modèle.

1. Dans le volet à gauche, sélectionnez **Travaux d’entraînement**.
2. Sélectionnez **Démarrer un travail d’entraînement**
3. Entraînez un nouveau modèle nommé `ExtractAds`
4. Choisissez **Séparer automatiquement le jeu de test des données d’entraînement**

    > **CONSEIL** : dans vos propres projets d’extraction, utilisez la division pour les tests qui convient le mieux à vos données. Pour des données plus cohérentes et des jeux de données plus grands, le service Azure AI Language va fractionner automatiquement le jeu de test selon un pourcentage. Avec des jeux de données plus petits, il est important d’effectuer l’entraînement avec la diversité pertinente des documents d’entrée possibles.

5. Cliquez sur **Entraîner**

    > **IMPORTANT** : l’entraînement de votre modèle peut parfois prendre plusieurs minutes. Vous recevez une notification une fois l’opération terminée.

## Évaluer votre modèle

Dans les applications réelles, il est important d’évaluer votre modèle et de l’améliorer pour vérifier qu’il fonctionne comme attendu. Deux pages à gauche vous montrent les détails de votre modèle entraîné et les tests qui ont échoué.

Sélectionnez **Performances du modèle** dans le menu de gauche, puis sélectionnez votre modèle `ExtractAds`. Ici, vous pouvez voir le scoring de votre modèle, ses métriques de performances et quand il a été entraîné. Vous serez en mesure de voir si des documents de test ont échoué, et ces échecs vous aideront à comprendre où le modèle doit être amélioré.

## Déployer votre modèle

Quand vous êtes satisfait de l’entraînement de votre modèle, vous pouvez le déployer, ce qui vous permet de commencer à extraire des entités via l’API.

1. Dans le volet de gauche, sélectionnez **Déploiement d’un modèle**.
2. Sélectionnez **Ajouter un déploiement**, entrez le nom `AdEntities`, puis sélectionnez le modèle **ExtractAds**.
3. Cliquez sur **Déployer** pour déployer votre modèle.

## Préparer le développement d’une application dans Cloud Shell

Pour tester les fonctionnalités d’extraction des entités personnalisées du service Azure AI Language, vous allez développer une application console simple dans Azure Cloud Shell.

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
    ```

1. After the repo has been cloned, navigate to the folder containing the application code files:  

    ```
    cd mslearn-ai-language/Labfiles/05-custom-entity-recognition/Python/custom-entities
    ```

## Configure your application

1. In the command line pane, run the following command to view the code files in the **custom-entities** folder:

    ```
   ls -a -l
    ```

    The files include a configuration file (**.env**) and a code file (**custom-entities.py**). The text your application will analyze is in the **ads** subfolder.

1. Create a Python virtual environment and install the Azure AI Language Text Analytics SDK package and other required packages by running the following command:

    ```
   python -m venv labenv ./labenv/bin/Activate.ps1 pip install -r requirements.txt azure-ai-textanalytics==5.3.0
    ```

1. Enter the following command to edit the application configuration file:

    ```
   code .env
    ```

    The file is opened in a code editor.

1. Update the configuration values to include the  **endpoint** and a **key** from the Azure Language resource you created (available on the **Keys and Endpoint** page for your Azure AI Language resource in the Azure portal).The file should already contain the project and deployment names for your custom entity extraction model.
1. After you've replaced the placeholders, within the code editor, use the **CTRL+S** command or **Right-click > Save** to save your changes and then use the **CTRL+Q** command or **Right-click > Quit** to close the code editor while keeping the cloud shell command line open.

## Add code to extract entities

1. Enter the following command to edit the application code file:

    ```
    code custom-entities.py
    ```

1. Review the existing code. You will add code to work with the AI Language Text Analytics SDK.

    > **Tip**: As you add code to the code file, be sure to maintain the correct indentation.

1. At the top of the code file, under the existing namespace references, find the comment **Import namespaces** and add the following code to import the namespaces you will need to use the Text Analytics SDK:

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

1. Notez que le code existant lit tous les fichiers du dossier **ads** et crée une liste contenant leur contenu. Recherchez ensuite le commentaire **Extraire des entités**, puis ajoutez le code suivant :

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

1. Enregistrez vos modifications (CTRL+S), puis entrez la commande suivante pour exécuter le programme (vous pouvez agrandir le volet Cloud Shell et redimensionner les volets pour afficher davantage de texte dans le volet de ligne de commande) :

    ```
   python custom-entities.py
    ```

1. Observez la sortie. L’application doit afficher les détails des entités trouvées dans chaque fichier texte.

## Nettoyage

Quand vous n’avez plus besoin de votre projet, vous pouvez le supprimer de la page **Projets** dans Language Studio. Vous pouvez aussi supprimer le service Azure AI Language et le compte de stockage associé dans le [portail Azure](https://portal.azure.com).
