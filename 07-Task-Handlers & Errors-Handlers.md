# Chapitre 7 : Gestion des Handlers et des Erreurs

Dans les playbooks Ansible réels, deux problématiques reviennent très souvent :

1. **Exécuter certaines actions uniquement lorsqu’un changement a eu lieu**
2. **Gérer proprement les erreurs sans arrêter toute l’automatisation**

Pour répondre à ces besoins, Ansible fournit deux mécanismes essentiels :

- **Les Task Handlers** → réagir aux changements
- **La gestion des erreurs** → contrôler les échecs avec élégance

Ces notions sont **fondamentales pour écrire des playbooks robustes**.

---

## 1. Task Handlers : 

Sans handler, on pourrait redémarrer un service à chaque exécution du playbook…  
Mais cela serait : inutile et risqué en production  

Un **handler** permet d’exécuter une tâche **uniquement si une autre tâche a modifié l’état du système**.

### Principe de fonctionnement

1. Une tâche modifie quelque chose (`changed`)
2. Elle déclenche un **notify**
3. Le handler correspondant est exécuté **à la fin du play**

Si aucune modification n’a eu lieu → le handler **n’est pas exécuté**

---

### Structure d’un handler

```yaml
tasks:
  - name: Exemple de tâche
    module:
      option: value
    notify: Nom du handler

handlers:
  - name: Nom du handler
    module:
      option: value
```

### Exemple : redémarrer un service

Installer Apache,

Modifier son fichier de configuration,

Redémarrer le service uniquement si nécessaire.

#### Playbook avec handler:

```yaml
- name: Apache configuration with handler
  hosts: web
  become: true

  tasks:
    - name: Installer Apache
      yum:
        name: httpd
        state: present
      notify: restart apache

    - name: Déployer la configuration Apache
      template:
        src: httpd.conf.j2
        dest: /etc/httpd/conf/httpd.conf
      notify: restart apache

  handlers:
    - name: restart apache
      service:
        name: httpd
        state: restarted
```


**Note** :

Le handler est exécuté une seule fois, même si plusieurs tâches le notifient.

Les handlers sont exécutés à la fin du play.

Si le fichier de configuration ne change pas → pas de redémarrage.

---

## 2. Errors Handlers : 

Par défaut, Ansible adopte une stratégie fail-fast :

 une erreur → arrêt du playbook

### Ignorer certaines erreurs

#### ignore_errors

Permet de continuer même si une tâche échoue.

```yaml
- name: Commande volontairement erronée
  command: /bin/false
  ignore_errors: yes
```

#### ignore_unreachable

Permet de continuer le play même si certains hôtes sont inaccessibles.

```yaml
- hosts: all
  ignore_unreachable: yes
  tasks:
    - name: Exécuter sur les hôtes joignables
      command: echo "Host reachable"
```

### Les blocs (block) : structurer et contrôler l’exécution

Un block permet de :

regrouper plusieurs tâches

appliquer une condition globale

gérer les erreurs proprement

##### Block avec condition

```yaml
- hosts: all
  tasks:
    - block:
        - name: Installer Apache
          yum:
            name: httpd
            state: present

        - name: Démarrer Apache
          service:
            name: httpd
            state: started
      when: ansible_os_family == "RedHat"
```

#### Gestion des erreurs avec rescue
Principe

**block** → tâches principales

**rescue** → exécuté si une erreur survient

```yaml
- hosts: localhost
  tasks:
    - block:
        - name: Commande risquée
          command: /bin/false

        - name: Ne sera pas exécutée
          command: echo "OK"
      rescue:
        - name: Gestion de l’erreur
          debug:
            msg: "Une erreur a été détectée"
```

#### Section always : garantir une action finale

Les tâches dans always sont exécutées quoi qu’il arrive.

```yaml
- hosts: localhost
  tasks:
    - block:
        - name: Tâche pouvant échouer
          command: /bin/false
      always:
        - name: Toujours exécutée
          debug:
            msg: "Fin du traitement"
```

#### block + rescue + always

```yaml
- hosts: localhost
  tasks:
    - block:
        - name: Étape principale
          command: /bin/false
      rescue:
        - name: En cas d’échec
          debug:
            msg: "Erreur détectée"
      always:
        - name: Nettoyage final
          debug:
            msg: "Toujours exécuté"
```

---

#### Exemple : Handler + Error Handling

Objectif :

Installer Apache,

Déployer une configuration,

Redémarrer Apache uniquement si nécessaire, 

Gérer une erreur d’installation secondaire.


```yaml
- name: Web service with handlers and error control
  hosts: web
  become: true

  tasks:
    - name: Installer Apache
      yum:
        name: httpd
        state: present
      notify: restart apache

    - name: Installer un paquet optionnel
      block:
        - name: Paquet pouvant échouer
          yum:
            name: fake_package
            state: present
      rescue:
        - name: Paquet de secours
          debug:
            msg: "Paquet non critique ignoré"
      always:
        - name: Log de fin
          debug:
            msg: "Traitement terminé"

  handlers:
    - name: restart apache
      service:
        name: httpd
        state: restarted
```
