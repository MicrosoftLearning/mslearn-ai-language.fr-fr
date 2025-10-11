---
lab:
  title: Traduire du contenu vocal
  description: Convertissez la parole en parole et mettez-la en œuvre dans votre propre application.
---

# Traduire du contenu vocal

Azure AI Speech inclut une API de traduction vocale que vous pouvez utiliser pour traduire un discours oral. Par exemple, supposons que vous souhaitiez développer une application de traduction que des utilisateurs peuvent utiliser lorsqu’ils voyagent dans des pays où ils ne parlent pas la langue locale. Ils seraient en mesure de dire des phrases telles que « Où se trouve la gare ? » ou « J’ai besoin de trouver une pharmacie » dans leur propre langue et de les traduire dans la langue locale. Dans cet exercice, vous allez utiliser le kit de développement logiciel (SDK) Azure AI Speech pour Python pour créer une application simple basée sur cet exemple.

Bien que cet exercice soit basé sur Python, vous pouvez développer des applications de traduction vocale à l’aide de plusieurs kits SDK spécifiques à une langue, notamment :

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

## Préparer le développement d’une application dans Cloud Shell

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

1. Une fois le référentiel cloné, accédez au dossier contenant les fichiers de code :

    ```
   cd mslearn-ai-language/Labfiles/08-speech-translation/Python/translator
    ```

1. Dans le volet de ligne de commande, exécutez la commande suivante pour afficher les fichiers de code dans le dossier **traducteur** :

    ```
   ls -a -l
    ```

    Les fichiers incluent un fichier de configuration (**.env**) et un fichier de code (**translator.py**).

1. Créez un environnement virtuel Python et installez le package du kit de développement logiciel (SDK) Azure AI Speech, ainsi que les autres packages requis en exécutant la commande suivante :

    ```
    python -m venv labenv
    ./labenv/bin/Activate.ps1
    pip install -r requirements.txt azure-cognitiveservices-speech==1.42.0
    ```

