# Chapitre 4 : Les Playbooks Ansible

## 1. Introduction

Les **playbooks Ansible** sont le cœur de l’automatisation avec Ansible.  
Contrairement aux **commandes ad-hoc**, qui servent à des actions rapides et ponctuelles, les playbooks permettent de :

- automatiser des tâches complexes
- appliquer des configurations reproductibles
- gérer plusieurs hôtes et rôles

Un playbook est écrit en **YAML** et décrit :
- **quoi faire**
- **sur quels hôtes**
- **et comment le faire**

---

<p align="center">
  <img src="images/Ad-Hoc commands.png" alt="Ansible Overview" width="600">
</p>


## 2. Généralités sur les Playbooks

### 2.1 Extension des fichiers

Les playbooks utilisent les extensions suivantes :

- `.yml`
- `.yaml`

Les deux sont valides, mais **`.yml` est le plus utilisé** dans la pratique et lors de l’examen **RHCE**.

---

### 2.2 Vérification et exécution d’un playbook

✔️ **Vérifier la syntaxe du playbook**  
(Cette étape est très importante à l’examen RHCE)

```bash
ansible-playbook --syntax-check playbook.yml
```

Pour exécuter le playbook on fait:
```bash
 ansible-playbook playbook.yml
```

## 3. Structure d’un Playbook Ansible
Structure générale:
```yaml
- name: Description du playbook
  hosts: all
  become: true
  tasks:
    - name: Description de la tâche 1
      module_name:
        argument1: value
        argument2: value

    - name: Description de la tâche 2
      module_name:
        argument: value
```

| Élément  | Description                                             |
| -------- | ------------------------------------------------------- |
| `name`   | Description lisible du playbook ou de la tâche          |
| `hosts`  | Groupe ou hôte ciblé                                    |
| `become` | Exécuter les tâches avec les privilèges root            |
| `tasks`  | Liste des tâches à exécuter                             |
| `module` | Action Ansible (`user`, `yum`, `service`, `file`, etc.) |

---

## 4. LABS : Comprendre et maîtriser les Playbooks Ansible (EX294)

Les laboratoires suivants sont conçus pour :
- comprendre la structure des playbooks
- manipuler différents modules essentiels
- apprendre à travailler avec plusieurs plays
---

### LAB 1 : Création et configuration d’un utilisateur système
Créer un utilisateur nommé **developer** sur tous les hôtes avec :
- un UID spécifique
- un shell restreint
- un répertoire personnel personnalisé

#### Playbook : `create_user.yml`
```yaml
- name: Create developer user
  hosts: all
  become: true
  tasks:
    - name: Create a developer user
      user:
        name: devops
        uid: 3500
        shell: /sbin/nologin
        home: /opt/devops
        state: present
```

---

### LAB 2 : Suppression propre d’un compte utilisateur
Supprimer l’utilisateur testuser ainsi que son répertoire personnel sur tous les hôtes.
```yaml
- name: Remove testuser account
  hosts: all
  become: true
  tasks:
    - name: Delete testuser and home directory
      user:
        name: testuser
        state: absent
        remove: yes
```

---

### LAB 3 : Installation et gestion d’un service réseau (chronyd)
Installer le service chrony, démarrer et activer le service, Vérifier son état
```yaml

- name: Install and manage chronyd
  hosts: all
  become: true
  tasks:
    - name: Install chrony
      yum:
        name: chrony
        state: present

    - name: Enable and start chronyd
      service:
        name: chronyd
        state: started
        enabled: yes
```

---

LAB 4 : Gestion d’un fichier de configuration avec contenu personnalisé
Créer un fichier /etc/motd contenant un message de bienvenue sur tous les hôtes.
```yaml
- name: Configure MOTD message
  hosts: all
  become: true
  tasks:
    - name: Create MOTD file
      copy:
        content: |
          Welcome to this server
          Managed by Ansible
        dest: /etc/motd
```
---

## 5. Playbooks avec plusieurs plays
Un playbook Ansible peut contenir plusieurs plays.
Chaque play peut cibler un groupe d’hôtes différent et représenter une étape distincte du déploiement.

Pourquoi utiliser plusieurs plays ?
appliquer des configurations sur différents hôtes,
séparer les responsabilités,
améliorer la lisibilité.

### LAB 5 : Playbook multi-plays – Configuration et validation
Premier play : installer et démarrer httpd
Deuxième play : vérifier l’accès au service web

```yaml
- name: Install web service
  hosts: webservers
  become: true
  tasks:
    - name: Install httpd
      yum:
        name: httpd
        state: present

    - name: Start and enable httpd
      service:
        name: httpd
        state: started
        enabled: yes

- name: Validate web service
  hosts: webservers
  become: false
  tasks:
    - name: Check web service
      uri:
        url: http://localhost
```

---

## LAB 6 : Playbook multi-plays – Déploiement et test d’un site web (Type RHCE)
Ce laboratoire a pour but de comprendre :
- l’utilisation des **playbooks multi-plays**
- le déploiement automatisé d’un **serveur web Apache**
- la gestion des **services système**
- la configuration du **firewall**
- la validation du service via un test HTTP

Ce scénario est **très proche des exercices RHCE (EX294)**.

---
### Premier play : Déploiement du site web
Le premier play permet de :
- installer les packages **httpd** et **firewalld**
- démarrer et activer les services
- créer une page web `index.html`
- autoriser le service HTTP dans le firewall

### Deuxième play : Test du site web
Le second play permet de :
- vérifier que le site web est accessible
- afficher le contenu de la page via une requête HTTP

---

### Pré-requis

Le module `firewalld` nécessite l’installation de la collection **ansible.posix**.

Installer la collection sur la machine de contrôle :

```bash
ansible-galaxy collection install ansible.posix
```

Playbook : web_multiplay.yml
```yaml
- hosts: all  
  become: true
  tasks:      
  - name: install packages
    yum:      
      name:   
      - firewalld 
      - httpd 
      state: present
  - name: enable httpd service
    service:  
      name:  httpd  
      state: started
      enabled: yes  
  - name: enable firewalld service
    service:         
      name: firewalld
      state: started 
      enabled: yes   
  - name: create index
    copy:            
      content: "hello"
      dest: /var/www/html/index.html
      setype: httpd_sys_content_t
  - name: allow httpd service
    firewalld:       
      service: http  
      permanent: yes 
      state: enabled 
      immediate: yes
- hosts: all
  become: false
  tasks:
  - name: webpage test
    uri:
      url: http://node1
```

Résultat attendu

Les services httpd et firewalld sont installés et actifs
Le firewall autorise le trafic HTTP
La page web est accessible via http://node1
Le contenu "hello" est affiché
