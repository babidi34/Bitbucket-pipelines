# README pour Configuration CI/CD avec Bitbucket Pipelines

Ce document explique comment configurer les variables d'environnement nécessaires dans Bitbucket, placer le fichier `bitbucket-pipelines.yml` dans le dépôt, et préparer les serveurs pour recevoir des connexions SSH pour CI/CD.

## Prérequis

1. Un dépôt Bitbucket configuré.
2. Accès SSH configuré et fonctionnel sur les serveurs de destination.
3. Préparation des serveurs de destination :
   - Projet déployé.
   - Utilisateur bitbucket-ci créé avec des droits pour lire et écrire dans le dossier du projet.
   - Clé autorisée pour l'utilisateur bitbucket-ci.

## Étapes de Configuration

### 1. Générer un Access Token Bitbucket

Pour générer un token Bitbucket :

1. Accédez aux paramètres de votre dépôt Bitbucket : https://bitbucket.org/account/settings/
2. Sous **Sécurité**, cliquez sur **Access Token**.
3. Cliquez sur **Create Repository Access Token**.
4. Donnez un nom à l'Access Token et sélectionnez les permissions nécessaires pour lire les dépôts (`Read` dans Repositories).
5. Une fois le token créé, Bitbucket fournit une URL dans la partie "How to use this token with your Git repository". Prenez uniquement l'URL (sans le 'git clone') et utilisez-la comme valeur de la variable `GIT_REPO` dans Bitbucket.

### 2. Encoder la Clé Privée SSH

Sur votre machine locale, encodez votre clé privée SSH en Base64 :

```bash
base64 -w 0 < ~/.ssh/chemin_de_la_clé
```


Copiez la sortie et définissez-la comme valeur de la variable `SSH_PRIVATE_KEY_BASE64` dans Bitbucket.

**Note : Il est fortement recommandé de stocker cette clé privée encodée en Base64 dans un gestionnaire de mots de passe sécurisé tel que Vault, Bitwarden ou tout autre outil de gestion de secrets afin de pouvoir la réutiliser pour d'autres pipelines.**

#### 3. Définir les Variables d'Environnement dans Bitbucket

Accédez aux paramètres de votre dépôt Bitbucket et définissez les variables d'environnement globale suivantes sous **Repository settings > Pipelines > Repository variables** :


- **COMMAND_ENDING** : Commande à exécuter après le déploiement (ex: "composer install --no-interaction --no-scripts && npm install && npm run build && php artisan migrate").
- **BECOME** : "true" pour qu'Ansible utilise le sudo (si pas de droit sudo alors la valeur doit être "false").
- **GIT_REPO** : URL git avec le token (fourni lors de la création du token).
- **ANSIBLE_EXECUTABLE** : (optionnel) chemin vers le shell qui doit être utilisé par ansible.

Maintenant il faut définir les variables suivantes pour chacun des environnements sous **Repository settings > Pipelines > Deployments** :

- **SSH_HOSTS** : Adresse(s) IP ou nom(s) de domaine des serveurs de destination.
- **SSH_PORT** : Port SSH des serveurs de destination (ex: 22, 2227, etc.).
- **SSH_PRIVATE_KEY_BASE64** : Clé privée SSH encodée en Base64. (Si vous utilisez la même clé SSH sur vos différents serveurs alors vous pouvez créer cette variable dans la partie **Repository variables**)
- **SSH_USER** : Nom d'utilisateur SSH pour la connexion.
- **PROJECT_DIR** : Répertoire du projet sur le serveur (ex: /var/www/webapp).


### 4. Placer le Fichier `bitbucket-pipelines.yml` dans le Dépôt

Ajoutez le fichier `bitbucket-pipelines.yml` à la racine de votre dépôt.

### 5. Préparer les Serveurs pour les Connexions SSH

Assurez-vous que les serveurs de destination sont configurés pour accepter les connexions SSH. Vous devez :

1. Ajouter la clé publique correspondante à la clé privée utilisée dans `~/.ssh/authorized_keys` sur chaque serveur.
2. Vérifier que les serveurs sont accessibles via le port SSH configuré (`SSH_PORT`).

**Note** : Si ces configurations ont déjà été effectuées par le passé, cette étape n'est pas nécessaire.

## Conclusion

Avec ces configurations, votre pipeline Bitbucket sera prêt à exécuter des déploiements CI/CD via SSH sur les serveurs configurés. Assurez-vous que les serveurs sont correctement configurés pour accepter les connexions SSH afin que le processus de déploiement se déroule sans problème.

Si vous rencontrez des problèmes ou avez des questions, veuillez consulter la documentation de Bitbucket Pipelines ou contacter votre administrateur système.


## Procédure de rollback via Bitbucket Pipelines

### Étapes à suivre :

1. **Créer une branche à partir du commit cible :**
   - Identifiez le commit sur lequel vous souhaitez revenir.
   - Créez une nouvelle branche sur ce commit, par exemple : `git checkout -b rollback-branch <commit_id>`.

2. **Pousser la branche nouvellement créée :**
   - Poussez la nouvelle branche sur le dépôt distant :  
     `git push origin rollback-branch`.

3. **Modifier la variable GIT_BRANCH dans Bitbucket :**
   - Accédez à Bitbucket, ouvrez les **paramètres de l'environnement** concerné.
   - Créez ou modifiez la variable `GIT_BRANCH`, et définissez comme valeur le nom de la branche nouvellement créée (`rollback-branch`).

4. **Exécuter le pipeline manuellement :**
   - Allez dans la section **Pipelines** du dépôt.
   - Lancez manuellement l'exécution du pipeline sur l'environnement en question.

---

Suivre ces étapes permet de revenir à un état antérieur de l'application.