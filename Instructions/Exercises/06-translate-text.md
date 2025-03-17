---
lab:
  title: Traduire du texte
  module: Module 3 - Getting Started with Natural Language Processing
---
{% assign site.title = page.lab.title %}

# Traduire du texte

**Azure AI Traducteur** est un service qui vous permet de traduire du texte d’une langue vers une autre. Dans cet exercice, vous allez l’utiliser pour créer une application simple qui traduit l’entrée dans n’importe quelle langue prise en charge vers la langue cible de votre choix.

## Configurer une ressource *Azure AI Traducteur*

Si vous n’en avez pas encore dans votre abonnement, vous devez configurer une ressource **Azure AI Traducteur**.

1. Ouvrez le portail Azure à l’adresse `https://portal.azure.com` et connectez-vous avec le compte Microsoft associé à votre abonnement Azure.
1. Dans le champ de recherche en haut, recherchez **Services Azure AI** et appuyez sur **Entrée**. Dans les résultats, sélectionnez **Créer** sous **Traducteur**.
1. Créez une ressource avec les paramètres suivants :
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *sélectionnez ou créez un groupe de ressources*.
    - **Région** : *choisissez n’importe quelle région disponible*
    - **Nom** : *Entrez un nom unique.*
    - **Niveau tarifaire** : sélectionnez **F0** (*gratuit*). Si cette option n’est pas disponible, sélectionnez **S** (*standard*).
    - **Mention sur l’IA responsable** : J’accepte.
1. Sélectionnez **Vérifier + créer**, puis **Créer** pour provisionner la ressource.
1. Attendez la fin du déploiement, puis accédez à la ressource déployée.
1. Consultez la page **Clés et points de terminaison**. Vous aurez besoin des informations de cette page plus loin dans l’exercice.

## Préparer le développement d’une application dans Visual Studio Code

Vous allez développer votre application de traduction de texte à l’aide de Visual Studio Code. Les fichiers de code de votre application ont été fournis dans un référentiel GitHub.

> **Conseil** : si vous avez déjà cloné le référentiel **mslearn-ai-language**, ouvrez-le dans Visual Studio Code. Dans le cas contraire, procédez comme suit pour le cloner dans votre environnement de développement.

1. Démarrez Visual Studio Code.
2. Ouvrez la palette (Maj+CTRL+P) et exécutez une commande **Git : Cloner** pour cloner le référentiel `https://github.com/MicrosoftLearning/mslearn-ai-language` vers un dossier local (peu importe quel dossier).
3. Lorsque le référentiel a été cloné, ouvrez le dossier dans Visual Studio Code.

    > **Remarque** : Si Visual Studio Code affiche un message contextuel qui vous invite à approuver le code que vous ouvrez, cliquez sur l’option **Oui, je fais confiance aux auteurs** dans la fenêtre contextuelle.

4. Attendez que des fichiers supplémentaires soient installés pour prendre en charge les projets de code C# dans le référentiel.

    > **Remarque** : si vous êtes invité à ajouter des ressources requises pour générer et déboguer, sélectionnez **Not Now** (Pas maintenant).

## Configuration de votre application

Des applications pour C# et Python sont fournies. Les deux applications présentent les mêmes fonctionnalités. Premièrement, vous allez terminer certaines parties clés de l’application pour lui permettre d’utiliser votre ressource Azure AI Traducteur.

1. Dans Visual Studio Code, dans le volet **Explorateur**, accédez au dossier **Labfiles/06b-translator-sdk** et développez le dossier **CSharp** ou **Python** en fonction de votre préférence de langage, ainsi que le dossier **translate-text** qu’il contient. Chaque dossier contient les fichiers de code propres au langage d’une application dans laquelle vous allez intégrer des fonctionnalités Azure AI Traducteur.
2. Cliquez avec le bouton droit sur le dossier **translate-text**, qui contient vos fichiers de code, et ouvrez un terminal intégré. Installez ensuite le package du kit de développement logiciel (SDK) d’Azure AI Traducteur en exécutant la commande appropriée en fonction de votre préférence de langage :

    **C# :**

    ```
    dotnet add package Azure.AI.Translation.Text --version 1.0.0-beta.1
    ```

    **Python** :

    ```
    pip install azure-ai-translation-text==1.0.0b1
    ```

3. Dans le volet **Explorateur**, dans le dossier **translate-text**, ouvrez le fichier de configuration correspondant à votre langage préféré.

    - **C#** : appsettings.json
    - **Python** : .env
    
4. Mettez à jour les valeurs de configuration de sorte à inclure une **région** et une **clé** de la ressource Azure AI Traducteur que vous avez créée (disponible sur la page **Clés et point de terminaison** de votre ressource Azure AI Traducteur dans le Portail Azure).

    > **REMARQUE** : veillez à ajouter la *région* de votre ressource, <u>et non</u> le point de terminaison !

5. Enregistrez le fichier de configuration.

## Ajouter le code permettant de traduire du texte

Vous pouvez maintenant utiliser Azure AI Traducteur pour traduire du texte.

