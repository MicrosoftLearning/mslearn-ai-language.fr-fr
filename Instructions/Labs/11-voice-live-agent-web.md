---
lab:
  title: Développer un assistant vocal Azure AI Voice Live
  description: Découvrez comment créer une application web permettant des interactions vocales en temps réel avec un assistant Azure AI Voice Live.
---

# Développer un assistant vocal Azure AI Voice Live

Dans cet exercice, vous allez créer une application web Python basée sur Flask qui permet des interactions vocales en temps réel avec un assistant. Vous allez ajouter le code permettant d’initialiser la session et de gérer les événements de session. Vous utiliserez un script de déploiement qui : déploie le modèle IA ; crée une image de l’application dans Azure Container Registry (ACR) à l’aide de tâches ACR ; puis crée une instance Azure App Service qui extrait l’image. Pour tester l’application, vous aurez besoin d’un périphérique audio avec microphone et haut-parleur.

Bien que cet exercice soit basé sur Python, vous pouvez développer des applications similaires avec d’autres kits de développement logiciel (SDK) spécifiques aux langages, notamment :

- [Bibliothèque cliente Azure VoiceLive pour .NET](https://www.nuget.org/packages/Azure.AI.VoiceLive/)

Tâches effectuées dans cet exercice :

* Téléchargez les fichiers de base de l’application
* Ajoutez du code pour terminer l’application web
* Passez en revue l’ensemble de la base de code
* Mettez à jour et exécutez le script de déploiement
* Affichez et testez l’application

Cet exercice dure environ **30** minutes.

## Lancez Azure Cloud Shell et téléchargez les fichiers

Dans cette section de l’exercice, vous téléchargez un fichier compressé contenant les fichiers de base de l’application.

1. Dans votre navigateur, accédez au portail Azure [https://portal.azure.com](https://portal.azure.com) et connectez-vous en utilisant vos informations d’identification Azure.

1. Cliquez sur le bouton **[\>_]** à droite de la barre de recherche, en haut de la page, pour créer un Cloud Shell dans le portail Azure, puis sélectionnez un environnement ***Bash***. Cloud Shell fournit une interface de ligne de commande dans un volet situé en bas du portail Azure.

    > **Note** : si vous avez déjà créé un Cloud Shell qui utilise un environnement *PowerShel*, basculez-le vers ***Bash***.

1. Dans la barre d’outils Cloud Shell, dans le menu **Paramètres**, sélectionnez **Accéder à la version classique** (cela est nécessaire pour utiliser l’éditeur de code).

1. Exécutez la commande suivante dans l’interpréteur de commandes **Bash** pour télécharger et décompresser les fichiers de l’exercice. La deuxième commande vous permettra également d’accéder au répertoire contenant les fichiers de l’exercice.

    ```bash
    wget https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/downloads/python/voice-live-web.zip
    ```

    ```
    unzip voice-live-web.zip && cd voice-live-web
    ```

## Ajoutez du code pour terminer l’application web

Maintenant que les fichiers de l’exercice sont téléchargés, l’étape suivante consiste à ajouter du code pour terminer l’application. Les étapes suivantes sont effectuées dans le Cloud Shell. 

>**Conseil :** Redimensionnez le Cloud Shell pour afficher plus d’informations et de code en faisant glisser la bordure supérieure. Vous pouvez également utiliser les boutons de réduction et d’agrandissement pour alterner entre le Cloud Shell et l’interface principale du portail.

Exécutez la commande suivante pour accéder au répertoire *src* avant de poursuivre l’exercice.

```bash
cd src
```

### Ajoutez du code pour implémenter l’assistant vocal en direct

Dans cette section, vous ajoutez du code pour implémenter l’assistant vocal en direct. La méthode **\_\_init\_\_** initialise l’assistant vocal en stockant les paramètres de connexion Azure VoiceLive (point de terminaison, informations d’identification, modèle, voix et instructions système) et en configurant des variables d’état d’exécution pour gérer le cycle de vie de la connexion et les interruptions de l’utilisateur pendant les conversations. La méthode **start** importe les composants nécessaires du kit de développement logiciel (SDK) Azure VoiceLive utilisés pour établir la connexion WebSocket et configurer la session vocale en temps réel.

1. Exécutez la commande suivante pour ouvrir le fichier *flask_app.py* pour modification.

    ```bash
    code flask_app.py
    ```

1. Recherchez le commentaire **# BEGIN VOICE LIVE ASSISTANT IMPLEMENTATION - ALIGN CODE WITH COMMENT** dans le code. Copiez le code ci-dessous et collez-le juste en dessous du commentaire. Veillez à vérifier la mise en retrait.

    ```python
    def __init__(
        self,
        endpoint: str,
        credential,
        model: str,
        voice: str,
        instructions: str,
        state_callback=None,
    ):
        # Store Azure Voice Live connection and configuration parameters
        self.endpoint = endpoint
        self.credential = credential
        self.model = model
        self.voice = voice
        self.instructions = instructions
        
        # Initialize runtime state - connection established in start()
        self.connection = None
        self._response_cancelled = False  # Used to handle user interruptions
        self._stopping = False  # Signals graceful shutdown
        self.state_callback = state_callback or (lambda *_: None)

    async def start(self):
        # Import Voice Live SDK components needed for establishing connection and configuring session
        from azure.ai.voicelive.aio import connect  # type: ignore
        from azure.ai.voicelive.models import (
            RequestSession,
            ServerVad,
            AzureStandardVoice,
            Modality,
            InputAudioFormat,
            OutputAudioFormat,
        )  # type: ignore
    ```

1. Appuyez sur **ctrl+s** pour enregistrer vos modifications et laissez l’éditeur ouvert pour la section suivante.

### Ajoutez du code pour implémenter l’assistant vocal en direct

Dans cette section, vous ajoutez du code pour configurer la session vocale en direct. Cela permet de spécifier les modalités (l’audio seul n’est pas pris en charge par l’API), les instructions système qui définissent le comportement de l’assistant, la voix Azure TTS pour les réponses, le format audio pour les flux d’entrée et de sortie, et la détection d’activité vocale (VAD) côté serveur qui indique comment le modèle détecte le début et la fin de la parole des utilisateurs.

1. Recherchez le commentaire **# BEGIN CONFIGURE VOICE LIVE SESSION - ALIGN CODE WITH COMMENT** dans le code. Copiez le code ci-dessous et collez-le juste en dessous du commentaire. Veillez à vérifier la mise en retrait.

    ```python
    # Configure VoiceLive session with audio/text modalities and voice activity detection
    session_config = RequestSession(
        modalities=[Modality.TEXT, Modality.AUDIO],
        instructions=self.instructions,
        voice=voice_cfg,
        input_audio_format=InputAudioFormat.PCM16,
        output_audio_format=OutputAudioFormat.PCM16,
        turn_detection=ServerVad(threshold=0.5, prefix_padding_ms=300, silence_duration_ms=500),
    )
    await conn.session.update(session=session_config)
    ```

1. Appuyez sur **ctrl+s** pour enregistrer vos modifications et laissez l’éditeur ouvert pour la section suivante.

### Ajoutez du code pour gérer les événements de session

Dans cette section, vous ajoutez du code pour ajouter des gestionnaires d’événements pour la session vocale en direct. Les gestionnaires d’événements répondent aux principaux événements de la session VoiceLive pendant le cycle de vie de la conversation : **_handle_session_updated** signale que la session est prête à recevoir les entrées de l’utilisateur, **_handle_speech_started** détecte lorsque l’utilisateur commence à parler et met en œuvre une logique d’interruption en arrêtant toute lecture audio en cours de l’assistant et en annulant les réponses en cours afin de permettre un flux de conversation naturel, et **_handle_speech_stopped** gère la fin de parole de l’utilisateur et le début du traitement de l’entrée par l’assistant.

1. Recherchez le commentaire **# BEGIN HANDLE SESSION EVENTS - ALIGN CODE WITH COMMENT** dans le code. Copiez le code ci-dessous et collez-le juste en dessous du commentaire, en veillant à respecter la mise en retrait.

    ```python
    async def _handle_event(self, event, conn, verbose=False):
        """Handle Voice Live events with clear separation by event type."""
        # Import event types for processing different Voice Live server events
        from azure.ai.voicelive.models import ServerEventType
        
        event_type = event.type
        if verbose:
            _broadcast({"type": "log", "level": "debug", "event_type": str(event_type)})
        
        # Route Voice Live server events to appropriate handlers
        if event_type == ServerEventType.SESSION_UPDATED:
            await self._handle_session_updated()
        elif event_type == ServerEventType.INPUT_AUDIO_BUFFER_SPEECH_STARTED:
            await self._handle_speech_started(conn)
        elif event_type == ServerEventType.INPUT_AUDIO_BUFFER_SPEECH_STOPPED:
            await self._handle_speech_stopped()
        elif event_type == ServerEventType.RESPONSE_AUDIO_DELTA:
            await self._handle_audio_delta(event)
        elif event_type == ServerEventType.RESPONSE_AUDIO_DONE:
            await self._handle_audio_done()
        elif event_type == ServerEventType.RESPONSE_DONE:
            # Reset cancellation flag but don't change state - _handle_audio_done already did
            self._response_cancelled = False
        elif event_type == ServerEventType.ERROR:
            await self._handle_error(event)

    async def _handle_session_updated(self):
        """Session is ready for conversation."""
        self.state_callback("ready", "Session ready. You can start speaking now.")

    async def _handle_speech_started(self, conn):
        """User started speaking - handle interruption if needed."""
        self.state_callback("listening", "Listening… speak now")
        
        try:
            # Stop any ongoing audio playback on the client side
            _broadcast({"type": "control", "action": "stop_playback"})
            
            # If assistant is currently speaking or processing, cancel the response to allow interruption
            current_state = assistant_state.get("state")
            if current_state in {"assistant_speaking", "processing"}:
                self._response_cancelled = True
                await conn.response.cancel()
                _broadcast({"type": "log", "level": "debug", 
                          "msg": f"Interrupted assistant during {current_state}"})
            else:
                _broadcast({"type": "log", "level": "debug", 
                          "msg": f"User speaking during {current_state} - no cancellation needed"})
        except Exception as e:
            _broadcast({"type": "log", "level": "debug", 
                      "msg": f"Exception in speech handler: {e}"})

    async def _handle_speech_stopped(self):
        """User stopped speaking - processing input."""
        self.state_callback("processing", "Processing your input…")

    async def _handle_audio_delta(self, event):
        """Stream assistant audio to clients."""
        if self._response_cancelled:
            return  # Skip cancelled responses
            
        # Update state when assistant starts speaking
        if assistant_state.get("state") != "assistant_speaking":
            self.state_callback("assistant_speaking", "Assistant speaking…")
        
        # Extract and broadcast Voice Live audio delta as base64 to WebSocket clients
        audio_data = getattr(event, "delta", None)
        if audio_data:
            audio_b64 = base64.b64encode(audio_data).decode("utf-8")
            _broadcast({"type": "audio", "audio": audio_b64})

    async def _handle_audio_done(self):
        """Assistant finished speaking."""
        self._response_cancelled = False
        self.state_callback("ready", "Assistant finished. You can speak again.")

    async def _handle_error(self, event):
        """Handle Voice Live errors."""
        error = getattr(event, "error", None)
        message = getattr(error, "message", "Unknown error") if error else "Unknown error"
        self.state_callback("error", f"Error: {message}")

    def request_stop(self):
        self._stopping = True
    ```

1. Appuyez sur **ctrl+s** pour enregistrer vos modifications et laissez l’éditeur ouvert pour la section suivante.

### Passez en revue le code dans l’application

Jusqu’à présent, vous avez ajouté du code à l’application pour implémenter l’assistant et gérer les événements de l’assistant. Prenez quelques minutes pour passer en revue l’intégralité du code et les commentaires afin de mieux comprendre comment l’application gère l’état et les opérations du client.

1. Lorsque vous avez terminé, appuyez sur **ctrl+q** pour quitter l’éditeur. 

## Mettez à jour et exécutez le script de déploiement

Dans cette section, vous apportez une petite modification au script de déploiement **azdeploy.sh**, puis vous exécutez le déploiement. 

### Mettez à jour le script de déploiement

Vous ne devez modifier que deux valeurs en haut du script de déploiement **azdeploy.sh**. 

* La valeur **rg** spécifie le groupe de ressources qui contiendra le déploiement. Vous pouvez accepter la valeur par défaut ou entrer votre propre valeur si vous devez effectuer le déploiement dans un groupe de ressources spécifique.

* La valeur **location** définit la région pour le déploiement. Le modèle *gpt-4o* utilisé dans l’exercice peut être déployé dans d’autres régions, mais il peut y avoir des limites dans certaines régions. Si le déploiement échoue dans la région que vous avez choisie, essayez **eastus2** ou **swedencentral**. 

    ```
    rg="rg-voicelive" # Replace with your resource group
    location="eastus2" # Or a location near you
    ```

1. Exécutez les commandes suivantes dans le Cloud Shell pour commencer à modifier le script de déploiement.

    ```bash
    cd ~/voice-live-web
    ```
    
    ```bash
    code azdeploy.sh
    ```

1. Mettez à jour les valeurs **rg** et **location** en fonction de vos besoins, puis appuyez sur **ctrl+s** pour enregistrer vos modifications et sur **ctrl+q** pour quitter l’éditeur.

### Exécuter le script de déploiement

Le script de déploiement déploie le modèle IA et crée les ressources nécessaires dans Azure pour exécuter une application conteneurisée dans App Service.

1. Exécutez la commande suivante dans le Cloud Shell pour commencer à déployer les ressources Azure et l’application.

    ```bash
    bash azdeploy.sh
    ```

1. Sélectionnez **option 1** pour le déploiement initial.

    Le déploiement devrait prendre entre 5 et 10 minutes. Pendant le déploiement, vous pouvez être invité à fournir les informations/effectuer les actions suivantes :
    
    * Si vous êtes invité à vous authentifier auprès d’Azure, suivez les instructions présentées.
    * Si vous êtes invité à sélectionner un abonnement, utilisez les flèches pour mettre en surbrillance votre abonnement et appuyez sur **Entrée**. 
    * Vous verrez probablement apparaître des avertissements pendant le déploiement, mais vous pouvez les ignorer.
    * Si le déploiement échoue pendant le déploiement du modèle IA, modifiez la région dans le script de déploiement et réessayez. 
    * Les régions Azure peuvent parfois être saturées, ce qui peut perturber le déroulement du déploiement. Si le déploiement échoue après le déploiement de modèle, réexécutez le script de déploiement.

## Affichez et testez l’application

Une fois le déploiement terminé, un message « Déploiement terminé ! » s’affichera dans l’interpréteur de commandes, accompagné d’un lien vers l’application web. Vous pouvez sélectionner ce lien ou accéder à la ressource App Service et lancer l’application à partir de là. Le chargement de l’application peut prendre quelques minutes. 

1. Sélectionnez le bouton **Démarrer la session** pour vous connecter au modèle.
1. Vous serez invité à autoriser l’application à accéder à vos périphériques audio.
1. Commencez à parler au modèle lorsque l’application vous invite à le faire.

Résolution des problèmes :

* Si l’application signale des variables d’environnement manquantes, redémarrez l’application dans App Service.
* Si vous voyez trop de messages *bloc audio* dans le journal affiché dans l’application, sélectionnez **Arrêter la session**, puis redémarrez la session. 
* Si l’application ne fonctionne pas du tout, vérifiez que vous avez bien ajouté tout le code et respecté la mise en retrait. Si vous devez apporter des modifications, réexécutez le déploiement et sélectionnez **option 2** pour mettre à jour uniquement l’image.

## Nettoyer les ressources

Exécutez la commande suivante dans le Cloud Shell pour supprimer toutes les ressources déployées pour cet exercice. Vous serez invité à confirmer la suppression des ressources.

```
azd down --purge
```