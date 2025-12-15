### Workflow GitHub Actions : Notification Discord

Ce document explique le fonctionnement du fichier `.github/workflows/discord.yml` et les principales variables utilisées.

---

### 1. Déclencheur du workflow

- **name** : `Notify Discord`  
  Nom du workflow, uniquement pour l’interface GitHub.

- **on.push.branches** : `main`  
  Le workflow se lance à chaque `push` sur la branche `main`.

---

### 2. Job principal

- **jobs.notify.runs-on** : `ubuntu-latest`  
  Le job s’exécute sur un runner GitHub basé sur Ubuntu.

- **jobs.notify.steps[0].name** : `Send message to Discord`  
  Étape unique qui envoie le message au webhook Discord.

---

### 3. Variables d’environnement

- **DISCORD_WEBHOOK**  
  - Défini par : `env: DISCORD_WEBHOOK: ${{ secrets.DISCORD_WEBHOOK }}`  
  - Source : secret GitHub `DISCORD_WEBHOOK` (à configurer dans `Settings > Secrets and variables > Actions`).  
  - Contenu : URL de ton webhook Discord (fourni par Discord).

---

### 4. Variables GitHub (`github.*`)

Ces variables sont injectées par GitHub dans le contexte `${{ github.* }}`.

- **`${{ github.repository }}`**  
  - Exemple : `glena/githubaction`  
  - Utilisée dans :
    - Le message principal : nom du dépôt
    - L’URL du commit : `https://github.com/${{ github.repository }}/commit/${{ github.sha }}`

- **`${{ github.ref_name }}`**  
  - Nom court de la ref (par ex. `main`).  
  - Affichée dans le message principal comme nom de la branche.

- **`${{ github.sha }}`**  
  - SHA complet du commit en cours.  
  - Utilisée pour construire l’URL vers la page du commit sur GitHub.

- **`${{ github.actor }}`**  
  - Utilisateur GitHub qui a déclenché le workflow (auteur de l’action, pas forcément du commit).

- **`${{ github.event_name }}`**  
  - Type d’événement GitHub qui a déclenché le workflow (ici `push`).

- **`${{ github.workflow }}`**  
  - Nom du workflow (ici `Notify Discord`).

- **`${{ github.job }}`**  
  - Nom interne du job (ici `notify`).

- **`${{ github.ref }}`**  
  - Ref complète, par ex. `refs/heads/main`.

- **`${{ github.event.head_commit.message }}`**  
  - Message du commit en tête du `push`.  
  - Fallback : `'Pas de message'` si absent.

- **`${{ github.event.head_commit.author.name }}`**  
  - Nom de l’auteur Git du commit.  
  - Fallback : `${{ github.actor }}` si non défini.

- **`${{ github.event.head_commit.author.email }}`**  
  - Email de l’auteur Git du commit.  
  - Fallback : `'n/a'` si non défini.

- **`${{ github.event.head_commit.timestamp }}`**  
  - Date/heure du commit, utilisée comme `timestamp` de l’embed Discord.

---

### 5. Contenu du message Discord

- **username** : `GitHub Actions`  
  Nom affiché du bot dans Discord.

- **content** :
  - Texte simple au-dessus de l’embed.  
  - Exemple : `🚀 Nouveau push sur \\`repo\\` (branche \\`main\\`)` avec interpolation du dépôt et de la branche.

- **embeds[0].title** :
  - `Commit ${GITHUB_SHA:0:7}`  
  - Utilise la variable d’environnement `GITHUB_SHA` fournie par GitHub dans le runner, raccourcie aux 7 premiers caractères.

- **embeds[0].url** :
  - Lien direct vers la page du commit sur GitHub.

- **embeds[0].description** :
  - Message du commit ou `Pas de message` si vide.

- **embeds[0].fields** : tableau d’informations structurées :
  - **Auteur** : `${{ github.actor }}` (compte GitHub ayant déclenché l’action).
  - **Auteur (git)** : `${{ github.event.head_commit.author.name || github.actor }}`.  
    - Priorité à l’auteur Git; sinon retombe sur l’acteur GitHub.
  - **Email** : `${{ github.event.head_commit.author.email || 'n/a' }}`.  
  - **Événement** : `${{ github.event_name }}`.  
  - **Workflow** : `${{ github.workflow }}`.  
  - **Job** : `${{ github.job }}`.  
  - **Ref** : `${{ github.ref }}`.  
  - **Repo** : `${{ github.repository }}`.

- **embeds[0].timestamp** :
  - Date/heure du commit, pour l’affichage temps relatif dans Discord.

---

### 6. Envoi de la requête HTTP

- **curl**  
  - Méthode : `POST`  
  - En-tête : `Content-Type: application/json`  
  - Corps : variable `payload` contenant le JSON formatté.  
  - URL : `$DISCORD_WEBHOOK` (URL du webhook Discord stockée en secret).

---

### 7. À retenir

- **À configurer absolument** : le secret `DISCORD_WEBHOOK` dans GitHub.  
- **Sécurité** : l’URL du webhook n’est jamais commitée dans le dépôt, elle reste cachée dans les secrets.  
- **Personnalisation** : tu peux adapter le `content`, les `fields` ou ajouter d’autres informations disponibles dans le contexte `${{ github.* }}`.
