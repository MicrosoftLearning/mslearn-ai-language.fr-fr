---
lab:
  title: Traduire du contenu vocal
  module: Module 8 - Translate speech with Azure AI Speech
---

# Traduire du contenu vocal

Azure AI Speech inclut une API de traduction vocale que vous pouvez utiliser pour traduire un discours oral. Par exemple, supposons que vous souhaitiez développer une application de traduction que des utilisateurs peuvent utiliser lorsqu’ils voyagent dans des pays où ils ne parlent pas la langue locale. Ils seraient en mesure de dire des phrases telles que « Où se trouve la gare ? » ou « J’ai besoin de trouver une pharmacie » dans leur propre langue et de les traduire dans la langue locale.

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
1. Consultez la page **Clés et points de terminaison**. Plus loin dans l’exercice, vous aurez besoin des informations disponibles sur cette page.

## Préparer le développement d’une application dans Visual Studio Code

Vous allez développer votre application vocale à l’aide de Visual Studio Code. Les fichiers de code de votre application ont été fournis dans un référentiel GitHub.

> **Conseil** : si vous avez déjà cloné le référentiel **mslearn-ai-language**, ouvrez-le dans Visual Studio Code. Dans le cas contraire, procédez comme suit pour le cloner dans votre environnement de développement.

1. Démarrez Visual Studio Code.
1. Ouvrez la palette (Maj+CTRL+P) et exécutez une commande **Git : Cloner** pour cloner le référentiel `https://github.com/MicrosoftLearning/mslearn-ai-language` vers un dossier local (peu importe quel dossier).
1. Lorsque le référentiel a été cloné, ouvrez le dossier dans Visual Studio Code.
1. Attendez que des fichiers supplémentaires soient installés pour prendre en charge les projets de code C# dans le référentiel.

    > **Remarque** : Si vous êtes invité à ajouter des ressources requises pour générer et déboguer, sélectionnez **Not Now** (Pas maintenant).

## Configuration de votre application

Des applications pour C# et Python sont fournies. Les deux applications présentent les mêmes fonctionnalités. Premièrement, vous allez terminer certaines parties clés de l’application pour lui permettre d’utiliser votre ressource Azure AI Speech.

1. Dans Visual Studio Code, dans le volet **Explorateur**, accédez au dossier **Labfiles/08-speech-translation** et développez le dossier **CSharp** ou **Python** en fonction de votre préférence de langage, ainsi que le dossier **translator** qu’il contient. Chaque dossier contient les fichiers de code propres au langage d’une application dans laquelle vous allez intégrer des fonctionnalités Azure AI Speech.
1. Cliquez avec le bouton droit sur le dossier **translator**, qui contient vos fichiers de code, et ouvrez un terminal intégré. Installez ensuite le package du kit de développement logiciel (SDK) d’Azure AI Speech en exécutant la commande appropriée en fonction de votre préférence de langage :

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.30.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-speech==1.30.0
    ```

1. Dans le volet **Explorateur**, dans le dossier **translator**, ouvrez le fichier de configuration correspondant à votre langage préféré.

    - **C#** : appsettings.json
    - **Python** : .env

1. Mettez à jour les valeurs de configuration de sorte à inclure une **région** et une **clé** de la ressource Azure AI Speech que vous avez créée (disponible sur la page **Clés et point de terminaison** de votre ressource Azure AI Speech dans le Portail Azure).

    > **REMARQUE** : veillez à ajouter la *région* de votre ressource, <u>et non</u> le point de terminaison !

1. Enregistrez le fichier de configuration.

## Ajouter du code pour utiliser le kit de développement logiciel (SDK) Speech

1. Notez également que le dossier **translator** contient un fichier de code pour l’application cliente :

    - **C#** : Program.cs
    - **Python** : translator.py

    Ouvrez le fichier de code et, en haut, sous les références d’espace de noms existantes, recherchez le commentaire **Importer des espaces de noms**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour importer les espaces de noms dont vous aurez besoin pour utiliser le kit de développement logiciel (SDK) d’Azure AI Speech :

    **C#** : Program.cs

    ```csharp
    // Import namespaces
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    using Microsoft.CognitiveServices.Speech.Translation;
    ```

    **Python** : translator.py

    ```python
    # Import namespaces
    import azure.cognitiveservices.speech as speech_sdk
    ```

1. Dans la fonction **Main**, notez que le code pour charger la clé et la région du service Azure AI Speech à partir du fichier de configuration a déjà été fourni. Vous devez utiliser ces variables pour créer un objet **SpeechTranslationConfig** pour votre ressource Azure AI Speech, que vous utiliserez pour traduire des entrées orales. Ajoutez le code suivant sous le commentaire **Configurer la traduction** :

    **C#** : Program.cs

    ```csharp
    // Configure translation
    translationConfig = SpeechTranslationConfig.FromSubscription(aiSvcKey, aiSvcRegion);
    translationConfig.SpeechRecognitionLanguage = "en-US";
    translationConfig.AddTargetLanguage("fr");
    translationConfig.AddTargetLanguage("es");
    translationConfig.AddTargetLanguage("hi");
    Console.WriteLine("Ready to translate from " + translationConfig.SpeechRecognitionLanguage);
    ```

    **Python** : translator.py

    ```python
    # Configure translation
    translation_config = speech_sdk.translation.SpeechTranslationConfig(ai_key, ai_region)
    translation_config.speech_recognition_language = 'en-US'
    translation_config.add_target_language('fr')
    translation_config.add_target_language('es')
    translation_config.add_target_language('hi')
    print('Ready to translate from',translation_config.speech_recognition_language)
    ```

1. Vous utiliserez l’objet **SpeechTranslationConfig** pour traduire la parole en texte, mais vous utiliserez également un objet **SpeechConfig** pour synthétiser des traductions en parole. Ajoutez le code suivant sous le commentaire **Configurer Speech** :

    **C#** : Program.cs

    ```csharp
    // Configure speech
    speechConfig = SpeechConfig.FromSubscription(aiSvcKey, aiSvcRegion);
    ```

    **Python** : translator.py

    ```python
    # Configure speech
    speech_config = speech_sdk.SpeechConfig(ai_key, ai_region)
    ```

1. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **translator** et entrez la commande suivante pour exécuter le programme :

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python translator.py
    ```

