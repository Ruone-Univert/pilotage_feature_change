# UNI-VERT Pilotage PO — Feature Change

Ce dépôt contient la version en cours de développement et de maintenance de l'application **UNI-VERT Pilotage PO**.

Il correspond à la reprise du développement et de la maintenance du projet PO par Ruone, à partir de l'application existante.

## Objectif

Ce dépôt constitue le point de départ pour les évolutions futures et la maintenance de l'application.

Les versions sont gérées avec **Git**. Les évolutions ne doivent plus être réalisées en créant un nouveau fichier HTML pour chaque version.

Le principe de travail est le suivant :

```text
master
  ↓
création d'une branche de travail
  ↓
modification et tests en local
  ↓
commit
  ↓
push de la branche
  ↓
Pull Request sur GitHub
  ↓
merge dans master
  ↓
déploiement sur le VPS
```

La branche `master` doit autant que possible rester dans un état stable et déployable.

## Dépôt historique

L'ancien dépôt `PILOT-PO` contient les versions historiques de l'application sous forme de fichiers HTML distincts (`v10`, `v12`, `v13`, etc.).

Le dépôt actuel :

`Ruone-Univert/pilotage_feature_change`

est utilisé pour les développements et la maintenance futurs.

## Fichier principal

L'application principale est contenue dans :

`UNI-VERT_PO_Centre_Pilotage.html`

L'application est actuellement principalement contenue dans ce fichier HTML unique.

## Architecture générale

L'application s'exécute principalement dans le navigateur et communique avec plusieurs workflows n8n pour la synchronisation, le stockage et l'archivage des données.

Architecture simplifiée :

```text
Navigateur
    ↓
Webhooks n8n
    ↓
Services de stockage et workflows associés
```

Le site web est hébergé sur un **VPS Hostinger** via **Docker** et **Nginx**.

Le fichier HTML utilisé par le site en production est monté dans le conteneur Nginx depuis :

`/docker/UNIVERT-PILOT-PO/html/index.html`

# Workflow de développement Git

## 1. Partir de la dernière version de `master`

Avant de commencer une nouvelle modification :

```bash
git switch master
git pull origin master
```

Vérifier ensuite :

```bash
git status
```

Le working tree doit normalement être propre avant de commencer une nouvelle modification.

## 2. Créer une branche

Une modification ou un problème doit être traité dans une branche dédiée.

Exemples :

```bash
git switch -c fix/jotform-producer-matching
```

ou :

```bash
git switch -c feature/nom-de-la-fonctionnalite
```

Principes :

- `fix/...` pour une correction ;
- `feature/...` pour une nouvelle fonctionnalité ;
- une branche doit correspondre autant que possible à un seul sujet ;
- éviter de mélanger plusieurs bugs ou fonctionnalités indépendants dans la même branche.

## 3. Modifier et tester en local

Les modifications sont réalisées dans :

`UNI-VERT_PO_Centre_Pilotage.html`

Tester le comportement en local avant de préparer le commit.

Avant le commit, vérifier les modifications :

```bash
git status
git diff
```

## 4. Créer le commit

Ajouter uniquement les fichiers concernés :

```bash
git add UNI-VERT_PO_Centre_Pilotage.html
```

Vérifier ce qui sera inclus :

```bash
git diff --cached
```

Puis créer le commit.

Les messages de commit sont rédigés en français.

Exemple :

```bash
git commit -m "Corrige l'identification des producteurs lors de l'import Jotform"
```

Vérifier ensuite :

```bash
git status
```

Le working tree doit normalement être `clean`.

## 5. Push de la branche sur GitHub

Lors du premier push d'une nouvelle branche :

```bash
git push -u origin nom-de-la-branche
```

Exemple :

```bash
git push -u origin fix/jotform-producer-matching
```

Les push suivants sur cette même branche peuvent ensuite être réalisés avec :

```bash
git push
```

Le push d'une branche de travail ne modifie pas directement `master` et ne déploie pas le site en production.

## 6. Créer une Pull Request

Sur GitHub, créer une Pull Request avec :

```text
base: master
compare: branche-de-travail
```

Avant le merge, vérifier notamment :

- les fichiers modifiés ;
- le nombre et le contenu des modifications ;
- l'absence de fichiers non liés au sujet ;
- l'absence de conflit avec `master` ;
- les tests réalisés.

La description de la Pull Request doit expliquer brièvement :

