### README pour Configuration CI/CD avec Bitbucket Pipelines

Ce document explique comment configurer les variables d'environnement nécessaires dans Bitbucket, placer le fichier `bitbucket-pipelines.yml` dans le dépôt, et préparer les serveurs pour recevoir des connexions SSH pour CI/CD.

#### Prérequis

1. Un dépôt Bitbucket configuré.
2. Accès SSH configuré et fonctionnel sur les serveurs de destination.
3. Les variables d'environnement suivantes doivent être définies dans Bitbucket :
   - `SSH_HOSTS`
   - `SSH_PORT`
   - `SSH_PRIVATE_KEY_BASE64`
   - `SSH_USER`
   - `PROJECT_DIR`
   - `GIT_REPO`
   - `GIT_BRANCH`
   - `COMMAND_ENDING`
   - `BITBUCKET_USERNAME`
   - `BITBUCKET_APP_PASSWORD`
4. Préparation des serveurs de destination :
   - Projet déployé.
   - Utilisateur bitbucket-ci créé.
   - Clé autorisée pour l'utilisateur bitbucket-ci.
   - Droits sudo pour l'utilisateur bitbucket-ci.

### Étapes de Configuration

#### 1. Définir les Variables d'Environnement dans Bitbucket

Accédez aux paramètres de votre dépôt Bitbucket et définissez les variables d'environnement suivantes sous **Repository settings > Pipelines > Repository variables** :

- **SSH_HOSTS** : Adresse(s) IP ou nom(s) de domaine des serveurs de destination.
- **SSH_PORT** : Port SSH des serveurs de destination (ex: 22, 2227, etc.).
- **SSH_PRIVATE_KEY_BASE64** : Clé privée SSH encodée en Base64.
- **SSH_USER** : Nom d'utilisateur SSH pour la connexion.
- **PROJECT_DIR** : Répertoire du projet sur le serveur (ex: /var/www/webapp).
- **COMMAND_ENDING** : Commande à exécuter après le déploiement (ex: "composer install --no-interaction --no-scripts && npm install && npm run build && php artisan migrate").
- **BITBUCKET_USERNAME** : Ton nom d'utilisateur Bitbucket.
- **BITBUCKET_APP_PASSWORD** : Mot de passe d'application pour l'accès au dépôt.
- **ENV_FILE_CONTENT_BASE64** : Contenu en base64 de .env.
- **BECOME** : "true" pour qu'Ansible utilise le sudo.

#### 2. Générer un Mot de Passe d'Application Bitbucket

Pour générer un mot de passe d'application Bitbucket :

1. Accédez aux paramètres de votre compte Bitbucket : https://bitbucket.org/account/settings/
2. Sous **Access management**, cliquez sur **App passwords**.
3. Créez un nouvel App password en cliquant sur **Create app password**.
4. Donnez un nom au mot de passe et sélectionnez les permissions nécessaires pour lire les dépôts (`Read`).
5. Cliquez sur **Create** et copiez le mot de passe généré. Utilisez ce mot de passe comme valeur de la variable `BITBUCKET_APP_PASSWORD` dans Bitbucket.

#### 3. Encoder la Clé Privée SSH et le .env en Base64

Sur votre machine locale, encodez votre clé privée SSH en Base64 :

```bash
base64 -w 0 < ~/.ssh/id_rsa
```


Copiez la sortie et définissez-la comme valeur de la variable `SSH_PRIVATE_KEY_BASE64` dans Bitbucket.

Sur votre machine locale, encodez votre fichier .env en Base64 :

```bash
base64 -w 0 < /chemin-projet/.env
```

Copiez la sortie et définissez-la comme valeur de la variable `ENV_FILE_CONTENT_BASE654` dans Bitbucket.

#### 4. Placer le Fichier `bitbucket-pipelines.yml` dans le Dépôt

Ajoutez le fichier `bitbucket-pipelines.yml` à la racine de votre dépôt.

### 5. Préparer les Serveurs pour les Connexions SSH

Assurez-vous que les serveurs de destination sont configurés pour accepter les connexions SSH. Vous devez :

1. Ajouter la clé publique correspondante à la clé privée utilisée dans `~/.ssh/authorized_keys` sur chaque serveur.
2. Vérifier que les serveurs sont accessibles via le port SSH configuré (`SSH_PORT`).

### Conclusion

Avec ces configurations, votre pipeline Bitbucket sera prêt à exécuter des déploiements CI/CD via SSH sur les serveurs configurés. Assurez-vous que les serveurs sont correctement configurés pour accepter les connexions SSH afin que le processus de déploiement se déroule sans problème.

Si vous rencontrez des problèmes ou avez des questions, veuillez consulter la documentation de Bitbucket Pipelines ou contacter votre administrateur système.
