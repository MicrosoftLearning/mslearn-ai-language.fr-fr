# Spécifications

## Exécuter dans Cloud Shell

* Abonnement Azure avec accès OpenAI
* Si vous exécutez Azure Cloud Shell, choisissez l’interpréteur de commandes Bash. Azure CLI et Azure Developer CLI sont inclus dans Cloud Shell.

## Exécution locale

* Vous pouvez exécuter l’application web localement après avoir exécuté le script de déploiement :
    * [Azure Developer CLI (azd)](https://learn.microsoft.com/en-us/azure/developer/azure-developer-cli/install-azd)
    * [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)
    * Abonnement Azure avec accès OpenAI


## Variables d'environnement

Le fichier `.env` est créé par le script *azdeploy.sh*. Le point de terminaison du modèle IA, la clé API et le nom du modèle sont ajoutés pendant le déploiement des ressources.

## Déploiement de ressources Azure

Le `azdeploy.sh` fourni crée les ressources requises dans Azure :

* Modifiez les deux variables en haut du script en fonction de vos besoins, ne modifiez rien d’autre.
* Le script :
    * Déploie le modèle *gpt-4o* à l’aide d’AZD.
    * Crée le service Azure Container Registry
    * Utilise les tâches ACR pour générer et déployer l’image Dockerfile vers ACR
    * Crée le plan App Service
    * Crée l’application web App Service
    * Configure l’application web pour l’image conteneur dans ACR
    * Configure les variables d’environnement de l’application web
    * Le script fournira le point de terminaison App Service

Le script propose deux options de déploiement : 1. Déploiement complet ; et 2. Redéployer l’image uniquement. L’option 2 est uniquement destinée à être utilisée après le déploiement, lorsque vous souhaitez tester des modifications dans l’application. 

> Note : Vous pouvez exécuter le script dans PowerShell ou Bash à l’aide de la commande `bash azdeploy.sh`. Cette commande vous permet également d’exécuter le script dans Bash sans avoir à le rendre exécutable.

## Développement local

### Provisionner un modèle IA sur Azure

Vous pouvez exécuter le projet localement et provisionner uniquement le modèle IA en suivant ces étapes :

1. **Initialiser l’environnement** (choisissez un nom descriptif) :

   ```bash
   azd env new gpt-realtime-lab --confirm
   # or: azd env new your-name-gpt-experiment --confirm
   ```
   
   **Important** : Ce nom fera partie des noms de vos ressources Azure !  
   L’indicateur `--confirm` définit cet environnement comme votre environnement par défaut sans confirmation.

1. **Définissez votre groupe de ressources** :

   ```bash
   azd env set AZURE_RESOURCE_GROUP "rg-your-name-gpt"
   ```

1. **Connectez-vous et provisionnez les ressources d’IA** :

   ```bash
   az login
   azd provision
   ```

    > **Important** : N’exécutez PAS `azd deploy` : l’application n’est pas configurée dans les modèles AZD.

Si vous avez uniquement provisionné le modèle à l’aide de la méthode `azd provision`, vous DEVEZ créer un fichier `.env` à la racine du répertoire avec les entrées suivantes :

```
AZURE_VOICE_LIVE_ENDPOINT=""
AZURE_VOICE_LIVE_API_KEY=""
VOICE_LIVE_MODEL=""
VOICE_LIVE_VOICE="en-US-JennyNeural"
VOICE_LIVE_INSTRUCTIONS="You are a helpful AI assistant with a focus on world history. Respond naturally and conversationally. Keep your responses concise but engaging."
VOICE_LIVE_VERBOSE="" #Suppresses excessive logging to the terminal if running locally
```

Remarques :

1. Le point de terminaison est le point de terminaison du modèle et ne doit inclure que `https://<proj-name>.cognitiveservices.azure.com`.
1. La clé API est la clé du modèle.
1. Le modèle est le nom du modèle utilisé pendant le déploiement.
1. Vous pouvez récupérer ces valeurs à partir du portail AI Foundry.

### Exécution locale du projet

Le projet a été créé et géré à l’aide de **uv**, mais ce n’est pas obligatoire pour l’exécution. 

Si vous avez **uv** installé :

* Exécutez `uv venv` pour créer l’environnement
* Exécutez `uv sync` pour ajouter des packages
* Un alias est créé pour l’application web : `uv run web` pour démarrer le script `flask_app.py`.
* fichier requirements.txt créé avec `uv pip compile pyproject.toml -o requirements.txt`

Si vous n’avez pas **uv** installé :

* Créez l’environnement : `python -m venv .venv`
* Activez l’environnement : `.\.venv\Scripts\Activate.ps1`
* Installez les dépendances : `pip install -r requirements.txt`
* Exécutez l’application (depuis la racine du projet) : `python .\src\real_time_voice\flask_app.py`
