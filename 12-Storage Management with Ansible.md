<h1 align="center" style="color: red;">Chapitre 12: Gestion de stockage</h1>

### Introduction
👋 Dans cette section, nous allons explorer comment gérer les partitions et les volumes logiques.

### Création de partition
On a besoin de la collection community.general pour les modules lvm
``` bash
ansible-galaxy collection install community.general
```

Pour créer une partition, nous utilisons le module `parted`.
``` bash
ansible-doc parted
```

``` bash
---
- name: Create a new partition
  hosts: all
  become: yes
  tasks:
    - name: Create partition
      parted:
        device: /dev/sda
        number: 1
        state: present
        part_end: 1GiB

```
### Créer plus qu'une partition
créer 2 partitions chacune a la taille 2G
``` bash

- hosts: all
  become: true
  tasks:
  - name: Create a new primary partition with a size of 1g
    parted:
      device: /dev/sdb
      number: 1
        #part_start: 1MiB
      state: present
      part_end: 2GiB
  - name: Create a new primary partition with a size of 1g
    parted:
      device: /dev/sdb
      number: 2
      part_start: 2GiB
      state: present
      part_end: 4GiB
```
### Formater la Partition 
Ensuite, nous formatons cette partition nouvellement créée en ext4 en utilisant le module `filesystem`.
``` bash
ansible-doc filesystem
```
``` bash
---
- name: Format the new partition
  hosts: all
  become: yes
  tasks:
    - name: Format partition to ext4
      filesystem:
        fstype: ext4
        dev: /dev/sda1

```
### Monter la Partition et l'Ajouter à fstab
``` bash
ansible-doc mount
```
``` bash
- name: Mount the new partition
  hosts: all
  become: yes
  tasks:
    - name: Create mount point directory
      file:
        path: /mnt/data
        state: directory

    - name: Mount partition
      mount:
        path: /mnt/data
        src: /dev/sda1
        fstype: ext4
        state: mounted

```
## LAB 1 – Gestion des partitions avec Ansible 

Créer un playbook nommé /home/ansible/partition.yml qui sera exécuté sur tous les nœuds gérés et qui devra effectuer les opérations suivantes :

a) Sur le disque /dev/sdb, créer une partition primaire numéro 1 d’une taille de 1200 Mo, la formater en système de fichiers ext4 et la monter dans le répertoire /srv.

b) Si l’espace disque disponible est insuffisant pour créer une partition de 1200 Mo, afficher le message suivant :
"Could not create partition of that size",
puis créer une partition de 800 Mo à la place.

c) Si le disque /dev/sdb n’existe pas sur le système, afficher le message suivant :
"This disk does not exist."

d) Si la partition est créée avec succès (quelle que soit sa taille), elle doit être formatée en ext4.

``` bash
- name: Partition Playbook
  hosts: all
  tasks:
  - name: check block stroage availablity.
    block: 
      - name: If sdb does not exist
        debug:
          msg: "this disk does not exist."
        when: "'sdb' not in ansible_devices"
      - name: Creating the 1200m partition
        parted:
          device: /dev/sdb
          number: 1
          part_end: 1200MiB
          state: present
        when: "'sdb' in ansible_devices"  
   rescue:
      - name: If there is not enough disk space
        debug:                            
          msg: "Could not create partition of that size"
      - name: Creating the smaller partition
        parted:
          device: /dev/sdb
          number: 1
          part_end: 800MiB
          state: present
    always:
      - name: Creating the ext4 filesystem
        filesystem:
          fstype: ext4
          dev: /dev/sdb1
        when: "'sdb' in ansible _devices"
      - name: Create mount point directory
        file:
          path: /srv
          state: directory
        when: "'sdb' in ansible_devices"
      - name: Mount partition
        mount:
          path: /srv
          src: /dev/sdb1
          fstype: ext4
          state: mounted
        when: "'sdb' in ansible_devices"
``` 

### Création d'un Groupe de Volumes avec lvg
``` bash
ansible-doc lvg
```
``` bash
---
- name: Create a Volume Group
  hosts: all
  become: yes
  tasks:
    - name: Create volume group
      lvg:
        vg: vg_data  # Nom du groupe de volumes
        pvs: /dev/sdb1  # La partition physique à inclure dans le groupe de volumes

```
### Créer un Volume Logique avec lvol
``` bash
ansible-doc lvol
```
``` bash
---
- name: Create a Logical Volume
  hosts: all
  become: yes
  tasks:
    - name: Create logical volume
      lvol:
        vg: vg_data  # Le groupe de volumes auquel appartient le LV
        lv: lv_data  # Nom du volume logique
        size: 500m  # Taille du volume logique

```
### Formater le Volume Logique avec filesystem
``` bash
ansible-doc filesystem
```
``` bash
---
- name: Format the Logical Volume
  hosts: all
  become: yes
  tasks:
    - name: Format logical volume
      filesystem:
        fstype: ext4  # Type de système de fichiers (ex: ext4, xfs)
        dev: /dev/vg_data/lv_data  # Dispositif à formater
```
### Monter le Volume en Utilisant le Module mount
``` bash
ansible-doc mount
```
``` bash
---
- name: Mount the Logical Volume
  hosts: all
  become: yes
  tasks:
    - name: Create mount point directory
      file:
        path: /mnt/data1  # Chemin du point de montage
        state: directory  # État souhaité (directory pour créer un répertoire)
    - name: Mount logical volume
      mount:
        path: /mnt/data1  # Chemin du point de montage
        src: /dev/vg_data/lv_data  # Source du dispositif à monter
        fstype: ext4  # Type de système de fichiers
        state: mounted  # État souhaité (mounted pour monter le volume)
```
## LAB 2 – Gestion du stockage logique (LVM) avec Ansible
Créer un playbook nommé /home/ansible/lvm.yml qui sera exécuté sur tous les nœuds gérés et qui devra réaliser les actions suivantes :

a) Créer un volume logique (Logical Volume) répondant aux exigences ci-dessous :

i. Le volume logique doit être créé dans un groupe de volumes existant (Volume Group).

ii. Le nom du volume logique doit être data.

iii. La taille du volume logique doit être de 1200 MiB.

iv. Le volume logique doit être formaté avec le système de fichiers ext4.

b) Si la taille demandée (1200 MiB) ne peut pas être créée, afficher le message d’erreur suivant :
"Could not create logical volume of that size",
puis créer le volume logique avec une taille de 800 MiB.

c) Si le groupe de volumes (Volume Group) requis n’existe pas, afficher le message suivant :
"Volume group does not exist".

d) Le volume logique ne doit pas être monté (aucune opération de montage ne doit être effectuée).

``` bash
- name: lvm playbook
  hosts: all
  become: true
  tasks:
  - name: checking details
    block:
      - name: if the volume research does not exist
        debug:
          msg: "volume group does not exist"
        when: "'research' not in ansible_lvm.vgs"
      - name: creating the 1200m lvm
        lvol:
          vg: research     
          Lv: data   
          size: 1200m
        when: "'research' in ansible_lvm.vgs"  
    rescue:
    - name: if the requested logical volume size cannot be cretaed
      debug:
        msg: "Could not create logical volume of that size"
    - name: creating the logical volume of 800m
      lvol:
        vg : research
        lv: data
        size: 800m
    always:
    - name: format filesystem
      filesystem:
        fstype: ext4
        dev: /dev/research/data
      when: "'data' in ansible_lvm.lvs"
```
