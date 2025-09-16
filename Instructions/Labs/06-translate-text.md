---
lab:
  title: Traduire du texte
  description: "Traduisez le texte fourni entre toutes les langues prises en charge par Azure\_AI\_Traducteur."
---

# Traduire du texte

**Azure AI Traducteur** est un service qui vous permet de traduire du texte d’une langue vers une autre. Dans cet exercice, vous allez l’utiliser pour créer une application simple qui traduit l’entrée dans n’importe quelle langue prise en charge vers la langue cible de votre choix.

Bien que cet exercice soit basé sur Python, vous pouvez développer des applications de traduction de texte à l’aide de plusieurs kits SDK spécifiques à une langue, notamment :

- [Bibliothèque de client Traduction d’IA Azure pour Python](https://pypi.org/project/azure-ai-translation-text/)
- [Bibliothèque de client Traduction Azure AI pour .NET](https://www.nuget.org/packages/Azure.AI.Translation.Text)
- [Bibliothèque de client Traduction d’IA Azure pour JavaScript](https://www.npmjs.com/package/@azure-rest/ai-translation-text)

Cet exercice prend environ **30** minutes.

## Configurer une ressource *Azure AI Traducteur*

Si vous n’en avez pas encore dans votre abonnement, vous devez configurer une ressource **Azure AI Traducteur**.

1. Ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.
1. Dans le champ de recherche en haut de la page, recherchez **Traducteurs**, puis sélectionnez **Traducteurs** dans les résultats.
1. Créez une ressource avec les paramètres suivants :
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *sélectionnez ou créez un groupe de ressources*.
    - **Région** : *choisissez n’importe quelle région disponible*
    - **Nom** : *Entrez un nom unique.*
    - **Niveau tarifaire** : sélectionnez **F0** (*gratuit*). Si cette option n’est pas disponible, sélectionnez **S** (*standard*).
1. Sélectionnez **Vérifier + créer**, puis **Créer** pour provisionner la ressource.
1. Attendez la fin du déploiement, puis accédez à la ressource déployée.
1. Consultez la page **Clés et points de terminaison**. Plus loin dans l’exercice, vous aurez besoin des informations disponibles sur cette page.

## Préparer le développement d’une application dans Cloud Shell

Pour tester les fonctionnalités de traduction de texte d’Azure AI Traducteur, vous devez développer une application console simple dans Azure Cloud Shell.

1. Dans le portail Azure, cliquez sur le bouton **[\>_]** à droite de la barre de recherche, en haut de la page, pour créer un environnement Cloud Shell dans le portail Azure, puis sélectionnez un environnement ***PowerShell***. Cloud Shell fournit une interface de ligne de commande dans un volet situé en bas du portail Azure.

    > **Remarque** : si vous avez déjà créé un Cloud Shell qui utilise un environnement *Bash*, basculez-le vers ***PowerShell***.

1. Dans la barre d’outils Cloud Shell, dans le menu **Paramètres**, sélectionnez **Accéder à la version classique** (cela est nécessaire pour utiliser l’éditeur de code).

    **<font color="red">Assurez-vous d’avoir basculé vers la version classique du Cloud Shell avant de continuer.</font>**

1. Dans le volet PowerShell, entrez les commandes suivantes pour cloner le référentiel GitHub pour cet exercice :

    ```
   rm -r mslearn-ai-language -f
   git clone https://github.com/microsoftlearning/mslearn-ai-language
    ```

    > **Conseil** : lorsque vous entrez des commandes dans le Cloud Shell, la sortie peut occuper une grande partie de la mémoire tampon d’écran. Vous pouvez effacer le contenu de l’écran en saisissant la commande `cls` pour faciliter le focus sur chaque tâche.

1. Une fois le référentiel cloné, accédez au dossier contenant les fichiers de code de l’application :  

    ```
   cd mslearn-ai-language/Labfiles/06-translator-sdk/Python/translate-text
    ```

## Configuration de votre application

1. Dans le volet de ligne de commande, exécutez la commande suivante pour afficher les fichiers de code dans le dossier **translate-text** :

    ```
   ls -a -l
    ```

    Les fichiers incluent un fichier de configuration (**.env**) et un fichier de code (**translate.py**).

1. Créez un environnement virtuel Python et installez le package du kit de développement logiciel (SDK) de traduction Azure AI, ainsi que les autres packages requis en exécutant la commande suivante :

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-translation-text==1.0.1
    ```

1. Entrez la commande suivante pour modifier le fichier de configuration de l’application :

    ```
   code .env
    ```

    Le fichier s’ouvre dans un éditeur de code.

1. Mettez à jour les valeurs de configuration de sorte à inclure une **région** et une **clé** de la ressource Azure AI Traducteur que vous avez créée (disponible sur la page **Clés et point de terminaison** de votre ressource Azure AI Traducteur dans le Portail Azure).

    > **REMARQUE** : veillez à ajouter la *région* de votre ressource, <u>et non</u> le point de terminaison !

1. Une fois que vous avez remplacé les espaces réservés, utilisez la commande **CTRL+S** ou **Clic droit > Enregistrer** dans l’éditeur de code pour enregistrer vos modifications, puis utilisez la commande **CTRL+Q** ou **Clic droit > Quitter** pour fermer l’éditeur tout en gardant la ligne de commande du Cloud Shell ouverte.

## Ajouter le code permettant de traduire du texte

1. Entrez la commande suivante pour modifier le fichier de code de l’application :

    ```
   code translate.py
    ```

1. Passez en revue le code existant. Vous allez ajouter du code pour travailler avec le kit de développement logiciel (SDK) de traduction Azure AI.

    > **Conseil** : lorsque vous ajoutez du code au fichier de code, veillez à conserver la bonne mise en retrait.

1. En haut du fichier de code, sous les références d’espace de noms existantes, recherchez le commentaire **Importer les espaces de noms** et ajoutez le code suivant pour importer les espaces de noms dont vous aurez besoin pour utiliser le Kit de développement logiciel (SDK) de traduction :

    ```python
   # import namespaces
   from azure.core.credentials import AzureKeyCredential
   from azure.ai.translation.text import *
   from azure.ai.translation.text.models import InputTextItem
    ```

1. Dans la fonction **Main**, notez que le code existant lit les paramètres de configuration.
1. Recherchez le commentaire **Create client using endpoint and key** (Créer un client à l’aide du point de terminaison et de la clé), puis ajoutez le code suivant :

    ```python
   # Create client using endpoint and key
   credential = AzureKeyCredential(translatorKey)
   client = TextTranslationClient(credential=credential, region=translatorRegion)
    ```

1. Recherchez le commentaire **Choisir la langue cible** et ajoutez le code suivant, qui utilise le service Traducteur de texte pour renvoyer la liste des langues prises en charge pour la traduction et invite l’utilisateur à sélectionner un code de langue pour la langue cible :

    ```python
   # Choose target language
   languagesResponse = client.get_supported_languages(scope="translation")
   print("{} languages supported.".format(len(languagesResponse.translation)))
   print("(See https://learn.microsoft.com/azure/ai-services/translator/language-support#translation)")
   print("Enter a target language code for translation (for example, 'en'):")
   targetLanguage = "xx"
   supportedLanguage = False
   while supportedLanguage == False:
        targetLanguage = input()
        if  targetLanguage in languagesResponse.translation.keys():
            supportedLanguage = True
        else:
            print("{} is not a supported language.".format(targetLanguage))
    ```

1. Recherchez le commentaire **Traduire le texte** et ajoutez le code suivant, qui invite à plusieurs reprises l’utilisateur à saisir le texte à traduire, utilise le service Azure AI Traducteur pour le traduire dans la langue cible (en détectant automatiquement la langue source) et affiche les résultats jusqu’à ce que l’utilisateur entre quitter :

    ```python
   # Translate text
   inputText = ""
   while inputText.lower() != "quit":
        inputText = input("Enter text to translate ('quit' to exit):")
        if inputText != "quit":
            input_text_elements = [InputTextItem(text=inputText)]
            translationResponse = client.translate(body=input_text_elements, to_language=[targetLanguage])
            translation = translationResponse[0] if translationResponse else None
            if translation:
                sourceLanguage = translation.detected_language
                for translated_text in translation.translations:
                    print(f"'{inputText}' was translated from {sourceLanguage.language} to {translated_text.to} as '{translated_text.text}'.")
    ```

1. Enregistrez vos modifications (CTRL+S), puis entrez la commande suivante pour exécuter le programme (vous pouvez agrandir le volet Cloud Shell et redimensionner les volets pour afficher davantage de texte dans le volet de ligne de commande) :

    ```
   python translate.py
    ```

1. Lorsque le système vous y invite, choisissez une langue cible valide dans la liste affichée.
1. Entrez une expression à traduire (par exemple `This is a test` ou `C'est un test`) et affichez le résultat. La langue source doit être détectée automatiquement et le texte doit être traduit dans la langue cible.
1. Lorsque vous avez terminé, entrez `quit`. Vous pouvez exécuter l’application à nouveau et choisir une autre langue cible.

## Nettoyer les ressources

Si vous avez terminé d’explorer le service Azure AI Traducteur, vous pouvez supprimer les ressources que vous avez créées dans cet exercice. Voici comment procéder :

1. Fermez le volet Azure Cloud Shell.
1. Dans le portail Azure, accédez à la ressource Azure AI Translator que vous avez créée dans cette activité.
1. Dans la page de la ressource, sélectionnez **Supprimer** et suivez les instructions pour supprimer la ressource.

## Plus d’informations

Pour plus d’informations sur l’utilisation d’**Azure AI Translator**, consultez la [documentation d’Azure AI Translator](https://learn.microsoft.com/azure/ai-services/translator/).
