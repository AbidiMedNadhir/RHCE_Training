# Chapitre 11 : Roles & Rhel-system-roles

### Introduction
Dans ce chapitre, nous allons découvrir l’un des concepts **les plus importants d’Ansible pour l’examen RHCE** :  
**les rôles Ansible** et les **rhel-system-roles** fournis par Red Hat.

Les rôles permettent :
- d’organiser le code Ansible de manière propre et réutilisable,
- de standardiser les configurations,
- de simplifier la maintenance et l’automatisation à grande échelle.

### Création de rôle Apache
``` bash
ansible-galaxy --help
```
Pour créer un rôle Ansible, vous utilisez la commande ansible-galaxy init. Cela génère une structure de répertoires pour le rôle avec des fichiers de configuration de base.
#### initialiser le rôle
``` bash
cd /path/to/your/custom/roles
ansible-galaxy init apache
```
or you can specify the path
``` bash
ansible-galaxy init --init-path= /path/to/your/custom/roles apache
```
``` bash
cd apache
tree
```
- `handlers/` : Contient des handlers, qui sont des tâches déclenchées par d'autres tâches (par exemple, pour redémarrer un service).
- `tasks/` : Contient les tâches principales du rôle.
- `templates/` : Contient les fichiers de template Jinja2 pour la génération dynamique de fichiers.
- `vars/` : Contient les variables spécifiques au rôle.
####  Définir les Tâches dans tasks/main.yml
``` bash


- name: install packages
  yum:
    name: "{{item}}"
    state: present
  loop: "{{packages}}"

- name: Enable services
  service:
    name: "{{item}}"
    state: started
    enabled: yes
  loop: "{{packages}}"
- name: Allow webserver service
  firewalld:
    service: http
    state: enabled
    permanent: yes
    immediate: yes

- name: Create index file from index.html.j2
  template:
    src: index.html.j2
    dest: /var/www/html/index.html
  notify:
    - restart_service
```
NB: `notify` notifie le handler restart_webservers lorsque le fichier de template change.
#### Créer le Template Jinja2 dans templates/index.html.j2
``` bash
Welcome to {{ ansible_fqdn }} on {{ ansible_default_ipv4.address }}
```
#### Définir le Handler pour Redémarrer le Service
``` bash
- name: restart_service
  service:
    name: httpd
    state: restarted
```
#### Créer variable packages dans vars/main.yml
``` bash
packages:
- firewalld
- httpd
```

#### Créer un Playbook Utilisant le Rôle
``` bash
vim apache.yml
```
``` bash
- name: Install apache from apache-role
  hosts: all
  become: true
  roles:
    - sample-apache
```
### Rhel-system-roles
`rhel-system-roles` est un ensemble de rôles Ansible fournis par Red Hat pour simplifier la gestion des systèmes RHEL (Red Hat Enterprise Linux). Voici un guide simple pour installer et utiliser ces rôles, et comprendre leur documentation.  
#### Installation des Rôles rhel-system-roles
``` bash
yum install rhel-system-roles
```
you can whether
``` bash

sudo cp -r /usr/share/ansible/roles/* /home/ansible/roles1
```
or add the path directly in the ansible.cfg
``` bash
[defaults]
remote_user=ansible
inventory=/home/ansible/inventory
roles_path=/home/ansible/roles1:/home/ansible/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles
[privilege_escalation]
become=true
```
Cette commande télécharge et installe les rôles dans le répertoire des rôles d'Ansible (souvent ~/.ansible/roles ou un répertoire défini dans votre configuration).
#### Documentation des Rôles
La documentation des rôles rhel-system-roles se trouve généralement dans `/usr/share/doc/rhel-system-roles`. Vous pouvez explorer cette documentation pour obtenir des informations sur les `variables disponibles ` et les `configurations possibles` dans le fichier`README.md`.
``` bash
ls /usr/share/doc/rhel-system-roles
```
#### Rôle timesync
Ce rôle configure le service de synchronisation de l'heure (NTP ou chrony).
Exemple de Playbook utilisant timesync :
``` bash
cd /usr/share/doc/rhel-system-roles/timesync
```
``` bash
cat example-timesync-playbook
```

``` bash
- hosts: all
  vars:
    timesync_ntp_servers:
    - hostname: @server
      iburst: yes
  roles:
  - rhel-system-roles.timesync
```
#### Rôle selinux
Ce rôle configure les paramètres de SELinux.
Exemple de Playbook utilisant selinux :
``` bash
cat example-selinux-playbook.yml
```
``` bash
- name: Manage SELinux Booleans
  hosts: all
  become: yes
  vars: 
    selinux_booleans:
    - name: httpd_can_network_connect
      state: true
  roles:
    - rhel-system-roles.selinux
```
un autre exemple:
``` bash
- name: Manage SELinux policy example
  hosts: all
  vars: 
    selinux_policy: targeted
    selinux_state: enforcing
  roles: 
    - rhel-system-roles.selinux
```
