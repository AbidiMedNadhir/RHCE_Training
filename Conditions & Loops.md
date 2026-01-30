# Chapitre 6 : Conditions et Boucles 

- **Les boucles (loops)** : pour répéter une tâche sur plusieurs éléments
- **Les conditions (when)** : pour exécuter une tâche uniquement si une règle est respectée

Ces notions permettent :
- d’éviter la duplication du code
- de rendre les playbooks plus dynamiques
- d’adapter l’exécution selon l’environnement ou les hôtes

---

## Les boucles (Loops) :

Une boucle permet d’exécuter **la même tâche plusieurs fois**, chaque fois avec une valeur différente.
Ansible utilise le mot-clé `loop` et la variable spéciale `item`.

`item` représente **l’élément courant** de la boucle.

---

### Boucle simple avec une liste définie dans le playbook

Exemple : afficher plusieurs valeurs sans répéter la tâche.

```yaml
- hosts: all
  tasks:
    - name: Afficher une liste simple
      debug:
        msg: "{{ item }}"
      loop:
        - alpha
        - beta
        - gamma
```
**loop** contient la liste
**item** change à chaque itération

### Boucle avec un fichier de variables

Il est recommandé de séparer les données du playbook.

Fichier values.yml:

```yaml
elements:
  - alpha
  - beta
  - gamma
```

Playbook:

```yaml
- hosts: all
  vars_files:
    - values.yml
  tasks:
    - name: Afficher les éléments depuis un fichier
      debug:
        msg: "{{ item }}"
      loop: "{{ elements }}"
```

### Boucle avec une liste de dictionnaires

Très utilisée pour gérer des utilisateurs, services ou configurations complexes.

Fichier users.yml :

```yaml
users:
  - name: user1
    role: dev
  - name: user2
    role: admin
  - name: user3
    role: test
```

Playbook :

```yaml
- hosts: all
  vars_files:
    - users.yml
  tasks:
    - name: Afficher les informations utilisateur
      debug:
        msg: "Utilisateur {{ item.name }} avec le rôle {{ item.role }}"
      loop: "{{ users }}"
```

Accès aux valeurs :
**item.name**
**item.role**

---

### Exemple : installation de services avec une boucle

Fichier services.yml:

```yaml
packages:
  - httpd
  - firewalld
```

Playbook:

```yaml
- hosts: all
  become: true
  vars_files:
    - services.yml

  tasks:
    - name: Installer les paquets requis
      yum:
        name: "{{ item }}"
        state: present
      loop: "{{ packages }}"

    - name: Activer et démarrer les services
      service:
        name: "{{ item }}"
        state: started
        enabled: yes
      loop: "{{ packages }}"
```

---

## Les conditions avec when

Le mot-clé when permet de restreindre l’exécution d’une tâche.
Si la condition est fausse, Ansible affiche skipping.

### Exemple : vérifier l’existence d’un disque

écrire la taille du disque s’il existe, sinon écrire NONE.

```yaml
- hosts: all
  become: true
  tasks:
    - name: Écrire la taille de sda si présent
      lineinfile:
        path: /root/report.txt
        line: "SDA={{ ansible_devices.sda.size }}"
      when: "'sda' in ansible_devices"

    - name: Écrire NONE si sda absent
      lineinfile:
        path: /root/report.txt
        line: "SDA=NONE"
      when: "'sda' not in ansible_devices"
```

**Note**: ansible_devices provient des facts système

---

## Exemple pratique : Déploiement d’un répertoire web avec permissions et SELinux

Créer un playbook Ansible `webdev.yml` permettant de :

- créer un utilisateur **webdev**
- créer un répertoire `/webdev` sur les hôtes du groupe **dev**
- définir les permissions :
  - utilisateur : `rwx`
  - groupe : `rw`
  - autres : aucun accès
- activer le **setgid** sur le répertoire
- créer un lien symbolique vers `/var/www/html/webdev`
- servir une page web contenant le texte **"Development"**
- configurer **SELinux** pour autoriser httpd
- tester l’accès via HTTP

```bash
ansible-galaxy collection install community.general
```

### Playbook : webdev.yml

```yaml
- name: Web development setup
  hosts: dev
  become: true

  tasks:
    - name: Créer l'utilisateur webdev
      user:
        name: webdev
        state: present

    - name: Créer le répertoire /webdev avec permissions et setgid
      file:
        path: /webdev
        state: directory
        owner: webdev
        group: webdev
        mode: '2760'

    - name: Créer un lien symbolique vers /var/www/html/webdev
      file:
        src: /webdev
        dest: /var/www/html/webdev
        state: link

    - name: Créer le fichier index.html
      copy:
        content: "Development"
        dest: /webdev/index.html
        owner: webdev
        group: webdev
        mode: '0644'

    - name: Autoriser httpd à accéder au répertoire /webdev (SELinux)
      community.general.sefcontext:
        target: '/webdev(/.*)?'
        setype: httpd_sys_content_t
        state: present

    - name: Appliquer le contexte SELinux
      command: restorecon -Rv /webdev

    - name: Tester l’accès HTTP
      uri:
        url: http://node1.example.com/webdev/
        status_code: 200
```

### Test manuel

Depuis le control node :

```bash
ansible node1 -m command -a "curl http://node1.example.com/webdev/"
```

Résultat attendu :

```json
Development
```
