# Chapitre 3 : Variables et Facts avec Ansible

## 1. Introduction

Dans ce chapitre, nous allons aborder deux concepts **fondamentaux pour l’examen RHCE (EX294)** et pour toute automatisation avancée avec Ansible :
* **Les variables**, qui permettent de rendre les playbooks dynamiques, réutilisables et maintenables.
* **Les facts**, qui sont des informations système collectées automatiquement sur les hôtes gérés.

---

## 2. Les Variables Ansible

### 2.1 Définition
Une **variable Ansible** est une paire **clé: valeur** utilisée pour stocker des données dynamiques comme :
* noms d’utilisateurs
* chemins de fichiers
* packages
* services
* adresses IP, ports, etc.

Syntaxe générale :
```yaml
variable_name: value
```

Pour utiliser une variable, on l’appelle avec les **doubles accolades** :
```yaml
{{ variable_name }}
```

---

### 2.2 Types de variables

#### a) Variables simples

```yaml
username: alice
shell: /bin/bash
```

#### b) Variables à valeurs multiples

**Liste (array)** :

```yaml
users:
  - alice
  - bob
  - lisa
```

Accès à un élément :

```yaml
{{ users[1] }}  # bob
```

**Dictionnaire** :

```yaml
user:
  name: alice
  uid: 2001
  shell: /bin/bash
```

Accès :

```yaml
{{ user.name }}
{{ user['uid'] }}
```

---

### 2.3 Où définir les variables

#### a) Dans le playbook (vars)

```yaml
- name: Exemple avec vars
  hosts: all
  become: true
  vars:
    username: alice
    user_shell: /bin/bash
  tasks:
    - name: Créer utilisateur
      user:
        name: "{{ username }}"
        shell: "{{ user_shell }}"
```

#### b) Dans un fichier de variables (vars_files)

Fichier `variables.yml` :

```yaml
username: alice
user_shell: /bin/bash
```

Playbook :

```yaml
vars_files:
  - variables.yml
```

#### c) Variables d’hôtes et de groupes

Structure recommandée :
```
inventory
host_vars/
  node1.yml
group_vars/
  webservers.yml
```

---

## 3. Variables spéciales et magiques

Les **variables magiques** sont définies automatiquement par Ansible.

Exemples importants :

| Variable           | Description                      |
| ------------------ | -------------------------------- |
| inventory_hostname | Nom de l’hôte courant            |
| groups             | Tous les groupes de l’inventaire |
| group_names        | Groupes de l’hôte courant        |
| hostvars           | Variables de tous les hôtes      |

Exemple :

```bash
ansible localhost -m debug -a "var=hostvars['node1']"
```

Note: Ces variables sont **en lecture seule**.

---

## 4. Les Facts Ansible
### 4.1 Qu’est-ce qu’un fact ?

Les **facts** sont des variables collectées automatiquement par Ansible sur les hôtes distants :
* système d’exploitation
* adresse IP
* interfaces réseau
* disques
* mémoire
Ils sont stockés dans la variable :
```yaml
ansible_facts
```

---

### 4.2 ansible_facts
Deux notations possibles :

```yaml
ansible_facts['default_ipv4']['address']
ansible_facts.default_ipv4.address
```

Facts courants :

| Fact                         | Utilisation        |
| ---------------------------- | ------------------ |
| ansible_hostname             | Nom de la machine  |
| ansible_distribution         | Distribution Linux |
| ansible_distribution_version | Version            |
| ansible_default_ipv4.address | Adresse IP         |

---

### 4.3 Module setup

Afficher tous les facts :
```bash
ansible node1 -m setup
```

Filtrer les facts :
```bash
ansible node1 -m setup -a "filter=ansible_default_ipv4"
```

Désactiver la collecte automatique :
```yaml
gather_facts: no
```
Puis collecter manuellement :

```yaml
- setup:
```

---

## 5. Utilisation des facts dans les commandes ad-hoc

Afficher le hostname :
```bash
ansible all -m debug -a "var=ansible_hostname"
```

Afficher l’adresse IP :
```bash
ansible all -m debug -a "var=ansible_facts.default_ipv4.address"
```

---

## 6. Lab : Variables & Facts
### LAB1 : Création d’un utilisateur à partir d’un fichier de variables

#### Objectif:
Créer un utilisateur sur tous les hôtes en utilisant **un fichier de variables externe**, afin d’éviter les valeurs en dur dans le playbook.

#### Étape 1 : Créer le fichier de variables
Créer un fichier nommé `user_vars.yml` :
```yaml
username: devuser
user_uid: 3001
user_shell: /bin/bash
```

