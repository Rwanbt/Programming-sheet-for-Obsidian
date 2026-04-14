# Automatisation avec Ansible

## Qu'est-ce qu'Ansible ?

**Ansible** est un outil d'automatisation open source qui permet de **configurer des serveurs**, **deployer des applications** et **orchestrer des taches** de maniere simple et repetable.

> [!tip] Analogie
> Imagine que tu es le **directeur d'une chaine de restaurants** :
> - Tu as 50 restaurants a gerer, et chacun doit avoir la **meme configuration** : meme menu, meme decoration, memes equipements.
> - **Sans Ansible** = tu te deplaces dans chaque restaurant, un par un, pour tout installer a la main. Ca prend des semaines et tu oublies forcement quelque chose.
> - **Avec Ansible** = tu ecris une **liste d'instructions** ("installer four X, accrocher tableau Y, imprimer menu Z") et tu l'envoies a **tous les restaurants en meme temps**. Chaque gerant execute les instructions localement.
>
> Ansible, c'est le **memo du directeur** envoye a tous les restaurants. Pas besoin d'installer un logiciel special dans chaque restaurant (agentless), il suffit de pouvoir les joindre (SSH).

---

## Ansible vs Terraform

| Critere | Terraform | Ansible |
|---|---|---|
| **Role principal** | Provisionner l'infrastructure | Configurer les serveurs |
| **Ce qu'il fait** | Cree serveurs, reseaux, BDD, DNS | Installe paquets, deploie apps, gere fichiers |
| **Approche** | Declaratif (HCL) | Imperatif / Procedurale (YAML) |
| **State** | Fichier terraform.tfstate | Pas de state (agentless) |
| **Idempotent** | Oui (natif) | Oui (si modules utilises correctement) |
| **Communication** | API cloud (HTTP) | SSH / WinRM |
| **Agent requis** | Non | Non (agentless) |
| **Quand l'utiliser** | "Je veux 3 serveurs Ubuntu" | "J'installe Nginx sur ces 3 serveurs" |

> [!info] Terraform + Ansible = le workflow complet
> 1. **Terraform** cree les serveurs, reseaux et BDD dans le cloud
> 2. **Ansible** se connecte en SSH aux serveurs et les configure
>
> Terraform = le **constructeur** qui batit la maison. Ansible = le **decorateur** qui installe les meubles et la peinture.

```
┌───────────────────────────────────────────────────────────────────┐
│                    WORKFLOW TERRAFORM + ANSIBLE                     │
│                                                                     │
│  Terraform          Infrastructure              Ansible             │
│  ┌──────────┐      ┌──────────────┐           ┌──────────┐        │
│  │ main.tf  │─────>│  3 serveurs  │<──── SSH ─│playbook  │        │
│  │          │      │  1 VPC       │           │  .yml    │        │
│  │ Cree     │      │  1 RDS       │           │          │        │
│  │ l'infra  │      │  1 S3        │           │Configure │        │
│  └──────────┘      └──────────────┘           │ l'infra  │        │
│                                                └──────────┘        │
│                                                                     │
│  Provisioning ──────────────> Configuration Management              │
└───────────────────────────────────────────────────────────────────┘
```

---

## Architecture d'Ansible

### Le modele Push (sans agent)

```
┌──────────────────────────────────────────────────────────────────┐
│                  ARCHITECTURE ANSIBLE                              │
│                                                                    │
│   Control Node                     Managed Nodes                  │
│   (ta machine)                     (les serveurs)                 │
│                                                                    │
│   ┌──────────────┐                ┌──────────────┐               │
│   │   Ansible    │──── SSH ──────>│  Serveur 1   │               │
│   │              │──── SSH ──────>│  Serveur 2   │               │
│   │  Playbooks   │──── SSH ──────>│  Serveur 3   │               │
│   │  Inventory   │──── SSH ──────>│  Serveur 4   │               │
│   └──────────────┘                └──────────────┘               │
│                                                                    │
│   ● Pas d'agent a installer sur les serveurs                      │
│   ● Seul SSH est necessaire (deja installe partout)               │
│   ● Le control node pousse les commandes (push model)             │
└──────────────────────────────────────────────────────────────────┘
```

