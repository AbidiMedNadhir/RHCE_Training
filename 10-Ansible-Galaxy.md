# Chapitre 10 : Gestion des rôles avec Ansible Galaxy

Ansible Galaxy est un composant d’Ansible qui permet d’utiliser des rôles prêts à l’emploi pour faciliter et accélérer l’automatisation des systèmes.
Avec la commande ansible-galaxy, vous pouvez télécharger, installer, organiser et gérer des rôles et des collections Ansible provenant de sources publiques ou privées.

##### Qu’est-ce qu’un rôle Ansible ?

Dans ce contexte, un rôle représente une unité modulable qui regroupe des tâches, variables, handlers et autres fichiers liés à une fonction spécifique.
Le fonctionnement détaillé des rôles sera présenté dans le chapitre suivant.

#### installation de Rôle
``` bash
ansible-config dump | grep -i galaxy_server
```

Pour installer un rôle depuis Ansible Galaxy, utilisez la commande ansible-galaxy install.  
Par exemple, pour installer le rôle geerlingguy.apache, vous pouvez exécuter :
``` bash
ansible-galaxy install geerlingguy.apache
```
**Note** :  
Par défaut, les rôles installés avec ansible-galaxy sont stockés dans le répertoire .ansible/roles. (ansible --version)  
#### Changer l'Emplacement par Défaut des Rôles
``` bash
ansible-config dump | grep -i roles
```

Vous pouvez changer l'emplacement par défaut où ansible-galaxy installe les rôles en utilisant le fichier de configuration `ansible.cfg`.  
Ajoutez cette ligne dans le fichier ansible.cfg :
``` bash
roles_path = /path/to/your/custom/roles
```
Dans cet exemple, les rôles seront installés dans /path/to/your/custom/roles au lieu du répertoire roles par défaut. 
``` bash
ansible-config dump | grep -i roles
``` 
#### Appeler un Rôle dans un Playbook
``` bash
- name: Configure web server
  hosts: all
  roles:
    - geerlingguy.apache
```
Dans ce cas, le rôle geerlingguy.apache est utilisé pour configurer un serveur web Apache.
#### Utilisation de requirements.yml pour Installer Plusieurs Rôles
`requirements.yml` est un fichier utilisé pour définir une liste de rôles et de collections à installer via ansible-galaxy. Ce fichier permet de gérer plusieurs rôles de manière centralisée et cohérente.  
Exemple de requirements.yml :


``` bash 
---
- name: geerlingguy.apache
  src: geerlingguy.apache

- name: geerlingguy.mysql
  src: geerlingguy.mysql

```
#### Installer les Rôles à partir de requirements.yml

``` bash 
ansible-galaxy install -r requirements.yml
```
Cette commande télécharge et installe tous les rôles listés dans le fichier requirements.yml et les place dans le répertoire roles par défaut ou dans le répertoire spécifié dans ansible.cfg.
#### Exemple de Playbook Utilisant Plusieurs Rôles
``` bash 
---
- name: Configure web and database servers
  hosts: all
  become: true
  roles:
    - geerlingguy.apache
    - geerlingguy.mysql
```
## Lab
Créez un fichier nommé requirements.yml dans le répertoire /home/ansible/roles pour installer deux rôles.

La source du premier rôle est **geerlingguy.haproxy** et celle du second rôle est **geerlingguy.php**.

Renommez le premier rôle en **haproxy-role** et le second en **php-role**.

Les rôles doivent être installés dans le répertoire /home/ansible/roles1.

``` bash 
echo "roles_path= /home/ansible/roles1" >> ansible.cfg
```
``` bash 
vim requirements.yml
```
``` bash 
- name: haproxy-role
  src: geerlingguy.haproxy
- name: php-role
  src: geerlingguy.php
```
``` bash 
ansible-galaxy install -r requirements.yml
```

Utilisez ces rôles dans un fichier nommé **role.yml** situé dans /home/ansible/.

Le rôle **haproxy-role** doit être appliqué sur l’hôte **proxy**.

Le rôle **php-role** doit être appliqué sur l’hôte **prod**.

``` bash 

- hosts: proxy
  become: true
  roles:
  - haproxy-role
- hosts: prod
  become: true
  roles:
  - php-role
```