1. Saisissez la commande suivante pour modifier le fichier de configuration fourni :

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
   code translator.py
    ```

1. En haut du fichier de code, sous les références de l’espace de noms existantes, recherchez le commentaire **Importer des espaces de noms**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour importer les espaces de noms dont vous aurez besoin pour utiliser le kit de développement logiciel (SDK) d’Azure AI Speech :

    ```python
   # Import namespaces
   from azure.core.credentials import AzureKeyCredential
   import azure.cognitiveservices.speech as speech_sdk
    ```

1. Dans la fonction **Main**, sous le commentaire **Obtenir les paramètres de configuration**, notez que le code charge la clé et la région que vous avez définies dans le fichier de configuration.

1. Recherchez le code suivant sous le commentaire **Configurer la traduction**et ajoutez le code suivant pour configurer votre connexion au point de terminaison Azure AI Services Speech :

    ```python
   # Configure translation
   translation_config = speech_sdk.translation.SpeechTranslationConfig(speech_key, speech_region)
   translation_config.speech_recognition_language = 'en-US'
   translation_config.add_target_language('fr')
   translation_config.add_target_language('es')
   translation_config.add_target_language('hi')
   print('Ready to translate from',translation_config.speech_recognition_language)
    ```

1. Vous utiliserez l’objet **SpeechTranslationConfig** pour traduire la parole en texte, mais vous utiliserez également un objet **SpeechConfig** pour synthétiser des traductions en parole. Ajoutez le code suivant sous le commentaire **Configurer Speech** :

    ```python
   # Configure speech
   speech_config = speech_sdk.SpeechConfig(speech_key, speech_region)
   print('Ready to use speech service in:', speech_config.region)
    ```

1. Enregistrez vos modifications (*Ctrl+S*), mais laissez l’éditeur de code ouvert.

## Exécuter l’application

Jusqu’à présent, l’application ne fait rien d’autre que de se connecter à votre ressource Azure AI Speech. Toutefois, il convient de l’exécuter et de vérifier qu’elle fonctionne avant d’ajouter des fonctionnalités vocales.

1. Dans la ligne de commande, entrez la commande suivante pour exécuter l’application du traducteur.

    ```
   python translator.py
    ```

    Le code doit afficher la région de la ressource du service vocal que l’application utilisera, un message indiquant qu’elle est prête à traduire de l’anglais américain et vous demander de choisir une langue cible. Une exécution réussie indique que l’application s’est connectée à votre service Azure AI Speech. Appuyez sur ENTRÉE pour mettre fin au programme.

## Implémenter la traduction vocale

Maintenant que vous disposez d’un objet **SpeechTranslationConfig** pour le service Azure AI Speech, vous pouvez utiliser l’API de traduction d’Azure AI Speech pour reconnaître et traduire le contenu vocal.

1. Dans le fichier de code, notez que le code utilise la fonction **Traduire** pour traduire les entrées vocales. Puis, dans la fonction **Translate**, sous le commentaire **Traduire du contenu vocal**, ajoutez le code suivant pour créer un client **TranslationRecognizer** qui peut être utilisé pour reconnaître et traduire du contenu vocal à partir d’un fichier.

    ```python
   # Translate speech
   current_dir = os.getcwd()
   audioFile = current_dir + '/station.wav'
   audio_config_in = speech_sdk.AudioConfig(filename=audioFile)
   translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config = audio_config_in)
   print("Getting speech from file...")
   result = translator.recognize_once_async().get()
   print('Translating "{}"'.format(result.text))
   translation = result.translations[targetLanguage]
   print(translation)
    ```

1. Enregistrez vos modifications (*CTRL+S*) et réexécutez le programme :

    ```
   python translator.py
    ```

1. Lorsque vous y êtes invité, entrez un code de langue valide (*fr*, *es* ou *hi*). Le programme doit transcrire votre fichier d’entrée et le traduire dans la langue que vous avez spécifiée (français, espagnol ou hindi). Répétez ce processus, en essayant chaque langue prise en charge par l’application.

    > **REMARQUE** : La traduction en hindi peut ne pas toujours être affichée correctement dans la fenêtre Console en raison de problèmes d’encodage de caractères.

1. Lorsque vous avez terminé, appuyez sur ENTRÉE pour mettre fin au programme.

> **REMARQUE** : le code de votre application traduit l’entrée en trois langues dans un seul appel. Seule la traduction de la langue spécifique est affichée, mais vous pouvez récupérer l’une des traductions en spécifiant le code de la langue cible dans la collection de **traductions** du résultat.

## Synthétiser la traduction en parole

Pour le moment, votre application traduit l’entrée parlée en texte, ce qui peut être suffisant si vous devez demander de l’aide pendant un voyage. Toutefois, il serait préférable d’avoir la traduction parlée à haute voix dans une voix appropriée.

> **Remarque** : en raison des limitations matérielles de Cloud Shell, nous dirigerons la sortie vocale synthétisée vers un fichier.

1. Dans la fonction **Traduire**, recherchez le commentaire **Synthétiser la traduction**, puis ajoutez le code suivant pour utiliser un client **SpeechSynthesizer** afin de synthétiser la traduction sous forme vocale et l’enregistrer en tant que fichier .wav :

    ```python
   # Synthesize translation
   output_file = "output.wav"
   voices = {
            "fr": "fr-FR-HenriNeural",
            "es": "es-ES-ElviraNeural",
            "hi": "hi-IN-MadhurNeural"
   }
   speech_config.speech_synthesis_voice_name = voices.get(targetLanguage)
   audio_config_out = speech_sdk.audio.AudioConfig(filename=output_file)
   speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config_out)
   speak = speech_synthesizer.speak_text_async(translation).get()
   if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
   else:
        print("Spoken output saved in " + output_file)
    ```

1. Enregistrez vos modifications (*CTRL+S*) et réexécutez le programme :

    ```
   python translator.py
    ```

1. Vérifiez le résultat de l’application, qui devrait indiquer que la traduction de la sortie vocale a été enregistrée dans un fichier. Lorsque vous avez terminé, appuyez sur **ENTRÉE** pour mettre fin au programme.
1. Si vous disposez d’un lecteur multimédia capable de lire les fichiers audio .wav, téléchargez le fichier généré en entrant la commande suivante :

    ```
   download ./output.wav
    ```

    La commande de téléchargement crée un lien contextuel en bas à droite de votre navigateur, que vous pouvez sélectionner pour télécharger et ouvrir le fichier.

> **REMARQUE : **
> *dans cet exemple, vous avez utilisé un objet **SpeechTranslationConfig** pour traduire de la parole en texte, puis utilisé un objet **SpeechConfig** afin de synthétiser la traduction en parole. De fait, vous pouvez utiliser l’objet **SpeechTranslationConfig** pour synthétiser directement la traduction, mais cela ne fonctionne uniquement que lorsque vous traduisez vers une seule langue, et retourne un flux audio généralement enregistré en tant que fichier.*

## Nettoyer les ressources

Si vous avez terminé d’explorer le service Azure AI Speech, vous pouvez supprimer les ressources que vous avez créées dans cet exercice. Voici comment procéder :

1. Fermez le volet Azure Cloud Shell.
1. Dans le portail Azure, accédez à la ressource Azure AI Speech que vous avez créée dans cette activité.
1. Dans la page de la ressource, sélectionnez **Supprimer** et suivez les instructions pour supprimer la ressource.

## Que se passe-t-il si vous avez un micro et un haut-parleur ?

Dans cet exercice, l’environnement Azure Cloud Shell que nous avons utilisé ne prend pas en charge le matériel audio. Vous avez donc utilisé des fichiers audio pour l’entrée et la sortie vocales. Voyons comment le code peut être modifié pour utiliser du matériel audio si vous en avez à disposition.

### Utilisation de la traduction vocale avec un microphone

1. Si vous avez un micro, vous pouvez utiliser le code suivant pour capturer l’entrée parlée pour la traduction vocale :

    ```python
   # Translate speech
   audio_config_in = speech_sdk.AudioConfig(use_default_microphone=True)
   translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config = audio_config_in)
   print("Speak now...")
   result = translator.recognize_once_async().get()
   print('Translating "{}"'.format(result.text))
   translation = result.translations[targetLanguage]
   print(translation)
    ```

> **Note** : le microphone par défaut du système est l’entrée audio par défaut. Vous pouvez donc également simplement omettre l’AudioConfig complètement.

### Utilisation de la synthèse vocale avec un haut-parleur

1. Si vous avez un haut-parleur, vous pouvez utiliser le code suivant pour synthétiser la voix.
    
    ```python
   # Synthesize translation
   voices = {
            "fr": "fr-FR-HenriNeural",
            "es": "es-ES-ElviraNeural",
            "hi": "hi-IN-MadhurNeural"
   }
   speech_config.speech_synthesis_voice_name = voices.get(targetLanguage)
   audio_config_out = speech_sdk.audio.AudioOutputConfig(use_default_speaker=True)
   speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config, audio_config_out)
   speak = speech_synthesizer.speak_text_async(translation).get()
   if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

> **Note** : le haut-parleur par défaut du système est la sortie audio par défaut. Vous pouvez donc également simplement omettre l’AudioConfig complètement.

## Plus d’informations

Pour plus d’informations sur l’utilisation de l’API de traduction d’Azure AI Speech, consultez la [documentation correspondante](https://learn.microsoft.com/azure/ai-services/speech-service/speech-translation).
