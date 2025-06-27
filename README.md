# WordPress Installation Role

Un rôle Ansible complet pour installer et configurer WordPress avec MariaDB sur Ubuntu et Rocky Linux.

## Description

Ce rôle automatise l'installation complète de WordPress avec :
- Installation des paquets nécessaires (Apache/HTTP, PHP, MariaDB)
- Configuration sécurisée de MariaDB
- Téléchargement et installation de WordPress
- Configuration d'Apache avec virtual host
- Support multi-OS (Ubuntu/Debian et Rocky Linux/RHEL)

## Prérequis

- Ansible >= 2.9
- Collection `community.mysql` installée :
  ```bash
  ansible-galaxy collection install community.mysql
  ```
- Accès root/sudo sur les serveurs cibles
- Python `pymysql` ou `mysqlclient` installé sur les serveurs cibles

## Variables du rôle

### Variables par défaut (`defaults/main.yml`)

```yaml
# Configuration de la base de données
wordpress_db_name: wordpress
wordpress_db_user: wpuser
wordpress_db_password: wppassword
wordpress_db_root_password: rootpassword

# Configuration WordPress
wordpress_version: latest
wordpress_admin_email: admin@localhost
wordpress_site_title: "WordPress Site"
wordpress_document_root: /var/www/html

# Configuration Apache
wordpress_apache_port: 80
```

### Variables système (définies automatiquement)

Le rôle détecte automatiquement l'OS et configure :
- Les noms de services (`apache2`/`httpd`, `mysql`/`mariadb`)
- Les utilisateurs système (`www-data`/`apache`)
- Les gestionnaires de paquets (`apt`/`dnf`)
- Les répertoires de configuration

## Handlers

- `restart apache` : Redémarre Apache
- `reload apache` : Recharge la configuration Apache
- `restart mariadb` : Redémarre MariaDB
- `start mariadb` : Démarre MariaDB
- `start apache` : Démarre Apache

## Exemple d'utilisation

### Playbook basique

```yaml
---
- hosts: wordpress_servers
  become: yes
  roles:
    - install_wordpress
```

### Playbook avec variables personnalisées

```yaml
---
- hosts: wordpress_servers
  become: yes
  vars:
    wordpress_db_name: mon_blog
    wordpress_db_user: bloguser
    wordpress_db_password: motdepasse_securise
    wordpress_db_root_password: root_securise
    wordpress_admin_email: admin@monsite.com
  roles:
    - install_wordpress
```

### Inventaire exemple

```ini
[wordpress_servers]
ubuntu-server ansible_host=192.168.1.10 ansible_user=ubuntu
rocky-server ansible_host=192.168.1.11 ansible_user=rocky

[wordpress_servers:vars]
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

## Installation et test

### 1. Installation du rôle

```bash
# Depuis Ansible Galaxy (une fois publié)
ansible-galaxy install votre_username.install_wordpress

# Ou clonage local
git clone <repo_url> roles/install_wordpress
```

### 2. Installation des dépendances

```bash
ansible-galaxy collection install community.mysql
```

### 3. Test avec des conteneurs

```bash
# Démarrer les conteneurs de test
docker run -d --name ubuntu-test --privileged ubuntu:20.04 /sbin/init
docker run -d --name rocky-test --privileged rockylinux:8 /sbin/init

# Installer Python dans les conteneurs
docker exec ubuntu-test apt update && apt install -y python3 python3-pip
docker exec rocky-test dnf install -y python3 python3-pip

