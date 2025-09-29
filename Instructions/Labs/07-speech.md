---
lab:
  title: Reconnaître et synthétiser du contenu vocal
  description: Mettre en place une horloge parlante qui convertit la parole en texte et le texte en parole.
---

# Reconnaître et synthétiser du contenu vocal

**Azure AI Speech** est un service qui fournit des fonctionnalités vocales, notamment les suivantes :

- Une API de *reconnaissance vocale* qui vous permet d’implémenter la reconnaissance vocale (conversion de mots parlés audibles en texte).
- Une API de *synthèse vocale* qui vous permet d’implémenter la synthèse vocale (conversion de texte en parole audible).

Dans cet exercice, vous allez utiliser ces deux API pour implémenter une application d’horloge parlante.

Bien que cet exercice soit basé sur Python, vous pouvez développer des applications vocales à l’aide de plusieurs kits SDK spécifiques à une langue, notamment :

- [Kit de développement logiciel (SDK) Azure AI Speech pour Python](https://pypi.org/project/azure-cognitiveservices-speech/)
- [Kit de développement logiciel (SDK) Azure AI Speech pour .NET](https://www.nuget.org/packages/Microsoft.CognitiveServices.Speech)
- [Kit de développement logiciel (SDK) Azure AI Speech pour JavaScript](https://www.npmjs.com/package/microsoft-cognitiveservices-speech-sdk)

Cet exercice prend environ **30** minutes.

> **NOTE** : cet exercice est conçu pour être effectué dans Azure Cloud Shell, où l’accès direct au matériel audio de votre ordinateur n’est pas pris en charge. Le labo utilise donc des fichiers audio pour les flux d’entrée vocale et de sortie. Le code permettant d’obtenir les mêmes résultats à l’aide d’un micro et d’un haut-parleur est fourni pour votre référence.

## Créer une ressource Azure AI Speech

Commençons par créer une ressource Azure AI Speech.

1. Ouvrez le [portail Azure](https://portal.azure.com) à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.
1. Dans le champ de recherche situé en haut de la page, recherchez le **service Speech**. Sélectionnez-le dans la liste, puis sélectionnez **Créer**.
1. Configurez la ressource avec les paramètres suivants :
    - **Abonnement** : *votre abonnement Azure*.
    - **Groupe de ressources** : *créez ou sélectionnez un groupe de ressources*.
    - **Région** : *choisissez n’importe quelle région disponible*.
    - **Nom** : *entrez un nom unique.*
    - **Niveau tarifaire** : sélectionnez **F0** (*gratuit*). Si cette option n’est pas disponible, sélectionnez **S** (*standard*).
1. Sélectionnez **Vérifier + créer**, puis **Créer** pour provisionner la ressource.
1. Attendez la fin du déploiement, puis accédez à la ressource déployée.
1. Affichez la page **Point de terminaison et clés** dans la section **Gestion des ressources**. Plus loin dans l’exercice, vous aurez besoin des informations disponibles sur cette page.

## Préparer et configurer l’application d’horloge parlante

1. Tout en laissant la page **Clés et point de terminaison** ouverte, utilisez le bouton **[\>_]** à droite de la barre de recherche en haut de la page pour créer un Cloud Shell dans le portail Azure, en sélectionnant un environnement ***PowerShell***. Cloud Shell fournit une interface de ligne de commande dans un volet situé en bas du portail Azure.

    > **Remarque** : si vous avez déjà créé un Cloud Shell qui utilise un environnement *Bash*, basculez-le vers ***PowerShell***.

1. Dans la barre d’outils Cloud Shell, dans le menu **Paramètres**, sélectionnez **Accéder à la version classique** (cela est nécessaire pour utiliser l’éditeur de code).

    **<font color="red">Assurez-vous d’avoir basculé vers la version classique du Cloud Shell avant de continuer.</font>**

1. Dans le volet PowerShell, entrez les commandes suivantes pour cloner le référentiel GitHub pour cet exercice :

    ```
   rm -r mslearn-ai-language -f
   git clone https://github.com/microsoftlearning/mslearn-ai-language
    ```

    > **Conseil** : lorsque vous entrez des commandes dans le Cloud Shell, la sortie peut occuper une grande partie de la mémoire tampon d’écran. Vous pouvez effacer le contenu de l’écran en saisissant la commande `cls` pour faciliter le focus sur chaque tâche.

1. Une fois le référentiel cloné, accédez au dossier contenant les fichiers de code de l’application d’horloge parlante :  

    ```
   cd mslearn-ai-language/Labfiles/07-speech/Python/speaking-clock
    ```

1. Dans le volet de ligne de commande, exécutez la commande suivante pour afficher les fichiers de code dans le dossier **speaking-clock** :

    ```
   ls -a -l
    ```

    Les fichiers incluent un fichier de configuration (**.env**) et un fichier de code (**speaking-clock.py**). Les fichiers audio que votre application utilisera se trouvent dans le sous-dossier **audio**.

1. Créez un environnement virtuel Python et installez le package du kit de développement logiciel (SDK) Azure AI Speech, ainsi que les autres packages requis en exécutant la commande suivante :

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-cognitiveservices-speech==1.42.0
    ```

1. Entrez la commande suivante pour modifier le fichier de configuration :

    ```
   code .env
    ```

    Le fichier s’ouvre dans un éditeur de code.

1. Mettez à jour les valeurs de configuration pour inclure la **région** et une **clé** de la ressource Azure AI Speech que vous avez créée (disponible dans la page **Clés et point de terminaison** de votre ressource Azure AI Traducteur dans le portail Azure).
1. Une fois que vous avez remplacé les espaces réservés, utilisez la commande **Ctrl+S** pour enregistrer vos modifications, puis utilisez la commande **Ctrl+Q** pour fermer l’éditeur de code tout en gardant la ligne de commande Cloud Shell ouverte.

## Ajouter du code pour utiliser le kit de développement logiciel (SDK) Azure AI Speech

> **Conseil** : lorsque vous ajoutez du code, veillez à conserver la mise en retrait correcte.

1. Saisissez la commande suivante pour modifier le fichier de code fourni :

    ```
   code speaking-clock.py
    ```

1. En haut du fichier de code, sous les références de l’espace de noms existantes, recherchez le commentaire **Importer des espaces de noms**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour importer les espaces de noms dont vous aurez besoin pour utiliser le kit de développement logiciel (SDK) d’Azure AI Speech :

    ```python
   # Import namespaces
   from azure.core.credentials import AzureKeyCredential
   import azure.cognitiveservices.speech as speech_sdk
    ```

1. Dans la fonction **Main**, sous le commentaire **Obtenir les paramètres de configuration**, notez que le code charge la clé et la région que vous avez définies dans le fichier de configuration.

1. Recherchez le commentaire **Configurer le service Speech** et ajoutez le code suivant pour utiliser la clé AI Services et votre région afin de configurer votre connexion au point de terminaison Azure AI Services Speech :

    ```python
   # Configure speech service
   speech_config = speech_sdk.SpeechConfig(speech_key, speech_region)
   print('Ready to use speech service in:', speech_config.region)
    ```

1. Enregistrez vos modifications (*Ctrl+S*), mais laissez l’éditeur de code ouvert.

## Exécuter l’application

Jusqu’à présent, l’application ne fait rien d’autre que de se connecter à votre service Azure AI Speech. Toutefois, il convient de l’exécuter et de vérifier qu’elle fonctionne avant d’ajouter des fonctionnalités vocales.

1. Dans la ligne de commande, entrez la commande suivante pour exécuter l’application d’horloge parlante :

    ```
   python speaking-clock.py
    ```

    Le code doit afficher la région de la ressource du service Speech que l’application utilisera. Une exécution réussie indique que l’application s’est connectée à votre ressource Azure AI Speech.

## Ajouter du code pour reconnaître la voix

Maintenant que vous disposez d’un objet **SpeechConfig** pour le service Speech dans votre ressource Azure AI Services, vous pouvez utiliser l’API **Reconnaissance vocale** pour reconnaître la voix et la transcrire en texte.

Dans cette procédure, l’entrée vocale est capturée à partir d’un fichier audio, que vous pouvez lire ici :

<video controls src="https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Instructions/media/Time.mp4" title="Quelle heure est-il ?" width="150"></video>

1. Dans le fichier de code, notez que le code utilise la fonction **TranscribeCommand** pour accepter l’entrée vocale. Puis, dans la fonction **TranscribeCommand**, rechercher le commentaire **Configurer la reconnaissance vocale**, et ajoutez le code approprié ci-dessous pour créer un client **SpeechRecognizer** qui peut être utilisé pour reconnaître et transcrire du contenu vocal à partir d’un fichier audio :

    ```python
   # Configure speech recognition
   current_dir = os.getcwd()
   audioFile = current_dir + '/time.wav'
   audio_config = speech_sdk.AudioConfig(filename=audioFile)
   speech_recognizer = speech_sdk.SpeechRecognizer(speech_config, audio_config)
    ```

1. Dans la fonction **TranscribeCommand**, sous le commentaire **Traiter l’entrée vocale**, ajoutez le code suivant pour écouter l’entrée parlée, en étant prudent de ne pas remplacer le code à la fin de la fonction qui retourne la commande :

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

1. Enregistrez vos modifications (*CTRL+S*), puis, dans la ligne de commande sous l’éditeur de code, réexécutez le programme :
1. Vérifiez le résultat, qui doit « entendre » correctement la parole dans le fichier audio et renvoyer une réponse appropriée (notez que votre Azure Cloud Shell peut s’exécuter sur un serveur qui se trouve dans un autre fuseau horaire que le vôtre !).

    > **Conseil** : si SpeechRecognizer rencontre une erreur, il génère un résultat « Annulé ». Le code de l’application affiche ensuite le message d’erreur. La cause la plus probable est une valeur de région incorrecte dans le fichier de configuration.

## Synthèse vocale

Votre application d’horloge parlante accepte l’entrée parlée, mais elle ne parle pas. Nous allons résoudre ce problème en ajoutant du code pour synthétiser du contenu vocal.

Une fois de plus, en raison des limitations matérielles de Cloud Shell, nous dirigerons la sortie vocale synthétisée vers un fichier.

1. Dans le fichier de code, notez que le code utilise la fonction **TellTime** pour indiquer à l’utilisateur l’heure actuelle.
1. Dans la fonction **TellTime**, sous le commentaire **Configurer la synthèse vocale**, ajoutez le code suivant pour créer un client **SpeechSynthesizer** qui peut être utilisé pour générer la sortie parlée :

    ```python
   # Configure speech synthesis
   output_file = "output.wav"
   speech_config.speech_synthesis_voice_name = "en-GB-RyanNeural"
   audio_config = speech_sdk.audio.AudioConfig(filename=output_file)
   speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config,)
    ```

1. Dans la fonction **TellTime**, sous le commentaire **Synthétiser la sortie parlée**, ajoutez le code suivant pour générer la sortie parlée, en étant prudent de ne pas remplacer le code à la fin de la fonction qui imprime la réponse :

    ```python
   # Synthesize spoken output
   speak = speech_synthesizer.speak_text_async(response_text).get()
   if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
   else:
        print("Spoken output saved in " + output_file)
    ```

1. Enregistrez vos modifications (*CTRL+S*) et réexécutez le programme, ce qui doit indiquer que la sortie vocale a été enregistrée dans un fichier.

1. Si vous disposez d’un lecteur multimédia capable de lire les fichiers audio .wav, téléchargez le fichier généré en entrant la commande suivante :

    ```
   download ./output.wav
    ```

    La commande de téléchargement crée un lien contextuel en bas à droite de votre navigateur, que vous pouvez sélectionner pour télécharger et ouvrir le fichier.

    Le fichier doit ressembler à ceci :

    <video controls src="https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Instructions/media/Output.mp4" title="L’heure est 2:15." width="150"></video>

## Utiliser le langage de balisage de synthèse vocale

Le langage SSML (Speech Synthesis Markup Language) vous permet de personnaliser la façon dont vos paroles sont synthétisées à l’aide d’un format XML.

1. Dans la fonction **TellTime**, remplacez tout le code actuel sous le commentaire **Synthétiser la sortie parlée** par le code suivant (laissez le code sous le commentaire **Imprimer la réponse**) :

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
       print("Spoken output saved in " + output_file)
    ```

1. Enregistrez vos modifications et réexécutez le programme, ce qui doit à nouveau indiquer que la sortie vocale a été enregistrée dans un fichier.
1. Téléchargez et lisez le fichier généré, qui doit ressembler à ceci :
    
    <video controls src="https://github.com/MicrosoftLearning/mslearn-ai-language/raw/refs/heads/main/Instructions/media/Output2.mp4" title="L’heure est 5:30. Heure de fin de ce labo." width="150"></video>

## Nettoyage

Si vous avez terminé d’explorer Azure AI Speech, vous devez supprimer les ressources que vous avez créées dans cet exercice pour éviter d’entraîner des coûts Azure inutiles.

1. Fermez le volet Azure Cloud Shell.
1. Dans le portail Azure, accédez à la ressource Azure AI Speech que vous avez créée dans cette activité.
1. Dans la page de la ressource, sélectionnez **Supprimer** et suivez les instructions pour supprimer la ressource.

## Que se passe-t-il si vous avez un micro et un haut-parleur ?

Dans cet exercice, l’environnement Azure Cloud Shell que nous avons utilisé ne prend pas en charge le matériel audio. Vous avez donc utilisé des fichiers audio pour l’entrée et la sortie vocales. Voyons comment le code peut être modifié pour utiliser du matériel audio si vous en avez à disposition.

### Utilisation de la reconnaissance vocale avec un microphone

Si vous avez un micro, vous pouvez utiliser le code suivant pour capturer l’entrée parlée pour la reconnaissance vocale :

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

> **Note** : le microphone par défaut du système est l’entrée audio par défaut. Vous pouvez donc également simplement omettre l’AudioConfig complètement.

### Utilisation de la synthèse vocale avec un haut-parleur

Si vous avez un haut-parleur, vous pouvez utiliser le code suivant pour synthétiser la voix.

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

> **Note** : le haut-parleur par défaut du système est la sortie audio par défaut. Vous pouvez donc également simplement omettre l’AudioConfig complètement.

## Plus d’informations

Pour plus d’informations sur l’utilisation des API **Reconnaissance vocale** et **Synthèse vocale**, consultez les articles [Documentation sur la reconnaissance vocale](https://learn.microsoft.com/azure/ai-services/speech-service/index-speech-to-text) et [Documentation sur la synthèse vocale](https://learn.microsoft.com/azure/ai-services/speech-service/index-text-to-speech).