| Composant | Description |
|---|---|
| **Control Node** | La machine depuis laquelle tu lances Ansible (ton PC, un serveur CI/CD) |
| **Managed Nodes** | Les serveurs cibles que tu veux configurer |
| **Inventory** | La liste des serveurs cibles (IPs, noms, groupes) |
| **Playbook** | Le fichier YAML qui decrit les taches a executer |
| **Module** | Une unite de travail (installer un paquet, copier un fichier, etc.) |
| **Role** | Un ensemble reutilisable de taches, templates, variables |

---

## Installer Ansible

### Sur macOS

```bash
pip3 install ansible
# ou
brew install ansible
```

### Sur Ubuntu/Debian

```bash
sudo apt update
sudo apt install -y software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install -y ansible
```

### Via pip (recommande, toutes plateformes)

```bash
pip3 install ansible

# Verifier l'installation
ansible --version
# ansible [core 2.17.x]
```

> [!warning] Ansible ne fonctionne PAS nativement sur Windows
> Le **control node** doit etre Linux ou macOS. Sur Windows, utilise **WSL2** (Windows Subsystem for Linux) pour lancer Ansible. Les managed nodes Windows sont supportes via WinRM.

---

## L'Inventory (inventaire)

L'**inventory** est le fichier qui liste les serveurs que tu veux gerer.

### Format INI (classique)

```ini
# inventory.ini

# Serveurs web
[webservers]
web1.example.com
web2.example.com
192.168.1.50

# Serveurs base de donnees
[dbservers]
db1.example.com ansible_host=10.0.0.10 ansible_user=admin
db2.example.com ansible_host=10.0.0.11 ansible_user=admin

# Groupe de groupes
[production:children]
webservers
dbservers

# Variables pour tout le groupe webservers
[webservers:vars]
http_port=80
nginx_version=1.24
```

### Format YAML (moderne)

```yaml
# inventory.yml
all:
  children:
    webservers:
      hosts:
        web1.example.com:
          http_port: 80
        web2.example.com:
          http_port: 8080
      vars:
        nginx_version: "1.24"

    dbservers:
      hosts:
        db1.example.com:
          ansible_host: 10.0.0.10
          ansible_user: admin
        db2.example.com:
          ansible_host: 10.0.0.11
          ansible_user: admin

    production:
      children:
        webservers:
        dbservers:
```

### Variables d'hote importantes

| Variable | Description | Exemple |
|---|---|---|
| `ansible_host` | Adresse IP reelle | `10.0.0.10` |
| `ansible_user` | Utilisateur SSH | `ubuntu` |
| `ansible_port` | Port SSH | `2222` |
| `ansible_ssh_private_key_file` | Cle SSH | `~/.ssh/id_rsa` |
| `ansible_become` | Utiliser sudo | `true` |
| `ansible_python_interpreter` | Chemin Python | `/usr/bin/python3` |

---

## Commandes Ad-Hoc

Les commandes **ad-hoc** permettent d'executer une action rapide sur un ou plusieurs serveurs sans ecrire un playbook.

```bash
# Syntaxe : ansible <pattern> -i <inventory> -m <module> -a "<arguments>"

# Tester la connexion a tous les serveurs
ansible all -i inventory.ini -m ping

# Verifier l'uptime
ansible all -i inventory.ini -m shell -a "uptime"

# Installer un paquet
ansible webservers -i inventory.ini -m apt -a "name=nginx state=present" --become

# Copier un fichier
ansible all -i inventory.ini -m copy -a "src=./app.conf dest=/etc/app.conf"

# Redemarrer un service
ansible webservers -i inventory.ini -m service -a "name=nginx state=restarted" --become

# Voir les facts (informations systeme)
ansible web1.example.com -i inventory.ini -m setup
```

> [!info] Le module ping
> `ansible all -m ping` ne fait PAS un ping ICMP. C'est un module Ansible qui verifie que :
> 1. La connexion SSH fonctionne
> 2. Python est installe sur le serveur cible
> 3. Ansible peut executer des modules
>
> Si ca retourne `pong`, tout est bon.

---

## Les Playbooks

Un **playbook** est un fichier YAML qui decrit une serie de taches a executer sur des serveurs.

### Structure d'un playbook

