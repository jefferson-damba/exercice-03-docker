# Exercice 03 : Déploiement Multi-Conteneurs WordPress avec Docker-Compose

Ce projet présente la mise en place d'une architecture Web hautement disponible, segmentée et conteneurisée pour héberger la dernière version de WordPress. Au lieu d'utiliser un conteneur monolithique pré-généré, l'infrastructure sépare de manière professionnelle le serveur web frontal, le processeur de scripts (FastCGI) et le système de gestion de base de données relationnelle.

---

## 📐 1. Architecture des Services & Réseau

L'application est découpée en 3 micro-services isolés au sein d'un réseau bridge dédié (`wp-network`) :
1. **`wp-nginx` (Nginx : latest) :** Serveur Web frontal recevant les connexions HTTP sur le port `80`. Il distribue le contenu statique (images, CSS) et transmet les requêtes dynamiques au moteur PHP.
2. **`wp-php` (PHP : 8.2-fpm) :** Processeur de script (FastCGI Process Manager). Il exécute le code WordPress et intègre dynamiquement l'extension native `mysqli` indispensable pour dialoguer avec MariaDB.
3. **`wp-database` (MariaDB : latest) :** Système de gestion de base de données relationnelle persistant.

### 📦 Gestion de la Persistance et Volume Commun
Pour répondre à l'exigence d'un **volume commun**, le fichier d'orchestration implémente un volume nommé partagé :
* **`wp_data` :** Ce volume est monté simultanément dans le conteneur Nginx et le conteneur PHP sur le chemin `/var/www/html`. Nginx y accède pour localiser et lire les fichiers statiques, tandis que PHP en a besoin pour interpréter le code source des fichiers `.php`.
* **`db_data` :** Assure la persistance des tables SQL de MariaDB de manière isolée dans `/var/lib/mysql`.

---

## 🚀 2. Guide d'Installation de A à Z (Linux Proxmox LXC)

L'intégralité du déploiement a été réalisée sur un conteneur **Proxmox LXC** (Debian Trixie) avec l'option **Nesting (Imbrication)** activée.

### Étape 1 : Préparation et installation du moteur Docker
Mise à jour des dépôts locaux, installation des utilitaires de sécurité réseau et déploiement du moteur Docker officiel ainsi que de son plugin Compose :

```bash
apt update && apt upgrade -y
apt install -y ca-certificates curl gnupg lsb-release wget
mkdir -p /etc/apt/keyrings
curl -fsSL [https://download.docker.com/linux/debian/gpg](https://download.docker.com/linux/debian/gpg) | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
apt update
apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

### 🐋 Étape 2 : Vérification de la santé du démon Docker
Validation opérationnelle du service système avant de configurer l'application :

```bash
systemctl status docker --no-pager
docker --version
docker compose version
📁 Étape 3 : Création de l'arborescence du projet sur l'hôte
Avant de rédiger les configurations, la structure des dossiers doit être créée sous /opt/ pour accueillir proprement les fichiers :

Bash
mkdir -p /opt/exercice-03-docker/nginx
cd /opt/exercice-03-docker
🌐 Étape 4 : Configuration du Serveur Web Nginx
Création du fichier d'hôte virtuel dans le sous-dossier dédié pour gérer la redirection FastCGI :

Bash
nano nginx/default.conf
Contenu du fichier nginx/default.conf :

Nginx
# Insère ici ta configuration Nginx personnalisée
📝 Étape 5 : Rédaction du descripteur d'orchestration Docker-Compose
Création du fichier d'orchestration à la racine du projet pour lier les conteneurs, les volumes et le réseau :

Bash
nano docker-compose.yaml
Contenu du fichier docker-compose.yaml :

YAML
# Copie-colle ton fichier ici en modifiant tes informations de connexion
📥 Étape 6 : Téléchargement et déploiement des sources de l'application
Bash
# Téléchargement de l'archive officielle
wget [https://wordpress.org/latest.tar.gz](https://wordpress.org/latest.tar.gz)

# Extraction et transfert vers le volume de stockage Docker de l'hôte
tar -xzf latest.tar.gz
cp -r wordpress/* /var/lib/docker/volumes/exercice-03-docker_wp_data/_data/

# Attribution des permissions à l'utilisateur www-data (UID 33) pour l'exécution web
chown -R 33:33 /var/lib/docker/volumes/exercice-03-docker_wp_data/_data/

# Nettoyage des résidus
rm -rf wordpress latest.tar.gz
🧪 3. Phase de Validation Graphique (Interface Web)
L'installation finale s'exécute depuis un navigateur à l'adresse IP locale du conteneur LXC : http://X.X.X.X.

A. Initialisation de l'assistant de configuration
L'affichage du sélecteur valide que Nginx transmet correctement les requêtes à PHP-FPM à travers le point de montage commun.

B. Appairage avec la Base de Données Centralisée
Saisie des identifiants de sécurité. La variable de l'hôte est définie par db (et non localhost), permettant au DNS de Docker d'aiguiller le flux directement vers le conteneur MariaDB.

C. Finalisation de l'identité du site
Création du profil d'administration du blog de l'entreprise avec le titre MiniLab DevOps et application d'un mot de passe fort.

D. Authentification et Accès au Panneau de Contrôle
Validation finale via la mire de connexion pour atteindre le tableau de bord d'administration général.