- le problème ou l'objectif ;
- la modification réalisée ;
- les principaux tests effectués.

## 7. Merge dans `master`

Lorsque la Pull Request a été vérifiée, elle peut être mergée dans `master`.

Le merge signifie que la modification est désormais acceptée dans la version principale du projet.

Le merge dans GitHub ne met cependant pas automatiquement à jour le site sur le VPS.

---

# Déploiement en production

## 1. Connexion au VPS

Le dépôt Git utilisé sur le VPS est situé dans :

`/docker/UNIVERT-PILOT-PO/source`

Se placer dans ce dossier :

```bash
cd /docker/UNIVERT-PILOT-PO/source
```

## 2. Vérifier l'état du dépôt

Avant toute mise à jour :

```bash
git status
```

Le dépôt doit normalement être sur `master` et le working tree doit être propre.

## 3. Récupérer le dernier `master`

```bash
git pull origin master
```

Vérifier que la mise à jour s'est effectuée correctement avant de déployer le fichier HTML.

## 4. Sauvegarder la version actuellement en production

Avant de remplacer le fichier utilisé par le site, créer une sauvegarde.

Pour une sauvegarde simple :

```bash
cp /docker/UNIVERT-PILOT-PO/html/index.html /docker/UNIVERT-PILOT-PO/html/index.html.bak
```

Pour conserver plusieurs sauvegardes, utiliser de préférence un nom daté :

```bash
cp /docker/UNIVERT-PILOT-PO/html/index.html /docker/UNIVERT-PILOT-PO/html/index.html.backup-$(date +%Y%m%d-%H%M%S)
```

## 5. Déployer le nouveau fichier HTML

Copier la version provenant du dépôt vers le fichier utilisé par le site :

```bash
cp /docker/UNIVERT-PILOT-PO/source/UNI-VERT_PO_Centre_Pilotage.html /docker/UNIVERT-PILOT-PO/html/index.html
```

Le dossier :

`/docker/UNIVERT-PILOT-PO/html`

est monté dans le conteneur Nginx.

Par conséquent, **aucun rebuild de l'image Docker ni redémarrage du conteneur n'est nécessaire pour une simple mise à jour du fichier HTML**.

## 6. Tester la production

Après le déploiement :

- ouvrir le site en production ;
- vérifier que l'application se charge normalement ;
- tester la fonctionnalité concernée par la modification ;
- vérifier les données concernées lorsque le test implique n8n, Excel ou SharePoint.

---

# Rollback

Si un problème important apparaît immédiatement après le déploiement, restaurer la sauvegarde créée avant la mise en production.

Avec une sauvegarde `index.html.bak` :

```bash
cp /docker/UNIVERT-PILOT-PO/html/index.html.bak /docker/UNIVERT-PILOT-PO/html/index.html
```

Avec une sauvegarde datée, remplacer le chemin source par le fichier de sauvegarde souhaité.

Comme `/docker/UNIVERT-PILOT-PO/html` est monté dans le conteneur Nginx, la restauration du fichier HTML ne nécessite normalement pas de rebuild du conteneur.

Après un rollback, vérifier le site en production.

---

# Après le déploiement

Pour remettre également le dépôt local à jour :

```bash
git switch master
git pull origin master
```

Une branche déjà mergée peut ensuite être supprimée lorsqu'elle n'est plus nécessaire.

La suppression d'une branche n'efface pas les commits déjà intégrés dans `master`.

---

# Maintenance du README

Ce README doit être mis à jour lorsqu'une modification importante affecte notamment :

- l'architecture de l'application ;
- le stockage des données ;
- les workflows ou actions n8n ;
- les chemins SharePoint ;
- le workflow Git ;
- le processus de déploiement ;
- la configuration Docker / Nginx ;
- les procédures de sauvegarde ou de restauration.

Les corrections mineures de bugs, modifications visuelles ou petits changements de logique ne nécessitent pas systématiquement une mise à jour du README.

Les détails historiques d'une correction spécifique n'ont pas vocation à rester dans ce document lorsqu'ils ne sont plus nécessaires pour comprendre l'architecture ou maintenir l'application.

---

# Sécurité

Ne jamais enregistrer dans ce dépôt :

- mots de passe ;
- tokens OAuth ;
- clés API ;
- secrets n8n ;
- liens Microsoft Graph temporaires contenant des jetons d'accès ;
- autres identifiants sensibles.