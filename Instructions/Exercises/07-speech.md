---
lab:
  title: Reconnaître et synthétiser du contenu vocal
  module: Module 4 - Create speech-enabled apps with Azure AI services
---

# Reconnaître et synthétiser du contenu vocal

**Azure AI Speech** est un service qui fournit des fonctionnalités vocales, notamment les suivantes :

- Une API de *reconnaissance vocale* qui vous permet d’implémenter la reconnaissance vocale (conversion de mots parlés audibles en texte).
- Une API de *synthèse vocale* qui vous permet d’implémenter la synthèse vocale (conversion de texte en parole audible).

Dans cet exercice, vous allez utiliser ces deux API pour implémenter une application d’horloge parlante.

> **REMARQUE** : cet exercice nécessite que vous utilisiez un ordinateur avec des haut-parleurs ou des écouteurs. Pour une expérience optimale, un microphone est également nécessaire. Certains environnements virtuels hébergés peuvent être en mesure de capturer l’audio à partir de votre microphone local, mais si cela ne fonctionne pas (ou si vous n’avez pas de microphone du tout), vous pouvez utiliser un fichier audio fourni pour l’entrée vocale. Suivez attentivement les instructions, car vous devez choisir différentes options selon que vous utilisez un microphone ou le fichier audio.

## Configurer une ressource *Azure AI Speech*

Si vous n’en avez pas encore dans votre abonnement, vous devez configurer une ressource **Azure AI Speech**.

1. Ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.
1. Dans le champ de recherche en haut, recherchez **Services Azure AI** et appuyez sur **Entrée**. Dans les résultats, sélectionnez **Créer** sous **Service Speech**.
1. Créez une ressource avec les paramètres suivants :
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *sélectionnez ou créez un groupe de ressources*.
    - **Région** : *choisissez n’importe quelle région disponible*
    - **Nom** : *Entrez un nom unique.*
    - **Niveau tarifaire** : sélectionnez **F0** (*gratuit*). Si cette option n’est pas disponible, sélectionnez **S** (*standard*).
    - **Mention sur l’IA responsable** : J’accepte.
1. Sélectionnez **Revoir + créer**.
1. Attendez la fin du déploiement, puis accédez à la ressource déployée.
1. Consultez la page **Clés et points de terminaison**. Vous aurez besoin des informations de cette page plus loin dans l’exercice.

## Préparer le développement d’une application dans Visual Studio Code

Vous allez développer votre application vocale à l’aide de Visual Studio Code. Les fichiers de code de votre application ont été fournis dans un référentiel GitHub.

> **Conseil** : si vous avez déjà cloné le référentiel **mslearn-ai-language**, ouvrez-le dans Visual Studio Code. Dans le cas contraire, procédez comme suit pour le cloner dans votre environnement de développement.

1. Démarrez Visual Studio Code.
1. Ouvrez la palette (Maj+CTRL+P) et exécutez une commande **Git : Cloner** pour cloner le référentiel `https://github.com/MicrosoftLearning/mslearn-ai-language` vers un dossier local (peu importe quel dossier).
1. Lorsque le référentiel a été cloné, ouvrez le dossier dans Visual Studio Code.
1. Attendez que des fichiers supplémentaires soient installés pour prendre en charge les projets de code C# dans le référentiel.

    > **Remarque** : si vous êtes invité à ajouter des ressources requises pour générer et déboguer, sélectionnez **Not Now** (Pas maintenant).

## Configuration de votre application

Des applications pour C# et Python sont fournies. Les deux applications présentent les mêmes fonctionnalités. Premièrement, vous allez terminer certaines parties clés de l’application pour lui permettre d’utiliser votre ressource Azure AI Speech.

