# Chapitre 8 : Gestion des tâches planifiées (Cron Jobs) 

Dans l’administration système, il est très courant d’avoir besoin d’exécuter des tâches automatiquement à des moments précis ou à intervalles réguliers :

sauvegardes,

nettoyage de fichiers temporaires,

vérifications périodiques,

génération de rapports,

tâches de maintenance.

Sous Linux, ces tâches sont gérées par le service cron.
Avec Ansible, il est possible d’administrer les cron jobs de manière centralisée et automatisée, sans modifier manuellement les fichiers crontab.

---

### Rappels sur Cron sous Linux

Un cron job est défini par :

```bash
minute heure jour_du_mois mois jour_de_la_semaine commande
```

Exemple :

*/5 * * * * /usr/bin/backup.sh

Chaque utilisateur possède sa propre crontab, et le service crond se charge de l’exécution.

---

## Le module cron

Ansible fournit le module :

```bash
ansible.builtin.cron
```

Ce module permet de :

créer un cron job

modifier un cron job existant

supprimer un cron job

gérer la crontab d’un utilisateur spécifique

garantir l’idempotence (pas de doublons)

### Paramètres du module cron

| Paramètre | Description                                     |
| --------- | ----------------------------------------------- |
| `name`    | Identifiant unique du cron job (très important) |
| `user`    | Utilisateur propriétaire du cron job            |
| `minute`  | Minute d’exécution                              |
| `hour`    | Heure d’exécution                               |
| `day`     | Jour du mois                                    |
| `month`   | Mois                                            |
| `weekday` | Jour de la semaine                              |
| `job`     | Commande à exécuter                             |
| `state`   | `present` ou `absent`                           |

**Note** : Le champ **name** est obligatoire : Ansible l’utilise pour retrouver et gérer le cron job.

### Exemple : créer un cron job périodique

Créer un cron job qui écrit un message dans les logs système toutes les 2 minutes.

Playbook : cron_logger.yml

```yaml
- name: Manage cron jobs with Ansible
  hosts: all
  become: true

  tasks:
    - name: Create a cron job that logs a message
      ansible.builtin.cron:
        name: "Log training status"
        user: student
        minute: "*/2"
        job: logger "Automation training in progress"
        state: present
```

Le cron job sera créé une seule fois.

Toute modification future sera automatiquement prise en compte.

### Suppression d’un cron job :

Supprimer un cron job existant sans toucher aux autres entrées.

```yaml
- name: Remove old cron job
  ansible.builtin.cron:
    name: "Weekly cleanup"
    user: root
    state: absent
```
Ansible supprime uniquement le cron job correspondant à ce nom.

### Gestion conditionnelle des cron jobs

Créer un cron job uniquement sur les serveurs du groupe dev.

```yaml
- name: Cron job only on dev servers
  ansible.builtin.cron:
    name: "Dev log task"
    user: student
    minute: "*/5"
    job: logger "Dev server cron job"
  when: "'dev' in group_names"
```

---

### Vérification sur les nœuds :

Pour vérifier la présence d’un cron job :

```bash
ansible all -m command -a "crontab -u student -l"
```

