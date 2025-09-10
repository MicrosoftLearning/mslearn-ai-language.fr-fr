---
lab:
  title: "Explorer l’API Voice\_Live"
  description: "Découvrez comment utiliser et personnaliser l’API Voice\_Live disponible dans le terrain de jeu Azure\_AI\_Foundry."
---

# Explorer l’API Voice Live

Dans cet exercice, vous créez un agent dans Azure AI Foundry et explorez l’API Voice Live dans le terrain de jeu Speech. 

Cet exercice dure environ **30** minutes.

> <span style="color:red">**Remarque** :</span> certaines des technologies utilisées dans cet exercice sont actuellement en aperçu ou en cours de développement. Il se peut que vous rencontriez des comportements, inattendus, des avertissements ou des erreurs.

> <span style="color:red">**Remarque** :</span> cet exercice est conçu pour être réalisé dans un environnement de navigateur disposant d’un accès direct au microphone de votre ordinateur. Bien que les concepts puissent être explorés dans Azure Cloud Shell, les fonctionnalités vocales interactives nécessitent un accès local au matériel audio.

## Créer un projet Azure AI Foundry

Commençons par créer un projet Azure AI Foundry.

1. Dans un navigateur web, ouvrez le [portail Azure AI Foundry](https://ai.azure.com) à l’adresse `https://ai.azure.com` et connectez-vous en utilisant vos informations d’identification Azure. Fermez les conseils ou les volets de démarrage rapide ouverts la première fois que vous vous connectez et, si nécessaire, utilisez le logo **Azure AI Foundry** en haut à gauche pour accéder à la page d’accueil, qui ressemble à l’image suivante (fermez le volet **Aide** s’il est ouvert) :

    ![Capture d’écran de la page d’accueil d’Azure AI Foundry avec l’option de création d’un assistant sélectionné.](../media/ai-foundry-new-home-page.png)

1. Sur la page d’accueil, sélectionnez **Créer un agent**.

1. Dans l’Assistant **Création d’un assistant**, entrez un nom valide pour votre assistant. 

1. Sélectionnez **Options avancées** et définissez les paramètres suivants :
    - **Ressource Azure AI Foundry** : *conservez le nom par défaut*
    - **Abonnement** : *votre abonnement Azure*
    - **Groupe de ressources** : *créez ou sélectionnez un groupe de ressources*
    - **Région** : sélectionnez aléatoirement une région parmi les options suivantes :\*
        - USA Est 2
        - Suède Centre

    > \* Au moment de l’écriture, l’API Voice Live est uniquement prise en charge dans les régions répertoriées précédemment. La sélection d’un emplacement de manière aléatoire permet de s’assurer qu’une seule région n’est pas submergée par le trafic et vous permet d’avoir une expérience plus fluide. En cas de limites de service atteintes, vous devrez peut-être créer un autre projet dans une autre région.

1. Vérifiez vos configurations, puis sélectionnez **Créer**. Attendez que le processus de configuration se termine.

    >**Remarque** : si vous recevez une erreur d’autorisation, sélectionnez le bouton **Corriger** pour ajouter les autorisations appropriées pour continuer.

1. Lorsque votre projet est créé, vous êtes redirigé par défaut vers le terrain de jeu des assistants dans le portail Azure AI Foundry, dont l’interface sera similaire à l’image suivante :

    ![Capture d’écran des détails d’un projet Azure AI dans le portail Azure AI Foundry.](../media/ai-foundry-project-2.png)

## Démarrer un exemple Voice Live

 Dans cette section de l’exercice, vous interagissez avec l’un des assistants. 

1. Dans le volet de navigation, sélectionnez **Terrains de jeu**.

1. Recherchez le groupe **Terrains de jeu Speech**, puis sélectionnez le bouton **Essayer le terrain de jeu Speech**.

1. Le terrain de jeu Speech offre de nombreuses options prédéfinies. Utilisez la barre de défilement horizontale pour atteindre la fin de la liste et sélectionnez la vignette **Voice Live**. 

    ![Capture d’écran de la vignette Voice Live.](../media/voice-live-tile.png)

1. Sélectionnez l’échantillon d’assistant **Conversation informelle** dans le volet **Essayer avec des échantillons**.

1. Vérifiez que votre microphone et vos haut-parleurs fonctionnent, puis sélectionnez le bouton **Démarrer** en bas de la page. 

    Pendant votre interaction avec l’assistant, notez que vous pouvez l’interrompre ; il marquera alors une pause pour écouter. Essayez de parler en marquant des pauses de différentes longueurs entre les mots et les phrases. Notez a rapidité avec laquelle l’assistant reconnaît ces pauses et relance la conversation. Lorsque vous avez terminé, sélectionnez le bouton **Terminer**.

1. Démarrez les autres assistants échantillons pour explorer leur comportement.

    En explorant les différents assistants, notez les modifications apportées dans la section **Instructions de réponse** du volet **Configuration**.

## Configurer l’agent 

Dans cette section, vous modifiez la voix de l’assistant et ajoutez un avatar à l’assistant **Conversation informelle**. Le volet **Configuration** est divisé en trois sections : **IA générative**, **Speech** et **Avatar**.

>**Remarque :** si vous modifiez ou interagissez avec l’une des options de configuration, vous devez sélectionner le bouton **Appliquer** en bas du volet **Configuration** pour activer l’assistant.

Sélectionnez l’assistant **Conversation informelle**. Ensuite, modifiez la voix de l’assistant et ajoutez un avatar en suivant ces instructions :

1. Sélectionnez **> Speech** pour développer la section et accéder aux options.

1. Sélectionnez le menu déroulant de l’option **Voix** et choisissez une voix différente.

1. Sélectionnez **Appliquer** pour enregistrer vos modifications, puis **Démarrer** pour lancer l’assistant et entendre votre changement.

    Répétez les étapes précédentes afin d’essayer plusieurs voix différentes. Passez à l’étape suivante lorsque vous avez terminé la sélection vocale.

1. Sélectionnez **> Avatar** pour développer la section et accéder aux options.

1. Activez le bouton bascule pour activer la fonctionnalité et sélectionnez l’un des avatars. 

1. Sélectionnez **Appliquer** pour enregistrer vos modifications, puis **Démarrez** pour lancer l’assistant. 

    Notez l’animation et la synchronisation de l’avatar avec l’audio.

1. Développez la section **> IA générative** et placez le bouton bascule **Engagement proactif** sur la position désactivée. Ensuite, sélectionnez **Appliquer** pour enregistrer vos modifications, puis **Démarrer** pour lancer l’assistant.

    Avec l’option **Engagement proactif** désactivée, l’assistant n’initie pas la conversation. Demandez à l’assistant : « Pouvez-vous me dire ce que vous faites ? » pour démarrer la conversation.

>**Conseil :** vous pouvez sélectionner **Rétablir les valeurs par défaut** puis **Appliquer** pour que l’assistant retrouve son comportement par défaut.

Lorsque vous avez terminé, passez à la section suivante.

## Créer un assistant vocal

Dans cette section, vous allez créer votre propre assistant vocal à partir de zéro.

1. Sélectionnez **Démarrer à partir d’un champ vide** dans la section **Essayer avec votre propre** section du volet. 

1. Développez la section **> IA générative** du volet **Configuration**.

1. Sélectionnez le menu déroulant **Modèle d’IA générative** et choisissez le modèle **GPT-4o Mini Realtime**.

1. Ajoutez le texte suivant dans la section **Instructions de réponse**.

    ```
    You are a voice agent named "Ava" who acts as a friendly car rental agent. 
    ```

1. Définissez le curseur **Température de réponse** sur une valeur de **0,8**. 

1. Placez le bouton bascule **Engagement proactif** sur la position activée.

1. Sélectionnez **Appliquer** pour enregistrer vos modifications, puis **Démarrez** pour lancer l’assistant.

    L’assistant va se présenter et demander comment il peut vous aider aujourd’hui. Demandez à l’assistant « Avez-vous des berlines disponibles à la location jeudi ? » Notez combien de temps l’assistant doit répondre. Posez d’autres questions à l’assistant pour évaluer ses réponses. Lorsque vous avez terminé, passez à l’étape suivante.

1. Développez la section **> Speech** du volet **Configuration**.

1. Placez le bouton bascule **Fin d’énoncé (EOU)** sur la position **activée**.

1. Placez le bouton bascule **Amélioration audio** en position **activée**.

1. Sélectionnez **Appliquer** pour enregistrer vos modifications, puis **Démarrez** pour lancer l’assistant.

    Lorsque l’assistant s’est présenté, demandez-lui : « Avez-vous des avions disponibles à la location ? » Notez que l’assistant répond plus rapidement qu’auparavant, une fois votre question terminée. Le paramètre **Fin d’énoncé (EOU)** configure l’assistant pour qu’il détecte les pauses et votre fin de parole en fonction du contexte et de la sémantique. Ce paramétrage lui permet de tenir une conversation plus naturelle.

Lorsque vous avez terminé, passez à la section suivante.

## Nettoyer les ressources

Maintenant que l’exercice est terminé, supprimez le projet que vous avez créé afin d’éviter une utilisation inutile des ressources.

1. Sélectionnez **Centre de gestion** dans le menu de navigation d’AI Foundry.
1. Sélectionnez **Supprimer le projet** dans le volet d’informations de droite, puis confirmez la suppression.