```yaml
# playbook.yml
---
# Un playbook contient un ou plusieurs PLAYS
- name: Configurer les serveurs web        # Play 1
  hosts: webservers                         # Sur quels serveurs
  become: yes                               # Utiliser sudo
  vars:                                     # Variables du play
    http_port: 80

  tasks:                                    # Liste des taches
    - name: Installer Nginx                 # Tache 1
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Demarrer Nginx                  # Tache 2
      service:
        name: nginx
        state: started
        enabled: yes

- name: Configurer les bases de donnees     # Play 2
  hosts: dbservers
  become: yes

  tasks:
    - name: Installer PostgreSQL
      apt:
        name: postgresql
        state: present
```

### Executer un playbook

```bash
# Lancer le playbook
ansible-playbook -i inventory.ini playbook.yml

# Mode verbose (debug)
ansible-playbook -i inventory.ini playbook.yml -v    # ou -vv, -vvv

# Limiter a certains hotes
ansible-playbook -i inventory.ini playbook.yml --limit web1.example.com

# Mode check (dry-run, ne modifie rien)
ansible-playbook -i inventory.ini playbook.yml --check

# Afficher les taches qui vont s'executer
ansible-playbook -i inventory.ini playbook.yml --list-tasks
```

```
PLAY [Configurer les serveurs web] *****************************************

TASK [Gathering Facts] *****************************************************
ok: [web1.example.com]
ok: [web2.example.com]

TASK [Installer Nginx] *****************************************************
changed: [web1.example.com]
changed: [web2.example.com]

TASK [Demarrer Nginx] ******************************************************
ok: [web1.example.com]
ok: [web2.example.com]

PLAY RECAP *****************************************************************
web1.example.com    : ok=3    changed=1    unreachable=0    failed=0
web2.example.com    : ok=3    changed=1    unreachable=0    failed=0
```

> [!info] Les statuts des taches
> - **ok** : la tache a ete verifiee et rien n'a change (deja dans l'etat voulu)
> - **changed** : la tache a modifie quelque chose sur le serveur
> - **failed** : la tache a echoue
> - **skipped** : la tache a ete ignoree (condition `when` non remplie)
> - **unreachable** : impossible de se connecter au serveur

---

## Les modules courants

### Gestion des paquets

```yaml
# apt (Debian/Ubuntu)
- name: Installer Nginx
  apt:
    name: nginx
    state: present          # present = installe, absent = desinstalle
    update_cache: yes

# yum (RHEL/CentOS)
- name: Installer httpd
  yum:
    name: httpd
    state: latest           # latest = derniere version

# Installer plusieurs paquets
- name: Installer des paquets
  apt:
    name:
      - nginx
      - curl
      - htop
      - git
    state: present
```

### Gestion des fichiers

```yaml
# Copier un fichier
- name: Copier la configuration Nginx
  copy:
    src: files/nginx.conf
    dest: /etc/nginx/nginx.conf
    owner: root
    group: root
    mode: '0644'

# Creer un repertoire
- name: Creer le dossier de l'application
  file:
    path: /var/www/myapp
    state: directory
    owner: www-data
    mode: '0755'

# Creer un lien symbolique
- name: Lien vers le site actif
  file:
    src: /etc/nginx/sites-available/myapp
    dest: /etc/nginx/sites-enabled/myapp
    state: link
```

### Gestion des services

```yaml
# Demarrer et activer un service
- name: Demarrer Nginx
  service:
    name: nginx
    state: started
    enabled: yes         # Demarrer au boot

# Redemarrer un service
- name: Redemarrer Nginx
  service:
    name: nginx
    state: restarted
```

### Autres modules utiles

```yaml
# Executer une commande
- name: Verifier la version de Python
  command: python3 --version
  register: python_version

# Cloner un repo Git
- name: Cloner l'application
  git:
    repo: https://github.com/org/myapp.git
    dest: /var/www/myapp
    version: main

# Gerer un conteneur Docker
- name: Lancer un conteneur Redis
  docker_container:
    name: redis
    image: redis:7
    state: started
    ports:
      - "6379:6379"

# Creer un utilisateur
- name: Creer l'utilisateur deploy
  user:
    name: deploy
    groups: sudo,www-data
    shell: /bin/bash
    create_home: yes
```

---

## Les Variables

### Ou definir les variables

