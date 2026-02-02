# Chapitre 9 : Sécurisation des données avec Ansible Vault

Ansible Vault est un mécanisme intégré à Ansible permettant de :

chiffrer des fichiers entiers,

chiffrer des variables,

protéger les secrets sans casser l’automatisation.

L’accès se fait via un mot de passe Vault.

---

### Principe de fonctionnement :

1. Le fichier est chiffré avec un mot de passe.

2. Ansible ne peut le lire que si le mot de passe est fourni.

3. Le déchiffrement est temporaire en mémoire (sécurisé).

Le fichier reste toujours chiffré sur le disque.

---

### Créer un fichier chiffré : 

```bash
ansible-vault create secrets.yml
```

Ansible demande un mot de passe,

Le fichier s’ouvre dans l’éditeur,

Le contenu est automatiquement chiffré à la sauvegarde.

Exemple de contenu :

```yaml
db_password: StrongPassword123
```

### Vérifier le chiffrement :
```bash
cat secrets.yml
```

Résultat :
```json
$ANSIBLE_VAULT;1.1;AES256
6132396533656333393161306130303430...
```

Les données sont illisibles.

### Consulter et modifier un fichier Vault

Lire sans modifier :
```bash
ansible-vault view secrets.yml
```

Modifier un fichier chiffré :
```bash
ansible-vault edit secrets.yml
```

### Chiffrer / Déchiffrer un fichier existant

**Chiffrer** un fichier en clair :

```bash
ansible-vault encrypt vars.yml
```

**Déchiffrer** un fichier : 

```bash
ansible-vault decrypt vars.yml
```

### Changer le mot de passe Vault (Rekey)

```bash
ansible-vault rekey secrets.yml
```

### Chiffrer une seule variable (encrypt_string)

```bash
ansible-vault encrypt_string 'P@ssw0rd!' --name 'user_password'
```

## Exécution d'un playbook avec des fichiers chiffrés :

Pour exécuter un playbook contenant des fichiers chiffrés, vous pouvez utiliser les options **--ask-vault-password** ou **--vault-password-file**.

#### Option --ask-vault-password

Cette option vous invite à entrer le mot de passe pour déchiffrer le fichier.

```bash
ansible-playbook playbook.yml --ask-vault-password
```

#### Option --vault-password-file 

Cette option utilise un fichier contenant le mot de passe pour déchiffrer le fichier.

```bash
ansible-playbook playbook.yml --vault-password-file vault_password
```

## Exemple : Création d’un utilisateur avec Ansible Vault

### Création du fichier chiffré (mot de passe)

Nous créons un fichier chiffré contenant le mot de passe de l’utilisateur.

```bash
ansible-vault create password.yml
```

Contenu du fichier :

```yaml
password: "StrongPassword123"
```

Playbook **create_user.yml** :

```yaml
- name: Création d’un utilisateur avec mot de passe sécurisé
  hosts: all
  become: true

  vars_files:
    - user.yml
    - password.yml

  tasks:
    - name: Créer l'utilisateur Linux
      user:
        name: "{{ username }}"
        password: "{{ password | password_hash('sha512') }}"
        state: present
```

### Exécution du playbook avec Ansible Vault

Méthode interactive:

```bash
ansible-playbook create_user.yml --ask-vault-pass
```

Ansible demande le mot de passe Vault pour déchiffrer password.yml.

#### Méthode avec fichier de mot de passe :

```bash
 ~/.vault_pass.txt
```

conntenu du fichier :

```bash
MyVaultPassword
```


```bash
chmod 600 ~/.vault_pass.txt
```

```bash
ansible-playbook create_user.yml --vault-password-file ~/.vault_pass.txt
```

