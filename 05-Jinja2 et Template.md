# Chapitre 5 : Jinja2 et Templates Ansible

## 1. Introduction

Dans Ansible, **Jinja2** est le moteur de template utilisé pour générer des fichiers dynamiques (configuration, rapports, fichiers système…).

Au lieu d’écrire des fichiers statiques avec le module `copy`, on utilise le module **`template`**, qui permet de :

* insérer des **variables** dans les fichiers
* utiliser des **conditions** (if / else)
* générer du contenu avec des **boucles**
* produire des fichiers adaptés à chaque hôte

---

## 2. Principe de fonctionnement

Un template Ansible est un fichier avec l’extension **`.j2`**.

Processus général :

1. Le template `.j2` est stocké sur le **control node**
2. Ansible remplace les expressions Jinja2 par leurs valeurs
3. Le fichier final est copié sur les **hôtes distants**

---

## 3. Syntaxe Jinja2 – Les bases

### Variables

Les variables sont entourées par `{{ }}`.

```jinja
Hello {{ username }}
```

Exemple :

```yaml
vars:
  username: alice
```

Résultat rendu :

```
Hello alice
```

---

## 4. Boucles Jinja2

Les boucles servent à générer du contenu répétitif dynamiquement.

### Boucle simple

```jinja
{% for item in services %}
- {{ item }}
{% endfor %}
```

Variables Ansible :

```yaml
services:
  - httpd
  - firewalld
  - sshd
```

Résultat :

```
- httpd
- firewalld
- sshd
```

---

### Boucle avec dictionnaire

```yaml
users:
  - name: alice
    role: admin
  - name: bob
    role: user
```

```jinja
{% for user in users %}
User={{ user.name }} Role={{ user.role }}
{% endfor %}
```

---

## 5. Conditions Jinja2

Les conditions permettent de contrôler le contenu généré.

### Condition simple

```jinja
{% if is_admin %}
ROLE=admin
{% else %}
ROLE=user
{% endif %}
```

---

### Condition avec facts Ansible

```jinja
{% if ansible_os_family == 'RedHat' %}
PACKAGE_MANAGER=yum
{% else %}
PACKAGE_MANAGER=apt
{% endif %}
```

---

## 6. Commentaires Jinja2

Les commentaires ne sont **pas rendus** dans le fichier final.

```jinja
{# Ceci est un commentaire Jinja2 #}
```

---

## 7. Le module template en Ansible

### Syntaxe générale

```yaml
- name: Render template
  template:
    src: template.j2
    dest: /path/to/file
```

---

### Exemple 

#### Template (`motd.j2`)

```jinja
Welcome on {{ ansible_hostname }}
```

#### Playbook

```yaml
- name: Generate MOTD
  hosts: all
  tasks:
    - name: Create motd file
      template:
        src: motd.j2
        dest: /etc/motd
```

---

## 8. Exemple : Génération dynamique du fichier /root/myhosts

### QUESTION (Type RHCE / EX294)

Créer un fichier **`/root/myhosts.j2`** sur le **control node** contenant les entrées locales suivantes :

```text
127.0.0.1 localhost localhost.localdomain localhost4 localhost4.localdomain4
::1 localhost localhost.localdomain localhost6 localhost6.localdomain6
```

Ensuite, créer un playbook Ansible nommé **`hosts.yml`** dans **`/home/ansible`**, permettant de générer sur les hôtes du groupe **dev** un fichier **`/root/myhosts`**
ayant le contenu suivant (l’ordre des hôtes n’a pas d’importance) :

```text
127.0.0.1 localhost localhost.localdomain localhost4 localhost4.localdomain4
::1 localhost localhost.localdomain localhost6 localhost6.localdomain6
10.0.2.1 node1.example.com node1
10.0.2.2 node2.example.com node2
10.0.2.3 node3.example.com node3
10.0.2.4 node4.example.com node4
10.0.2.5 node5.example.com node5
```

**Note** : utilisez la variable spéciale **`hostvars`** pour récupérer les informations (IP, FQDN, hostname) de tous les hôtes de l’inventaire.

---

### Solution

### Template Jinja2 (`myhosts.j2`)

```jinja
127.0.0.1 localhost localhost.localdomain localhost4 localhost4.localdomain4
::1 localhost localhost.localdomain localhost6 localhost6.localdomain6

{% for host in groups.all %}
{{ hostvars[host].ansible_default_ipv4.address }} {{ hostvars[host].ansible_fqdn }} {{ hostvars[host].ansible_hostname }}
{% endfor %}
```

---

### Playbook Ansible (`hosts.yml`)

```yaml
- name: Generate hosts file using template
  hosts: all
  tasks:
    - name: Deploy hosts file on dev nodes
      template:
        src: /home/ansible/myhosts.j2
        dest: /root/myhosts
      when: "'dev' in group_names"
```

---

## 9. Variables spéciales :

### hostvars

`hostvars` est un dictionnaire contenant les variables de **tous les hôtes**.

```jinja
{{ hostvars['node1'].ansible_hostname }}
```

### inventory_hostname vs ansible_hostname

| Variable             | Description                     |
| -------------------- | ------------------------------- |
| `inventory_hostname` | Nom défini dans l’inventaire    |
| `ansible_hostname`   | Nom réel de la machine distante |