1. Notez également que le dossier **translate-text** contient un fichier de code pour l’application cliente :

    - **C#** : Program.cs
    - **Python** : translate.py

    Ouvrez le fichier de code et, en haut, sous les références d’espace de noms existantes, recherchez le commentaire **Importer des espaces de noms**. Ensuite, sous ce commentaire, ajoutez le code spécifique au langage suivant pour importer les espaces de noms dont vous aurez besoin pour utiliser le kit de développement logiciel (SDK) d’analyse de texte :

    **C#**  : Programs.cs

    ```csharp
    // import namespaces
    using Azure;
    using Azure.AI.Translation.Text;
    ```

    **Python** : translate.py

    ```python
    # import namespaces
    from azure.ai.translation.text import *
    from azure.ai.translation.text.models import InputTextItem
    ```

1. Dans la fonction **Main**, notez que le code existant lit les paramètres de configuration.
1. Recherchez le commentaire **Create client using endpoint and key** (Créer un client à l’aide du point de terminaison et de la clé), puis ajoutez le code suivant :

    **C#**  : Programs.cs

    ```csharp
    // Create client using endpoint and key
    AzureKeyCredential credential = new(translatorKey);
    TextTranslationClient client = new(credential, translatorRegion);
    ```

    **Python** : translate.py

    ```python
    # Create client using endpoint and key
    credential = TranslatorCredential(translatorKey, translatorRegion)
    client = TextTranslationClient(credential)
    ```

1. Recherchez le commentaire **Choose target language** (Choisir la langue cible) et ajoutez le code suivant, qui utilise le service de traduction de texte pour renvoyer la liste des langues prises en charge pour la traduction, puis invite l’utilisateur à sélectionner le code de la langue cible.

    **C#**  : Programs.cs

    ```csharp
    // Choose target language
    Response<GetLanguagesResult> languagesResponse = await client.GetLanguagesAsync(scope:"translation").ConfigureAwait(false);
    GetLanguagesResult languages = languagesResponse.Value;
    Console.WriteLine($"{languages.Translation.Count} languages available.\n(See https://learn.microsoft.com/azure/ai-services/translator/language-support#translation)");
    Console.WriteLine("Enter a target language code for translation (for example, 'en'):");
    string targetLanguage = "xx";
    bool languageSupported = false;
    while (!languageSupported)
    {
        targetLanguage = Console.ReadLine();
        if (languages.Translation.ContainsKey(targetLanguage))
        {
            languageSupported = true;
        }
        else
        {
            Console.WriteLine($"{targetLanguage} is not a supported language.");
        }

    }
    ```

    **Python** : translate.py

    ```python
    # Choose target language
    languagesResponse = client.get_languages(scope="translation")
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

1. Recherchez le commentaire **Translate text** (Traduire le texte) et ajoutez le code suivant, qui invite l’utilisateur à fournir le texte à traduire, utilise le service Azure AI Traducteur pour le traduire dans la langue cible (la langue source est détectée automatiquement) et affiche le résultat, puis recommence jusqu’à ce que l’utilisateur entre le mot-clé *quit*.

    **C#**  : Programs.cs

    ```csharp
    // Translate text
    string inputText = "";
    while (inputText.ToLower() != "quit")
    {
        Console.WriteLine("Enter text to translate ('quit' to exit)");
        inputText = Console.ReadLine();
        if (inputText.ToLower() != "quit")
        {
            Response<IReadOnlyList<TranslatedTextItem>> translationResponse = await client.TranslateAsync(targetLanguage, inputText).ConfigureAwait(false);
            IReadOnlyList<TranslatedTextItem> translations = translationResponse.Value;
            TranslatedTextItem translation = translations[0];
            string sourceLanguage = translation?.DetectedLanguage?.Language;
            Console.WriteLine($"'{inputText}' translated from {sourceLanguage} to {translation?.Translations[0].To} as '{translation?.Translations?[0]?.Text}'.");
        }
    } 
    ```

    **Python** : translate.py

    ```python
    # Translate text
    inputText = ""
    while inputText.lower() != "quit":
        inputText = input("Enter text to translate ('quit' to exit):")
        if inputText != "quit":
            input_text_elements = [InputTextItem(text=inputText)]
            translationResponse = client.translate(content=input_text_elements, to=[targetLanguage])
            translation = translationResponse[0] if translationResponse else None
            if translation:
                sourceLanguage = translation.detected_language
                for translated_text in translation.translations:
                    print(f"'{inputText}' was translated from {sourceLanguage.language} to {translated_text.to} as '{translated_text.text}'.")
    ```

1. Enregistrez les modifications apportées à votre fichier de code.

## Tester votre application

Votre application est maintenant prête à être testée.

1. Revenez au terminal intégré du dossier **Translate text**, puis entrez la commande suivante pour exécuter le programme :

    - **C#**  : `dotnet run`
    - **Python** : `python translate.py`

    > **Conseil** : vous pouvez utiliser l’icône **Agrandir le volet** (**^**) dans la barre d’outils du terminal pour mieux voir le texte de la console.

1. Lorsque le système vous y invite, choisissez une langue cible valide dans la liste affichée.
1. Entrez une expression à traduire (par exemple `This is a test` ou `C'est un test`) et affichez le résultat. La langue source doit être détectée automatiquement et le texte doit être traduit dans la langue cible.
1. Lorsque vous avez terminé, entrez `quit`. Vous pouvez exécuter l’application à nouveau et choisir une autre langue cible.

## Nettoyage

Quand vous n’avez plus besoin de votre projet, vous pouvez supprimer la ressource Azure AI Traducteur dans le [Portail Azure](https://portal.azure.com).