1. Si vous utilisez C#, vous pouvez ignorer tous les avertissements concernant l’utilisation de l’opérateur **await** dans les méthodes asynchrones. Nous les corrigerons ultérieurement. Le code doit afficher un message indiquant que le programme est prêt à traduire à partir de la langue en-US et de fournir le résultat en langue cible. Appuyez sur ENTRÉE pour mettre fin au programme.

## Implémenter la traduction vocale

Maintenant que vous disposez d’un objet **SpeechTranslationConfig** pour le service Azure AI Speech, vous pouvez utiliser l’API de traduction d’Azure AI Speech pour reconnaître et traduire le contenu vocal.

> **IMPORTANT** : cette section inclut des instructions pour deux procédures différentes. Suivez la première procédure si vous avez un microphone et qu’il fonctionne. Suivez la deuxième procédure si vous souhaitez simuler une entrée vocale à l’aide d’un fichier audio.

### Si vous disposez d’un microphone de travail

1. Dans la fonction **Main** de votre programme, notez que le code utilise la fonction **Translate** pour traduire l’entrée parlée.
1. Dans la fonction **Translate**, sous le commentaire **Traduire du contenu vocal**, ajoutez le code suivant pour créer un client **TranslationRecognizer** qui peut être utilisé pour reconnaître et traduire du contenu vocal à l’aide du microphone système par défaut pour l’entrée.

    **C#** : Program.cs

    ```csharp
    // Translate speech
    using AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    using TranslationRecognizer translator = new TranslationRecognizer(translationConfig, audioConfig);
    Console.WriteLine("Speak now...");
    TranslationRecognitionResult result = await translator.RecognizeOnceAsync();
    Console.WriteLine($"Translating '{result.Text}'");
    translation = result.Translations[targetLanguage];
    Console.OutputEncoding = Encoding.UTF8;
    Console.WriteLine(translation);
    ```

    **Python** : translator.py

    ```python
    # Translate speech
    audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
    translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config = audio_config)
    print("Speak now...")
    result = translator.recognize_once_async().get()
    print('Translating "{}"'.format(result.text))
    translation = result.translations[targetLanguage]
    print(translation)
    ```

    > **REMARQUE** : le code de votre application traduit l’entrée en trois langues dans un seul appel. Seule la traduction de la langue spécifique est affichée, mais vous pouvez récupérer l’une des traductions en spécifiant le code de la langue cible dans la collection de **traductions** du résultat.

1. Passez maintenant à la section **Exécuter le programme** ci-dessous.

---

### Vous pouvez également utiliser l’entrée audio à partir d’un fichier

1. Dans la fenêtre de terminal, entrez la commande suivante pour installer une bibliothèque que vous pouvez utiliser pour lire le fichier audio :

    **C#** : Program.cs

    ```csharp
    dotnet add package System.Windows.Extensions --version 4.6.0 
    ```

    **Python** : translator.py

    ```python
    pip install playsound==1.3.0
    ```

1. Dans le fichier de code de votre programme, sous les importations d’espaces de noms existants, ajoutez le code suivant pour importer la bibliothèque que vous venez d’installer :

    **C#** : Program.cs

    ```csharp
    using System.Media;
    ```

    **Python** : translator.py

    ```python
    from playsound import playsound
    ```