# Exécuter le playbook
ansible-playbook -i inventory playbook.yml
```

## Structure du rôle

```
install_wordpress/
├── defaults/
│   └── main.yml          # Variables par défaut
├── files/                # Fichiers statiques (vide)
├── handlers/
│   └── main.yml          # Handlers pour services
├── meta/
│   └── main.yml          # Métadonnées du rôle
├── tasks/
│   └── main.yml          # Tâches principales
├── templates/
│   └── wordpress.conf.j2 # Template Apache
├── tests/
│   ├── inventory         # Inventaire de test
│   └── test.yml          # Playbook de test
├── vars/
│   └── main.yml          # Variables système
└── README.md             # Cette documentation
```

## Fonctionnalités

### Idempotence
- Toutes les tâches sont idempotentes
- Réexécution sans problème
- Vérification des états existants

### Multi-OS
- Support Ubuntu/Debian (18.04, 20.04, 22.04)
- Support Rocky Linux/RHEL (8, 9)
- Détection automatique de l'OS

### Sécurité
- Suppression des utilisateurs MySQL anonymes
- Suppression de la base de test
- Configuration sécurisée des mots de passe
- Permissions appropriées sur les fichiers

### Maintenance
- Code modulaire et lisible
- Variables configurables
- Handlers pour les redémarrages
- Documentation complète

## Accès à WordPress après déploiement

### Accès depuis le serveur local

Une fois le déploiement terminé, WordPress est accessible :

```bash
# Sur le serveur où WordPress est installé
curl http://localhost
# ou
curl http://127.0.0.1
```

### Accès depuis un navigateur

#### Serveur physique ou VM
```
http://<adresse_ip_du_serveur>
```

#### Environnement conteneurisé (Docker)

Si vous utilisez des conteneurs Docker avec port mapping :

```bash
# Vérifier les ports mappés
docker ps

# Accès depuis l'hôte Docker
http://localhost:<port_mappé>

# Exemples avec la configuration de test :
http://localhost:8083  # client1 (Ubuntu)
http://localhost:8084  # client2 (Ubuntu)  
http://localhost:8085  # client3 (Rocky)
http://localhost:8086  # client4 (Rocky)
```

#### Accès depuis un autre conteneur

```bash
# Depuis un conteneur dans le même réseau Docker
curl http://<nom_container>
curl http://<ip_interne_container>

# Exemple depuis le conteneur ansible
curl http://client1
curl http://172.19.0.3  # IP interne du conteneur
```

### Configuration post-installation

Après l'accès initial, vous devrez :

1. **Choisir la langue** d'installation
2. **Configurer la base de données** (déjà fait automatiquement)
3. **Créer le compte administrateur** WordPress :
   - Nom d'utilisateur admin
   - Mot de passe sécurisé
   - Adresse email
4. **Finaliser l'installation** et accéder au tableau de bord

### URLs importantes

```
Site principal     : http://<serveur>/
Administration     : http://<serveur>/wp-admin/
API REST          : http://<serveur>/wp-json/
Fichiers          : http://<serveur>/wp-content/
```

### Vérification de l'installation

```bash
# Vérifier que tous les services fonctionnent
systemctl status apache2  # Ubuntu
systemctl status httpd    # Rocky
systemctl status mysql    # Ubuntu  
systemctl status mariadb  # Rocky

# Vérifier les logs en cas de problème
tail -f /var/log/apache2/error.log      # Ubuntu
tail -f /var/log/httpd/error_log        # Rocky
tail -f /var/log/mysql/error.log        # MySQL/MariaDB
```

## Dépannage

### Problèmes courants

1. **Erreur de connexion MySQL** :
   ```bash
   # Vérifier que MariaDB fonctionne
   systemctl status mariadb
   ```

2. **Permissions Apache** :
   ```bash
   # Vérifier les permissions
   ls -la /var/www/html/
   chown -R www-data:www-data /var/www/html/  # Ubuntu
   chown -R apache:apache /var/www/html/      # Rocky
   ```

3. **Module Python MySQL manquant** :
   ```bash
   # Installer sur la cible
   pip3 install pymysql
   ```

4. **WordPress ne s'affiche pas** :
   ```bash
   # Vérifier Apache
   systemctl status apache2  # Ubuntu
   systemctl status httpd    # Rocky
   
   # Vérifier la configuration du virtual host
   apache2ctl configtest     # Ubuntu
   httpd -t                  # Rocky
   ```

5. **Erreur de base de données** :
   ```bash
   # Tester la connexion MySQL
   mysql -u root -p
   mysql -u {{ wordpress_db_user }} -p {{ wordpress_db_name }}
   ```

## Publication sur Ansible Galaxy

### 1. Préparation

```bash
# Vérifier la syntaxe
ansible-playbook --syntax-check test.yml

# Linter le rôle
ansible-lint .
```

### 2. Publication

```bash
# Se connecter à Galaxy
ansible-galaxy login

# Publier le rôle
ansible-galaxy import <github_username> <repo_name>
```

## Auteur

Loic Steve - Entreprise d'hébergement de sites web

## Support

Pour signaler des bugs ou demander des fonctionnalités, ouvrez une issue sur le repository GitHub.