```
Priorite (du moins au plus prioritaire) :
                                                    
  1. Role defaults (defaults/main.yml)              ← Moins prioritaire
  2. Inventory vars (group_vars/, host_vars/)       
  3. Play vars                                      
  4. Role vars (vars/main.yml)                      
  5. set_fact / register                            
  6. Extra vars (-e "var=value")                    ← Plus prioritaire
```

### Variables dans le playbook

```yaml
- name: Deployer l'application
  hosts: webservers
  vars:
    app_name: myapp
    app_port: 8080
    deploy_path: /var/www/{{ app_name }}

  tasks:
    - name: Creer le dossier
      file:
        path: "{{ deploy_path }}"
        state: directory
```

### group_vars et host_vars

```
inventaire/
├── inventory.yml
├── group_vars/
│   ├── all.yml              # Variables pour TOUS les serveurs
│   ├── webservers.yml       # Variables pour le groupe webservers
│   └── dbservers.yml        # Variables pour le groupe dbservers
└── host_vars/
    ├── web1.example.com.yml # Variables pour web1 uniquement
    └── db1.example.com.yml  # Variables pour db1 uniquement
```

```yaml
# group_vars/webservers.yml
nginx_version: "1.24"
http_port: 80
app_root: /var/www/html

# host_vars/web1.example.com.yml
http_port: 8080    # Override pour ce serveur specifique
```

### register et set_fact

```yaml
# register : capturer le resultat d'une tache
- name: Verifier si Nginx est installe
  command: nginx -v
  register: nginx_result
  ignore_errors: yes

- name: Afficher le resultat
  debug:
    msg: "Nginx version: {{ nginx_result.stdout }}"
  when: nginx_result.rc == 0

# set_fact : creer une variable dynamiquement
- name: Definir l'URL complete
  set_fact:
    app_url: "http://{{ ansible_default_ipv4.address }}:{{ http_port }}"

- name: Afficher l'URL
  debug:
    msg: "Application accessible sur {{ app_url }}"
```

### ansible_facts

```yaml
# Ansible collecte automatiquement des facts sur chaque serveur
- name: Afficher des informations systeme
  debug:
    msg: |
      OS: {{ ansible_distribution }} {{ ansible_distribution_version }}
      IP: {{ ansible_default_ipv4.address }}
      CPU: {{ ansible_processor_cores }} cores
      RAM: {{ ansible_memtotal_mb }} MB
      Hostname: {{ ansible_hostname }}
```

---

## Les Templates (Jinja2)

Ansible utilise le moteur de templates **Jinja2** pour generer des fichiers dynamiques.

### Syntaxe Jinja2

```
{{ variable }}              → Afficher une variable
{% if condition %}          → Condition
{% for item in list %}      → Boucle
{# commentaire #}          → Commentaire (ignore)
{{ var | default("val") }}  → Filtre (valeur par defaut)
{{ var | upper }}           → Filtre (majuscules)
```

### Exemple de template Nginx

```yaml
# templates/nginx.conf.j2

server {
    listen {{ http_port }};
    server_name {{ server_name }};

    root {{ app_root }};
    index index.html;

{% if ssl_enabled %}
    listen 443 ssl;
    ssl_certificate /etc/ssl/certs/{{ domain }}.crt;
    ssl_certificate_key /etc/ssl/private/{{ domain }}.key;
{% endif %}

    # Upstream servers
{% for backend in backend_servers %}
    upstream backend_{{ loop.index }} {
        server {{ backend.host }}:{{ backend.port }};
    }
{% endfor %}

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### Utiliser le module template

```yaml
- name: Deployer la configuration Nginx
  template:
    src: templates/nginx.conf.j2
    dest: /etc/nginx/sites-available/{{ app_name }}
    owner: root
    group: root
    mode: '0644'
  notify: Redemarrer Nginx
