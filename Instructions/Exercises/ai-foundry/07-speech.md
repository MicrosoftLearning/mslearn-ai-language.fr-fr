---
lab:
  title: "Reconnaître et synthétiser du contenu vocal (version d’Azure\_AI\_Foundry)"
  module: Module 4 - Create speech-enabled apps with Azure AI services
---

<!--
Possibly update to use standalone AI Service instead of Foundry?
-->

# Reconnaître et synthétiser du contenu vocal

**Azure AI Speech** est un service qui fournit des fonctionnalités vocales, notamment les suivantes :

- Une API de *reconnaissance vocale* qui vous permet d’implémenter la reconnaissance vocale (conversion de mots parlés audibles en texte).
- Une API de *synthèse vocale* qui vous permet d’implémenter la synthèse vocale (conversion de texte en parole audible).

Dans cet exercice, vous allez utiliser ces deux API pour implémenter une application d’horloge parlante.

> **NOTE** : cet exercice est conçu pour être effectué dans Azure Cloud Shell, où l’accès direct au matériel audio de votre ordinateur n’est pas pris en charge. Le labo utilise donc des fichiers audio pour les flux d’entrée vocale et de sortie. Le code permettant d’obtenir les mêmes résultats à l’aide d’un micro et d’un haut-parleur est fourni pour votre référence.

## Créer un projet Azure AI Foundry

Commençons par créer un projet Azure AI Foundry.

