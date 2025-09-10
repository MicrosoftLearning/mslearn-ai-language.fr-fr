---
lab:
  title: Développer une application de conversation avec fonction audio.
  description: "Apprenez à utiliser Azure\_AI\_Foundry pour créer une application d’IA générative prenant en charge l’entrée audio."
---

# Développer une application de conversation avec fonction audio.

Dans cet exercice, vous utiliserez le modèle génératif d’IA *Phi-4-multimodal-instruct* pour générer des réponses à des invites comprenant des fichiers audio. Vous allez développer une application qui fournit une assistance IA pour une entreprise de grossiste en produits frais en utilisant Azure AI Foundry et le kit de développement logiciel (SDK) Python OpenAI pour résumer les messages vocaux laissés par les clients.

Bien que cet exercice soit basé sur Python, vous pouvez développer des applications similaires à l'aide de plusieurs SDK spécifiques à un langage, notamment :

- [Projets Azure AI pour Python](https://pypi.org/project/azure-ai-projects)
- [Bibliothèque OpenAI pour Python](https://pypi.org/project/openai/)
- [Projets Azure AI pour Microsoft .NET](https://www.nuget.org/packages/Azure.AI.Projects)
- [Bibliothèque de client Azure OpenAI pour Microsoft .NET](https://www.nuget.org/packages/Azure.AI.OpenAI)
- [Projets Azure AI pour JavaScript](https://www.npmjs.com/package/@azure/ai-projects)
- [Bibliothèque Azure OpenAI pour TypeScript](https://www.npmjs.com/package/@azure/openai)

Cet exercice prend environ **30** minutes.

## Créer un projet Azure AI Foundry

Commençons par déployer un projet Azure AI Foundry.

1. Dans un navigateur web, ouvrez le [portail Azure AI Foundry](https://ai.azure.com) à l’adresse `https://ai.azure.com` et connectez-vous en utilisant vos informations d’identification Azure. Fermez les conseils ou les volets de démarrage rapide ouverts la première fois que vous vous connectez et, si nécessaire, utilisez le logo **Azure AI Foundry** en haut à gauche pour accéder à la page d’accueil, qui ressemble à l’image suivante :

    ![Capture d’écran du portail Azure AI Foundry.](../media/ai-foundry-home.png)

1. Dans la page d’accueil, dans la section **Explorer les modèles et les fonctionnalités**, recherchez le modèle `Phi-4-multimodal-instruct`, que nous utiliserons dans notre projet.
1. Dans les résultats de recherche, sélectionnez le modèle **Phi-4-multimodal-instruct** pour consulter ses détails, puis, en haut de la page du modèle, sélectionnez **Utiliser ce modèle**.
1. Lorsque vous êtes invité à créer un projet, entrez un nom valide pour votre projet et développez les **options avancées**.
1. Sélectionnez **Personnaliser** et spécifiez les paramètres suivants pour votre hub :
    - **Ressource Azure AI Foundry** : *un nom valide pour votre ressource Azure AI Foundry.*
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *créez ou sélectionnez un groupe de ressources*
    - **Région** : *Sélectionnez n’importe quelle **recommandation d’AI Foundry***\*

    > \* Certaines ressources Azure AI sont limitées par des quotas de modèles régionaux. Si une limite de quota est atteinte plus tard dans l’exercice, vous devrez peut-être créer une autre ressource dans une autre région. Vous pouvez consulter la disponibilité régionale la plus récente pour des modèles spécifiques dans la [documentation Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/deploy-models-serverless-availability#region-availability).

1. Sélectionnez **Créer** et attendez que votre projet soit créé.

    L’opération peut prendre quelques instants pour s’achever.

1. Sélectionnez **Accepter et continuer** pour accepter les conditions du modèle, puis sélectionnez **Déployer** pour finaliser le modèle de déploiement Phi.

1. Lorsque votre projet est créé, les détails du modèle s’ouvrent automatiquement. Notez le nom de votre modèle de déploiement ; il doit être **Phi-4-multimodal-instruct**.

1. Dans le volet de navigation à gauche, sélectionnez **Vue d’ensemble** pour accéder à la page principale de votre projet ; elle se présente comme suit :

    > **Remarque** : si une erreur *Autorisations insuffisantes** s’affiche, utilisez le bouton **Corriger** pour la résoudre.

    ![Capture d’écran d’une page de présentation d’un projet Azure AI Foundry.](../media/ai-foundry-project.png)

## Créer une application cliente

Votre modèle étant déployé, vous pouvez à présent utiliser Azure AI Foundry et les kits de développement logiciel (SDK) d’inférence de modèle Azure AI pour développer une application permettant de converser avec lui.

> **Conseil** : vous pouvez choisir de développer votre solution à l’aide de Python ou de Microsoft C#. Suivez les instructions de la section appropriée pour votre langue choisie.

### Préparer la configuration de l’application

1. Dans le portail Azure AI Foundry, affichez la page **Vue d’ensemble** de votre projet.
1. Dans la zone **Détails du projet**, notez le **point de terminaison du projet Azure AI Foundry**. Vous utiliserez ce point de terminaison pour connecter votre projet à une application cliente.
1. Ouvrez un nouvel onglet de navigateur (en gardant le portail Azure AI Foundry ouvert dans l’onglet existant). Dans un nouvel onglet du navigateur, ouvrez le [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com` et connectez-vous en utilisant vos informations d’identification Azure.

    Fermez les notifications de bienvenue pour afficher la page d’accueil du portail Azure.

1. Utilisez le bouton **[\>_]** à droite de la barre de recherche, en haut de la page, pour créer un environnement Cloud Shell dans le portail Azure, puis sélectionnez un environnement ***PowerShell*** avec aucun stockage dans votre abonnement.

    Cloud Shell fournit une interface de ligne de commande via un volet situé en bas du portail Azure. Vous pouvez redimensionner ou agrandir ce volet pour faciliter le travail.

    > **Remarque** : si vous avez déjà créé un Cloud Shell qui utilise un environnement *Bash*, basculez-le vers ***PowerShell***.

1. Dans la barre d’outils Cloud Shell, dans le menu **Paramètres**, sélectionnez **Accéder à la version classique** (cela est nécessaire pour utiliser l’éditeur de code).

    **<font color="red">Assurez-vous d’avoir basculé vers la version classique du Cloud Shell avant de continuer.</font>**

1. Dans le volet Cloud Shell, saisissez les commandes suivantes pour cloner le référentiel GitHub contenant les fichiers de code pour cet exercice (saisissez la commande, ou copiez-la dans le presse-papiers puis effectuez un clic droit dans la ligne de commande pour la coller en texte brut) :

    ```
   rm -r mslearn-ai-audio -f
   git clone https://github.com/MicrosoftLearning/mslearn-ai-language
    ```

    > **Conseil** : lorsque vous collez des commandes dans cloudshell, la sortie peut prendre une grande quantité de mémoire tampon d’écran. Vous pouvez effacer le contenu de l’écran en saisissant la commande `cls` pour faciliter le focus sur chaque tâche.

1. Une fois le référentiel cloné, accédez au dossier contenant les fichiers de code de l’application :  

    ```
   cd mslearn-ai-language/Labfiles/09-audio-chat/Python
    ````

1. Dans le volet de ligne de commande Cloud Shell, saisissez la commande suivante pour installer les bibliothèques que vous utiliserez, c’est-à-dire :

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-identity azure-ai-projects openai
    ```

1. Saisissez la commande suivante pour modifier le fichier de configuration fourni :

    ```
   code .env
    ```

    Le fichier devrait s’ouvrir dans un éditeur de code.

1. Dans le fichier de code, remplacez l’espace réservé **your_project_endpoint** par le point de terminaison de votre projet (copié depuis la page **Vue d’ensemble** du portail Azure AI Foundry), et l’espace réservé **your_model_deployment** par le nom que vous avez attribué à votre modèle de déploiement Phi-4-multimodal-instruct.

1. Après avoir remplacé les espaces réservés, utilisez la commande **CTRL+S** ou **Clic droit > Enregistrer** dans l’éditeur de code pour enregistrer vos modifications, puis la commande **CTRL+Q** ou **Clic droit > Quitter** pour fermer l’éditeur tout en laissant la ligne de commande Cloud Shell ouverte.

### Écrire du code pour vous connecter à votre projet et obtenir un client de conversation instantanée pour votre modèle

> **Conseil** : lorsque vous ajoutez du code, veillez à conserver la mise en retrait correcte.

1. Saisissez la commande suivante pour modifier le fichier de code :

    ```
   code audio-chat.py
    ```

1. Dans le fichier de code, notez les instructions existantes qui ont été ajoutées en haut du fichier pour importer les espaces de noms SDK nécessaires. Ensuite, repérez le commentaire **Ajouter des références**, puis ajoutez le code suivant pour référencer les espaces de noms des bibliothèques que vous avez installées précédemment :

    ```python
   # Add references
   from azure.identity import DefaultAzureCredential
   from azure.ai.projects import AIProjectClient
    ```

1. Dans la fonction **main**, sous le commentaire **Obtenir les paramètres de configuration**, notez que le code charge les valeurs de chaîne de connexion du projet et de nom de déploiement du modèle que vous avez définies dans le fichier de configuration.

1. Recherchez le commentaire **Initialiser le client du projet** et ajoutez le code suivant pour connecter votre projet Azure AI Foundry :

    > **Conseil** : veillez à respecter le niveau d’indentation correct dans votre code.

    ```python
   # Initialize the project client
   project_client = AIProjectClient(            
       credential=DefaultAzureCredential(
           exclude_environment_credential=True,
           exclude_managed_identity_credential=True
       ),
       endpoint=project_endpoint,
   )
    ```

1. Recherchez le commentaire **Obtenir un client de conversation instantanée** et ajoutez le code suivant pour créer un objet client afin de converser avec votre modèle :

    ```python
   # Get a chat client
   openai_client = project_client.get_openai_client(api_version="2024-10-21")
    ```

### Écrivez du code pour envoyer une requête audio.

Avant de soumettre la requête, nous devons encoder le fichier audio pour l’envoi. Nous pouvons ensuite joindre les données audio au message de l’utilisateur avec une instruction destinée au LLM. Notez que le code inclut une boucle qui permet à l’utilisateur de saisir une instruction jusqu’à ce qu’il saisisse « quitter ». 

1. Sous le commentaire **Encoder le fichier audio**, saisissez le code suivant pour préparer le fichier audio suivant :

    <video controls src="https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Instructions/media/avocados.mp4" title="une demande d’avocats" width="150"></video>

    ```python
   # Encode the audio file
   file_path = "https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Labfiles/09-audio-chat/data/avocados.mp3"
   response = requests.get(file_path)
   response.raise_for_status()
   audio_data = base64.b64encode(response.content).decode('utf-8')
    ```

1. Sous le commentaire **Voir une réponse à l’entrée audio**, ajoutez le code suivant pour soumettre une requête :

    ```python
   # Get a response to audio input
   response = openai_client.chat.completions.create(
       model=model_deployment,
       messages=[
           {"role": "system", "content": system_message},
           { "role": "user",
               "content": [
               { 
                   "type": "text",
                   "text": prompt
               },
               {
                   "type": "input_audio",
                   "input_audio": {
                       "data": audio_data,
                       "format": "mp3"
                   }
               }
           ] }
       ]
   )
   print(response.choices[0].message.content)
    ```

1. Utilisez la commande **Ctrl+S** pour enregistrer les modifications que vous avez apportées au fichier de code. Vous pouvez également fermer l’éditeur de code (**Ctrl+Q**) si vous le souhaitez.

### Se connecter à Azure et exécuter l’application

1. Dans le volet de la ligne de commande Cloud Shell, entrez la commande suivante pour vous connecter à Azure.

    ```
   az login
    ```

    **<font color="red">Vous devez vous connecter à Azure, même si la session Cloud Shell est déjà authentifiée.</font>**

    > **Remarque** :dans la plupart des scénarios, l’utilisation d’*az login* suffit. Toutefois, si vous avez des abonnements dans plusieurs locataires, vous devrez peut-être spécifier le locataire à l’aide du paramètre *--tenant*. Pour plus d’informations, consultez [Se connecter à Azure de manière interactive à l’aide d’Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively).
    
1. Lorsque l’invite apparaît, suivez les instructions pour ouvrir la page de connexion dans un nouvel onglet et entrez le code d’authentification fourni ainsi que vos informations d’identification Azure. Effectuez ensuite le processus de connexion dans la ligne de commande, en sélectionnant l’abonnement contenant votre hub Azure AI Foundry si nécessaire.

1. Dans le volet en ligne de commande du Cloud Shell, saisissez la commande suivante pour exécuter l’application :

    ```
   python audio-chat.py
    ```

1. Quand vous y êtes invité, entrez l’invite 

    ```
   Can you summarize this customer's voice message?
    ```

1. Passez en revue la réponse.

### Utiliser un autre fichier audio

1. Dans l’éditeur de code de votre application, trouvez le code que vous avez précédemment ajouté sous le commentaire **Encoder le fichier audio**. Modifiez ensuite l’URL du chemin d’accès du fichier comme indiqué ci-dessous pour utiliser un fichier audio différent dans la requête (sans modifier le code existant suivant le chemin d’accès) :

    ```python
   # Encode the audio file
   file_path = "https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Labfiles/09-audio-chat/data/fresas.mp3"
    ```

    Le nouveau fichier se présente comme suit :

    <video controls src="https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Instructions/media/fresas.mp4" title="une demande de fraises" width="150"></video>

 1. Utilisez la commande **Ctrl+S** pour enregistrer les modifications que vous avez apportées au fichier de code. Vous pouvez également fermer l’éditeur de code (**Ctrl+Q**) si vous le souhaitez.

1. Dans le volet de ligne de commande Cloud Shell sous l’Éditeur de code, entrez la commande suivante pour exécuter l’application :

    ```
   python audio-chat.py
    ```

1. Quand vous y êtes invité, entrez l’invite suivante : 
    
    ```
   Can you summarize this customer's voice message? Is it time-sensitive?
    ```

1. Passez en revue la réponse. Ensuite, entrez `quit` pour quitter le programme.

    > **Remarque** : dans cette application simple, nous n‘avons pas implémenté de logique pour conserver l‘historique des conversations. Par conséquent, le modèle traite chaque invite comme une nouvelle requête sans contexte de l‘invite précédente.

1. Vous pouvez continuer à exécuter l‘application, en choisissant différents types d‘invites et en essayant différentes invites. Lorsque vous avez terminé, entrez `quit` pour quitter le programme.

    Si vous avez le temps, vous pouvez modifier le code pour utiliser une autre invite du système et vos propres fichiers audio et image accessibles sur Internet.

    > **Remarque** : dans cette application simple, nous n‘avons pas implémenté de logique pour conserver l‘historique des conversations. Par conséquent, le modèle traite chaque invite comme une nouvelle requête sans contexte de l‘invite précédente.

## Résumé

Dans cet exercice, vous avez utilisé Azure AI Foundry et le kit de développement logiciel (SDK) Azure AI Inference pour créer une application cliente utilisant un modèle multimodal afin de générer des réponses à l‘audio.

## Nettoyage

.Si vous avez terminé d‘explorer Azure AI Foundry, vous devez supprimer les ressources que vous avez créées dans cet exercice pour éviter d‘entraîner des coûts Azure inutiles.

1. Revenez à l’onglet du navigateur contenant le portail Azure (ou ouvrez à nouveau le [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com` dans un nouvel onglet de navigateur) et affichez le contenu du groupe de ressources dans lequel vous avez déployé les ressources utilisées dans cet exercice.
1. Dans la barre d’outils, sélectionnez **Supprimer le groupe de ressources**.
1. Entrez le nom du groupe de ressources et confirmez que vous souhaitez le supprimer.