```

> [!example] Template vs Copy
> - **copy** : copie un fichier tel quel, sans modification
> - **template** : copie un fichier en remplacant les variables `{{ }}` par leurs valeurs
>
> Utilise `template` des que ton fichier de configuration contient des valeurs qui changent selon l'environnement.

---

## Les Handlers

Un **handler** est une tache speciale qui ne s'execute que si elle est **notifiee** par une autre tache, et uniquement **une fois** a la fin du play.

```yaml
- name: Configurer le serveur web
  hosts: webservers
  become: yes

  tasks:
    - name: Installer Nginx
      apt:
        name: nginx
        state: present

    - name: Deployer la configuration
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/sites-available/default
      notify: Redemarrer Nginx              # Notifie le handler

    - name: Deployer le certificat SSL
      copy:
        src: files/ssl.crt
        dest: /etc/ssl/certs/app.crt
      notify: Redemarrer Nginx              # Meme notification

  handlers:
    - name: Redemarrer Nginx                # Execute UNE SEULE FOIS
      service:                              # meme si notifie 2 fois
        name: nginx
        state: restarted
```

> [!info] Pourquoi les handlers ?
> Si la config Nginx ET le certificat SSL changent, on ne veut pas redemarrer Nginx **deux fois**. Le handler s'execute une seule fois a la fin, meme s'il est notifie plusieurs fois. C'est plus efficace et plus sur.

---

## Les Roles

Un **role** est un moyen d'organiser un playbook en composants reutilisables avec une structure standardisee.

### Structure d'un role

```
roles/
└── nginx/
    ├── tasks/
    │   └── main.yml          # Taches principales
    ├── handlers/
    │   └── main.yml          # Handlers
    ├── templates/
    │   └── nginx.conf.j2     # Templates Jinja2
    ├── files/
    │   └── index.html        # Fichiers statiques
    ├── vars/
    │   └── main.yml          # Variables du role (haute priorite)
    ├── defaults/
    │   └── main.yml          # Valeurs par defaut (basse priorite)
    └── meta/
        └── main.yml          # Metadonnees et dependances
```

### Creer un role avec ansible-galaxy

```bash
# Creer la structure d'un role
ansible-galaxy init roles/nginx

# Structure creee automatiquement
roles/nginx/
├── README.md
├── defaults/
│   └── main.yml
├── files/
├── handlers/
│   └── main.yml
├── meta/
│   └── main.yml
├── tasks/
│   └── main.yml
├── templates/
├── tests/
│   ├── inventory
│   └── test.yml
└── vars/
    └── main.yml
```

### Exemple de role Nginx

```yaml
# roles/nginx/defaults/main.yml
nginx_port: 80
nginx_server_name: localhost
nginx_root: /var/www/html

# roles/nginx/tasks/main.yml
---
- name: Installer Nginx
  apt:
    name: nginx
    state: present
    update_cache: yes

- name: Deployer la configuration
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/sites-available/default
  notify: Redemarrer Nginx

- name: Activer le site
  file:
    src: /etc/nginx/sites-available/default
    dest: /etc/nginx/sites-enabled/default
    state: link

- name: Demarrer Nginx
  service:
    name: nginx
    state: started
    enabled: yes

# roles/nginx/handlers/main.yml
---
- name: Redemarrer Nginx
  service:
    name: nginx
    state: restarted
```

### Utiliser un role dans un playbook

```yaml
# playbook.yml
---
- name: Configurer les serveurs web
  hosts: webservers
  become: yes

  roles:
    - nginx
    - { role: app, app_name: "monapp", app_port: 8080 }
```

---

## Conditionnels avec when

```yaml
# Condition simple
- name: Installer Nginx (Debian uniquement)
  apt:
    name: nginx
    state: present
  when: ansible_os_family == "Debian"

# Condition avec variable
- name: Activer SSL
  template:
    src: ssl.conf.j2
    dest: /etc/nginx/conf.d/ssl.conf
  when: ssl_enabled | default(false)

# Conditions multiples (AND)
- name: Installer sur Ubuntu 22.04+
  apt:
    name: nginx
    state: present
  when:
    - ansible_distribution == "Ubuntu"
    - ansible_distribution_major_version | int >= 22

# Condition OR
- name: Installer sur Debian ou Ubuntu
  apt:
    name: nginx
    state: present
  when: ansible_distribution == "Debian" or ansible_distribution == "Ubuntu"

# Condition basee sur le resultat d'une tache
- name: Verifier si le fichier existe
  stat:
    path: /etc/nginx/nginx.conf
  register: nginx_conf