1. Dans Visual Studio Code, dans le volet **Explorateur**, accédez au dossier **Labfiles/07-speech** et développez le dossier **CSharp** ou **Python** en fonction de votre préférence de langage, ainsi que le dossier **speaking-clock** qu’il contient. Chaque dossier contient les fichiers de code propres au langage d’une application dans laquelle vous allez intégrer des fonctionnalités Azure AI Speech.
1. Cliquez avec le bouton droit sur le dossier **speaking-clock**, qui contient vos fichiers de code, et ouvrez un terminal intégré. Installez ensuite le package du kit de développement logiciel (SDK) d’Azure AI Speech en exécutant la commande appropriée en fonction de votre préférence de langage :

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.30.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-speech==1.30.0
    ```

1. Dans le volet **Explorateur**, dans le dossier **speaking-clock**, ouvrez le fichier de configuration correspondant à votre langage préféré.

    - **C#** : appsettings.json
    - **Python** : .env

1. Mettez à jour les valeurs de configuration de sorte à inclure une **région** et une **clé** de la ressource Azure AI Speech que vous avez créée (disponible sur la page **Clés et point de terminaison** de votre ressource Azure AI Speech dans le Portail Azure).

    > **REMARQUE** : veillez à ajouter la *région* de votre ressource, <u>et non</u> le point de terminaison !

1. Enregistrez le fichier de configuration.

## Ajouter du code pour utiliser le kit de développement logiciel (SDK) Azure AI Speech

1. Notez que le dossier **speaking-clock** contient un fichier de code pour l’application cliente :

    - **C#** : Program.cs
    - **Python** : speaking-clock.py

    Ouvrez le fichier de code et, en haut, sous les références d’espace de noms existantes, recherchez le commentaire **Importer des espaces de noms**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour importer les espaces de noms dont vous aurez besoin pour utiliser le kit de développement logiciel (SDK) d’Azure AI Speech :

    **C#** : Program.cs

    ```csharp
    // Import namespaces
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    ```

    **Python** : speaking-clock.py

    ```python
    # Import namespaces
    import azure.cognitiveservices.speech as speech_sdk
    ```

1. Dans la fonction **Main**, notez que le code pour charger la clé et la région du service à partir du fichier de configuration a déjà été fourni. Vous devez utiliser ces variables pour créer un objet **SpeechConfig** pour votre ressource Azure AI Speech. Ajoutez le code suivant sous le commentaire **Configurer le service Speech** :

    **C#** : Program.cs

    ```csharp
    // Configure speech service
    speechConfig = SpeechConfig.FromSubscription(aiSvcKey, aiSvcRegion);
    Console.WriteLine("Ready to use speech service in " + speechConfig.Region);
    
    // Configure voice
    speechConfig.SpeechSynthesisVoiceName = "en-US-AriaNeural";
    ```

    **Python** : speaking-clock.py

    ```python
    # Configure speech service
    speech_config = speech_sdk.SpeechConfig(ai_key, ai_region)
    print('Ready to use speech service in:', speech_config.region)
    ```

1. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **speaking-clock** et entrez la commande suivante pour exécuter le programme :

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. Si vous utilisez C#, vous pouvez ignorer tous les avertissements concernant l’utilisation de l’opérateur **await** dans les méthodes asynchrones. Nous les corrigerons ultérieurement. Le code doit afficher la région de la ressource du service Speech que l’application utilisera.

## Ajouter du code pour reconnaître la voix

Maintenant que vous disposez d’un objet **SpeechConfig** pour le service de reconnaissance vocale dans votre ressource Azure AI Speech, vous pouvez utiliser l’API **Reconnaissance vocale** pour reconnaître le discours audible et le transcrire en texte.

> **IMPORTANT** : cette section inclut des instructions pour deux procédures différentes. Suivez la première procédure si vous avez un microphone et qu’il fonctionne. Suivez la deuxième procédure si vous souhaitez simuler une entrée vocale à l’aide d’un fichier audio.

### Si vous disposez d’un microphone de travail

1. Dans la fonction **Main** de votre programme, notez que le code utilise la fonction **TranscribeCommand** pour accepter l’entrée parlée.
1. Dans la fonction **TranscribeCommand**, sous le commentaire **Configurer la reconnaissance vocale**, ajoutez le code approprié ci-dessous pour créer un client **SpeechRecognizer** qui peut être utilisé pour reconnaître et transcrire du contenu vocal à l’aide du microphone système par défaut :

    **C#**

    ```csharp
    // Configure speech recognition
    using AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    Console.WriteLine("Speak now...");
    ```

    **Python**

    ```python
    # Configure speech recognition
    audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
    speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    print('Speak now...')
    ```

1. Passez maintenant à la section **Ajouter du code pour traiter la commande transcrite** ci-dessous.

---

### Vous pouvez également utiliser l’entrée audio à partir d’un fichier

1. Dans la fenêtre de terminal, entrez la commande suivante pour installer une bibliothèque que vous pouvez utiliser pour lire le fichier audio :

    **C#**

    ```
    dotnet add package System.Windows.Extensions --version 4.6.0 
    ```

    **Python**

    ```
    pip install playsound==1.2.2
    ```

1. Dans le fichier de code de votre programme, sous les importations d’espaces de noms existants, ajoutez le code suivant pour importer la bibliothèque que vous venez d’installer :

    **C#** : Program.cs

    ```csharp
    using System.Media;
    ```

    **Python** : speaking-clock.py

    ```python
    from playsound import playsound
    ```

1. Dans la fonction **Main**, notez que le code utilise la fonction **TranscribeCommand** pour accepter l’entrée parlée. Puis, dans la fonction **TranscribeCommand**, sous le commentaire **Configurer la reconnaissance vocale**, ajoutez le code approprié ci-dessous pour créer un client **SpeechRecognizer** qui peut être utilisé pour reconnaître et transcrire du contenu vocal à partir d’un fichier audio :

    **C#** : Program.cs

    ```csharp
    // Configure speech recognition
    string audioFile = "time.wav";
    SoundPlayer wavPlayer = new SoundPlayer(audioFile);
    wavPlayer.Play();
    using AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
    using SpeechRecognizer speechRecognizer = new SpeechRecognizer(speechConfig, audioConfig);
    ```

    **Python** : speaking-clock.py

    ```python
    # Configure speech recognition
    current_dir = os.getcwd()
    audioFile = current_dir + '\\time.wav'
    playsound(audioFile)
    audio_config = speech_sdk.AudioConfig(filename=audioFile)
    speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    ```

---

### Ajouter du code pour traiter la commande transcrite

1. Dans la fonction **TranscribeCommand**, sous le commentaire **Traiter l’entrée vocale**, ajoutez le code suivant pour écouter l’entrée parlée, en étant prudent de ne pas remplacer le code à la fin de la fonction qui retourne la commande :

    **C#** : Program.cs

    ```csharp
    // Process speech input
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

    **Python** : speaking-clock.py

    ```python
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

1. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **speaking-clock** et entrez la commande suivante pour exécuter le programme :

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. Si vous utilisez un microphone, parlez clairement et dites « quelle heure est-il ? ». Le programme doit transcrire votre entrée parlée et afficher l’heure (en fonction de l’heure locale de l’ordinateur sur lequel le code est en cours d’exécution, qui peut ne pas être l’heure exacte de votre emplacement).

    SpeechRecognizer vous donne environ 5 secondes pour parler. S’il ne détecte aucune entrée parlée, il génère un résultat « Aucune correspondance ».

    Si SpeechRecognizer rencontre une erreur, il génère un résultat « Annulé ». Le code de l’application affiche ensuite le message d’erreur. La cause la plus probable est une clé ou une région incorrecte dans le fichier de configuration.

## Synthèse vocale

Votre application d’horloge parlante accepte l’entrée parlée, mais elle ne parle pas. Nous allons résoudre ce problème en ajoutant du code pour synthétiser du contenu vocal.

1. Dans la fonction **Main** de votre programme, notez que le code utilise la fonction **TellTime** pour indiquer l’heure actuelle à l’utilisateur.
1. Dans la fonction **TellTime**, sous le commentaire **Configurer la synthèse vocale**, ajoutez le code suivant pour créer un client **SpeechSynthesizer** qui peut être utilisé pour générer la sortie parlée :

    **C#** : Program.cs

    ```csharp
    // Configure speech synthesis
    speechConfig.SpeechSynthesisVoiceName = "en-GB-RyanNeural";
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    ```

    **Python** : speaking-clock.py

    ```python
    # Configure speech synthesis
    speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    ```

    > **REMARQUE** : la configuration audio standard utilise l’appareil par défaut du système pour la sortie. Vous n’avez donc pas besoin de fournir explicitement un objet **AudioConfig**. Si vous devez rediriger la sortie audio vers un fichier, vous pouvez utiliser un objet **AudioConfig** en indiquant un chemin.

1. Dans la fonction **TellTime**, sous le commentaire **Synthétiser la sortie parlée**, ajoutez le code suivant pour générer la sortie parlée, en étant prudent de ne pas remplacer le code à la fin de la fonction qui imprime la réponse :

    **C#** : Program.cs

    ```csharp
    // Synthesize spoken output
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(responseText);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```

    **Python** : speaking-clock.py

    ```python
    # Synthesize spoken output
    speak = speech_synthesizer.speak_text_async(response_text).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

1. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **speaking-clock** et entrez la commande suivante pour exécuter le programme :

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. Lorsque vous y êtes invité, parlez clairement dans le microphone et dites « quelle heure est-il ? ». Le programme doit parler et vous indiquer l’heure.

## Utiliser une autre voix

Votre application d’horloge parlante utilise une voix par défaut, que vous pouvez modifier. Le service Speech prend en charge une gamme de voix *standard* ainsi que des voix *neuronales* quasi humaines. Vous pouvez également créer des voix *personnalisées*.

> **Remarque** : pour obtenir la liste des voix neuronales et standard, consultez la [galerie de voix](https://speech.microsoft.com/portal/voicegallery) dans Speech Studio.

1. Dans la fonction **TellTime**, sous le commentaire **Configurer la synthèse vocale**, modifiez le code comme suit pour spécifier une autre voix avant de créer le client **SpeechSynthesizer** :

   **C#** : Program.cs

    ```csharp
    // Configure speech synthesis
    speechConfig.SpeechSynthesisVoiceName = "en-GB-LibbyNeural"; // change this
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    ```

    **Python** : speaking-clock.py

    ```python
    # Configure speech synthesis
    speech_config.speech_synthesis_voice_name = 'en-GB-LibbyNeural' # change this
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    ```

1. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **speaking-clock** et entrez la commande suivante pour exécuter le programme :

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. Lorsque vous y êtes invité, parlez clairement dans le microphone et dites « quelle heure est-il ? ». Le programme doit parler dans la voix spécifiée, vous indiquant l’heure.

## Utiliser le langage de balisage de synthèse vocale

Le langage SSML (Speech Synthesis Markup Language) vous permet de personnaliser la façon dont vos paroles sont synthétisées à l’aide d’un format XML.

1. Dans la fonction **TellTime**, remplacez tout le code actuel sous le commentaire **Synthétiser la sortie parlée** par le code suivant (laissez le code sous le commentaire **Imprimer la réponse**) :

   **C#** : Program.cs

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
    ```

    **Python** : speaking-clock.py

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
    ```

1. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **speaking-clock** et entrez la commande suivante pour exécuter le programme :

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock.py
    ```

1. Lorsque vous y êtes invité, parlez clairement dans le microphone et dites « quelle heure est-il ? ». Le programme doit parler dans la voix qui est spécifiée dans le langage SSML (en remplaçant la voix spécifiée dans l’objet SpeechConfig). Il doit vous indiquer l’heure puis, après une pause, vous dire qu’il est temps de conclure ce labo, ce qui est le cas !

## Plus d’informations

Pour plus d’informations sur l’utilisation des API **Reconnaissance vocale** et **Synthèse vocale**, consultez les articles [Documentation sur la reconnaissance vocale](https://learn.microsoft.com/azure/ai-services/speech-service/index-speech-to-text) et [Documentation sur la synthèse vocale](https://learn.microsoft.com/azure/ai-services/speech-service/index-text-to-speech).
