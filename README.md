# UNI-VERT Pilotage PO — Feature Change

Ce dépôt contient la version en cours de développement et de maintenance de l'application UNI-VERT Pilotage PO.

Il correspond à la reprise du développement et de la maintenance du projet PO par Ruone, à partir de l'application existante.

## Objectif

Ce dépôt est utilisé pour poursuivre le développement de l'application à partir de la version existante, avec une gestion des versions basée sur Git.

Les évolutions sont désormais suivies par des commits Git plutôt que par la création d'un nouveau fichier HTML pour chaque version.

## Dépôt historique

L'ancien dépôt `PILOT-PO` contient les versions historiques de l'application sous forme de fichiers HTML distincts (`v10`, `v12`, `v13`, etc.).

Ce dépôt constitue le nouveau point de départ pour les évolutions futures et la maintenance de l'application.

## Fichier principal

L'application principale est contenue dans :

`UNI-VERT_PO_Centre_Pilotage.html`

## Architecture générale

L'application est actuellement une application web principalement contenue dans un fichier HTML unique.

Architecture générale :

```text
Navigateur
    ↓
n8n
    ↓
Microsoft Graph
    ↓
SharePoint
```

Le site web est hébergé sur un VPS Hostinger via Docker et Nginx.

## Développement actuel

### Référentiel PO partagé

Une synchronisation partagée du Référentiel PO a été mise en place.

Le Référentiel PO est désormais stocké séparément de l'AppState dans le fichier :

`PILOT - PO/BASE DE DONNEE/referentiel.json`

Ce fichier est stocké dans la bibliothèque SharePoint existante.

Deux actions n8n sont utilisées :

- `load_referentiel` : charge le référentiel depuis SharePoint ;
- `save_referentiel` : enregistre le référentiel dans SharePoint.

Lorsqu'un utilisateur importe un PDF du Référentiel PO :

```text
PDF
    ↓
Parsing dans le navigateur
    ↓
save_referentiel
    ↓
n8n
    ↓
Microsoft Graph
    ↓
referentiel.json sur SharePoint
```

Les autres utilisateurs récupèrent ensuite le même référentiel via `load_referentiel`.

Le navigateur conserve également une copie locale de secours dans `localStorage`.

### Pourquoi le Référentiel est séparé de l'AppState

Le Référentiel PO ne doit pas être stocké dans le champ JSON de l'AppState.

La taille du Référentiel peut dépasser la capacité d'une cellule Excel. Il est donc stocké dans un fichier JSON séparé sur SharePoint.

L'AppState continue d'être utilisé pour les autres données de l'application.

## Déploiement

Le dépôt GitHub utilisé pour le développement actuel est :

`Ruone-Univert/pilotage_feature_change`

Sur le VPS, le dépôt Git est situé dans :

`/docker/UNIVERT-PILOT-PO/source`

Pour récupérer la dernière version :

```bash
cd /docker/UNIVERT-PILOT-PO/source
git pull
```

Le fichier HTML doit ensuite être copié vers le fichier utilisé par le site :

```bash
cp /docker/UNIVERT-PILOT-PO/source/UNI-VERT_PO_Centre_Pilotage.html /docker/UNIVERT-PILOT-PO/html/index.html
```

Le dossier :

`/docker/UNIVERT-PILOT-PO/html`

est monté dans le conteneur Nginx.

Par conséquent, aucun redémarrage du conteneur n'est nécessaire pour une simple mise à jour du fichier HTML.

### Processus de déploiement

```text
Développement local
    ↓
git commit
    ↓
git push
    ↓
GitHub
    ↓
git pull sur le VPS
    ↓
Copie du HTML vers /html/index.html
    ↓
Nginx
    ↓
Site en production
```

## Sauvegarde / rollback

Une sauvegarde du site réalisée avant la mise en place du Référentiel partagé est disponible ici :

`/docker/UNIVERT-PILOT-PO/html/index.html.backup-20261005-before-referentiel`

Elle peut être utilisée en cas de besoin de retour à la version précédente.

## Tests du Référentiel partagé

Le fonctionnement suivant a été validé :

```text
Import du PDF
    ↓
Parsing du Référentiel
    ↓
Sauvegarde via save_referentiel
    ↓
SharePoint
    ↓
Chargement via load_referentiel
    ↓
Affichage sur un autre navigateur / ordinateur
```

Le Référentiel partagé a été testé depuis plusieurs navigateurs / postes.

## Hors périmètre actuel

La gestion des permissions et des rôles utilisateurs n'a pas été modifiée lors de la mise en place du Référentiel partagé.

Une gestion plus stricte des droits de modification pourra être ajoutée ultérieurement.

## Maintenance

Ce README doit être mis à jour lorsqu'une modification importante affecte notamment :

- l'architecture de l'application ;
- le stockage des données ;
- les workflows ou actions n8n ;
- les chemins SharePoint ;
- le processus de déploiement ;
- la configuration Docker / Nginx ;
- les procédures de sauvegarde ou de restauration.

Les corrections mineures de bugs, modifications visuelles ou petits changements de logique ne nécessitent pas systématiquement une mise à jour du README.

## Sécurité

Ne jamais enregistrer dans ce dépôt :

- mots de passe ;
- tokens OAuth ;
- clés API ;
- secrets n8n ;
- liens Microsoft Graph temporaires contenant des jetons d'accès ;
- autres identifiants sensibles.