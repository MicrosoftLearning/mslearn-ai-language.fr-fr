---
lab:
  title: Développer une application de conversation avec fonction audio.
  description: "Apprenez à utiliser Azure\_AI\_Foundry pour créer une application d’IA générative prenant en charge l’entrée audio."
---

# Développer une application de conversation avec fonction audio.

Dans cet exercice, vous utiliserez le modèle génératif d’IA *Phi-4-multimodal-instruct* pour générer des réponses à des invites comprenant des fichiers audio. Vous allez développer une application qui aide l’IA à fournir des produits frais dans une épicerie grâce á Azure AI Foundry et au service d’inférence de modèle Azure AI.

Cet exercice prend environ **30** minutes.

## Créer un projet Azure AI Foundry

Commençons par créer un projet Azure AI Foundry.

1. Dans un navigateur web, ouvrez le [portail Azure AI Foundry](https://ai.azure.com) à l’adresse `https://ai.azure.com` et connectez-vous en utilisant vos informations d’identification Azure. Fermez les conseils ou les volets de démarrage rapide ouverts la première fois que vous vous connectez et, si nécessaire, utilisez le logo **Azure AI Foundry** en haut à gauche pour accéder à la page d’accueil, qui ressemble à l’image suivante :

    ![Capture d’écran du portail Azure AI Foundry.](../media/ai-foundry-home.png)

1. Sur la page d’accueil, sélectionnez **+Créer un projet**.
1. Dans l’assistant **Créer un projet**, saisissez un nom valide et, si un hub existant est suggéré, choisissez l’option permettant d’en créer un. Passez ensuite en revue les ressources Azure qui seront créées automatiquement pour prendre en charge votre hub et votre projet.
1. Sélectionnez **Personnaliser** et spécifiez les paramètres suivants pour votre hub :
    - **Nom du hub** : *un nom valide pour votre hub*
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *créez ou sélectionnez un groupe de ressources*
    - **Région** : Sélectionnez l’une des régions suivantes\* :
        - USA Est
        - USA Est 2
        - Centre-Nord des États-Unis
        - États-Unis - partie centrale méridionale
        - Suède Centre
        - USA Ouest
        - USA Ouest 3
    - **Connecter Azure AI Services ou Azure OpenAI** : *créer une nouvelle ressource AI Services*
    - **Connecter la Recherche Azure AI** : ignorer la connexion

    > \* Au moment de l’écriture, le modèle Microsoft *Phi-4-multimodal-instruct* que nous allons utiliser dans cet exercice est disponible dans ces régions. Vous pouvez consulter les dernières disponibilités régionales de certains modèles dans la [documentation Azure AI Foundry](https://learn.microsoft.com/azure/ai-foundry/how-to/deploy-models-serverless-availability#region-availability). Si une limite de quota régionale est atteinte plus tard dans l’exercice, vous devrez peut-être créer une autre ressource dans une autre région.

1. Sélectionnez **Suivant** et passez en revue votre configuration. Sélectionnez **Créer** et patientez jusqu’à ce que l’opération se termine.
1. Une fois votre projet créé, fermez les conseils affichés et passez en revue la page du projet dans le portail Azure AI Foundry, qui doit ressembler à l’image suivante :

    ![Capture d’écran des détails d’un projet Azure AI dans le portail Azure AI Foundry.](../media/ai-foundry-project.png)

## Déployer un modèle multimodal

Vous êtes maintenant prêt à déployer un modèle multimodal qui peut prendre en charge l’entrée basée sur l’audio. Vous pouvez choisir parmi plusieurs modèles, notamment le modèle OpenAI *gpt-4o*. Dans cet exercice, nous allons utiliser un modèle *Phi-4-multimodal-instruct*.

1. Dans la barre d’outils située en haut à droite de la page de votre projet Azure AI Foundry, utilisez l’icône **Fonctionnalités en préversion** (**&#9215;**) pour vérifier que la fonctionnalité **Déployer des modèles vers le service d’inférence de modèles Azure AI** est activée. Cette fonctionnalité garantit que votre déploiement de modèle est disponible pour le service Inférence Azure AI, que vous utiliserez dans votre code d’application.
1. Dans le volet de gauche de votre projet, dans la section **Mes ressources**, sélectionnez la page **Modèles + points de terminaison**.
1. Sur la page **Modèles + points de terminaison**, dans l’onglet **Déploiements de modèles**, dans le menu **+ Déployer un modèle**, sélectionnez **Déployer le modèle de base**.
1. Recherchez le modèle **Phi-4-multimodal-instruct** dans la liste, puis sélectionnez-le et confirmez-le.
1. Acceptez le contrat de licence si vous y êtes invité, puis déployez le modèle avec les paramètres suivants en sélectionnant **Personnaliser** dans les détails du déploiement :
    - **Nom du déploiement** : *nom valide pour votre modèle de déploiement*
    - **Type de déploiement** : standard global
    - **Détails du déploiement** : *utilisez les paramètres par défaut*
1. Attendez que l’état d’approvisionnement du déploiement soit **Terminé**.

## Créer une application cliente

Maintenant que vous avez déployé le modèle, vous pouvez utiliser le déploiement dans une application cliente.

> **Conseil** : vous pouvez choisir de développer votre solution à l’aide de Python ou de Microsoft C#. Suivez les instructions de la section appropriée pour votre langue choisie.

### Préparer la configuration de l’application

1. Dans le portail Azure AI Foundry, affichez la page **Vue d’ensemble** de votre projet.
1. Dans la zone **Détails du projet**, notez la **chaîne de connexion du projet**. Vous utiliserez cette chaîne de connexion pour vous connecter à votre projet dans une application cliente.
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
    git clone https://github.com/MicrosoftLearning/mslearn-ai-language mslearn-ai-audio
    ```

    > **Conseil** : lorsque vous collez des commandes dans cloudshell, la sortie peut prendre une grande quantité de mémoire tampon d’écran. Vous pouvez effacer le contenu de l’écran en saisissant la commande `cls` pour faciliter le focus sur chaque tâche.

1. Une fois le référentiel cloné, accédez au dossier contenant les fichiers de code de l’application :  

    **Python**

    ```
    cd mslearn-ai-audio/Labfiles/09-audio-chat/python
    ```

    **C#**

    ```
    cd mslearn-ai-audio/Labfiles/09-audio-chat/c-sharp
    ```

1. Dans le volet de ligne de commande Cloud Shell, saisissez la commande suivante pour installer les bibliothèques que vous utiliserez, c’est-à-dire :

    **Python**

    ```
    python -m venv labenv
    ./labenv/bin/Activate.ps1
    pip install python-dotenv azure-identity azure-ai-projects azure-ai-inference
    ```

    **C#**

    ```
    dotnet add package Azure.Identity
    dotnet add package Azure.AI.Inference --version 1.0.0-beta.3
    dotnet add package Azure.AI.Projects --version 1.0.0-beta.3
    ```

1. Saisissez la commande suivante pour modifier le fichier de configuration fourni :

    **Python**

    ```
    code .env
    ```

    **C#**

    ```
    code appsettings.json
    ```

    Le fichier devrait s’ouvrir dans un éditeur de code.

1. Dans le fichier de code, remplacez l’espace réservé **your_project_connection_string** par la chaîne de connexion de votre projet (copiée depuis la page **Vue d'ensemble** du projet dans le portail Azure AI Foundry), et remplacez **your_model_deployment** par le nom que vous avez attribué à votre déploiement du modèle Phi-4-multimodal-instruct.

1. Après avoir remplacé les espaces réservés, utilisez la commande **CTRL+S** ou **Clic droit > Enregistrer** dans l’éditeur de code pour enregistrer vos modifications, puis la commande **CTRL+Q** ou **Clic droit > Quitter** pour fermer l’éditeur tout en laissant la ligne de commande Cloud Shell ouverte.

### Écrire du code pour vous connecter à votre projet et obtenir un client de conversation instantanée pour votre modèle

> **Conseil** : lorsque vous ajoutez du code, veillez à conserver la mise en retrait correcte.

1. Saisissez la commande suivante pour modifier le fichier de code fourni :

    **Python**

    ```
    code audio-chat.py
    ```

    **C#**

    ```
    code Program.cs
    ```

1. Dans le fichier de code, notez les instructions existantes qui ont été ajoutées en haut du fichier pour importer les espaces de noms SDK nécessaires. Ensuite, repérez le commentaire **Ajouter des références**, puis ajoutez le code suivant pour référencer les espaces de noms des bibliothèques que vous avez installées précédemment :

    **Python**

    ```python
    # Add references
    from dotenv import load_dotenv
    from azure.identity import DefaultAzureCredential
    from azure.ai.projects import AIProjectClient
    from azure.ai.inference.models import (
        SystemMessage,
        UserMessage,
        TextContentItem,
        AudioContentItem,
        InputAudio,
        AudioContentFormat,
    )
    ```

    **C#**

    ```csharp
    // Add references
    using Azure.Identity;
    using Azure.AI.Projects;
    using Azure.AI.Inference;
    ```

1. Dans la fonction **main**, sous le commentaire **Obtenir les paramètres de configuration**, notez que le code charge les valeurs de chaîne de connexion du projet et de nom de déploiement du modèle que vous avez définies dans le fichier de configuration.

1. Recherchez le commentaire **Initialiser le client du projet**, puis ajoutez le code suivant pour vous connecter à votre projet Azure AI Foundry à l’aide des informations d’identification Azure avec lesquelles vous êtes actuellement connecté.

    **Python**

    ```python
    # Initialize the project client
    project_client = AIProjectClient.from_connection_string(
        conn_str=project_connection,
        credential=DefaultAzureCredential())
    ```

    **C#**

    ```csharp
    // Initialize the project client
    var projectClient = new AIProjectClient(project_connection,
                        new DefaultAzureCredential());
    ```

1. Recherchez le commentaire **Obtenir un client de conversation instantanée** et ajoutez le code suivant pour créer un objet client afin de converser avec votre modèle :

    **Python**

    ```python
    # Get a chat client
    chat_client = project_client.inference.get_chat_completions_client(model=model_deployment)
    ```

    **C#**

    ```csharp
    // Get a chat client
    ChatCompletionsClient chat = projectClient.GetChatCompletionsClient();
    ```
    

### Écrire du code pour envoyer une invite basée sur une URL

1. Dans l’éditeur de code, pour le fichier **audio-chat.py**, dans la section de boucle, sous le commentaire **Obtenir une réponse à l’entrée audio**, ajoutez le code suivant pour soumettre une invite contenant l’audio suivant :

    <video controls src="../media/manzanas.mp4" title="une demande de pommes" width="150"></video>

    **Python**

    ```python
    # Get a response to audio input
    file_path = "https://github.com/microsoftlearning/mslearn-ai-language/raw/refs/heads/main/labfiles/09-audio-chat/data/manzanas.mp3"
    response = chat_client.complete(
        messages=[
            SystemMessage(system_message),
            UserMessage(
                [
                    TextContentItem(text=prompt),
                    {
                        "type": "audio_url",
                        "audio_url": {"url": file_path}
                    }
                ]
            )
        ]
    )
    print(response.choices[0].message.content)
    ```

    **C#**

    ```csharp
    // Get a response to audio input
    string audioUrl = "https://github.com/microsoftlearning/mslearn-ai-language/raw/refs/heads/main/labfiles/09-audio-chat/data/manzanas.mp3";
    var requestOptions = new ChatCompletionsOptions()
    {
        Messages =
        {
            new ChatRequestSystemMessage(system_message),
            new ChatRequestUserMessage(
                new ChatMessageTextContentItem(prompt),
                new ChatMessageAudioContentItem(new Uri(audioUrl))),
        },
        Model = model_deployment
    };
    var response = chat.Complete(requestOptions);
    Console.WriteLine(response.Value.Content);
    ```

1. Utilisez la commande **Ctrl+S** pour enregistrer les modifications que vous avez apportées au fichier de code. Vous pouvez également fermer l’éditeur de code (**Ctrl+Q**) si vous le souhaitez.

1. Dans le volet de ligne de commande Cloud Shell, sous l’éditeur de code, entrez la commande suivante pour exécuter l’application :

    **Python**

    ```
    python audio-chat.py
    ```

    **C#**

    ```
    dotnet run
    ```

1. Quand vous y êtes invité, entrez l’invite suivante : `What is this customer saying in English?`.

1. Vérifiez la réponse.

### Utiliser une autre invite

1. Dans l’éditeur de code, recherchez le code que vous avez ajouté précédemment sous le commentaire **Obtenir une réponse à l’entrée audio**. Modifiez ensuite le code comme suit pour sélectionner un autre fichier audio :

    <video controls src="../media/caomei.mp4" title="une demande de fraises" width="150"></video>

    **Python**

    ```python
    # Get a response to audio input
    file_path = "https://github.com/microsoftlearning/mslearn-ai-language/raw/refs/heads/main/labfiles/09-audio-chat/data/caomei.mp3"
    response = chat_client.complete(
        messages=[
            SystemMessage(system_message),
            UserMessage(
                [
                    TextContentItem(text=prompt),
                    {
                        "type": "audio_url",
                        "audio_url": {"url": file_path}
                    }
                ]
            )
        ]
    )
    print(response.choices[0].message.content)
    ```

    **C#**

    ```csharp
    // Get a response to audio input
    string audioUrl = "https://github.com/microsoftlearning/mslearn-ai-language/raw/refs/heads/main/labfiles/09-audio-chat/data/caomei.mp3";
    var requestOptions = new ChatCompletionsOptions()
    {
        Messages =
        {
            new ChatRequestSystemMessage(system_message),
            new ChatRequestUserMessage(
                new ChatMessageTextContentItem(prompt),
                new ChatMessageAudioContentItem(new Uri(audioUrl))),
        },
        Model = model_deployment
    };
    var response = chat.Complete(requestOptions);
    Console.WriteLine(response.Value.Content);
    ```

1. Utilisez la commande **Ctrl+S** pour enregistrer les modifications que vous avez apportées au fichier de code. Vous pouvez également fermer l’éditeur de code (**Ctrl+Q**) si vous le souhaitez.

1. Dans le volet de ligne de commande Cloud Shell sous l’Éditeur de code, entrez la commande suivante pour exécuter l’application :

    **Python**

    ```
    python audio-chat.py
    ```

    **C#**

    ```
    dotnet run
    ```

1. Quand vous y êtes invité, entrez l’invite suivante :

    ```
    A customer left this voice message, can you summarize it?
    ```

1. Vérifiez la réponse. Ensuite, entrez `quit` pour quitter le programme.

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
