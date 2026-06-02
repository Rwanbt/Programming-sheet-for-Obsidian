# Linux Security et Hardening

La sécurité d'un système Linux ne se résume pas à l'installation d'un antivirus : elle repose sur une compréhension profonde des mécanismes de permissions, d'authentification et de confinement intégrés au noyau. Ce cours couvre l'ensemble de la surface d'attaque d'un serveur Linux moderne, depuis les bits de permission jusqu'au chiffrement complet du disque, en passant par les frameworks MAC (Mandatory Access Control) que sont SELinux et AppArmor. Maîtriser ces outils, c'est passer du statut d'administrateur réactif à celui d'architecte de sécurité proactif.

---

## 1. Modèle de permissions Linux

### 1.1 Les trois catégories d'identité

Chaque processus et chaque fichier en Linux est associé à trois identités :

| Catégorie | Abréviation | Description |
|-----------|-------------|-------------|
| Propriétaire | `u` (user) | L'utilisateur qui possède le fichier |
| Groupe | `g` (group) | Le groupe propriétaire du fichier |
| Autres | `o` (others) | Tous les autres utilisateurs |
| Tous | `a` (all) | Raccourci u+g+o |

Chaque identité dispose de trois bits de permission :

| Bit | Valeur octale | Sur un fichier | Sur un répertoire |
|-----|---------------|----------------|-------------------|
| `r` (read) | 4 | Lire le contenu | Lister les entrées (`ls`) |
| `w` (write) | 2 | Modifier le contenu | Créer/supprimer des entrées |
| `x` (execute) | 1 | Exécuter comme programme | Traverser (accéder aux fichiers à l'intérieur) |

```bash
# Afficher les permissions d'un fichier
ls -la /etc/passwd
# -rw-r--r-- 1 root root 2847 Jan 15 09:12 /etc/passwd
#  ^^^ ^^^ ^^^
#  |   |   |
#  |   |   └── others : r-- = 4 (lecture seule)
#  |   └────── group  : r-- = 4 (lecture seule)
#  └────────── owner  : rw- = 6 (lecture + écriture)
```

### 1.2 Représentation octale

La notation octale est la plus utilisée dans les scripts et les commandes :

```bash
# Structure : chmod [permissions_octales] fichier
# Chaque chiffre = somme des bits pour une catégorie

chmod 755 script.sh
# 7 = 4+2+1 = rwx  pour le propriétaire
# 5 = 4+0+1 = r-x  pour le groupe
# 5 = 4+0+1 = r-x  pour les autres

chmod 644 document.txt
# 6 = 4+2+0 = rw-  pour le propriétaire
# 4 = 4+0+0 = r--  pour le groupe
# 4 = 4+0+0 = r--  pour les autres

chmod 600 private_key.pem
# 6 = rw-  propriétaire uniquement
# 0 = ---  groupe : aucun accès
# 0 = ---  autres : aucun accès
```

### 1.3 Notation symbolique

```bash
# Ajouter le droit d'exécution au propriétaire
chmod u+x script.sh

# Retirer le droit d'écriture au groupe et aux autres
chmod go-w fichier.conf

# Donner rwx au propriétaire, r-x au groupe, rien aux autres
chmod u=rwx,g=rx,o= programme

# Copier les permissions du propriétaire vers le groupe
chmod g=u fichier.txt
```

### 1.4 Propriétaires et groupes

```bash
# Changer le propriétaire
chown alice fichier.txt

# Changer propriétaire ET groupe
chown alice:developers fichier.txt

# Changer uniquement le groupe
chown :developers fichier.txt
# ou
chgrp developers fichier.txt

# Récursivement sur un répertoire
chown -R www-data:www-data /var/www/html

# Voir les groupes d'un utilisateur
groups alice
id alice
# uid=1001(alice) gid=1001(alice) groups=1001(alice),27(sudo),1002(developers)
```

### 1.5 L'umask — masque de création

L'umask définit les permissions **retirées** lors de la création de nouveaux fichiers et répertoires. C'est un masque soustractif.

```bash
# Afficher l'umask courant
umask
# 0022

# Afficher en notation symbolique
umask -S
# u=rwx,g=rx,o=rx
```

**Calcul des permissions effectives :**

```
Permissions maximales fichier    : 666 (rw-rw-rw-)
Permissions maximales répertoire : 777 (rwxrwxrwx)

Avec umask 022 :
  Fichier    : 666 - 022 = 644 (rw-r--r--)
  Répertoire : 777 - 022 = 755 (rwxr-xr-x)

Avec umask 027 :
  Fichier    : 666 - 027 = 640 (rw-r-----)
  Répertoire : 777 - 027 = 750 (rwxr-x---)

Avec umask 077 (très restrictif) :
  Fichier    : 666 - 077 = 600 (rw-------)
  Répertoire : 777 - 077 = 700 (rwx------)
```

> [!warning]
> Les fichiers ne reçoivent jamais le bit `x` par défaut, même si l'umask l'autorise. Le maximum pour les fichiers est 666, pas 777. Pour rendre un fichier exécutable, il faut toujours un `chmod +x` explicite.

```bash
# Configurer l'umask pour la session courante
umask 027

# Configurer l'umask de manière permanente
# Pour un utilisateur spécifique : ~/.bashrc ou ~/.profile
echo "umask 027" >> ~/.bashrc

# Pour tout le système : /etc/profile ou /etc/login.defs
# Dans /etc/login.defs :
# UMASK 027

# Vérification
umask -S
# u=rwx,g=rx,o=
```

> [!tip]
> Pour un serveur de production, `umask 027` est recommandé : le groupe a accès en lecture (utile pour les services qui tournent sous un groupe commun), mais les autres n'ont aucun accès. Pour un environnement très sensible, utiliser `umask 077`.

### 1.6 Les ACL (Access Control Lists)

Les permissions classiques (rwx) ne permettent de définir des droits que pour un propriétaire, un groupe et les autres. Les ACL étendent ce modèle pour des permissions granulaires par utilisateur ou groupe.

```bash
# Vérifier que les ACL sont supportées (le système de fichiers doit être monté avec acl)
mount | grep acl
# Si non : ajouter l'option "acl" dans /etc/fstab pour la partition concernée

# Installer les outils ACL
apt-get install acl

# Ajouter une permission ACL pour l'utilisateur bob
setfacl -m u:bob:rx /shared/project

# Ajouter une permission ACL pour le groupe marketing
setfacl -m g:marketing:rw /shared/documents

# Afficher les ACL d'un fichier
getfacl /shared/project
# file: shared/project
# owner: alice
# group: developers
# user::rwx
# user:bob:r-x
# group::r-x
# group:marketing:rw-
# mask::rwx
# other::---

# Supprimer une ACL spécifique
setfacl -x u:bob /shared/project

# Supprimer toutes les ACL
setfacl -b /shared/project

# ACL par défaut sur un répertoire (héritées par les nouveaux fichiers)
setfacl -d -m u:bob:rx /shared/project
```

---

## 2. SUID, SGID et Sticky Bit

### 2.1 Le bit SUID (Set User ID)

Quand le bit SUID est positionné sur un **exécutable**, le programme s'exécute avec les privilèges du **propriétaire du fichier**, et non de l'utilisateur qui le lance.

```bash
# Identifier les fichiers avec SUID (représenté par 's' à la place du 'x' du propriétaire)
ls -la /usr/bin/passwd
# -rwsr-xr-x 1 root root 68208 Mar 14 2022 /usr/bin/passwd
#    ^
#    └── 's' = SUID activé + x présent
#         S  = SUID activé + x ABSENT (problème !)

# Cas d'usage légitime : passwd modifie /etc/shadow
# qui appartient à root, mais un utilisateur normal
# doit pouvoir changer son propre mot de passe

# Trouver TOUS les fichiers SUID sur le système
find / -perm -4000 -type f 2>/dev/null
find / -perm -u=s -type f 2>/dev/null

# Résultat typique (à vérifier et réduire au minimum) :
# /usr/bin/passwd
# /usr/bin/sudo
# /usr/bin/su
# /usr/bin/mount
# /usr/bin/umount
# /usr/bin/newgrp
# /usr/lib/openssh/ssh-keysign
```

**Positionner et retirer le SUID :**

```bash
# Activer le SUID
chmod u+s /chemin/vers/programme
chmod 4755 /chemin/vers/programme
# 4 = bit SUID

# Retirer le SUID
chmod u-s /chemin/vers/programme
chmod 0755 /chemin/vers/programme
```

> [!warning]
> Le SUID sur des programmes interprétés (scripts shell, Python...) est **ignoré par le noyau Linux** depuis longtemps pour des raisons de sécurité. Il ne fonctionne que sur les binaires ELF compilés. Sur certains systèmes, il est même complètement ignoré sur les scripts, même si le bit est positionné.

### 2.2 Le bit SGID (Set Group ID)

**Sur un exécutable :** le programme s'exécute avec les privilèges du **groupe propriétaire** du fichier, et non du groupe de l'utilisateur.

**Sur un répertoire :** tous les nouveaux fichiers créés dans ce répertoire héritent du **groupe du répertoire**, et non du groupe primaire de l'utilisateur qui crée le fichier.

```bash
# SGID sur un répertoire partagé
ls -la /var/www/
# drwxrwsr-x 2 root www-data 4096 Jan 15 2024 html
#       ^
#       └── 's' = SGID activé

# Créer un répertoire partagé avec SGID
mkdir /shared/team
chown :developers /shared/team
chmod 2775 /shared/team
# 2 = bit SGID
# 775 = rwxrwxr-x

# Vérification : tout fichier créé dans /shared/team
# appartiendra automatiquement au groupe 'developers'

# Trouver tous les fichiers/répertoires SGID
find / -perm -2000 -type f 2>/dev/null   # fichiers
find / -perm -2000 -type d 2>/dev/null   # répertoires
```

### 2.3 Le Sticky Bit

Sur un **répertoire**, le sticky bit empêche un utilisateur de supprimer ou renommer des fichiers qu'il ne possède pas, même s'il a le droit d'écriture sur le répertoire.

```bash
# Exemple canonique : /tmp
ls -la /
# drwxrwxrwt 12 root root 4096 Jan 15 2024 tmp
#          ^
#          └── 't' = sticky bit activé + x présent
#               T  = sticky bit activé + x ABSENT

# Sans sticky bit sur /tmp :
# alice pourrait supprimer les fichiers de bob dans /tmp !
# Avec sticky bit : seul le propriétaire d'un fichier peut le supprimer

# Activer le sticky bit
chmod +t /shared/uploads
chmod 1777 /shared/uploads
# 1 = sticky bit

# Vérifier
ls -la /shared/uploads
# drwxrwxrwt 2 root root 4096 Jan 15 2024 uploads
```

### 2.4 Tableau récapitulatif des bits spéciaux

| Bit | Valeur octale | Sur fichier | Sur répertoire | Représentation |
|-----|---------------|-------------|----------------|----------------|
| SUID | 4000 | Exécute comme propriétaire | Sans effet (ou setuid sur certains systèmes) | `s` à la place de `x` du propriétaire |
| SGID | 2000 | Exécute comme groupe propriétaire | Héritage de groupe pour nouveaux fichiers | `s` à la place de `x` du groupe |
| Sticky | 1000 | Obsolète sur fichiers | Seul le propriétaire peut supprimer ses fichiers | `t` à la place de `x` des autres |

### 2.5 Risques de sécurité et exploitation

> [!warning]
> Les bits SUID/SGID sont une surface d'attaque majeure. Un binaire SUID root vulnérable à une injection de commande ou un buffer overflow donne immédiatement un accès root à l'attaquant.

**Exemple d'exploitation (à des fins éducatives) :**

```bash
# Si un attaquant trouve un binaire SUID root non standard
find / -perm -4000 -user root -type f 2>/dev/null

# Exemple : find lui-même avec SUID root (mauvaise configuration)
# find . -exec /bin/sh -p \; -quit
# -p = preserve privileges = on obtient un shell root !

# Autre exemple : vim avec SUID root
# vim -c ':!/bin/sh'

# Consulter GTFOBins pour les techniques d'exploitation connues
# https://gtfobins.github.io/
```

**Durcissement — retirer les SUID/SGID inutiles :**

```bash
# Script d'audit des SUID non standards
#!/bin/bash
# Liste des SUID légitimes sur Debian/Ubuntu
LEGIT_SUID=(
    "/usr/bin/passwd"
    "/usr/bin/sudo"
    "/usr/bin/su"
    "/usr/bin/mount"
    "/usr/bin/umount"
    "/usr/bin/newgrp"
    "/usr/bin/chsh"
    "/usr/bin/chfn"
    "/usr/lib/openssh/ssh-keysign"
    "/usr/lib/dbus-1.0/dbus-daemon-launch-helper"
)

echo "=== Fichiers SUID trouvés ==="
while IFS= read -r -d '' fichier; do
    legitime=false
    for leg in "${LEGIT_SUID[@]}"; do
        if [[ "$fichier" == "$leg" ]]; then
            legitime=true
            break
        fi
    done
    if $legitime; then
        echo "[OK]      $fichier"
    else
        echo "[SUSPECT] $fichier"
    fi
done < <(find / -perm -4000 -type f -print0 2>/dev/null)
```

```bash
# Retirer le SUID d'un binaire non nécessaire
chmod u-s /usr/bin/at
chmod u-s /usr/bin/wall
chmod u-s /usr/sbin/pppd

# Monter les partitions avec nosuid pour interdire le SUID dessus
# Dans /etc/fstab :
# /dev/sdb1 /data ext4 defaults,nosuid,noexec,nodev 0 2
```

---

## 3. sudo et sudoers

### 3.1 Fonctionnement de sudo

`sudo` (Superuser DO) permet à des utilisateurs autorisés d'exécuter des commandes avec les privilèges d'un autre utilisateur (généralement root), avec traçabilité complète.

```
Utilisateur lance : sudo commande
        │
        ▼
sudo lit /etc/sudoers (et /etc/sudoers.d/)
        │
        ▼
L'utilisateur est-il autorisé ?
        │
    OUI │                    NON
        ▼                     ▼
Demande le mot        Refuse et loggue
de passe de           dans /var/log/auth.log
l'UTILISATEUR
(pas de root)
        │
        ▼
Exécute la commande
avec les droits requis
et loggue dans
/var/log/auth.log
```

### 3.2 Syntaxe du fichier sudoers

```bash
# TOUJOURS éditer avec visudo (vérifie la syntaxe avant de sauvegarder)
visudo
# ou pour un fichier dans sudoers.d
visudo -f /etc/sudoers.d/developers
```

**Syntaxe générale :**

```
Qui   OùSur=(CommeQui)  Commande(s)
```

```bash
# /etc/sudoers - exemples commentés

# root peut tout faire depuis n'importe où
root    ALL=(ALL:ALL) ALL

# alice peut exécuter toutes les commandes en tant que root
alice   ALL=(ALL) ALL

# bob peut redémarrer apache depuis n'importe quel hôte
bob     ALL=(root) /usr/bin/systemctl restart apache2

# Le groupe sudo peut tout faire (Ubuntu par défaut)
%sudo   ALL=(ALL:ALL) ALL

# Le groupe wheel peut tout faire (Red Hat/CentOS par défaut)
%wheel  ALL=(ALL) ALL

# charlie peut exécuter certaines commandes SANS mot de passe
charlie ALL=(root) NOPASSWD: /usr/bin/systemctl status *, /usr/sbin/service * status

# diane ne peut PAS utiliser un shell interactif avec sudo
diane   ALL=(ALL) ALL, !/bin/bash, !/bin/sh, !/bin/dash

# eve peut uniquement gérer les utilisateurs
eve     ALL=(root) /usr/sbin/useradd, /usr/sbin/userdel, /usr/sbin/usermod
```

**Alias dans sudoers :**

```bash
# Alias d'hôtes
Host_Alias  WEBSERVERS = web1, web2, web3
Host_Alias  DBSERVERS  = db1, db2

# Alias d'utilisateurs
User_Alias  WEBADMINS  = alice, bob, carol
User_Alias  DBADMINS   = dan, eve

# Alias de commandes
Cmnd_Alias  WEBSERVICES = /usr/bin/systemctl restart nginx, \
                          /usr/bin/systemctl restart apache2, \
                          /usr/bin/systemctl reload nginx

Cmnd_Alias  DBSERVICES  = /usr/bin/systemctl restart mysql, \
                          /usr/bin/mysqldump, \
                          /usr/bin/mysql

# Application des alias
WEBADMINS   WEBSERVERS=(root) WEBSERVICES
DBADMINS    DBSERVERS=(root)  DBSERVICES
```

### 3.3 Options de configuration globale

```bash
# Dans /etc/sudoers, section Defaults

# Enregistrer toutes les commandes exécutées via sudo
Defaults    log_output
Defaults    logfile="/var/log/sudo.log"

# Obliger la saisie du mot de passe à chaque fois (ignorer le cache 15 min)
Defaults    timestamp_timeout=0

# Timeout du cache de mot de passe (en minutes)
Defaults    timestamp_timeout=5

# Envoyer un mail à root en cas de tentative non autorisée
Defaults    mail_badpass
Defaults    mailto="security@company.com"

# Afficher un message en cas de mauvais mot de passe
Defaults    badpass_message="Accès refusé. Cet incident sera signalé."

# Limiter le nombre de tentatives de mot de passe
Defaults    passwd_tries=3

# Désactiver la variable TERM (prévient certaines escalades)
Defaults    !visiblepw

# Sécuriser les variables d'environnement
Defaults    env_reset
Defaults    env_keep += "HOME LANG LANGUAGE LC_*"

# Enregistrer les logs dans syslog
Defaults    syslog=auth
```

### 3.4 Abus courants et vecteurs d'escalade de privilèges

> [!warning]
> Ces exemples sont fournis à des fins éducatives pour comprendre comment durcir sudo. Ne jamais tester sur des systèmes sans autorisation.

**1. Commandes dangereuses autorisées dans sudoers :**

```bash
# Si sudoers contient : user ALL=(root) /usr/bin/vim
# Un attaquant peut obtenir un shell root depuis vim :
sudo vim -c ':!/bin/bash'

# Si sudoers contient : user ALL=(root) /usr/bin/find
sudo find . -exec /bin/bash \; -quit

# Si sudoers contient : user ALL=(root) /usr/bin/python3
sudo python3 -c 'import os; os.execv("/bin/bash", ["/bin/bash"])'

# Si sudoers contient : user ALL=(root) /usr/bin/less
sudo less /etc/passwd
# Puis dans less : !bash

# Règle : ne JAMAIS autoriser des éditeurs, interpréteurs,
# ou commandes avec -exec dans sudoers
```

**2. Wildcard dangereux :**

```bash
# Configuration dangereuse dans sudoers :
user ALL=(root) /usr/bin/systemctl * apache2
# Un attaquant peut faire :
sudo systemctl enable --user apache2 --force /tmp/evil.service

# Configuration sécurisée :
user ALL=(root) /usr/bin/systemctl restart apache2, \
                /usr/bin/systemctl reload apache2, \
                /usr/bin/systemctl status apache2
# Les wildcards * dans les arguments sont DANGEREUX
```

**3. Sudo avec LD_PRELOAD :**

```bash
# Si env_reset n'est pas configuré et LD_PRELOAD est dans env_keep :
# Compiler un .so malveillant et le précharger avec sudo
# sudo LD_PRELOAD=/tmp/evil.so commande_autorisée
# Solution : toujours avoir Defaults env_reset dans sudoers
```

### 3.5 Bonnes pratiques sudo

```bash
# 1. Principe du moindre privilège : n'autoriser que ce qui est strictement nécessaire
# 2. Éviter les wildcards dans les chemins de commandes
# 3. Toujours utiliser les chemins absolus dans sudoers
# 4. Utiliser NOPASSWD avec extrême parcimonie (uniquement pour des scripts non interactifs)
# 5. Auditer régulièrement le fichier sudoers

# Vérifier ce qu'un utilisateur peut faire avec sudo
sudo -l -U alice
# ou connecté en tant qu'alice :
sudo -l

# Vérifier la syntaxe de sudoers sans l'ouvrir
visudo -c
# sudoers file: /etc/sudoers: parsed OK
# sudoers file: /etc/sudoers.d/developers: parsed OK
```

---

## 4. PAM (Pluggable Authentication Modules)

### 4.1 Architecture PAM

PAM est un framework qui permet de configurer l'authentification de manière modulaire, sans modifier les applications elles-mêmes. Chaque service (ssh, sudo, login, su...) a son propre fichier de configuration PAM.

```
Application (sshd, sudo, login...)
        │
        ▼
┌────────────────────────────┐
│    Interface PAM (libpam)  │
└────────────────────────────┘
        │
        ▼
Lit /etc/pam.d/[service]
        │
        ▼
┌─────────────────────────────────────────┐
│  Pile de modules PAM (exécutés en ordre) │
│                                          │
│  pam_unix.so    (mots de passe système)  │
│  pam_ldap.so    (authentification LDAP)  │
│  pam_google_auth.so  (2FA Google)        │
│  pam_faillock.so (blocage brute-force)   │
│  pam_limits.so  (limites ressources)     │
│  ...                                     │
└─────────────────────────────────────────┘
```

### 4.2 Structure d'un fichier de configuration PAM

```
type    control     module-path     module-args
```

**Les types (stacks) :**

| Type | Rôle |
|------|------|
| `auth` | Authentifier l'utilisateur (vérifier son identité) |
| `account` | Vérifier si le compte est valide, non expiré, autorisé |
| `password` | Mettre à jour le facteur d'authentification (changer mot de passe) |
| `session` | Actions au début/fin de session (monter home, env, logs) |

**Les contrôles :**

| Contrôle | Comportement |
|----------|-------------|
| `required` | Doit réussir. Si échec, continue les modules suivants mais retourne échec à la fin |
| `requisite` | Doit réussir. Si échec, arrête immédiatement et retourne échec |
| `sufficient` | Si réussite ET aucun module `required` précédent n'a échoué → succès immédiat |
| `optional` | Résultat ignoré sauf si c'est le seul module du type |
| `include` | Inclure une autre configuration PAM |

```bash
# Exemple : /etc/pam.d/sshd (Debian/Ubuntu)
cat /etc/pam.d/sshd
```

```
# /etc/pam.d/sshd
@include common-auth
account    required     pam_nologin.so
@include common-account
session [success=ok ignore=ignore module_unknown=ignore default=bad] pam_selinux.so close
session    required     pam_loginuid.so
session    optional     pam_keyinit.so force revoke
@include common-session
session    optional     pam_motd.so  motd=/run/motd.dynamic
session    optional     pam_motd.so noupdate
session    optional     pam_mail.so standard noenv
session    required     pam_limits.so
session    required     pam_env.so user_readenv=1 envfile=/etc/default/locale
@include common-password
```

### 4.3 Modules PAM importants pour le durcissement

**pam_faillock — Blocage après tentatives échouées :**

```bash
# /etc/pam.d/common-auth (Debian) ou /etc/pam.d/system-auth (RHEL)
# Ajouter avant pam_unix.so :

auth    required      pam_faillock.so preauth silent audit deny=5 unlock_time=1800
auth    [success=1 default=bad]  pam_unix.so
auth    [default=die] pam_faillock.so authfail audit deny=5 unlock_time=1800
auth    sufficient    pam_faillock.so authsucc audit deny=5 unlock_time=1800
```

```bash
# Configuration dans /etc/security/faillock.conf
deny = 5                  # Bloquer après 5 tentatives
fail_interval = 900       # Dans une fenêtre de 15 minutes
unlock_time = 1800        # Déverouiller après 30 minutes
even_deny_root            # Appliquer aussi à root
root_unlock_time = 60     # root déverrouillé après 1 minute
audit                     # Logger dans syslog
silent                    # Ne pas révéler si le compte est bloqué

# Vérifier les comptes bloqués
faillock

# Déverouiller manuellement un compte
faillock --user alice --reset
```

**pam_pwquality — Politique de mots de passe :**

```bash
# /etc/pam.d/common-password
password    requisite    pam_pwquality.so retry=3

# /etc/security/pwquality.conf
minlen = 12           # Longueur minimale
dcredit = -1          # Au moins 1 chiffre
ucredit = -1          # Au moins 1 majuscule
lcredit = -1          # Au moins 1 minuscule
ocredit = -1          # Au moins 1 caractère spécial
maxrepeat = 3         # Max 3 caractères identiques consécutifs
difok = 8             # Au moins 8 caractères différents du précédent
gecoscheck = 1        # Vérifier que le mdp ne contient pas l'info GECOS
badwords = password,admin,letmein  # Mots interdits
enforcing = 1         # Activer l'application (0 = warning uniquement)
```

**pam_limits — Limites de ressources :**

```bash
# /etc/security/limits.conf
# Format : domain  type  item  value

# Limiter le nombre de processus pour alice
alice    hard    nproc    100

# Limiter les fichiers ouverts pour le groupe developers
@developers   soft    nofile   4096
@developers   hard    nofile   8192

# Limiter la taille des fichiers core (désactiver les core dumps)
*        hard    core     0

# Limiter la mémoire verrouillée (protection contre les attaques swap)
*        hard    memlock  unlimited   # Pour les clés cryptographiques

# Limiter le temps CPU
@students  hard    cpu      3600    # 1 heure CPU max

# Limiter les connexions simultanées
alice    -    maxlogins  3
```

**pam_time — Restrictions horaires :**

```bash
# /etc/security/time.conf
# Format : services;ttys;users;times
# Restreindre la connexion SSH de bob aux heures ouvrées (lun-ven 8h-18h)
sshd;*;bob;MoTuWeThFr0800-1800

# Interdire la connexion à tout le monde la nuit (sauf root et admin)
login;*;!root&!admin;!Al0000-2400
# Al = tous les jours
# ! = négation

# Dans /etc/pam.d/sshd, ajouter :
account   required   pam_time.so
```

### 4.4 Durcissement PAM complet

```bash
# Vérifier la configuration PAM actuelle
ls /etc/pam.d/
cat /etc/pam.d/common-auth

# Désactiver les comptes de service dans PAM
# Dans /etc/pam.d/login :
account    required     pam_nologin.so
# Ce module vérifie /etc/nologin : si le fichier existe,
# seul root peut se connecter (utile pour la maintenance)

# Créer /etc/nologin pour la maintenance
echo "Système en maintenance, revenez dans 30 minutes." > /etc/nologin
# Supprimer pour réactiver les connexions
rm /etc/nologin

# Auditer les modules PAM chargés
# Les modules sont des .so dans /lib/security/ ou /lib/x86_64-linux-gnu/security/
ls /lib/x86_64-linux-gnu/security/
```

> [!info]
> PAM est le gardien de toute authentification sur un système Linux. Une mauvaise configuration PAM peut rendre un système inaccessible (lock-out). Toujours tester les modifications PAM depuis une session root existante avant de fermer la session courante.

---

## 5. auditd — Surveillance des événements système

### 5.1 Architecture d'auditd

```
Événement système (appel système, accès fichier, modification config...)
        │
        ▼
┌──────────────────────┐
│  Noyau Linux (audit) │  ← Hooks dans le noyau, performance minimale
└──────────────────────┘
        │
        ▼  (netlink socket)
┌──────────────────────┐
│      auditd          │  ← Daemon espace utilisateur
└──────────────────────┘
        │
        ├── /var/log/audit/audit.log   (log brut)
        │
        └── /etc/audit/auditd.conf    (configuration)
            /etc/audit/rules.d/       (règles)
```

### 5.2 Installation et configuration de base

```bash
# Installation
apt-get install auditd audispd-plugins   # Debian/Ubuntu
yum install audit audit-libs             # RHEL/CentOS

# Démarrer et activer
systemctl enable --now auditd

# Vérifier le statut
auditctl -s
# enabled 1
# failure 1
# pid 1234
# rate_limit 0
# backlog_limit 8192
# lost 0
# backlog 0
```

**Configuration principale (/etc/audit/auditd.conf) :**

```bash
# /etc/audit/auditd.conf

# Taille maximale du fichier de log (en Mo)
max_log_file = 50

# Action quand le fichier log est plein
max_log_file_action = ROTATE

# Nombre de rotations à conserver
num_logs = 10

# Espace disque minimum avant actions (en Mo)
space_left = 75
space_left_action = SYSLOG

# Espace critique (en Mo)
admin_space_left = 50
admin_space_left_action = SUSPEND

# Action en cas de disque plein
disk_full_action = SUSPEND

# Action en cas d'erreur d'écriture
disk_error_action = SUSPEND

# Écrire les logs de façon synchrone (plus sûr mais plus lent)
flush = INCREMENTAL_ASYNC

# Format de log
log_format = ENRICHED

# Résoudre les UID en noms d'utilisateurs dans les logs
log_group = root
priority_boost = 4
```

### 5.3 Règles auditd

```bash
# Syntaxe générale des règles
auditctl -a action,filter -S syscall -F field=value -k key_name

# Types d'actions :
# always  = toujours enregistrer
# never   = ne jamais enregistrer

# Filtres :
# task    = lors de la création d'un processus
# exit    = à la sortie d'un appel système
# user    = événements espace utilisateur
# exclude = exclure des événements

# Surveillance des fichiers sensibles
auditctl -w /etc/passwd -p wa -k user_modification
auditctl -w /etc/shadow -p wa -k shadow_modification
auditctl -w /etc/sudoers -p wa -k sudoers_modification
auditctl -w /etc/ssh/sshd_config -p rwa -k sshd_config

# Types de permissions à surveiller :
# r = lecture
# w = écriture
# x = exécution
# a = modification des attributs

# Surveillance d'un répertoire récursivement
auditctl -w /etc/ -p wa -k etc_modification

# Surveillance des appels système critiques
# Surveiller toutes les exécutions de commandes (par uid 1000)
auditctl -a always,exit -F arch=b64 -S execve -F uid=1000 -k user_commands

# Surveiller les changements de permissions
auditctl -a always,exit -F arch=b64 -S chmod,fchmod,chown,fchown -k permission_changes

# Surveiller l'accès aux fichiers SUID
auditctl -a always,exit -F arch=b64 -S execve -F perm=x -F auid>=1000 -F auid!=-1 -k suid_execution

# Surveiller les connexions réseau
auditctl -a always,exit -F arch=b64 -S connect,accept -k network_connections

# Surveiller les montages de systèmes de fichiers
auditctl -a always,exit -F arch=b64 -S mount -k mount_operations

# Surveiller les suppressions de fichiers
auditctl -a always,exit -F arch=b64 -S unlink,unlinkat,rename,renameat -k file_deletion

# Surveiller les modifications de l'heure système
auditctl -a always,exit -F arch=b64 -S adjtimex,settimeofday,clock_settime -k time_change

# Surveiller les tentatives d'accès non autorisées
auditctl -a always,exit -F arch=b64 -S open,openat -F exit=-EACCES -k access_denied
auditctl -a always,exit -F arch=b64 -S open,openat -F exit=-EPERM -k access_denied

# Afficher les règles actives
auditctl -l
```

**Fichiers de règles persistants :**

```bash
# Les règles dans /etc/audit/rules.d/ sont chargées au démarrage
# Créer un fichier de règles de durcissement

cat > /etc/audit/rules.d/hardening.rules << 'EOF'
# Règles de durcissement - Holberton Security

# Supprimer toutes les règles au chargement
-D

# Augmenter le buffer pour éviter les pertes d'événements
-b 8192

# En cas d'erreur : 0=silent, 1=printk, 2=panic
-f 1

# === Fichiers d'identité et d'authentification ===
-w /etc/passwd -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/group -p wa -k identity
-w /etc/gshadow -p wa -k identity
-w /etc/security/opasswd -p wa -k identity

# === Escalade de privilèges ===
-w /etc/sudoers -p wa -k privilege_escalation
-w /etc/sudoers.d/ -p wa -k privilege_escalation
-a always,exit -F arch=b64 -S setuid -S setgid -k privilege_escalation

# === Connexions SSH ===
-w /etc/ssh/sshd_config -p wa -k sshd_config
-w /root/.ssh -p wa -k root_ssh

# === PAM ===
-w /etc/pam.d/ -p wa -k pam_modification

# === Modules noyau ===
-w /sbin/insmod -p x -k kernel_modules
-w /sbin/rmmod -p x -k kernel_modules
-w /sbin/modprobe -p x -k kernel_modules
-a always,exit -F arch=b64 -S init_module,finit_module,delete_module -k kernel_modules

# === Exécutions (utilisateurs non root) ===
-a always,exit -F arch=b64 -S execve -F auid>=1000 -F auid!=-1 -k user_execution

# === Accès refusés ===
-a always,exit -F arch=b64 -S open -F exit=-EACCES -F auid>=1000 -k access_denied
-a always,exit -F arch=b64 -S open -F exit=-EPERM -F auid>=1000 -k access_denied

# Rendre les règles immutables (impossible de modifier sans reboot)
-e 2
EOF

# Charger les règles
augenrules --load
# ou
service auditd reload
```

### 5.4 Analyse des logs avec ausearch et aureport

```bash
# Rechercher par clé de règle
ausearch -k identity
ausearch -k privilege_escalation

# Rechercher par utilisateur
ausearch -ua alice
ausearch -ui 1001

# Rechercher par commande
ausearch -x /usr/bin/sudo

# Rechercher dans un intervalle de temps
ausearch -ts yesterday -te now
ausearch -ts "11/15/2024 08:00:00" -te "11/15/2024 18:00:00"

# Afficher de manière lisible (interprète les codes)
ausearch -k sudoers_modification -i
# -i = interpréter (uid→nom, syscall→nom, etc.)

# Filtrer par type d'événement
ausearch -m USER_LOGIN
ausearch -m SYSCALL
ausearch -m AVC        # SELinux Access Vector Cache (refus SELinux)

# Format de sortie
ausearch -k identity --format text    # Format texte lisible
ausearch -k identity --format json    # Format JSON

# Rapports avec aureport
aureport --summary              # Résumé général
aureport --auth                 # Rapport d'authentification
aureport --login                # Rapport de connexions
aureport --user                 # Rapport par utilisateur
aureport --failed               # Uniquement les échecs
aureport --syscall               # Appels système audités
aureport -x --failed            # Exécutions échouées
aureport --file                 # Accès aux fichiers

# Rapport des N dernières heures
aureport --start recent --end now
```

**Exemple d'entrée de log auditd :**

```
# Exemple de log brut dans /var/log/audit/audit.log
type=SYSCALL msg=audit(1705312800.123:4567): arch=c000003e syscall=59 success=yes exit=0 \
a0=7f1234 a1=7f5678 a2=7fab12 a3=7fff00 items=2 ppid=2345 pid=3456 \
auid=1001 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 \
tty=pts0 ses=5 comm="bash" exe="/bin/bash" key="user_execution"

# Décodé :
# - arch=c000003e : x86_64
# - syscall=59 : execve (exécution d'une commande)
# - success=yes : réussi
# - auid=1001 : UID réel de l'utilisateur original (audit user ID)
# - uid=0 : UID effectif (root via sudo)
# - comm="bash" : nom du processus
# - key="user_execution" : clé de règle qui a déclenché l'enregistrement
```

---

## 6. SELinux — Security-Enhanced Linux

### 6.1 Concept : DAC vs MAC

```
DAC (Discretionary Access Control) = Modèle Unix classique
  ├── L'utilisateur DÉCIDE des permissions sur ses fichiers
  ├── chmod, chown → contrôle total du propriétaire
  └── Problème : un programme compromis hérite des droits de l'utilisateur

MAC (Mandatory Access Control) = SELinux
  ├── Une POLITIQUE SYSTÈME définie par l'administrateur
  ├── S'applique en PLUS du DAC (les deux doivent autoriser l'accès)
  ├── Un processus compromis ne peut accéder qu'aux ressources
  │   que sa politique autorise EXPLICITEMENT
  └── Le propriétaire d'un fichier ne peut PAS changer la politique MAC
```

### 6.2 Les trois modes SELinux

```bash
# Vérifier le mode actuel
getenforce
# Enforcing

sestatus
# SELinux status:                 enabled
# SELinuxfs mount:                /sys/fs/selinux
# SELinux mount point:            /sys/fs/selinux
# Loaded policy name:             targeted
# Current mode:                   enforcing
# Mode from config file:          enforcing
# Policy MLS status:              enabled
# Policy deny_unknown status:     allowed
# Memory protection checking:     actual (secure)
# Max kernel policy version:      33
```

| Mode | Comportement | Utilisation |
|------|-------------|-------------|
| `enforcing` | Applique la politique, bloque et loggue les violations | Production |
| `permissive` | Loggue seulement, n'applique pas | Debugging, développement de politiques |
| `disabled` | SELinux complètement désactivé | À éviter absolument |

```bash
# Changer le mode temporairement (jusqu'au prochain reboot)
setenforce 0   # Permissive
setenforce 1   # Enforcing

# Changer le mode de manière permanente
# /etc/selinux/config (ou /etc/sysconfig/selinux)
SELINUX=enforcing    # ou permissive, ou disabled
SELINUXTYPE=targeted  # politique (targeted, mls, minimum)

# ATTENTION : passer de disabled à enforcing nécessite un re-labelling complet
# Créer ce fichier vide pour forcer le re-labelling au prochain boot
touch /.autorelabel
reboot
```

### 6.3 Les contextes SELinux

Chaque objet (fichier, processus, port réseau, utilisateur) possède un **contexte SELinux** de la forme :

```
utilisateur:rôle:type:niveau_mls
   user    :role :type: level
```

```bash
# Afficher le contexte d'un fichier
ls -Z /var/www/html/index.html
# -rw-r--r--. root root system_u:object_r:httpd_sys_content_t:s0 /var/www/html/index.html
#                       ──────── ──────── ─────────────────────── ──
#                       user     role     type (la plus importante) level MLS

# Afficher le contexte d'un processus
ps auxZ | grep httpd
# system_u:system_r:httpd_t:s0  root  1234  ... /usr/sbin/httpd

# Afficher le contexte de l'utilisateur courant
id -Z
# unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

# Afficher tous les contextes de fichiers dans un répertoire
ls -laZ /etc/ssh/
```

**Les types les plus courants :**

| Type | Description |
|------|-------------|
| `httpd_sys_content_t` | Contenu web servi par Apache/Nginx |
| `httpd_sys_rw_content_t` | Contenu web modifiable par Apache |
| `sshd_key_t` | Clés SSH |
| `var_log_t` | Fichiers de logs |
| `etc_t` | Fichiers de configuration système |
| `bin_t` | Binaires système |
| `user_home_t` | Fichiers dans les homes utilisateurs |
| `unconfined_t` | Type non confiné (processus sans restriction SELinux) |

### 6.4 Gestion des contextes

```bash
# Changer le contexte d'un fichier
chcon -t httpd_sys_content_t /var/www/mon-site/index.html

# Changer le contexte récursivement
chcon -R -t httpd_sys_content_t /var/www/mon-site/

# Restaurer le contexte par défaut (selon la politique)
restorecon /var/www/mon-site/index.html
restorecon -R /var/www/mon-site/

# Définir un contexte par défaut permanent pour un répertoire
# (sera appliqué par restorecon)
semanage fcontext -a -t httpd_sys_content_t "/var/www/mon-site(/.*)?"
restorecon -R /var/www/mon-site/

# Vérifier les contextes par défaut configurés
semanage fcontext -l | grep httpd
```

### 6.5 Les booléens SELinux

Les booléens permettent d'ajuster la politique sans la réécrire.

```bash
# Lister tous les booléens
getsebool -a

# Lister les booléens httpd
getsebool -a | grep httpd
# httpd_can_network_connect --> off
# httpd_can_network_connect_db --> off
# httpd_can_sendmail --> off
# httpd_execmem --> off
# httpd_use_nfs --> off

# Activer un booléen temporairement
setsebool httpd_can_network_connect on

# Activer un booléen de manière permanente (-P = permanent)
setsebool -P httpd_can_network_connect on
setsebool -P httpd_can_network_connect_db on

# Cas d'usage concret : permettre à Apache de se connecter à une base de données
setsebool -P httpd_can_network_connect_db on

# Permettre à Apache d'utiliser des répertoires NFS
setsebool -P httpd_use_nfs on
```

### 6.6 Gestion des ports

```bash
# Lister les ports autorisés pour http
semanage port -l | grep http
# http_port_t    tcp    80, 443, 488, 8008, 8009, 8443

# Autoriser Apache à écouter sur le port 8080
semanage port -a -t http_port_t -p tcp 8080

# Vérifier
semanage port -l | grep http_port
```

### 6.7 Troubleshooting SELinux

```bash
# Les refus SELinux sont dans /var/log/audit/audit.log
# Chercher les AVC (Access Vector Cache) denials
ausearch -m AVC -ts recent
grep "avc: denied" /var/log/audit/audit.log | tail -20

# Exemple de refus :
# type=AVC msg=audit(1705312800.123:123): avc:  denied  { read } for
# pid=1234 comm="httpd" name="secret.conf" dev="sda1" ino=12345
# scontext=system_u:system_r:httpd_t:s0
# tcontext=unconfined_u:object_r:admin_home_t:s0 tclass=file permissive=0

# Outil d'analyse et de suggestion de correction
audit2why < /var/log/audit/audit.log
# Was caused by:
#         Missing type enforcement (TE) allow rule.
#         You can use audit2allow to generate a loadable module to allow this access.

# Générer une politique pour autoriser une action refusée
audit2allow -a -M mon_module
# Génère mon_module.te (source) et mon_module.pp (compilé)

# Charger la politique générée
semodule -i mon_module.pp

# Vérifier les modules chargés
semodule -l | head -20

# Outil graphique d'analyse (si disponible)
sealert -a /var/log/audit/audit.log
```

> [!tip]
> Ne jamais utiliser `setenforce 0` pour "corriger" un problème SELinux en production. Identifier la cause avec `audit2why`, comprendre pourquoi l'accès est refusé, puis soit corriger le contexte du fichier (`restorecon`), soit activer le bon booléen, soit créer une règle ciblée avec `audit2allow`. Le mode permissive est uniquement pour le debugging temporaire.

---

## 7. AppArmor

### 7.1 AppArmor vs SELinux

| Critère | SELinux | AppArmor |
|---------|---------|----------|
| Modèle | Labels (contextes) sur tous les objets | Chemins de fichiers dans les profils |
| Complexité | Élevée — courbe d'apprentissage importante | Modérée — profils en syntaxe lisible |
| Granularité | Très fine (type enforcement, MLS) | Fine mais moins que SELinux |
| Distribution | Red Hat, Fedora, CentOS, RHEL | Ubuntu, Debian, SUSE, OpenSUSE |
| Approche | Politique système globale | Profil par application |
| Debugging | `ausearch`, `audit2why`, `sealert` | `aa-status`, `dmesg`, journalctl |
| Portabilité | Lié aux labels, pose problème avec NFS | Basé sur les chemins, plus portable |

### 7.2 Modes AppArmor

```bash
# Vérifier le statut AppArmor
aa-status
# apparmor module is loaded.
# 35 profiles are loaded.
# 33 profiles are in enforce mode.
# 2 profiles are in complain mode.
# 0 processes have profiles defined.
# 0 processes are in enforce mode.
# 0 processes are in complain mode.
# 0 processes are unconfined but have a defined profile.

# Modes disponibles :
# enforce  : applique la politique, bloque et loggue
# complain : loggue seulement (comme SELinux permissive)
# disabled : profil désactivé

# Vérifier un profil spécifique
aa-status | grep nginx
```

### 7.3 Structure d'un profil AppArmor

```bash
# Les profils sont dans /etc/apparmor.d/
ls /etc/apparmor.d/
# abstractions/    usr.bin.firefox   usr.sbin.nginx   ...

# Exemple de profil nginx simplifié
cat /etc/apparmor.d/usr.sbin.nginx
```

```
# /etc/apparmor.d/usr.sbin.nginx
#include <tunables/global>

/usr/sbin/nginx {
    # Inclure les abstractions communes
    #include <abstractions/base>
    #include <abstractions/nameservice>

    # Capacités réseau
    capability net_bind_service,
    capability setuid,
    capability setgid,

    # Binaire nginx lui-même
    /usr/sbin/nginx mr,

    # Fichiers de configuration
    /etc/nginx/** r,
    /etc/nginx/nginx.conf r,

    # Fichiers de log
    /var/log/nginx/ rw,
    /var/log/nginx/** rw,

    # Contenu web
    /var/www/html/** r,
    /usr/share/nginx/html/** r,

    # PID file
    /run/nginx.pid rw,

    # Sockets
    /run/nginx.sock rw,

    # Proc et sys (nécessaire pour certaines opérations)
    /proc/*/net/if_inet6 r,
    @{PROC}/sys/kernel/ngroups_max r,

    # Refuser tout accès à /etc/shadow (explicite pour la clarté)
    deny /etc/shadow r,
}
```

**Permissions dans les profils :**

| Permission | Description |
|-----------|-------------|
| `r` | Lecture |
| `w` | Écriture |
| `x` | Exécution |
| `m` | Mapping en mémoire (mmap) |
| `k` | Locking de fichier |
| `l` | Création de liens symboliques |
| `ix` | Exécution avec héritage du profil courant |
| `px` | Transition vers un profil spécifique |
| `ux` | Exécution non confinée |

### 7.4 Créer un profil avec aa-genprof

```bash
# Installer les outils AppArmor
apt-get install apparmor-utils apparmor-profiles apparmor-profiles-extra

# Générer un profil pour une application
# 1. Lancer aa-genprof (met l'application en mode complain automatiquement)
aa-genprof /usr/sbin/mon-service

# 2. Dans un autre terminal, exercer l'application normalement
# (faire toutes les opérations que le service doit faire)
systemctl start mon-service
# Utiliser le service normalement...

# 3. Retourner dans aa-genprof et appuyer sur S pour scanner les logs
# L'outil propose des règles basées sur les accès observés

# Mettre en mode complain pour une période de test
aa-complain /usr/sbin/nginx
aa-complain /etc/apparmor.d/usr.sbin.nginx

# Observer les violations dans le mode complain
journalctl -xe | grep apparmor
dmesg | grep apparmor

# Mettre en mode enforce
aa-enforce /usr/sbin/nginx

# Désactiver un profil
aa-disable /usr/sbin/nginx

# Recharger tous les profils
systemctl reload apparmor
# ou
apparmor_parser -r /etc/apparmor.d/usr.sbin.nginx
```

### 7.5 Debugging AppArmor

```bash
# Voir les violations AppArmor en temps réel
journalctl -f | grep -i apparmor

# Exemple de violation :
# kernel: apparmor="DENIED" operation="open" profile="/usr/sbin/nginx"
# name="/etc/secret/api.key" pid=1234 comm="nginx"
# requested_mask="r" denied_mask="r" fsuid=33 ouid=0

# Analyser les logs et suggérer des corrections
aa-logprof
# L'outil lit les logs et propose des modifications de profil

# Vérifier la syntaxe d'un profil
apparmor_parser -p /etc/apparmor.d/usr.sbin.nginx

# Charger un profil sans l'appliquer (test)
apparmor_parser -C /etc/apparmor.d/usr.sbin.nginx
```

> [!info]
> AppArmor et SELinux ne peuvent pas être actifs simultanément sur le même système (ils utilisent tous les deux le framework Linux Security Modules). Choisir l'un ou l'autre selon la distribution. Sur Ubuntu/Debian : AppArmor est le choix naturel. Sur RHEL/CentOS/Fedora : SELinux est le choix naturel.

---

## 8. Hardening checklist système

### 8.1 Désactiver les services inutiles

```bash
# Lister tous les services actifs
systemctl list-units --type=service --state=active

# Services souvent inutiles sur un serveur de production
# (à adapter selon le contexte)

# Désactiver des services non nécessaires
SERVICES_TO_DISABLE=(
    "bluetooth"
    "cups"           # Impression
    "avahi-daemon"   # mDNS/Bonjour
    "rpcbind"        # NFS (si non utilisé)
    "nfs-server"
    "vsftpd"         # FTP
    "telnet"         # Telnet (JAMAIS sur un serveur)
    "xinetd"
    "nis"
)

for service in "${SERVICES_TO_DISABLE[@]}"; do
    if systemctl is-active "$service" &>/dev/null; then
        echo "Désactivation de $service..."
        systemctl stop "$service"
        systemctl disable "$service"
        systemctl mask "$service"   # mask = impossible de démarrer même manuellement
    fi
done

# Vérifier les services démarrés automatiquement
systemctl list-unit-files --state=enabled
```

### 8.2 Audit des ports ouverts

```bash
# Lister les ports en écoute (avec les processus associés)
ss -tlnp         # TCP en écoute, avec processus
ss -ulnp         # UDP en écoute, avec processus
ss -tlnp | grep LISTEN

# Alternative avec netstat (si disponible)
netstat -tlnp
netstat -ulnp

# Exemple de sortie :
# Netid  State   Recv-Q  Send-Q  Local Address:Port  Process
# tcp    LISTEN  0       128     0.0.0.0:22           sshd(1234)
# tcp    LISTEN  0       128     127.0.0.1:5432       postgres(2345)
# tcp    LISTEN  0       128     0.0.0.0:80           nginx(3456)

# Scanner depuis l'extérieur (depuis une autre machine)
nmap -sV -O 192.168.1.100
nmap -p- 192.168.1.100   # Scanner TOUS les ports

# Fermer les ports inutiles en arrêtant les services
# ou en configurant le pare-feu

# Configuration du pare-feu avec ufw (Ubuntu)
ufw default deny incoming
ufw default allow outgoing
ufw allow 22/tcp comment "SSH"
ufw allow 80/tcp comment "HTTP"
ufw allow 443/tcp comment "HTTPS"
ufw enable
ufw status verbose

# Configuration avec iptables
iptables -F                          # Flush des règles
iptables -P INPUT DROP               # Politique par défaut : bloquer tout en entrée
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

iptables -A INPUT -i lo -j ACCEPT                                    # Loopback
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT  # Connexions établies
iptables -A INPUT -p tcp --dport 22 -j ACCEPT                       # SSH
iptables -A INPUT -p tcp --dport 80 -j ACCEPT                       # HTTP
iptables -A INPUT -p tcp --dport 443 -j ACCEPT                      # HTTPS

# Sauvegarder les règles iptables
iptables-save > /etc/iptables/rules.v4
```

### 8.3 Audit des packages installés

```bash
# Lister tous les packages installés
dpkg -l | grep "^ii" | awk '{print $2}'   # Debian/Ubuntu
rpm -qa                                    # RHEL/CentOS

# Identifier les packages potentiellement dangereux
DANGEROUS_PACKAGES=(
    "telnet"
    "rsh-client"
    "rsh-server"
    "talk"
    "talkd"
    "xinetd"
    "nis"
    "yp-tools"
    "tftp"
    "atftpd"
)

for pkg in "${DANGEROUS_PACKAGES[@]}"; do
    if dpkg -l "$pkg" &>/dev/null; then
        echo "[ATTENTION] Package installé : $pkg"
    fi
done

# Désinstaller un package et ses dépendances orphelines
apt-get purge telnet
apt-get autoremove

# Mettre à jour le système (patches de sécurité)
apt-get update && apt-get upgrade -y
# Ou uniquement les patches de sécurité
apt-get update && apt-get upgrade -t $(lsb_release -cs)-security -y

# Automatiser les mises à jour de sécurité
apt-get install unattended-upgrades
dpkg-reconfigure unattended-upgrades

# Configuration dans /etc/apt/apt.conf.d/50unattended-upgrades
# Activer uniquement les mises à jour de sécurité automatiques
```

### 8.4 Sécurisation du système de fichiers

```bash
# Options de montage sécurisées dans /etc/fstab
cat /etc/fstab
```

```
# /etc/fstab - Options de sécurité recommandées

# Partition /tmp : noexec, nosuid, nodev
tmpfs  /tmp  tmpfs  defaults,noexec,nosuid,nodev,size=2G  0 0

# Partition /var/tmp : noexec, nosuid, nodev
tmpfs  /var/tmp  tmpfs  defaults,noexec,nosuid,nodev,size=1G  0 0

# Partition /home : nosuid, nodev
/dev/sda3  /home  ext4  defaults,nosuid,nodev  0 2

# Partition /var : nosuid, nodev
/dev/sda4  /var  ext4  defaults,nosuid,nodev  0 2

# /proc : avec options de sécurité
proc  /proc  proc  defaults,hidepid=2  0 0
# hidepid=2 : les utilisateurs ne voient que leurs propres processus dans /proc
```

```bash
# Remontage à chaud sans reboot (test)
mount -o remount,noexec,nosuid,nodev /tmp

# Vérifier les options de montage
mount | grep -E '/tmp|/home|/var'
cat /proc/mounts

# Sécuriser les permissions sur les répertoires critiques
chmod 1777 /tmp /var/tmp        # Sticky bit obligatoire
chmod 750 /root                  # Home de root inaccessible
chmod 700 /etc/cron.d            # Cron accessible uniquement à root
chmod 600 /etc/crontab
chmod 640 /etc/shadow
chmod 644 /etc/passwd
chmod 644 /etc/group
chmod 640 /etc/gshadow
```

### 8.5 Paramètres noyau (sysctl)

```bash
# Créer un fichier de configuration sysctl pour le durcissement
cat > /etc/sysctl.d/99-hardening.conf << 'EOF'
# === Réseau ===
# Désactiver le forwarding IP (sauf si serveur routeur)
net.ipv4.ip_forward = 0
net.ipv6.conf.all.forwarding = 0

# Protection contre les attaques SYN flood
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_syn_retries = 2
net.ipv4.tcp_synack_retries = 2

# Désactiver les redirections ICMP
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0

# Désactiver l'acceptation des paquets source-routed
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0

# Activer la protection contre les adresses IP usurpées (anti-spoofing)
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1

# Loguer les paquets avec adresses suspectes
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.default.log_martians = 1

# Désactiver les ping broadcasts (Smurf attacks)
net.ipv4.icmp_echo_ignore_broadcasts = 1

# Ignorer les réponses ICMP invalides
net.ipv4.icmp_ignore_bogus_error_responses = 1

# === Mémoire et processus ===
# Désactiver les core dumps pour les SUID
fs.suid_dumpable = 0

# Activer l'ASLR (Address Space Layout Randomization)
kernel.randomize_va_space = 2

# Restreindre l'accès aux dmesg (empêche la fuite d'infos noyau)
kernel.dmesg_restrict = 1

# Restreindre ptrace (empêche l'espionnage entre processus)
kernel.yama.ptrace_scope = 1

# Désactiver les magic SysRq keys
kernel.sysrq = 0

# Restreindre l'accès aux performances hardwares
kernel.perf_event_paranoid = 3

# Protéger les liens symboliques et durs
fs.protected_symlinks = 1
fs.protected_hardlinks = 1
EOF

# Appliquer immédiatement
sysctl -p /etc/sysctl.d/99-hardening.conf

# Vérifier un paramètre
sysctl net.ipv4.tcp_syncookies
# net.ipv4.tcp_syncookies = 1
```

### 8.6 Comptes utilisateurs et mots de passe

```bash
# Auditer les comptes sans mot de passe
awk -F: '($2 == "" || $2 == "!!" ) {print $1}' /etc/shadow

# Verrouiller les comptes système inutiles
# Les comptes avec shell /bin/false ou /usr/sbin/nologin ne peuvent pas se connecter
grep -E "^[a-z].*:(bash|sh|dash|zsh)$" /etc/passwd
# Tout compte système avec un vrai shell est suspect

# Verrouiller un compte (ajouter ! devant le hash dans /etc/shadow)
passwd -l compte_suspect
usermod -L compte_suspect   # Même effet

# Vérifier les comptes UID 0 (root) supplémentaires
awk -F: '($3 == 0) {print $1}' /etc/passwd
# Doit retourner uniquement "root"

# Forcer la politique de mot de passe dans /etc/login.defs
# PASS_MAX_DAYS  90    # Expiration tous les 90 jours
# PASS_MIN_DAYS  7     # Minimum 7 jours avant de changer
# PASS_WARN_AGE  14    # Avertissement 14 jours avant expiration
# PASS_MIN_LEN   12    # Longueur minimale

# Configurer l'expiration pour un utilisateur existant
chage -M 90 -m 7 -W 14 alice

# Voir la politique de mot de passe d'un utilisateur
chage -l alice
# Last password change                    : Jan 15, 2024
# Password expires                        : Apr 14, 2024
# Password inactive                       : never
# Account expires                         : never
# Minimum number of days between password change  : 7
# Maximum number of days between password change  : 90
# Number of days of warning before password expires : 14

# Désactiver l'historique des mots de passe utilisés
# (via pam_pwhistory dans /etc/pam.d/common-password)
# password required pam_pwhistory.so remember=12 use_authtok
```

---

## 9. SSH sécurisé

### 9.1 Configuration sécurisée de sshd

```bash
# Fichier de configuration principal
# /etc/ssh/sshd_config

# TOUJOURS faire une sauvegarde avant modification
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
```

```
# /etc/ssh/sshd_config — Configuration sécurisée

# === Protocole et port ===
Protocol 2                      # SSH v2 uniquement (v1 est cassé cryptographiquement)
Port 2222                       # Port non standard (réduction du bruit des scans)
AddressFamily inet              # IPv4 uniquement si IPv6 non nécessaire

# === Authentification ===
PermitRootLogin no              # JAMAIS de connexion root directe
PermitEmptyPasswords no         # Pas de connexion sans mot de passe
MaxAuthTries 3                  # Max 3 tentatives avant déconnexion
MaxSessions 10                  # Max sessions simultanées par connexion

# Authentification par clé uniquement (recommandé)
PubkeyAuthentication yes
PasswordAuthentication no       # Désactiver les mots de passe
ChallengeResponseAuthentication no
KbdInteractiveAuthentication no

# === Clés autorisées ===
AuthorizedKeysFile .ssh/authorized_keys .ssh/authorized_keys2

# === Timeouts ===
LoginGraceTime 30               # 30 secondes pour s'authentifier
ClientAliveInterval 300         # Ping toutes les 5 minutes
ClientAliveCountMax 2           # Fermer après 2 pings sans réponse
TCPKeepAlive yes

# === Restrictions d'accès ===
AllowUsers alice bob deploy@192.168.1.0/24  # Whitelist d'utilisateurs
# AllowGroups ssh-users                      # Ou restriction par groupe
DenyUsers attacker                           # Blacklist explicite

# === Fonctionnalités à désactiver ===
X11Forwarding no                # Désactiver le forwarding X11
AllowTcpForwarding no           # Désactiver le tunneling TCP
AllowAgentForwarding no         # Désactiver l'agent forwarding
PermitTunnel no                 # Désactiver les tunnels VPN SSH
GatewayPorts no                 # Refuser le binding sur toutes les interfaces
PrintMotd no                    # Ne pas afficher le MOTD via SSH

# === Bannière ===
Banner /etc/ssh/banner.txt      # Message légal avant connexion

# === Logging ===
LogLevel VERBOSE                # Logs détaillés (y compris les clés utilisées)
SyslogFacility AUTH

# === Algorithmes cryptographiques (durcissement) ===
# Utiliser uniquement des algorithmes forts
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org,diffie-hellman-group16-sha512,diffie-hellman-group18-sha512
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr
MACs hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com,umac-128-etm@openssh.com

# HostKey algorithms
HostKeyAlgorithms ssh-ed25519,ssh-rsa
PubkeyAcceptedKeyTypes ssh-ed25519,rsa-sha2-512,rsa-sha2-256

# === SFTP seulement pour certains utilisateurs ===
Match Group sftp-only
    ChrootDirectory /var/sftp/%u
    ForceCommand internal-sftp
    AllowTcpForwarding no
    X11Forwarding no
```

```bash
# Vérifier la syntaxe avant de recharger
sshd -t
# Si aucun message : configuration valide

# Recharger sshd (sans couper les connexions existantes)
systemctl reload sshd

# Tester la connexion depuis une autre session AVANT de fermer la session courante !
ssh -p 2222 alice@serveur
```

### 9.2 Gestion des clés SSH

```bash
# Générer une paire de clés SSH forte
# Ed25519 (recommandé — algorithmiquement supérieur à RSA)
ssh-keygen -t ed25519 -C "alice@machine-perso" -f ~/.ssh/id_ed25519

# RSA 4096 bits (si Ed25519 non supporté par le serveur cible)
ssh-keygen -t rsa -b 4096 -C "alice@machine-perso" -f ~/.ssh/id_rsa_4096

# TOUJOURS utiliser une passphrase sur la clé privée
# (protège la clé en cas de vol)

# Copier la clé publique sur le serveur
ssh-copy-id -i ~/.ssh/id_ed25519.pub -p 2222 alice@serveur

# Manuellement :
cat ~/.ssh/id_ed25519.pub >> ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

# Vérifier les permissions (SSH refuse les clés avec des permissions trop ouvertes)
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519          # Clé privée : 600 obligatoire
chmod 644 ~/.ssh/id_ed25519.pub      # Clé publique : 644 acceptable

# Utiliser ssh-agent pour éviter de saisir la passphrase à chaque fois
eval $(ssh-agent -s)
ssh-add ~/.ssh/id_ed25519
# Saisir la passphrase une seule fois par session

# Revocation de clé : supprimer de authorized_keys
# Puis forcer la déconnexion des sessions actives
who                                           # Voir les sessions actives
pkill -KILL -u alice                          # Tuer toutes les sessions d'alice
```

### 9.3 fail2ban — Protection contre le bruteforce

```bash
# Installation
apt-get install fail2ban

# Configuration : NE PAS modifier /etc/fail2ban/jail.conf directement
# Créer /etc/fail2ban/jail.local pour les surcharges

cat > /etc/fail2ban/jail.local << 'EOF'
[DEFAULT]
# Temps de bannissement (en secondes) - 1 heure
bantime  = 3600

# Fenêtre d'observation (en secondes) - 10 minutes
findtime = 600

# Nombre de tentatives avant bannissement
maxretry = 5

# Whitelist (ne jamais bannir ces IPs)
ignoreip = 127.0.0.1/8 ::1 192.168.1.0/24

# Méthode de bannissement
banaction = iptables-multiport

# Notification mail
destemail = security@company.com
sendername = Fail2Ban
mta = sendmail
action = %(action_mwl)s  # ban + mail avec logs

[sshd]
enabled  = true
port     = 2222          # Adapter au port SSH configuré
filter   = sshd
logpath  = /var/log/auth.log
maxretry = 3
bantime  = 86400         # Bannissement 24h pour SSH (plus strict)

[nginx-http-auth]
enabled  = true
port     = http,https
filter   = nginx-http-auth
logpath  = /var/log/nginx/error.log
maxretry = 5

[nginx-botsearch]
enabled  = true
port     = http,https
filter   = nginx-botsearch
logpath  = /var/log/nginx/access.log
maxretry = 2
bantime  = 86400
EOF

# Démarrer fail2ban
systemctl enable --now fail2ban

# Vérifier le statut
fail2ban-client status
fail2ban-client status sshd

# Voir les IPs bannies
fail2ban-client status sshd | grep "Banned IP"

# Débannir une IP manuellement
fail2ban-client set sshd unbanip 1.2.3.4

# Tester un filtre
fail2ban-regex /var/log/auth.log /etc/fail2ban/filter.d/sshd.conf

# Logs fail2ban
tail -f /var/log/fail2ban.log
```

### 9.4 Port Knocking

Le port knocking permet de cacher un service (comme SSH) derrière une séquence secrète de connexions sur des ports fermés.

```bash
# Installation
apt-get install knockd

# Configuration /etc/knockd.conf
cat > /etc/knockd.conf << 'EOF'
[options]
    UseSyslog
    Interface = eth0

[openSSH]
    sequence    = 7000,8000,9000
    seq_timeout = 10
    command     = /usr/sbin/ufw allow from %IP% to any port 2222
    tcpflags    = syn

[closeSSH]
    sequence    = 9000,8000,7000
    seq_timeout = 10
    command     = /usr/sbin/ufw delete allow from %IP% to any port 2222
    tcpflags    = syn
EOF

# Activer knockd au démarrage
systemctl enable --now knockd

# Par défaut, SSH doit être bloqué par le pare-feu
ufw deny 2222

# Pour se connecter :
# 1. Envoyer la séquence de knock
knock serveur 7000 8000 9000
# 2. Se connecter en SSH (SSH est maintenant ouvert uniquement pour votre IP)
ssh -p 2222 alice@serveur
# 3. Fermer le port après connexion (optionnel)
knock serveur 9000 8000 7000
```

> [!warning]
> Le port knocking n'est PAS un mécanisme de sécurité fort — il améliore la discrétion mais un attaquant qui capture le trafic réseau peut déduire la séquence. Il doit être utilisé en complément, pas à la place d'une authentification forte par clé SSH.

### 9.5 Authentification à deux facteurs SSH (2FA)

```bash
# Utiliser Google Authenticator (TOTP) avec SSH

# Installation
apt-get install libpam-google-authenticator

# Configuration utilisateur (à exécuter pour chaque utilisateur concerné)
# Se connecter en tant que l'utilisateur
google-authenticator
# Répondre yes aux questions pour :
# - Tokens basés sur le temps (oui)
# - Mettre à jour le fichier .google_authenticator (oui)
# - Interdire les utilisations multiples du même token (oui)
# - Fenêtre de tolérance étendue (non sauf réseau peu fiable)
# - Limite de taux (oui)
# Scanner le QR code avec Google Authenticator ou Authy

# Configuration PAM
# /etc/pam.d/sshd — ajouter en première ligne :
auth required pam_google_authenticator.so

# Configuration SSH
# /etc/ssh/sshd_config :
ChallengeResponseAuthentication yes
UsePAM yes
AuthenticationMethods publickey,keyboard-interactive

# Avec cette configuration :
# 1. L'utilisateur présente sa clé SSH (publickey)
# 2. Puis saisit le code TOTP (keyboard-interactive)
# Les deux sont requis

systemctl reload sshd
```

---

## 10. Chiffrement disque : LUKS et dm-crypt

### 10.1 Architecture LUKS

```
Application
    │
    ▼
┌─────────────────────┐
│   /dev/mapper/data  │  ← Périphérique virtuel déchiffré (dm-crypt)
└─────────────────────┘
    │ (clé de chiffrement en mémoire)
    ▼
┌─────────────────────┐
│    LUKS Header      │  ← 2 Mo : magic, uuid, algorithme, 8 key slots
│  (sur /dev/sdb1)    │
│                     │
│   Données           │  ← Données chiffrées AES-XTS-256
│   chiffrées         │
└─────────────────────┘
    │
    ▼
Disque physique /dev/sdb1

Key Slot 0: Dérivé de la passphrase via PBKDF2/Argon2
Key Slot 1: Clé de récupération (escrow)
Key Slot 2-7: Disponibles (ex: clé USB, utilisateurs supplémentaires)
```

**Avantages de LUKS :**

| Fonctionnalité | Description |
|---|---|
| Format standardisé | Compatible entre distributions Linux |
| 8 key slots | Jusqu'à 8 passphrases/clés différentes |
| Algorithmes modernes | AES-XTS-512 (AES-256 en mode XTS) |
| Résistance brute-force | PBKDF2 ou Argon2id pour la dérivation de clé |
| Détachable | Header séparable du disque (protection supplémentaire) |

### 10.2 Chiffrement d'une partition avec LUKS

```bash
# Prérequis
apt-get install cryptsetup

# ATTENTION : Cette opération DÉTRUIT toutes les données sur la partition
# Faire une sauvegarde complète avant de procéder

# Étape 1 : Formater la partition avec LUKS2 (version recommandée)
cryptsetup luksFormat --type luks2 \
    --cipher aes-xts-plain64 \
    --key-size 512 \
    --hash sha512 \
    --pbkdf argon2id \
    --pbkdf-memory 1048576 \
    --pbkdf-parallel 4 \
    /dev/sdb1

# Saisir et confirmer la passphrase quand demandé

# Vérifier le header LUKS
cryptsetup luksDump /dev/sdb1
# LUKS header information
# Version:        2
# Epoch:          3
# Metadata area:  16384 [bytes]
# Keyslots area:  16744448 [bytes]
# UUID:           a1b2c3d4-...
# Label:          (no label)
# Subsystem:      (no subsystem)
# Flags:          (no flags)
# ...

# Étape 2 : Ouvrir le volume chiffré
cryptsetup luksOpen /dev/sdb1 data_chiffre
# Saisir la passphrase
# Crée /dev/mapper/data_chiffre

# Étape 3 : Créer un système de fichiers sur le volume déchiffré
mkfs.ext4 /dev/mapper/data_chiffre
# ou
mkfs.xfs /dev/mapper/data_chiffre

# Étape 4 : Monter le volume
mkdir /mnt/data_securise
mount /dev/mapper/data_chiffre /mnt/data_securise

# Utiliser normalement...

# Étape 5 : Démonter et fermer (chiffre à nouveau)
umount /mnt/data_securise
cryptsetup luksClose data_chiffre
```

### 10.3 Montage automatique au démarrage avec /etc/crypttab

```bash
# /etc/crypttab — Format : nom  source  clé  options
# Exemple de montage avec passphrase interactive au boot

# Obtenir l'UUID de la partition
blkid /dev/sdb1
# /dev/sdb1: UUID="a1b2c3d4-e5f6-7890-abcd-ef1234567890" TYPE="crypto_LUKS"

# Ajouter dans /etc/crypttab
echo "data_chiffre UUID=a1b2c3d4-e5f6-7890-abcd-ef1234567890 none luks" >> /etc/crypttab

# Ajouter dans /etc/fstab pour le montage automatique après déverrouillage
echo "/dev/mapper/data_chiffre /mnt/data_securise ext4 defaults 0 2" >> /etc/fstab

# Tester la configuration sans reboot
cryptdisks_start data_chiffre
mount /mnt/data_securise
```

### 10.4 Gestion des key slots LUKS

```bash
# Ajouter un second slot (clé de secours)
cryptsetup luksAddKey /dev/sdb1
# Saisir la passphrase existante, puis la nouvelle

# Ajouter un keyfile (fichier clé)
# Générer un keyfile aléatoire de 4096 octets
dd if=/dev/random of=/root/luks-keyfile bs=512 count=8
chmod 400 /root/luks-keyfile

# Enregistrer le keyfile comme clé LUKS
cryptsetup luksAddKey /dev/sdb1 /root/luks-keyfile

# Utiliser le keyfile pour déverrouiller
cryptsetup luksOpen --key-file /root/luks-keyfile /dev/sdb1 data_chiffre

# Lister les key slots actifs
cryptsetup luksDump /dev/sdb1 | grep "Key Slot"
# Key Slot 0: ENABLED
# Key Slot 1: ENABLED
# Key Slot 2: DISABLED
# ...

# Supprimer un key slot (ex: slot 1 si on veut retirer la clé de secours)
cryptsetup luksKillSlot /dev/sdb1 1

# Changer la passphrase du slot 0
cryptsetup luksChangeKey /dev/sdb1
```

### 10.5 Chiffrement du répertoire home avec ecryptfs

```bash
# Alternative légère à LUKS pour chiffrer uniquement le home d'un utilisateur
apt-get install ecryptfs-utils

# Chiffrer le home d'un utilisateur existant
# (l'utilisateur doit être déconnecté)
ecryptfs-migrate-home -u alice

# Créer un nouvel utilisateur avec home chiffré
useradd -m -G sudo alice
ecryptfs-setup-private    # Exécuté par l'utilisateur alice lui-même

# Monter le home chiffré (automatique à la connexion avec pam_ecryptfs)
# Dans /etc/pam.d/common-auth :
# auth    optional    pam_ecryptfs.so unwrap

# Vérifier l'état du chiffrement
ecryptfs-verify
```

### 10.6 Sauvegarde du header LUKS

> [!warning]
> Le header LUKS contient les informations nécessaires au déchiffrement. Si le header est endommagé (secteur défectueux, manipulation accidentelle), les données sont irrécupérables même avec la bonne passphrase. Toujours sauvegarder le header.

```bash
# Sauvegarder le header LUKS
cryptsetup luksHeaderBackup /dev/sdb1 --header-backup-file /root/backup-luks-header-sdb1.bin
chmod 400 /root/backup-luks-header-sdb1.bin

# Stocker cette sauvegarde HORS du système chiffré (support externe, cloud chiffré)

# Restaurer le header en cas de corruption
cryptsetup luksHeaderRestore /dev/sdb1 --header-backup-file /root/backup-luks-header-sdb1.bin
```

---

## 11. Challenges pratiques

### Challenge 1 — Audit de permissions

```bash
# Écrire un script qui :
# 1. Trouve tous les fichiers SUID/SGID sur le système
# 2. Compare avec une whitelist de référence
# 3. Alerte sur les fichiers non whitelistés
# 4. Génère un rapport dans /var/log/suid-audit.log
# Bonus : envoyer une alerte mail si des fichiers suspects sont trouvés
```

### Challenge 2 — Configuration sudo sécurisée

```bash
# Scénario : Vous êtes admin système dans une startup.
# - Alice est développeuse : elle peut redémarrer nginx et voir les logs système
# - Bob est DBA : il peut redémarrer postgresql et exécuter mysqldump
# - Carol est DevOps : elle peut tout faire mais PAS supprimer des utilisateurs
# - Le groupe "monitoring" peut voir le statut de tous les services (systemctl status)
#
# Écrire la configuration sudoers complète avec les alias appropriés.
# Tester chaque règle et documenter les vecteurs d'escalade résiduels.
```

### Challenge 3 — SELinux troubleshooting

```bash
# 1. Installer et démarrer nginx sur une CentOS/RHEL
# 2. Modifier la racine web vers /srv/web au lieu de /var/www/html
# 3. Constater l'erreur 403 Forbidden
# 4. Utiliser ausearch et audit2why pour identifier la cause
# 5. Corriger avec semanage fcontext + restorecon (PAS avec setenforce 0)
# 6. Documenter chaque commande utilisée et son output
```

### Challenge 4 — Hardening d'un serveur SSH

```bash
# À partir d'un serveur SSH avec configuration par défaut :
# 1. Appliquer toutes les bonnes pratiques du cours
# 2. Configurer fail2ban avec des seuils appropriés
# 3. Mettre en place l'authentification par clé uniquement
# 4. Ajouter le 2FA TOTP
# 5. Vérifier avec un scan nmap depuis l'extérieur
# 6. Vérifier avec ssh-audit (outil de vérification des algorithmes SSH)
apt-get install ssh-audit
ssh-audit localhost
```

### Challenge 5 — Chiffrement disque complet

```bash
# 1. Créer une machine virtuelle avec un disque supplémentaire
# 2. Partitionner le disque en deux partitions
# 3. Chiffrer avec LUKS2 en utilisant Argon2id
# 4. Configurer le montage automatique au démarrage (avec passphrase)
# 5. Sauvegarder le header LUKS sur un stockage externe
# 6. Simuler une corruption du header et restaurer depuis la sauvegarde
```

---

## 12. Récapitulatif et checklist de hardening

### 12.1 Checklist complète

```
PERMISSIONS ET COMPTES
□ umask 027 configuré dans /etc/profile
□ Permissions /etc/shadow : 640
□ Permissions /etc/passwd : 644
□ Aucun compte UID 0 autre que root
□ Aucun compte sans mot de passe
□ Comptes système avec shell /sbin/nologin
□ Sticky bit sur /tmp et /var/tmp
□ Politique de mot de passe (pam_pwquality) : minlen=12
□ Expiration des mots de passe configurée (chage)
□ Blocage après 5 tentatives (pam_faillock)

SUID / SGID
□ Inventaire des fichiers SUID/SGID réalisé
□ SUID retiré des binaires non nécessaires
□ Partitions /tmp et /home montées avec nosuid

SUDO
□ Pas de wildcard dans les commandes sudoers
□ env_reset activé dans Defaults
□ Logs sudo activés
□ Pas d'éditeurs/interpréteurs dans sudoers
□ NOPASSWD uniquement pour des commandes read-only

SSH
□ PermitRootLogin no
□ PasswordAuthentication no
□ Authentification par clé Ed25519
□ Port non standard (≠ 22)
□ MaxAuthTries 3
□ fail2ban configuré et actif
□ Algorithmes cryptographiques durcis

RÉSEAU
□ Pare-feu actif (ufw/iptables)
□ Seuls les ports nécessaires ouverts
□ Services non nécessaires désactivés/masqués
□ Paramètres sysctl anti-spoofing, anti-SYN flood

SYSTÈME
□ Mises à jour de sécurité automatiques
□ Packages inutiles supprimés
□ Partitions /tmp, /var, /home avec noexec/nosuid/nodev
□ Core dumps désactivés (fs.suid_dumpable=0)
□ ASLR activé (kernel.randomize_va_space=2)
□ ptrace restreint (kernel.yama.ptrace_scope=1)

SURVEILLANCE
□ auditd actif avec règles de durcissement
□ Surveillance /etc/passwd, /etc/shadow, /etc/sudoers
□ Surveillance des exécutions utilisateurs
□ Logs centralisés (rsyslog/syslog-ng vers serveur distant)
□ Rotation des logs configurée

MAC (SELinux ou AppArmor)
□ SELinux en mode enforcing (RHEL/CentOS)
□ AppArmor actif avec profils enforce (Ubuntu/Debian)
□ Aucun setenforce 0 en production
□ Contextes SELinux corrects sur les répertoires web

CHIFFREMENT
□ Données sensibles sur partitions LUKS
□ Header LUKS sauvegardé hors système
□ Clés SSH protégées par passphrase
□ Fichiers de configuration sensibles en 640 ou 600
```

### 12.2 Commandes de vérification rapide

```bash
#!/bin/bash
# Script de vérification rapide du niveau de hardening

echo "=== AUDIT RAPIDE HARDENING ==="
echo ""

echo "[1] Vérification umask"
grep "umask" /etc/profile /etc/bash.bashrc 2>/dev/null

echo ""
echo "[2] Comptes UID 0"
awk -F: '($3==0){print "[UID 0] " $1}' /etc/passwd

echo ""
echo "[3] Comptes sans mot de passe"
awk -F: '($2=="" || $2=="!!"){print "[NO PASS] " $1}' /etc/shadow 2>/dev/null

echo ""
echo "[4] Fichiers SUID non standards"
find /usr/bin /usr/sbin /bin /sbin -perm -4000 2>/dev/null | while read f; do
    echo "[SUID] $f"
done

echo ""
echo "[5] Services SSH"
grep -E "^(PermitRootLogin|PasswordAuthentication|MaxAuthTries)" /etc/ssh/sshd_config

echo ""
echo "[6] Pare-feu"
ufw status 2>/dev/null || iptables -L -n --line-numbers | head -20

echo ""
echo "[7] fail2ban"
fail2ban-client status 2>/dev/null | head -5

echo ""
echo "[8] SELinux / AppArmor"
if command -v getenforce &>/dev/null; then
    echo "SELinux: $(getenforce)"
elif command -v aa-status &>/dev/null; then
    echo "AppArmor: $(aa-status --enabled && echo 'actif' || echo 'inactif')"
fi

echo ""
echo "[9] auditd"
systemctl is-active auditd

echo ""
echo "[10] Paramètres sysctl critiques"
sysctl kernel.randomize_va_space fs.suid_dumpable net.ipv4.tcp_syncookies 2>/dev/null

echo ""
echo "=== FIN DE L'AUDIT ==="
```

> [!info]
> Ce script de vérification est un point de départ. Pour un audit de sécurité complet, utiliser des outils spécialisés comme **Lynis** (`apt-get install lynis && lynis audit system`), **OpenSCAP** ou **CIS-CAT** qui évaluent des centaines de points de contrôle basés sur les benchmarks CIS.

### 12.3 Ressources complémentaires

| Ressource | Description | Lien |
|-----------|-------------|------|
| CIS Benchmarks | Standards de hardening par OS | cisecurity.org |
| NIST SP 800-123 | Guide pour la sécurisation des serveurs | nvlpubs.nist.gov |
| GTFOBins | Techniques d'exploitation des binaires Unix | gtfobins.github.io |
| Linux Audit | Documentation auditd | linux-audit.com |
| LUKS on-disk format | Spécification technique LUKS2 | gitlab.com/cryptsetup |

> [!tip]
> **Lien avec les autres cours Holberton :** Ce cours s'appuie sur les concepts vus en *Linux Fundamentals* (permissions de base, gestion des utilisateurs) et *Networking* (pare-feu, ports). Les techniques de ce cours sont prérequises pour les cours avancés de *Penetration Testing* (où vous serez de l'autre côté) et *DevSecOps* (intégration de ces contrôles dans les pipelines CI/CD). La compréhension de SELinux/AppArmor est également fondamentale pour la sécurité des conteneurs Docker et Kubernetes.