1. Dans la fonction **Main** de votre programme, notez que le code utilise la fonction **Translate** pour traduire l’entrée parlée. Puis, dans la fonction **Translate**, sous le commentaire **Traduire du contenu vocal**, ajoutez le code suivant pour créer un client **TranslationRecognizer** qui peut être utilisé pour reconnaître et traduire du contenu vocal à partir d’un fichier.

    **C#** : Program.cs

    ```csharp
    // Translate speech
    string audioFile = "station.wav";
    SoundPlayer wavPlayer = new SoundPlayer(audioFile);
    wavPlayer.Play();
    using AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
    using TranslationRecognizer translator = new TranslationRecognizer(translationConfig, audioConfig);
    Console.WriteLine("Getting speech from file...");
    TranslationRecognitionResult result = await translator.RecognizeOnceAsync();
    Console.WriteLine($"Translating '{result.Text}'");
    translation = result.Translations[targetLanguage];
    Console.OutputEncoding = Encoding.UTF8;
    Console.WriteLine(translation);
    ```

    **Python** : translator.py

    ```python
    # Translate speech
    audioFile = 'station.wav'
    playsound(audioFile)
    audio_config = speech_sdk.AudioConfig(filename=audioFile)
    translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config = audio_config)
    print("Getting speech from file...")
    result = translator.recognize_once_async().get()
    print('Translating "{}"'.format(result.text))
    translation = result.translations[targetLanguage]
    print(translation)
    ```

---

### Exécuter le programme

1. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **translator** et entrez la commande suivante pour exécuter le programme :

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python translator.py
    ```

1. Lorsque vous y êtes invité, entrez un code de langue valide (*fr*, *es* ou *hi*), puis, si vous utilisez un microphone, parlez distinctement et dites « où se trouve la gare ? » ou une autre phrase que vous pourriez utiliser lors d’un voyage à l’étranger. Le programme doit transcrire votre entrée parlée et la traduire dans la langue que vous avez spécifiée (français, espagnol ou hindi). Répétez ce processus, en essayant chaque langue prise en charge par l’application. Lorsque vous avez terminé, appuyez sur ENTRÉE pour terminer le programme.

    TranslationRecognizer vous donne environ 5 secondes pour parler. S’il ne détecte aucune entrée parlée, il génère un résultat « Aucune correspondance ». La traduction en hindi peut ne pas toujours être affichée correctement dans la fenêtre Console en raison de problèmes d’encodage de caractères.

> **REMARQUE** : le code de votre application traduit l’entrée en trois langues dans un seul appel. Seule la traduction de la langue spécifique est affichée, mais vous pouvez récupérer l’une des traductions en spécifiant le code de la langue cible dans la collection de **traductions** du résultat.

## Synthétiser la traduction en parole

Pour le moment, votre application traduit l’entrée parlée en texte, ce qui peut être suffisant si vous devez demander de l’aide pendant un voyage. Toutefois, il serait préférable d’avoir la traduction parlée à haute voix dans une voix appropriée.

1. Dans la fonction **Translate**, sous le commentaire **Synthétiser la traduction**, ajoutez le code suivant pour utiliser un client **SpeechSynthesizer** afin de synthétiser la traduction en tant que parole par le biais du haut-parleur par défaut :

    **C#** : Program.cs

    ```csharp
    // Synthesize translation
    var voices = new Dictionary<string, string>
                    {
                        ["fr"] = "fr-FR-HenriNeural",
                        ["es"] = "es-ES-ElviraNeural",
                        ["hi"] = "hi-IN-MadhurNeural"
                    };
    speechConfig.SpeechSynthesisVoiceName = voices[targetLanguage];
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(translation);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```

    **Python** : translator.py

    ```python
    # Synthesize translation
    voices = {
            "fr": "fr-FR-HenriNeural",
            "es": "es-ES-ElviraNeural",
            "hi": "hi-IN-MadhurNeural"
    }
    speech_config.speech_synthesis_voice_name = voices.get(targetLanguage)
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    speak = speech_synthesizer.speak_text_async(translation).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

1. Enregistrez vos modifications et revenez au terminal intégré pour le dossier **translator** et entrez la commande suivante pour exécuter le programme :

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python translator.py
    ```

1. Lorsque vous y êtes invité, entrez un code de langue valide (*fr*, *es* ou *hi*), puis parlez distinctement dans le microphone et dites une phrase que vous pourriez utiliser lors d’un voyage à l’étranger. Le programme doit transcrire votre entrée parlée et répondre avec une traduction parlée. Répétez ce processus, en essayant chaque langue prise en charge par l’application. Lorsque vous avez terminé, appuyez sur **ENTRÉE** pour mettre fin au programme.

> **REMARQUE**
> *Dans cet exemple, vous avez utilisé un objet **SpeechTranslationConfig** pour traduire de la parole en texte, puis un objet **SpeechConfig** afin de synthétiser la traduction en parole. De fait, vous pouvez utiliser l’objet **SpeechTranslationConfig** pour synthétiser directement la traduction, mais cela ne fonctionne que lorsque vous traduisez vers une seule langue, et retourne un flux audio généralement enregistré en tant que fichier, au lieu de l’envoyer vers un haut-parleur.*

## Plus d’informations

Pour plus d’informations sur l’utilisation de l’API de traduction d’Azure AI Speech, consultez la [documentation correspondante](https://learn.microsoft.com/azure/ai-services/speech-service/speech-translation).