- name: Sauvegarder la config existante
  copy:
    src: /etc/nginx/nginx.conf
    dest: /etc/nginx/nginx.conf.backup
    remote_src: yes
  when: nginx_conf.stat.exists
```

---

## Boucles avec loop

```yaml
# Boucle simple
- name: Installer plusieurs paquets
  apt:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - curl
    - htop
    - git

# Boucle avec dictionnaires
- name: Creer des utilisateurs
  user:
    name: "{{ item.name }}"
    groups: "{{ item.groups }}"
    shell: "{{ item.shell }}"
  loop:
    - { name: alice, groups: sudo, shell: /bin/bash }
    - { name: bob, groups: www-data, shell: /bin/sh }
    - { name: charlie, groups: "sudo,www-data", shell: /bin/bash }

# Boucle avec with_items (ancienne syntaxe, toujours supportee)
- name: Copier des fichiers
  copy:
    src: "{{ item }}"
    dest: /etc/app/
  with_items:
    - app.conf
    - db.conf
    - cache.conf

# Boucle avec index
- name: Creer des fichiers numerotes
  file:
    path: "/tmp/file_{{ idx }}.txt"
    state: touch
  loop: "{{ range(1, 6) | list }}"
  loop_control:
    loop_var: idx
```

---

## Les Tags

Les **tags** permettent d'executer ou d'ignorer certaines taches d'un playbook.

```yaml
- name: Configurer le serveur
  hosts: webservers
  become: yes

  tasks:
    - name: Installer Nginx
      apt:
        name: nginx
        state: present
      tags:
        - install
        - nginx

    - name: Deployer la configuration
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/sites-available/default
      tags:
        - config
        - nginx

    - name: Deployer l'application
      git:
        repo: https://github.com/org/app.git
        dest: /var/www/app
      tags:
        - deploy
        - app
```

```bash
# Executer uniquement les taches taguees "config"
ansible-playbook playbook.yml --tags config

# Executer "install" et "deploy"
ansible-playbook playbook.yml --tags "install,deploy"

# Tout executer SAUF "deploy"
ansible-playbook playbook.yml --skip-tags deploy

# Lister les taches et leurs tags
ansible-playbook playbook.yml --list-tags
```

---

## Ansible Vault

**Ansible Vault** permet de **chiffrer** des fichiers ou des variables contenant des secrets (mots de passe, cles API, certificats).

```bash
# Chiffrer un fichier entier
ansible-vault encrypt group_vars/production/secrets.yml

# Dechiffrer un fichier
ansible-vault decrypt group_vars/production/secrets.yml

# Editer un fichier chiffre
ansible-vault edit group_vars/production/secrets.yml

# Chiffrer une seule variable (inline)
ansible-vault encrypt_string 'SuperSecretPassword123' --name 'db_password'
# Resultat :
# db_password: !vault |
#   $ANSIBLE_VAULT;1.1;AES256
#   61626364656667...
```

### Utiliser un fichier chiffre

```yaml
# group_vars/production/secrets.yml (chiffre)
db_password: SuperSecretPassword123
api_key: sk-abc123def456
```

```bash
# Lancer le playbook avec le mot de passe vault
ansible-playbook playbook.yml --ask-vault-pass

# Ou avec un fichier contenant le mot de passe
ansible-playbook playbook.yml --vault-password-file .vault_pass
```

> [!warning] Ne commite JAMAIS .vault_pass dans Git
> Le fichier `.vault_pass` contient le mot de passe en clair. Ajoute-le a `.gitignore`. En CI/CD, passe le mot de passe via une variable d'environnement ou un secret manager.

---

## L'idempotence

### Qu'est-ce que l'idempotence ?

Un module est **idempotent** si l'executer **une fois ou cent fois** produit le **meme resultat**. Ansible ne modifie le serveur que si necessaire.

```
┌────────────────────────────────────────────────────────────────────┐
│                         IDEMPOTENCE                                 │
│                                                                      │
│   1ere execution :     "Installer Nginx"    → changed (installe)    │
│   2eme execution :     "Installer Nginx"    → ok (deja installe)    │
│   3eme execution :     "Installer Nginx"    → ok (deja installe)    │
│                                                                      │
│   → Nginx est installe UNE SEULE FOIS, meme si on lance le         │
│     playbook 100 fois. C'est ca l'idempotence.                      │
└────────────────────────────────────────────────────────────────────┘
```

> [!warning] Les modules command et shell ne sont PAS idempotents
> `command` et `shell` executent la commande a chaque fois, meme si le resultat est deja atteint. Prefere les modules specifiques (`apt`, `service`, `file`) qui savent verifier l'etat avant d'agir.

```yaml
# NON idempotent (s'execute a chaque fois)
- name: Installer Nginx
  command: apt-get install -y nginx

