# Chapitre 2 : Commandes Ad-Hoc avec Ansible

## Introduction

Les **commandes ad-hoc** sont une fonctionnalité essentielle d’Ansible permettant de **lancer rapidement des tâches ponctuelles** sur vos hôtes gérés, sans écrire un playbook complet.

Elles sont idéales pour :

- Vérifier la connectivité avec vos hôtes,
- Déployer rapidement des fichiers ou packages,
- Gérer utilisateurs, groupes et services,
- Tester des commandes ou scripts sur vos machines distantes.

> 💡 Astuce RHCE : Les commandes ad-hoc sont très utilisées pour la **validation rapide des configurations et le débogage**, souvent avant de créer un playbook.

---

## 1 Syntaxe Générale

```bash
ansible <hosts> -m <module> -a "<arguments>" -i <inventory> [options]
```

| Paramètre          | Description                                                                                                                 |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------- |
| `<hosts>`          | Cibles de la commande (`all` pour tous, un groupe ou un hôte spécifique).                                                   |
| `-m <module>`      | Module Ansible à utiliser (`ping`, `shell`, `command`, `copy`, `file`, `user`, etc.).                                       |
| `-a "<arguments>"` | Arguments à passer au module choisi.                                                                                        |
| `-i <inventory>`   | Fichier d’inventaire contenant les hôtes et leurs groupes.                                                                  |
| `[options]`        | Options supplémentaires :<br>- `-u <user>` : utilisateur SSH<br>- `-b` : activer sudo<br>- `-k` : demander mot de passe SSH |

---

<p align="center">
  <img src="images/Ad-hoc structure.png" alt="Ansible Overview" width="600">
</p>

---
### Explorer les modules
Lister tous les modules :
```bash
ansible-doc -l
```
Documentation détaillée d’un module :
```bash
ansible-doc <module_name>
```
Rechercher un module par mot-clé :
```bash
ansible-doc -s user
```

**Astuce** : Les modules Ansible peuvent varier selon le système (ex : yum pour RHEL/CentOS, apt pour Debian/Ubuntu).

## 2 Modules Ad-Hoc Essentiels
Voici les modules les plus utilisés avec leurs exemples pratiques :

### 2.1. Ping
Vérifie la connectivité et la présence de Python sur l’hôte distant.
```bash
ansible all -m ping
```

### 2.2. Command
Exécute une commande simple sur les hôtes distants. ⚠️ Pas de redirection ni de variables shell.
```bash
ansible all -m command -a "ls -l /etc"
```

### 2.3. Shell
Exécute des commandes shell avec redirections et variables.
```bash
ansible all -m shell -a "echo $HOME > homes_directory"
```

### 2.4. Copy
Copie un fichier depuis le Control Node vers les Managed Nodes.
```bash
ansible all -m copy -a "src=/local/file dest=/remote/file"
```

### 2.5. Fetch
Récupère un fichier depuis un hôte distant vers le Control Node.
```bash
ansible all -m fetch -a "src=/remote/file dest=/local/destination"
```

### 2.6. Yum / Apt
Installe ou supprime des packages sur vos systèmes Linux.
```bash
ansible all -m yum -a "name=httpd state=present"
```
```bash
ansible all -m apt -a "name=nginx state=present"
```

### 2.7. File
Gère les fichiers et répertoires (création, suppression, permissions).
```bash
ansible all -m file -a "path=/backup state=directory mode=0755 owner=root group=root"
```

### 2.8. Lineinfile
Ajoute, modifie ou supprime une ligne dans un fichier texte.
```bash
ansible all -m lineinfile -a "path=/etc/hosts line='192.168.1.100 newhost' state=present"
```

### 2.9. Service
Gère l’état des services : démarrer, arrêter, redémarrer, activer.
```bash
ansible all -m service -a "name=httpd state=restarted"
```

### 2.10. User / Group
Gère les utilisateurs et groupes sur les hôtes distants.
```bash
ansible all -m user -a "name=developer uid=2001 state=present"
```
```bash
ansible all -m group -a "name=developers gid=2002 state=present"
```

### 2.11. Git / Template
Git : clone ou met à jour un dépôt Git.
Template : déploie un fichier Jinja2 sur les hôtes.
```bash
ansible all -m git -a "repo=https://github.com/user/repo.git dest=/path/to/dest"
```
```bash
ansible all -m template -a "src=/template.j2 dest=/remote/dest"
```