1. Dans un navigateur web, ouvrez le [portail Azure AI Foundry](https://ai.azure.com) à l’adresse `https://ai.azure.com` et connectez-vous en utilisant vos informations d’identification Azure. Fermez les conseils ou les volets de démarrage rapide ouverts la première fois que vous vous connectez et, si nécessaire, utilisez le logo **Azure AI Foundry** en haut à gauche pour accéder à la page d’accueil, qui ressemble à l’image suivante :

    ![Capture d’écran du portail Azure AI Foundry.](./media/ai-foundry-home.png)

1. Sur la page d’accueil, sélectionnez **+Créer un projet**.
1. Dans l’assistant **Créer un projet**, entrez un nom de projet approprié pour (par exemple, `my-ai-project`), puis passez en revue les ressources Azure qui seront automatiquement créées pour prendre en charge votre projet.
1. Sélectionnez **Personnaliser** et spécifiez les paramètres suivants pour votre hub :
    - **Nom du hub** : *nom unique, par exemple `my-ai-hub`*
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *créez un groupe de ressources avec un nom unique ( par exemple, `my-ai-resources`) ou sélectionnez un groupe de ressources existant.*
    - **Emplacement** : choisir n’importe quelle région disponible
    - **Connecter Azure AI Services ou Azure OpenAI** : *créer une ressource AI Services avec un nom approprié (par exemple, `my-ai-services`) ou utiliser une ressource existante*
    - **Connecter la Recherche Azure AI** : ignorer la connexion

1. Sélectionnez **Suivant** et passez en revue votre configuration. Sélectionnez **Créer** et patientez jusqu’à ce que l’opération se termine.
1. Une fois votre projet créé, fermez les conseils affichés et passez en revue la page du projet dans le portail Azure AI Foundry, qui doit ressembler à l’image suivante :

    ![Capture d’écran des détails d’un projet Azure AI dans le portail Azure AI Foundry.](./media/ai-foundry-project.png)

## Préparer et configurer l’application d’horloge parlante

1. Dans le portail Azure AI Foundry, affichez la page **Vue d’ensemble** de votre projet.
1. Dans la zone **Détails du projet**, notez la **chaîne de connexion du projet** et **l’emplacement** de votre projet. Vous utiliserez la chaîne de connexion pour vous connecter à votre projet dans une application cliente, et vous aurez besoin de l’emplacement pour vous connecter au point de terminaison Azure AI Services Speech.
1. Ouvrez un nouvel onglet de navigateur (en gardant le portail Azure AI Foundry ouvert dans l’onglet existant). Dans un nouvel onglet du navigateur, ouvrez le [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com` et connectez-vous en utilisant vos informations d’identification Azure.
1. Cliquez sur le bouton **[\>_]** à droite de la barre de recherche, en haut de la page, pour créer un environnement Cloud Shell dans le portail Azure, puis sélectionnez un environnement ***PowerShell***. Cloud Shell fournit une interface de ligne de commande dans un volet situé en bas du portail Azure.

    > **Remarque** : si vous avez déjà créé un Cloud Shell qui utilise un environnement *Bash*, basculez-le vers ***PowerShell***.

1. Dans la barre d’outils Cloud Shell, dans le menu **Paramètres**, sélectionnez **Accéder à la version classique** (cela est nécessaire pour utiliser l’éditeur de code).

    > **Conseil** : lorsque vous collez des commandes dans cloudshell, la sortie peut prendre une grande quantité de mémoire tampon d’écran. Vous pouvez effacer le contenu de l’écran en saisissant la commande `cls` pour faciliter le focus sur chaque tâche.

1. Dans le volet PowerShell, entrez les commandes suivantes pour cloner le référentiel GitHub pour cet exercice :

    ```
   rm -r mslearn-ai-language -f
   git clone https://github.com/microsoftlearning/mslearn-ai-language mslearn-ai-language
    ```

    ***Suivez maintenant les étapes de votre langage de programmation choisi.***

1. Une fois le référentiel cloné, accédez au dossier contenant les fichiers de code de l’application d’horloge parlante :  

    **Python**

    ```
   cd mslearn-ai-language/labfiles/07b-speech/python/speaking-clock
    ```

    **C#**

    ```
   cd mslearn-ai-language/labfiles/07b-speech/c-sharp/speaking-clock
    ```

1. Dans le volet de ligne de commande Cloud Shell, saisissez la commande suivante pour installer les bibliothèques que vous utiliserez, c’est-à-dire :

    **Python**

    ```
   pip install python-dotenv azure-identity azure-ai-projects azure-cognitiveservices-speech==1.42.0
    ```

    **C#**

    ```
   dotnet add package Azure.Identity
   dotnet add package Azure.AI.Projects --prerelease
   dotnet add package Microsoft.CognitiveServices.Speech --version 1.42.0
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

    Le fichier s’ouvre dans un éditeur de code.

1. Dans le fichier de code, remplacez les espaces réservés **your_project_endpoint** et **your_location** par la chaîne de connexion et l’emplacement de votre projet (copiés depuis la page **Vue d’ensemble** du portail Azure AI Foundry).
1. Une fois que vous avez remplacé les espaces réservés, utilisez la commande **Ctrl+S** pour enregistrer vos modifications, puis utilisez la commande **Ctrl+Q** pour fermer l’éditeur de code tout en gardant la ligne de commande Cloud Shell ouverte.

## Ajouter du code pour utiliser le kit de développement logiciel (SDK) Azure AI Speech

> **Conseil** : lorsque vous ajoutez du code, veillez à conserver la mise en retrait correcte.

1. Saisissez la commande suivante pour modifier le fichier de code fourni :

    **Python**

    ```
   code speaking-clock.py
    ```

    **C#**

    ```
   code Program.cs
    ```

1. En haut du fichier de code, sous les références de l’espace de noms existantes, recherchez le commentaire **Importer des espaces de noms**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour importer les espaces de noms dont vous aurez besoin pour utiliser le kit de développement logiciel (SDK) Azure AI Speech avec la ressource Azure AI Services dans votre projet Azure AI Foundry :

    **Python**

    ```python
   # Import namespaces
   from dotenv import load_dotenv
   from azure.ai.projects.models import ConnectionType
   from azure.identity import DefaultAzureCredential
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.projects import AIProjectClient
   import azure.cognitiveservices.speech as speech_sdk
    ```

    **C#**

    ```csharp
   // Import namespaces
   using Azure.Identity;
   using Azure.AI.Projects;
   using Microsoft.CognitiveServices.Speech;
   using Microsoft.CognitiveServices.Speech.Audio;
    ```

1. Dans la fonction **main**, sous le commentaire **Obtenir les paramètres de configuration**, notez que le code charge la chaîne de connexion du projet et l’emplacement que vous avez définis dans le fichier de configuration.

1. Ajoutez le code suivant sous le commentaire **Obtenir le point de terminaison et la clé AI Speech du projet** :

    **Python**

    ```python
   # Get AI Services key from the project
   project_client = AIProjectClient.from_connection_string(
        conn_str=project_connection,
        credential=DefaultAzureCredential())

   ai_svc_connection = project_client.connections.get_default(
      connection_type=ConnectionType.AZURE_AI_SERVICES,
      include_credentials=True, 
    )

   ai_svc_key = ai_svc_connection.key

    ```

    **C#**

    ```csharp
   // Get AI Services key from the project
   var projectClient = new AIProjectClient(project_connection,
                        new DefaultAzureCredential());

   ConnectionResponse aiSvcConnection = projectClient.GetConnectionsClient().GetDefaultConnection(ConnectionType.AzureAIServices, true);

   var apiKeyAuthProperties = aiSvcConnection.Properties as ConnectionPropertiesApiKeyAuth;

   var aiSvcKey = apiKeyAuthProperties.Credentials.Key;
    ```

    Ce code se connecte à votre projet Azure AI Foundry, obtient sa ressource connectée par défaut à AI Services et récupère la clé d’authentification nécessaire pour l’utiliser.

1. Sous le commentaire **Configurer le service Speech**, ajoutez le code suivant pour utiliser la clé AI Services et la région de votre projet pour configurer votre connexion au point de terminaison Azure AI Services Speech.

   **Python**

    ```python
   # Configure speech service
   speech_config = speech_sdk.SpeechConfig(ai_svc_key, location)
   print('Ready to use speech service in:', speech_config.region)
    ```

    **C#**

    ```csharp
   // Configure speech service
   speechConfig = SpeechConfig.FromSubscription(aiSvcKey, location);
   Console.WriteLine("Ready to use speech service in " + speechConfig.Region);
    ```

1. Enregistrez vos modifications (*Ctrl+S*), mais laissez l’éditeur de code ouvert.

## Exécuter l’application

Jusqu’à présent, l’application ne fait rien d’autre que de se connecter à votre projet Azure AI Foundry pour récupérer les détails nécessaires à l’utilisation du service Speech, mais il est utile de l’exécuter et de vérifier qu’elle fonctionne avant d’ajouter des fonctionnalités vocales.

1. Dans la ligne de commande sous l’éditeur de code, entrez la commande Azure CLI suivante pour déterminer le compte Azure connecté pour la session :

    ```
   az account show
    ```

    La sortie JSON résultante doit inclure les détails de votre compte Azure et l’abonnement dans lequel vous travaillez (qui doit être le même abonnement dans lequel vous avez créé votre projet Azure AI Foundry).

    Votre application utilise les informations d’identification Azure pour le contexte dans lequel elle est exécutée pour authentifier la connexion à votre projet. Dans un environnement de production, l’application peut être configurée pour s’exécuter à l’aide d’une identité managée. Dans cet environnement de développement, elle utilisera vos informations d’identification de session Cloud Shell authentifiées.

    > **Note** : vous pouvez vous connecter à Azure dans votre environnement de développement à l’aide de la commande Azure CLI `az login`. Dans ce cas, Cloud Shell s’est déjà connecté à l’aide des informations d’identification Azure avec lesquelles vous vous êtes connecté au portail ; il n’est donc pas nécessaire de se connecter explicitement. Pour en savoir plus sur l’utilisation d’Azure CLI pour s’authentifier auprès d’Azure, consultez [S’authentifier auprès d’Azure à l’aide d’Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli).

1. Dans la ligne de commande, entrez la commande spécifique au langage suivante pour exécuter l’application d’horloge parlante :

    **Python**

    ```
   python speaking-clock.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. Si vous utilisez C#, vous pouvez ignorer tous les avertissements concernant l’utilisation de l’opérateur **await** dans les méthodes asynchrones. Nous les corrigerons ultérieurement. Le code doit afficher la région de la ressource du service Speech que l’application utilisera. Une exécution réussie indique que l’application s’est connectée à votre projet Azure AI Foundry et a récupéré la clé dont elle a besoin pour utiliser le service Azure AI Speech.

## Ajouter du code pour reconnaître la voix

Maintenant que vous disposez d’un objet **SpeechConfig** pour le service Speech dans votre ressource Azure AI Services, vous pouvez utiliser l’API **Reconnaissance vocale** pour reconnaître la voix et la transcrire en texte.

Dans cette procédure, l’entrée vocale est capturée à partir d’un fichier audio, que vous pouvez lire ici :

<video controls src="media/Time.mp4" title="Quelle heure est-il ?" width="150"></video>

1. Dans la fonction **Main**, notez que le code utilise la fonction **TranscribeCommand** pour accepter l’entrée parlée. Puis, dans la fonction **TranscribeCommand**, sous le commentaire **Configurer la reconnaissance vocale**, ajoutez le code approprié ci-dessous pour créer un client **SpeechRecognizer** qui peut être utilisé pour reconnaître et transcrire du contenu vocal à partir d’un fichier audio :

    **Python**

    ```python
   # Configure speech recognition
   current_dir = os.getcwd()
   audioFile = current_dir + '/time.wav'
   audio_config = speech_sdk.AudioConfig(filename=audioFile)
   speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    ```

    **C#**

    ```csharp
   // Configure speech recognition
   string audioFile = "time.wav";
   using AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
   using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    ```

1. Dans la fonction **TranscribeCommand**, sous le commentaire **Traiter l’entrée vocale**, ajoutez le code suivant pour écouter l’entrée parlée, en étant prudent de ne pas remplacer le code à la fin de la fonction qui retourne la commande :

    **Python**

    ```python
   # Process speech input
   print("Listening...")
   speech = speech_recognizer.recognize_once_async().get()
   if speech.reason == speech_sdk.ResultReason.RecognizedSpeech:
       command = speech.text
       print(command)
   else:
       print(speech.reason)
       if speech.reason == speech_sdk.ResultReason.Canceled:
           cancellation = speech.cancellation_details
           print(cancellation.reason)
           print(cancellation.error_details)
    ```

    **C#**

    ```csharp
   // Process speech input
   Console.WriteLine("Listening...");
   SpeechRecognitionResult speech = await speechRecognizer.RecognizeOnceAsync();
   if (speech.Reason == ResultReason.RecognizedSpeech)
   {
       command = speech.Text;
       Console.WriteLine(command);
   }
   else
   {
       Console.WriteLine(speech.Reason);
       if (speech.Reason == ResultReason.Canceled)
       {
           var cancellation = CancellationDetails.FromResult(speech);
           Console.WriteLine(cancellation.Reason);
           Console.WriteLine(cancellation.ErrorDetails);
       }
   }
    ```

1. Enregistrez vos modifications (*Ctrl+S*), puis, dans la ligne de commande sous l’éditeur de code, entrez la commande suivante pour exécuter le programme :

    **Python**

    ```
   python speaking-clock.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. Passez en revue la sortie de l’application, qui doit « entendre » correctement la parole dans le fichier audio et renvoyer une réponse appropriée (notez que votre Azure Cloud Shell peut s’exécuter sur un serveur qui se trouve dans un autre fuseau horaire que le vôtre).

    > **Conseil** : si SpeechRecognizer rencontre une erreur, il génère un résultat « Annulé ». Le code de l’application affiche ensuite le message d’erreur. La cause la plus probable est une valeur de région incorrecte dans le fichier de configuration.

## Synthèse vocale

Votre application d’horloge parlante accepte l’entrée parlée, mais elle ne parle pas. Nous allons résoudre ce problème en ajoutant du code pour synthétiser du contenu vocal.

Une fois de plus, en raison des limitations matérielles de Cloud Shell, nous dirigerons la sortie vocale synthétisée vers un fichier.

1. Dans la fonction **Main** de votre programme, notez que le code utilise la fonction **TellTime** pour indiquer l’heure actuelle à l’utilisateur.
1. Dans la fonction **TellTime**, sous le commentaire **Configurer la synthèse vocale**, ajoutez le code suivant pour créer un client **SpeechSynthesizer** qui peut être utilisé pour générer la sortie parlée :

    **Python**

    ```python
   # Configure speech synthesis
   output_file = "output.wav"
   speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
   audio_config = speech_sdk.audio.AudioConfig(filename=output_file)
   speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config,)
    ```

    **C#**

    ```csharp
   // Configure speech synthesis
   var outputFile = "output.wav";
   speechConfig.SpeechSynthesisVoiceName = "en-GB-RyanNeural";
   using var audioConfig = AudioConfig.FromWavFileOutput(outputFile);
   using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig, audioConfig);
    ```

1. Dans la fonction **TellTime**, sous le commentaire **Synthétiser la sortie parlée**, ajoutez le code suivant pour générer la sortie parlée, en étant prudent de ne pas remplacer le code à la fin de la fonction qui imprime la réponse :

    **Python**

    ```python
   # Synthesize spoken output
   speak = speech_synthesizer.speak_text_async(response_text).get()
   if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
       print(speak.reason)
   else:
       print("Spoken output saved in " + outputFile)
    ```

    **C#**

    ```csharp
   // Synthesize spoken output
   SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(responseText);
   if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
   {
       Console.WriteLine(speak.Reason);
   }
   else
   {
       Console.WriteLine("Spoken output saved in " + outputFile);
   }
    ```

1. Enregistrez vos modifications (*Ctrl+S*), puis, dans la ligne de commande sous l’éditeur de code, entrez la commande suivante pour exécuter le programme :

   **Python**

    ```
   python speaking-clock.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. Passez en revue la sortie de l’application, qui doit indiquer que la sortie parlée a été enregistrée dans un fichier.
1. Si vous disposez d’un lecteur multimédia capable de lire des fichiers audio .wav, dans la barre d’outils du volet Cloud Shell, utilisez le bouton **Charger/Télécharger des fichiers** pour télécharger le fichier audio à partir de votre dossier d’application, puis lisez-le :

    **Python**

    /home/*user*`/mslearn-ai-language/Labfiles/07b-speech/Python/speaking-clock/output.wav`

    **C#**

    /home/*user*`/mslearn-ai-language/Labfiles/07b-speech/C-Sharp/speaking-clock/output.wav`

    Le fichier doit ressembler à ceci :

    <video controls src="./media/Output.mp4" title="L’heure est 2:15." width="150"></video>

## Utiliser le langage de balisage de synthèse vocale

Le langage SSML (Speech Synthesis Markup Language) vous permet de personnaliser la façon dont vos paroles sont synthétisées à l’aide d’un format XML.

1. Dans la fonction **TellTime**, remplacez tout le code actuel sous le commentaire **Synthétiser la sortie parlée** par le code suivant (laissez le code sous le commentaire **Imprimer la réponse**) :

    **Python**

    ```python
   # Synthesize spoken output
   responseSsml = " \
       <speak version='1.0' xmlns='http://www.w3.org/2001/10/synthesis' xml:lang='en-US'> \
           <voice name='en-GB-LibbyNeural'> \
               {} \
               <break strength='weak'/> \
               Time to end this lab! \
           </voice> \
       </speak>".format(response_text)
   speak = speech_synthesizer.speak_ssml_async(responseSsml).get()
   if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
       print(speak.reason)
   else:
       print("Spoken output saved in " + outputFile)
    ```

   **C#**

    ```csharp
   // Synthesize spoken output
   string responseSsml = $@"
       <speak version='1.0' xmlns='http://www.w3.org/2001/10/synthesis' xml:lang='en-US'>
           <voice name='en-GB-LibbyNeural'>
               {responseText}
               <break strength='weak'/>
               Time to end this lab!
           </voice>
       </speak>";
   SpeechSynthesisResult speak = await speechSynthesizer.SpeakSsmlAsync(responseSsml);
   if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
   {
       Console.WriteLine(speak.Reason);
   }
   else
   {
        Console.WriteLine("Spoken output saved in " + outputFile);
   }
    ```

1. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **speaking-clock** et entrez la commande suivante pour exécuter le programme :

    **Python**

    ```
   python speaking-clock.py
    ```

    **C#**

    ```
   dotnet run
    ```

1. Passez en revue la sortie de l’application, qui doit indiquer que la sortie parlée a été enregistrée dans un fichier.
1. Une fois de plus, si vous disposez d’un lecteur multimédia capable de lire des fichiers audio .wav, dans la barre d’outils du volet Cloud Shell, utilisez le bouton **Charger/Télécharger des fichiers** pour télécharger le fichier audio à partir de votre dossier d’application, puis lisez-le :

    **Python**

    /home/*user*`/mslearn-ai-language/Labfiles/07b-speech/Python/speaking-clock/output.wav`

    **C#**

    /home/*user*`/mslearn-ai-language/Labfiles/07b-speech/C-Sharp/speaking-clock/output.wav`

    Le fichier doit ressembler à ceci :
    
    <video controls src="./media/Output2.mp4" title="L’heure est 5:30. Heure de fin de ce labo." width="150"></video>

## Nettoyage

Si vous avez terminé d’explorer Azure AI Speech, vous devez supprimer les ressources que vous avez créées dans cet exercice pour éviter d’entraîner des coûts Azure inutiles.

1. Revenez à l’onglet du navigateur contenant le portail Azure (ou ouvrez à nouveau le [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com` dans un nouvel onglet de navigateur) et affichez le contenu du groupe de ressources dans lequel vous avez déployé les ressources utilisées dans cet exercice.
1. Dans la barre d’outils, sélectionnez **Supprimer le groupe de ressources**.
1. Entrez le nom du groupe de ressources et confirmez que vous souhaitez le supprimer.

## Que se passe-t-il si vous avez un micro et un haut-parleur ?

Dans cet exercice, vous avez utilisé des fichiers audio pour l’entrée vocale et la sortie. Voyons comment le code peut être modifié pour utiliser du matériel audio.

### Utilisation de la reconnaissance vocale avec un microphone

Si vous avez un micro, vous pouvez utiliser le code suivant pour capturer l’entrée parlée pour la reconnaissance vocale :

**Python**

```python
# Configure speech recognition
audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
print('Speak now...')

# Process speech input
speech = speech_recognizer.recognize_once_async().get()
if speech.reason == speech_sdk.ResultReason.RecognizedSpeech:
    command = speech.text
    print(command)
else:
    print(speech.reason)
    if speech.reason == speech_sdk.ResultReason.Canceled:
        cancellation = speech.cancellation_details
        print(cancellation.reason)
        print(cancellation.error_details)

```

**C#**

```csharp
// Configure speech recognition
using AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
Console.WriteLine("Speak now...");

SpeechRecognitionResult speech = await speechRecognizer.RecognizeOnceAsync();
if (speech.Reason == ResultReason.RecognizedSpeech)
{
    command = speech.Text;
    Console.WriteLine(command);
}
else
{
    Console.WriteLine(speech.Reason);
    if (speech.Reason == ResultReason.Canceled)
    {
        var cancellation = CancellationDetails.FromResult(speech);
        Console.WriteLine(cancellation.Reason);
        Console.WriteLine(cancellation.ErrorDetails);
    }
}
```

> **Note** : le microphone par défaut du système est l’entrée audio par défaut. Vous pouvez donc également simplement omettre l’AudioConfig complètement.

### Utilisation de la synthèse vocale avec un haut-parleur

Si vous avez un haut-parleur, vous pouvez utiliser le code suivant pour synthétiser la voix.

**Python**

```python
response_text = 'The time is {}:{:02d}'.format(now.hour,now.minute)

# Configure speech synthesis
speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
audio_config = speech_sdk.audio.AudioOutputConfig(use_default_speaker=True)
speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config)

# Synthesize spoken output
speak = speech_synthesizer.speak_text_async(response_text).get()
if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
    print(speak.reason)
```

**C#**

```csharp
var now = DateTime.Now;
string responseText = "The time is " + now.Hour.ToString() + ":" + now.Minute.ToString("D2");

// Configure speech synthesis
speechConfig.SpeechSynthesisVoiceName = "en-GB-RyanNeural";
using var audioConfig = AudioConfig.FromDefaultSpeakerOutput();
using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig, audioConfig);

// Synthesize spoken output
SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(responseText);
if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
{
    Console.WriteLine(speak.Reason);
}
```

> **Note** : le haut-parleur par défaut du système est la sortie audio par défaut. Vous pouvez donc également simplement omettre l’AudioConfig complètement.

## Plus d’informations

Pour plus d’informations sur l’utilisation des API **Reconnaissance vocale** et **Synthèse vocale**, consultez les articles [Documentation sur la reconnaissance vocale](https://learn.microsoft.com/azure/ai-services/speech-service/index-speech-to-text) et [Documentation sur la synthèse vocale](https://learn.microsoft.com/azure/ai-services/speech-service/index-text-to-speech).