# Idempotent (verifie avant d'agir)
- name: Installer Nginx
  apt:
    name: nginx
    state: present
```

### Le mode check (--check)

```bash
# Dry-run : montre ce qui SERAIT change sans rien modifier
ansible-playbook playbook.yml --check

# Check + diff : montre les differences dans les fichiers
ansible-playbook playbook.yml --check --diff
```

---

## Exemple pratique : configurer un serveur web

### Structure du projet

```
projet-ansible/
├── inventory.yml
├── playbook.yml
├── group_vars/
│   └── webservers.yml
├── roles/
│   └── webserver/
│       ├── tasks/
│       │   └── main.yml
│       ├── handlers/
│       │   └── main.yml
│       ├── templates/
│       │   ├── nginx.conf.j2
│       │   └── app.conf.j2
│       ├── files/
│       │   └── index.html
│       └── defaults/
│           └── main.yml
└── ansible.cfg
```

### inventory.yml

```yaml
all:
  children:
    webservers:
      hosts:
        web1:
          ansible_host: 192.168.1.10
          ansible_user: ubuntu
        web2:
          ansible_host: 192.168.1.11
          ansible_user: ubuntu
```

### group_vars/webservers.yml

```yaml
app_name: monapp
app_port: 8080
domain: example.com
deploy_path: /var/www/{{ app_name }}
repo_url: https://github.com/org/monapp.git
repo_branch: main
```

### roles/webserver/defaults/main.yml

```yaml
nginx_port: 80
nginx_worker_connections: 1024
ssl_enabled: false
```

### roles/webserver/tasks/main.yml

```yaml
---
- name: Mettre a jour le cache apt
  apt:
    update_cache: yes
    cache_valid_time: 3600

- name: Installer les paquets necessaires
  apt:
    name:
      - nginx
      - git
      - curl
    state: present

- name: Creer le repertoire de l'application
  file:
    path: "{{ deploy_path }}"
    state: directory
    owner: www-data
    group: www-data
    mode: '0755'

- name: Cloner le depot de l'application
  git:
    repo: "{{ repo_url }}"
    dest: "{{ deploy_path }}"
    version: "{{ repo_branch }}"
  notify: Redemarrer Nginx

- name: Deployer la configuration Nginx
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/sites-available/{{ app_name }}
    owner: root
    group: root
    mode: '0644'
  notify: Redemarrer Nginx

- name: Activer le site
  file:
    src: /etc/nginx/sites-available/{{ app_name }}
    dest: /etc/nginx/sites-enabled/{{ app_name }}
    state: link
  notify: Redemarrer Nginx

- name: Supprimer le site par defaut
  file:
    path: /etc/nginx/sites-enabled/default
    state: absent
  notify: Redemarrer Nginx

- name: S'assurer que Nginx est demarre
  service:
    name: nginx
    state: started
    enabled: yes
```

### roles/webserver/handlers/main.yml

```yaml
---
- name: Redemarrer Nginx
  service:
    name: nginx
    state: restarted