### 2.12. Setup
Récupère toutes les facts (informations système) sur les hôtes.
```bash
ansible all -m setup
```

### 2.13. Firewalld / SELinux
Gère le firewall et les politiques SELinux.
```bash
ansible all -m firewalld -a "service=http permanent=yes state=enabled"
```
```bash
ansible all -m selinux -a "policy=targeted state=enforcing"
```

## 3 Lab : Commandes Ad-Hoc Pratiques

### Objectif du Lab
Ce lab a pour but de pratiquer les **commandes ad-hoc Ansible** pour :

- Gérer des utilisateurs et groupes.
- Créer, modifier et supprimer des fichiers et répertoires.
- Installer et gérer des services.
- Copier et récupérer des fichiers.
- Vérifier l’état des hôtes.

Ce lab permet de se rapprocher des tâches RHCE EX294.

---

### 3.1 : Vérification de la connectivité et récupération d’informations

1. **Tester la connexion de tous les hôtes avec le module ping** :

```bash
ansible all -m ping
```
Explication : Permet de vérifier que tous les hôtes sont accessibles et que Python est installé.

### 3.2 : Gestion des utilisateurs et groupes

Créer un groupe devops :
```bash
ansible all -m group -a "name=devops gid=3001 state=present"
```
Ajouter un utilisateur alice dans le groupe devops avec un shell spécifique et UID :
```bash
ansible all -m user -a "name=alice uid=3001 group=devops shell=/bin/bash state=present"
```
Vérifier l’ajout de l’utilisateur :
```bash
ansible all -m command -a "id alice"
```
Modifier l’utilisateur pour changer son home directory et shell :
```bash
ansible all -m user -a "name=alice home=/home/alice_new shell=/sbin/nologin"
```
Supprimer l’utilisateur et le groupe après test :
```bash
ansible all -m user -a "name=alice state=absent remove=yes"
```
```bash
ansible all -m group -a "name=devops state=absent"
```
💡 Note : L’option remove=yes supprime également le home directory de l’utilisateur.

### 3.3 : Gestion des fichiers et répertoires

Créer un répertoire /opt/project avec permissions spécifiques :
```bash
ansible all -m file -a "path=/opt/project state=directory mode=0755 owner=root group=root"
```
Créer un fichier vide file.txt dans ce répertoire :
```bash
ansible all -m file -a "path=/opt/project/file.txt state=touch mode=0644 owner=root group=root"
```
Ajouter une ligne dans le fichier avec lineinfile :
```bash
ansible all -m lineinfile -a "path=/opt/project/file.txt line='LAB Ansible RHCE' state=present"
```
Supprimer une ligne spécifique si nécessaire :
```bash
ansible all -m lineinfile -a "path=/opt/project/file.txt regexp='LAB Ansible RHCE' state=absent"
```

### 3.4 : Gestion des services et packages

Installer le package httpd sur tous les hôtes (RHEL/CentOS) :
```bash
ansible all -m yum -a "name=httpd state=present"
```
Démarrer et activer le service httpd :
```bash
ansible all -m service -a "name=httpd state=started enabled=yes"
```
Vérifier l’état du service :
```bash
ansible all -m service -a "name=httpd state=started"
```
Arrêter le service si nécessaire :
```bash
ansible all -m service -a "name=httpd state=stopped"
```
**Note** Le module service est idempotent, il ne redémarrera pas un service déjà démarré.

### 3.5 : Copier et récupérer des fichiers

Copier un fichier depuis le Control Node vers les Managed Nodes :
```bash
ansible all -m copy -a "src=/home/ansible/config.txt dest=/etc/config.txt mode=0644 owner=root group=root"
```
Récupérer un fichier depuis les Managed Nodes vers le Control Node :
```bash
ansible all -m fetch -a "src=/var/log/messages dest=/home/ansible/logs/{{ inventory_hostname }}/messages"
```

### 3.6 : Nettoyage et vérification finale

Vérifier tous les utilisateurs existants :
```bash
ansible all -m command -a "tail -n 5 /etc/passwd"
```
Vérifier tous les services actifs :
```bash
ansible all -m shell -a "systemctl list-units --type=service --state=running"
```
Supprimer les fichiers et répertoires créés pour le lab :
```bash
ansible all -m file -a "path=/opt/project state=absent"
```
```bash
ansible all -m file -a "path=/home/ansible/logs state=absent"
```
