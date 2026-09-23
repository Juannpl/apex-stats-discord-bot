<div align="center">

# 🎮 Bot Apex

**Tes statistiques Apex Legends, directement dans Discord.**

Un bot Python pour consulter un profil, retrouver sa légende principale et synchroniser son surnom sur le serveur avec le pseudo Apex recherché.

![Python 3.12](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)
![discord.py](https://img.shields.io/badge/discord.py-2.6.4-5865F2?logo=discord&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-ready-2496ED?logo=docker&logoColor=white)

**PC · PlayStation · Xbox**

</div>

---

## ✨ Fonctionnalités

- **Statistiques du profil** : niveau, rang, division et points de classement (RP).
- **Légendes** : légende actuellement sélectionnée et légende principale estimée à partir des éliminations connues.
- **Totaux du compte** : éliminations et dégâts, selon les données fournies par l’API.
- **Classements ALS** : position et pourcentage sur la plateforme et au global, avec l’indicateur de fiabilité renvoyé par l’API.
- **Réponse privée** : les résultats s’affichent dans un embed visible uniquement par la personne qui lance la commande.
- **Surnom Discord** : après une réponse réussie de l’API principale, le bot tente d’appliquer le pseudo Apex recherché au membre ayant lancé la commande.
- **API de secours** : une réponse simplifiée peut être proposée lorsque des champs requis manquent dans la réponse principale.

## ⚡ Utilisation

Dans le serveur Discord configuré, lance la commande et sélectionne une plateforme :

```text
/statapex platform:PC player_name:MonPseudoApex
```

| Choix dans Discord | Identifiant envoyé à l’API |
| --- | --- |
| PC | `PC` |
| Playstation | `PS4` |
| Xbox | `X1` |

> Le surnom modifié est celui de la personne qui lance la commande, même si elle recherche le profil d’un autre joueur. Ce changement est visible sur le serveur ; seule la réponse contenant les statistiques est privée.

## 🚀 Installation locale

### 1. Préparer l’environnement

Utilise **Python 3.12**, la version employée dans l’image Docker du projet. Depuis le dossier du dépôt :

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

Sous Windows PowerShell, remplace la commande d’activation par :

```powershell
.\venv\Scripts\Activate.ps1
```

### 2. Configurer le bot Discord

Dans le portail développeur Discord, prépare une application avec un bot, puis :

1. Active les intents **Server Members Intent** et **Message Content Intent**, utilisés par le code.
2. Invite le bot sur ton serveur avec les scopes `bot` et `applications.commands`.
3. Donne-lui accès au salon où la commande sera utilisée et la permission **Gérer les surnoms** pour permettre le renommage.
4. Place son rôle au-dessus des membres qu’il doit pouvoir renommer.

### 3. Ajouter les variables d’environnement

Crée un fichier `.env` à la racine :

```dotenv
DISCORD_TOKEN=ton_token_discord
GUILD_ID=identifiant_du_serveur_discord
APEX_API_KEY=ta_cle_api_apex
```

| Variable | Description |
| --- | --- |
| `DISCORD_TOKEN` | Token du bot Discord. |
| `GUILD_ID` | Identifiant numérique du serveur où enregistrer `/statapex`. |
| `APEX_API_KEY` | Clé pour l’API Apex Legends Status utilisée par le bot (`api.mozambiquehe.re`). |

Le fichier `.env` est ignoré par Git. Garde les tokens et clés privés.

### 4. Démarrer

```bash
python bot_apex.py
```

Le bot affiche sa connexion dans la console, puis tente de synchroniser les commandes avec le serveur configuré.

## 🐳 Exécution avec Docker

Le `Dockerfile` utilise `python:3.12-slim`.

Pour construire l’image à partir des fichiers suivis par Git, sans inclure le `.env` local :

```bash
git archive --format=tar HEAD | docker build -t bot_apex:latest -
```

Cette commande utilise le dernier commit : pense à committer les changements de code que tu souhaites embarquer.

Démarre ensuite le conteneur avec tes variables :

```bash
docker run -d \
  --name bot_apex \
  --restart unless-stopped \
  --env-file .env \
  bot_apex:latest
```

Consulter les logs ou arrêter le bot :

```bash
docker logs -f bot_apex
docker stop bot_apex
```

Aucun port entrant n’est à publier pour ce bot.

## 🔄 Déploiement automatique

Le workflow [deploy.yml](.github/workflows/deploy.yml) se déclenche à chaque push sur `main`.

Il nécessite un **runner GitHub Actions auto-hébergé** disposant de Docker et capable d’exécuter les commandes `sudo docker` sans interaction. Ajoute ces secrets dans le dépôt GitHub :

| Secret GitHub Actions | Valeur |
| --- | --- |
| `DISCORD_TOKEN` | Token du bot |
| `GUILD_ID` | Identifiant du serveur |
| `APEX_API_KEY` | Clé de l’API Apex |

Le workflow récupère le code, construit l’image `bot_apex:latest`, arrête et supprime l’ancien conteneur, puis lance son remplaçant avec les secrets injectés comme variables d’environnement. Ce remplacement entraîne une courte interruption du bot.

## 🗂️ Structure du projet

```text
.
├── bot_apex.py                  # Bot Discord, commande, appels API et embeds
├── apex_utils.py                # Sélection de la légende ayant le plus de kills
├── requirements.txt             # Dépendances Python et versions fixées
├── Dockerfile                   # Image Python 3.12
├── .github/workflows/deploy.yml # Déploiement sur le runner auto-hébergé
└── .env                         # Configuration locale, non versionnée
```

## 🧩 Données et limites

La légende principale est estimée à partir du tracker `kills` disponible pour chaque légende, en excluant l’entrée `Global`. Il ne s’agit pas d’une mesure du temps de jeu. Si aucun total strictement positif n’est trouvé, le bot affiche qu’aucune donnée n’est disponible.

Les statistiques dépendent des données et trackers exposés par les API. Certaines valeurs peuvent donc être absentes et apparaître sous la forme `N/A`.

Le secours via `lil2-gateway.apexlegendsstatus.com` est déclenché lorsqu’une `KeyError` survient pendant le traitement de la réponse principale. Il ne prend pas le relais lors d’une erreur HTTP ou réseau. Sa réponse affiche moins de détails et ne déclenche pas le renommage.

## 🔧 Dépannage

| Symptôme | À vérifier |
| --- | --- |
| Le bot ne démarre pas | Les trois variables sont renseignées, `GUILD_ID` est numérique et les dépendances sont installées. |
| `/statapex` n’apparaît pas | Le serveur correspond à `GUILD_ID`, l’invitation inclut `applications.commands` et les logs indiquent la synchronisation. |
| La connexion Discord échoue | Le token est correct et les deux intents demandés sont activés. |
| Le surnom reste inchangé | Le bot dispose de la permission et d’un rôle suffisamment haut. Les erreurs de renommage sont ignorées par le code. |
| Le joueur est introuvable | Le pseudo et la plateforme sélectionnée sont corrects. |
| Une erreur API ou réseau apparaît | Vérifie la clé API, l’accès réseau et la disponibilité du service. Chaque requête a un délai maximal de 10 secondes. |

---

<div align="center">

Développé avec **Python**, **discord.py** et **aiohttp** · Statistiques fournies par les API **Apex Legends Status**.

</div>