```

### roles/webserver/templates/nginx.conf.j2

```yaml
server {
    listen {{ nginx_port }};
    server_name {{ domain }};

    root {{ deploy_path }};
    index index.html;

    access_log /var/log/nginx/{{ app_name }}_access.log;
    error_log /var/log/nginx/{{ app_name }}_error.log;

    location / {
        try_files $uri $uri/ =404;
    }

{% if app_port is defined %}
    location /api {
        proxy_pass http://127.0.0.1:{{ app_port }};
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
{% endif %}
}
```

### playbook.yml

```yaml
---
- name: Configurer les serveurs web
  hosts: webservers
  become: yes

  roles:
    - webserver
```

### Execution

```bash
# Tester la connexion
ansible all -i inventory.yml -m ping

# Dry-run
ansible-playbook -i inventory.yml playbook.yml --check --diff

# Deployer
ansible-playbook -i inventory.yml playbook.yml

# Deployer uniquement sur web1
ansible-playbook -i inventory.yml playbook.yml --limit web1
```

---

## Bonnes pratiques

| Pratique | Pourquoi |
|---|---|
| **Utiliser des roles** | Organisation claire, code reutilisable |
| **Variables pour la flexibilite** | Pas de valeurs en dur dans les taches |
| **Vault pour les secrets** | Ne jamais stocker de mots de passe en clair |
| **Mode check avant apply** | `--check --diff` pour verifier avant de modifier |
| **Modules plutot que command/shell** | Idempotence garantie avec les modules natifs |
| **Nommer chaque tache** | `name:` descriptif pour un output lisible |
| **Tags pour le controle** | Executer un sous-ensemble de taches |
| **group_vars / host_vars** | Separer les variables de la logique |
| **Tester avec un inventaire local** | Vagrant ou Docker pour tester les playbooks |
| **Versionner les roles** | Un depot Git par role pour le partage |

> [!tip] Checklist avant de deployer en production
> 1. `ansible-playbook --check --diff` → verifier les changements
> 2. `ansible-playbook --limit un_seul_serveur` → tester sur un serveur
> 3. Si tout est bon → lancer sur tous les serveurs
> 4. Verifier les logs et les statuts (`changed` vs `ok`)

---

## Carte Mentale

```
                          Automatisation
                            Ansible
                               |
           ┌───────────────────┼───────────────────┐
           |                   |                    |
       Fondamentaux         Playbooks             Avance
           |                   |                    |
    ┌──────┼──────┐     ┌─────┼─────┐      ┌──────┼──────┐
    |      |      |     |     |     |      |      |      |
 Agentless Inventory  Tasks Modules Handlers Roles  Vault  Templates
    |      |      |     |           |      |      |      |
   SSH    INI   Ad-hoc apt/yum   notify  Structure Encrypt Jinja2
   Push   YAML  ping   copy     restart  galaxy   decrypt {{ var }}
   Model  Groups       service           init     string  {% if %}
                        template
                        file
                           |
                    ┌──────┼──────┐
                    |      |      |
                Variables Loops  Conditionnels
                    |      |      |
                 vars/   loop    when
                 facts   with_   register
                 register items
                 set_fact
```

---

## Exercices

### Exercice 1 : Premier playbook

Ecris un playbook Ansible qui :
1. Cible le groupe `webservers` de ton inventaire
2. Installe `nginx`, `curl` et `htop`
3. Copie un fichier `index.html` vers `/var/www/html/`
4. S'assure que Nginx est demarre et active au boot
5. Utilise `become: yes` pour les privileges root

### Exercice 2 : Variables et templates

Ameliore l'exercice 1 en :
1. Definissant les variables `app_name`, `http_port` et `server_name` dans `group_vars/`
2. Creant un template `nginx.conf.j2` qui utilise ces variables
3. Ajoutant un handler qui redemarre Nginx quand la config change
4. Ajoutant une condition `when` pour n'installer SSL que si `ssl_enabled` est `true`

### Exercice 3 : Role complet

Transforme l'exercice 2 en un role `webserver` :
1. Cree la structure avec `ansible-galaxy init`
2. Deplace les taches, handlers, templates et variables dans le role
3. Utilise le role depuis un playbook principal
4. Ajoute des tags (`install`, `config`, `deploy`) sur les taches

### Exercice 4 : Deploiement multi-environnement

Cree un projet Ansible avec :
1. Un inventaire avec deux groupes : `staging` et `production`
2. Des `group_vars` differents pour chaque environnement (port, domaine, nombre de workers)
3. Un secret chiffre avec `ansible-vault` pour le mot de passe de la BDD
4. Un playbook qui deploie une application avec les bonnes variables selon l'environnement

---

## Liens

- [[05 - Infrastructure as Code Terraform]] - Creer l'infrastructure avant de la configurer avec Ansible
- [[02 - Services Cloud Essentiels]] - Comprendre les services cloud que tu vas configurer
- [[04 - CI-CD avec GitHub Actions]] - Automatiser le lancement d'Ansible dans un pipeline CI/CD
