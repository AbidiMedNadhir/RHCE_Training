<h1 align="center" style="color: red;"> Chapitre 13: Configuration de Yum Repositories</h1>

## Introduction

Dans ce chapitre, nous allons apprendre à configurer des dépôts Yum sur plusieurs nœuds à l’aide d’Ansible.
L’objectif est de créer et gérer deux dépôts Yum essentiels pour RHEL : baseos et appstream, puis de montrer comment exécuter des playbooks Ansible pour automatiser cette configuration sur l’ensemble des systèmes gérés.
  
## Configuration des Repositories Yum

Nous allons créer un playbook Ansible pour configurer deux dépôts Yum : **BaseOS** et **AppStream**.

### Création du playbook 

faire un playbook qui crée et configure les deux dépôts Yum sur tous les nœuds définis dans l'inventaire d'Ansible.  
Le playbook se compose de deux tâches principales :

1. **Créer le dépôt BaseOS** :
   - Nom : `baseos`
   - Description : `Baseos Description`
   - URL : `http://content/rhel9.0/x86_64/dvd/BaseOS`
   - GPG : Activé
   - Clé GPG : `http://content.example.com/rhel9.0/x86_64/dvd/RPM-GPG-KEY-redhatrelease`
   - Le dépôt est activé.

2. **Créer le dépôt AppStream** :
   - Nom : `appstream`
   - Description : `App Description`
   - URL : `http://content/rhel9.0/x86_64/dvd/AppStream`
   - GPG : Activé
   - Clé GPG : `http://content.example.com/rhel9.0/x86_64/dvd/RPM-GPG-KEY-redhatrelease`
   - Le dépôt est activé.

```yaml
vim yumrepo.yml

- name: Creating yum repository
  hosts: all
  tasks:
    - name: Create BaseOS Repository
      ansible.builtin.yum_repository:
        name: "baseos"
        description: "Baseos Description"
        baseurl: http://content/rhel9.0/x86_64/dvd/BaseOS
        gpgcheck: yes
        gpgkey: http://content.example.com/rhel9.0/x86_64/dvd/RPM-GPG-KEY-redhatrelease
        enabled: yes

    - name: Create Appstream Repository
      ansible.builtin.yum_repository:
        name: "appstream"
        description: "App Description"
        baseurl: http://content/rhel9.0/x86_64/dvd/AppStream
        gpgcheck: yes
        gpgkey: http://content.example.com/rhel9.0/x86_64/dvd/RPM-GPG-KEY-redhatrelease
        enabled: yes
```
