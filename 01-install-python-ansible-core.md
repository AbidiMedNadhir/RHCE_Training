# 1. Install Python & Ansible Core

## Introduction

Dans ce chapitre, nous allons préparer l'environnement pratique d'Ansible requis pour la certification **RHCE (EX294)**.

L'objectif est de mettre en place un **Control Node fonctionnel**, des **Managed Nodes accessibles via SSH**, et de configurer les outils nécessaires pour commencer à écrire vos premiers **playbooks Ansible**.

---

## Notions clés d'Ansible

Avant de configurer l'environnement, il est important de comprendre les termes essentiels :

- **Control Node (Serveur Ansible)** : machine principale où Ansible est installé et depuis laquelle toutes les commandes et automatisations sont exécutées.
- **Managed Nodes (Target Nodes)** : serveurs cibles gérés par Ansible, accessibles via SSH, Python installé mais Ansible non requis.
- **Playbooks** : fichiers YAML décrivant les tâches à exécuter sur les hôtes (installation de paquets, services, fichiers, utilisateurs…).
- **Inventaire** : fichier listant tous les hôtes gérés par Ansible, organisé en groupes pour faciliter la gestion.
- **ansible.cfg** : fichier de configuration global d'Ansible qui définit l'inventaire, l'utilisateur distant et les paramètres sudo.
- **SSH sans mot de passe** : connexion sécurisée entre le Control Node et les Managed Nodes, permettant à Ansible d'exécuter des tâches sans demander de mot de passe.

---

## Scénario pratique

Pour rendre les étapes plus concrètes, utilisons l'infrastructure suivante :

| Rôle         |   Nom de la machine  |  Adresse IP  |
|--------------|----------------------|--------------|
| Control Node |     control-node     | 192.168.1.10 |
| Managed Node |        node1         | 192.168.1.11 |
| Managed Node |        node2         | 192.168.1.12 |
| Managed Node |        node3         | 192.168.1.13 |

Le **Control Node** exécutera toutes les tâches Ansible et se connectera aux **Managed Nodes** via SSH sans mot de passe.

---

## Étape 1 : Installation de Python 3

Ansible nécessite **Python 3** sur toutes les machines.
Vérifiez la présence de Python 3 :

```bash
python3 --version
```
Si Python 3 n'est pas installé sur une machine (Control Node ou Managed Node), exécutez la commande suivante :
```bash
sudo yum install -y python3
```

## Étape 2 : Installation d'Ansible Core (Control Node uniquement)
Sur le Control Node, installez Ansible Core :
```bash
yum install ansible-core
```
Vérifiez l'installation :
```bash
ansible --version
```

## Étape 3 : Création de l'utilisateur Ansible
Pour une automatisation sécurisée, créez un utilisateur dédié nommé ansible sur toutes les machines (Control Node et Managed Nodes).
```bash
useradd ansible
passwd ansible
```
```bash
vim /etc/sudoers.d/ansible
ansible     ALL=(ALL) 	NOPASSWD: ALL
```

## Étape 4 : Définition des noms des machines
Pour simplifier l'inventaire et améliorer la lisibilité, éditez le fichier /etc/hosts sur le Control Node :
```bash
vim /etc/hosts
192.168.1.10 control-node
192.168.1.11 node1
192.168.1.12 node2
192.168.1.13 node3

```

## Étape 5 : Configuration SSH sans mot de passe
Sur le Control Node, connectez-vous avec l'utilisateur ansible :
```bash
su - ansible
```
Génération de la clé SSH
```bash
ssh-keygen
ls .ssh
```
Copie de la clé vers les Managed Nodes
```bash
ssh-copy-id ansible@node1
ssh-copy-id ansible@node2
ssh-copy-id ansible@node3

```
Test de la connexion SSH
```bash
ssh ansible@node1
ssh ansible@node2
ssh ansible@node3
```
⚠️ Aucun mot de passe ne doit être demandé à cette étape.

## Étape 6 : Création du fichier d'inventaire
Sur le Control Node, créez le fichier d'inventaire :
```bash
vim ~/inventory
```
Exemple d'inventaire simple :
```bash
[Group1]
node1
node2

[Group2]
node3
```

## Étape 7 : Création du fichier ansible.cfg
Sur le Control Node, créez le fichier de configuration Ansible :
```bash
vim ~/ansible.cfg
```
Ajoutez le contenu suivant :
```bash
[defaults]
remote_user=ansible
inventory=/home/ansible/inventory
[privilege_escalation]
become=true
```

Ce fichier permet :
de définir l'inventaire par défaut,
d'utiliser automatiquement l'utilisateur ansible,
d'activer l'élévation de privilèges (sudo),
d'exécuter les commandes en tant que root sans mot de passe.

## Étape 8 : Test de l'environnement
Test SSH

```bash
ssh ansible@node1
```
Vérification de l'inventaire Ansible
```bash
ansible all --list-hosts
```
Test Ansible avec le module ping
```bash
ansible node1 -m ping
```
Résultat attendu :
```json
node1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```