#### Étape 2 : Créer le playbook
Créer un fichier create_user.yml :
```yaml
- name: Create user using variables
  hosts: all
  become: true
  vars_files:
    - user_vars.yml
  tasks:
    - name: Create user
      user:
        name: "{{ username }}"
        uid: "{{ user_uid }}"
        shell: "{{ user_shell }}"
        state: present
```

#### Étape 3 : Exécuter le playbook
```bash
ansible-playbook create_user.yml
```

Résultat attendu:
L’utilisateur devuser est créé sur tous les hôtes
Le playbook reste inchangé si les valeurs sont modifiées dans le fichier de variables

### Lab 2 : Facts – Affichage de la distribution et de l’adresse IP

#### Étape 1 : Créer le playbook
Créer un fichier show_facts.yml :
```yaml
- name: Display system facts
  hosts: all
  gather_facts: yes
  tasks:
    - name: Show Linux distribution
      debug:
        var: ansible_distribution

    - name: Show IPv4 address
      debug:
        var: ansible_facts.default_ipv4.address
```

#### Étape 2 : Exécuter le playbook
```bash
ansible-playbook show_facts.yml
```

Résultat attendu:
Pour chaque hôte, Ansible affiche :
le nom de la distribution (RHEL, CentOS, Ubuntu…)
l’adresse IPv4 principale

### Lab 3 : Vérification rapide des facts avec des commandes ad-hoc (optionnel)

Afficher la distribution Linux :
```bash
ansible all -m setup -a "filter=ansible_distribution"
```

Afficher l’adresse IPv4 :
```bash
ansible all -m setup -a "filter=ansible_default_ipv4"
```


### Lab 4 : Génération d’un rapport système avec Ansible Facts

Créer un playbook Ansible permettant de générer automatiquement un **rapport système** sur les hôtes gérés (`node1` et `node2`) en utilisant les **facts Ansible**.
Le rapport sera stocké dans un fichier `report.txt` et contiendra des informations dynamiques propres à chaque hôte.

---

Créer un fichier nommé `report.yml` dans `/home/ansible`.
À l’aide de ce playbook :
1. Créer (ou récupérer) un fichier `report.txt` sur `node1` et `node2`
2. Remplir le fichier avec les informations suivantes :
HOST=inventory hostname
MEMORY=total memory in MB
BIOS=bios version
SDA_DISK_SIZE=disk size
SDB_DISK_SIZE=disk size


3. Modifier chaque ligne pour afficher les **valeurs réelles** de chaque hôte en utilisant les **facts Ansible**.

Facts Ansible utilisés:

|     Information      |        Fact Ansible        |
|----------------------|----------------------------|
|      Hostname        |     `ansible_hostname`     |
|  Mémoire totale (MB) |    `ansible_memtotal_mb`   |
|     Version BIOS     |   `ansible_bios_version`   |
|  Taille disque sda   | `ansible_devices.sda.size` |
|  Taille disque sdb   | `ansible_devices.sdb.size` |

---

### Fichier : `/home/ansible/report.yml`

```yaml
- name: Generate system report using Ansible Facts
  hosts: all
  become: true
  gather_facts: yes

  tasks:
    - name: Create report file
      file:
        path: /home/ansible/report.txt
        state: touch
        owner: ansible
        group: ansible
        mode: '0644'

    - name: Set hostname
      lineinfile:
        path: /home/ansible/report.txt
        regexp: '^HOST='
        line: "HOST={{ inventory_hostname }}"
        state: present

    - name: Set total memory
      lineinfile:
        path: /home/ansible/report.txt
        regexp: '^MEMORY='
        line: "MEMORY={{ ansible_memtotal_mb }} MB"
        state: present

    - name: Set BIOS version
      lineinfile:
        path: /home/ansible/report.txt
        regexp: '^BIOS='
        line: "BIOS={{ ansible_bios_version }}"
        state: present

    - name: Set SDA disk size
      lineinfile:
        path: /home/ansible/report.txt
        regexp: '^SDA_DISK_SIZE='
        line: "SDA_DISK_SIZE={{ ansible_devices.sda.size }}"
        state: present
      when: ansible_devices.sda is defined

    - name: Set SDB disk size
      lineinfile:
        path: /home/ansible/report.txt
        regexp: '^SDB_DISK_SIZE='
        line: "SDB_DISK_SIZE={{ ansible_devices.sdb.size }}"
        state: present
      when: ansible_devices.sdb is defined
```

