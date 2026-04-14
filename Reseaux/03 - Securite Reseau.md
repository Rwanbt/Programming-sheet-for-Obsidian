# 03 - Securite Reseau

## Introduction : la defense en profondeur

La securite reseau ne repose **jamais** sur un seul mecanisme. Le principe de la **defense en profondeur** (Defense in Depth) consiste a empiler plusieurs couches de protection. Si une couche est compromise, les suivantes continuent de proteger le systeme.

```
┌─────────────────────────────────────────────────────────────────┐
│                    DEFENSE EN PROFONDEUR                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  Couche 1 : Securite physique                             │  │
│  │  (acces aux serveurs, datacenter, badges)                 │  │
│  │  ┌───────────────────────────────────────────────────┐    │  │
│  │  │  Couche 2 : Securite reseau                       │    │  │
│  │  │  (firewalls, segmentation, VPN, IDS/IPS)          │    │  │
│  │  │  ┌───────────────────────────────────────────┐    │    │  │
│  │  │  │  Couche 3 : Securite des hotes             │    │    │  │
│  │  │  │  (SSH hardening, mises a jour, fail2ban)   │    │    │  │
│  │  │  │  ┌───────────────────────────────────┐     │    │    │  │
│  │  │  │  │  Couche 4 : Securite applicative  │     │    │    │  │
│  │  │  │  │  (auth, HTTPS, validation input)  │     │    │    │  │
│  │  │  │  │  ┌───────────────────────────┐    │     │    │    │  │
│  │  │  │  │  │  Couche 5 : Donnees       │    │     │    │    │  │
│  │  │  │  │  │  (chiffrement, backups)   │    │     │    │    │  │
│  │  │  │  │  └───────────────────────────┘    │     │    │    │  │
│  │  │  │  └───────────────────────────────────┘     │    │    │  │
│  │  │  └───────────────────────────────────────────┘    │    │  │
│  │  └───────────────────────────────────────────────────┘    │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

> [!tip] Analogie
> Pensez a un chateau medieval : il y a les **douves** (firewalls), les **murs d'enceinte** (segmentation reseau), les **portes avec gardes** (authentification), le **donjon** (chiffrement des donnees), et des **sentinelles** (monitoring/logs). Chaque couche ralentit ou arrete un attaquant, meme si une couche est franchie.

> [!warning] La securite est un processus, pas un produit
> Aucun pare-feu, aucun outil ne rend un systeme "100% securise". La securite est un **processus continu** : mises a jour, audits, monitoring, formation des equipes.

---

## Firewalls (Pare-feu)

Un firewall filtre le trafic reseau selon des **regles** predefinies. Il decide quel trafic est autorise a **entrer** ou **sortir** d'un reseau ou d'une machine.

### Types de firewalls

```
┌──────────────────────────────────────────────────────────────┐
│                    TYPES DE FIREWALLS                         │
├──────────────────┬───────────────────────────────────────────┤
│  Stateless       │  Examine chaque paquet independamment.    │
│  (sans etat)     │  Regles basees sur : IP source/dest,     │
│                  │  port source/dest, protocole.             │
│                  │  Rapide mais moins intelligent.           │
├──────────────────┼───────────────────────────────────────────┤
│  Stateful        │  Suit les connexions (sessions).          │
│  (avec etat)     │  Sait si un paquet fait partie d'une     │
│                  │  connexion existante ou est nouveau.      │
│                  │  Plus intelligent, standard aujourd'hui.  │
├──────────────────┼───────────────────────────────────────────┤
│  Application     │  Analyse le contenu (couche 7).           │
│  (WAF)           │  Peut bloquer des requetes SQL injection, │
│                  │  XSS, etc. Ex: ModSecurity, Cloudflare.  │
└──────────────────┴───────────────────────────────────────────┘
```

### iptables : le firewall classique de Linux

`iptables` est l'outil historique de filtrage de paquets sous Linux. Il organise les regles en **chaines** :

```bash
# Voir les regles actuelles
sudo iptables -L -n -v

# Les 3 chaines principales :
# INPUT   : paquets entrants (destines a cette machine)
# OUTPUT  : paquets sortants (emis par cette machine)
# FORWARD : paquets transites (cette machine fait routeur)

# Politique par defaut : tout bloquer, puis autoriser au cas par cas
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT

# Autoriser le trafic loopback (localhost)
sudo iptables -A INPUT -i lo -j ACCEPT

# Autoriser les connexions deja etablies
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Autoriser SSH (port 22)
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Autoriser HTTP et HTTPS
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Autoriser le ping (ICMP)
sudo iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT

# Bloquer un IP specifique
sudo iptables -A INPUT -s 203.0.113.50 -j DROP

# Limiter les connexions SSH (anti brute-force)
sudo iptables -A INPUT -p tcp --dport 22 -m limit --limit 3/min -j ACCEPT

# Sauvegarder les regles (Debian/Ubuntu)
sudo iptables-save > /etc/iptables.rules
```

### nftables : le successeur d'iptables

`nftables` est le remplacant moderne d'iptables, avec une syntaxe plus coherente :

```bash
# Voir les regles
sudo nft list ruleset

# Exemple de configuration basique
sudo nft add table inet filter
sudo nft add chain inet filter input { type filter hook input priority 0 \; policy drop \; }
sudo nft add rule inet filter input iif lo accept
sudo nft add rule inet filter input ct state established,related accept
sudo nft add rule inet filter input tcp dport 22 accept
sudo nft add rule inet filter input tcp dport { 80, 443 } accept
```

### ufw : le firewall simplifie pour Ubuntu

**ufw** (Uncomplicated Firewall) est un frontend simplifie pour iptables/nftables :

```bash
# Activer ufw
sudo ufw enable

# Regles de base
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Autoriser des services
sudo ufw allow ssh          # port 22
sudo ufw allow http         # port 80
sudo ufw allow https        # port 443
sudo ufw allow 3306/tcp     # MySQL

# Autoriser depuis une IP specifique
sudo ufw allow from 192.168.1.0/24 to any port 22

# Refuser un port
sudo ufw deny 8080

# Voir le statut et les regles
sudo ufw status verbose

# Supprimer une regle
sudo ufw delete allow 8080
```

> [!info] Quel outil choisir ?
> - **ufw** : ideal pour un serveur simple (VPS, serveur web). Simple et rapide.
> - **iptables** : quand vous avez besoin de regles plus complexes ou de scripts automatises.
> - **nftables** : le futur standard Linux, remplace progressivement iptables.

---

## SSH (Secure Shell)

SSH est le protocole standard pour l'**acces distant securise** aux serveurs Linux/Unix. Il chiffre toute la communication et supporte l'authentification par **cle publique**.

### Comment fonctionne SSH

```
Client SSH                                    Serveur SSH (port 22)
    │                                               │
    │── 1. Connexion TCP ────────────────────────→ │
    │                                               │
    │← 2. Echange de cles (Diffie-Hellman) ──────→│
    │   → Canal chiffre etabli                      │
    │                                               │
    │── 3. Authentification ───────────────────── →│
    │   (mot de passe OU cle publique)              │
    │                                               │
    │←─ 4. Session ouverte ──────────────────────→│
    │   Shell interactif ou commande               │
    │                                               │
```

### Generer une paire de cles SSH

```bash
# Generer une paire de cles (algorithme Ed25519, recommande)
ssh-keygen -t ed25519 -C "alice@example.com"

# Resultat :
# ~/.ssh/id_ed25519       → Cle PRIVEE (ne JAMAIS la partager)
# ~/.ssh/id_ed25519.pub   → Cle publique (a copier sur les serveurs)

# Ou avec RSA (si Ed25519 non supporte)
ssh-keygen -t rsa -b 4096 -C "alice@example.com"
```

> [!warning] Protegez votre cle privee !
> La cle privee (`id_ed25519`) est comme la **cle de votre maison**. Si quelqu'un l'obtient, il a acces a tous les serveurs ou votre cle publique est autorisee. Utilisez toujours une **passphrase** lors de la generation et ne partagez **jamais** le fichier de cle privee.

### Copier la cle publique sur un serveur

```bash
# Methode automatique (recommandee)
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@serveur

# Ce que fait ssh-copy-id en coulisse :
# 1. Se connecte au serveur avec le mot de passe
# 2. Ajoute la cle publique dans ~/.ssh/authorized_keys sur le serveur
# 3. Desormais, la connexion par cle est possible

# Connexion sans mot de passe
ssh user@serveur
```

### Fichier de configuration SSH (~/.ssh/config)

Le fichier `~/.ssh/config` permet de definir des **alias** et des **options** pour chaque serveur :

```bash
# ~/.ssh/config

# Serveur de production
Host prod
    HostName 203.0.113.10
    User deploy
    Port 2222
    IdentityFile ~/.ssh/id_ed25519_prod
    ForwardAgent no

# Serveur de staging
Host staging
    HostName 203.0.113.20
    User deploy
    Port 22
    IdentityFile ~/.ssh/id_ed25519

# Serveur de base de donnees (via le jump host)
Host db
    HostName 10.0.1.50
    User admin
    ProxyJump prod

# Configuration par defaut pour tous les hotes
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
    AddKeysToAgent yes
```

Utilisation :

```bash
# Au lieu de :
ssh -p 2222 -i ~/.ssh/id_ed25519_prod deploy@203.0.113.10

# On tape simplement :
ssh prod
```

### Tunnels SSH

Les tunnels SSH permettent de faire passer du trafic reseau **a travers** une connexion SSH chiffree. Trois types de tunnels :

#### Tunnel local (-L) : acceder a un service distant comme s'il etait local

```bash
# Scenario : la base de donnees MySQL (port 3306) n'est accessible
# que depuis le serveur, pas depuis votre machine locale.

ssh -L 3306:localhost:3306 user@serveur

# Desormais, sur votre machine locale :
mysql -h 127.0.0.1 -P 3306 -u root
# → se connecte en fait au MySQL du serveur, via le tunnel SSH

# Schema :
# [Votre PC :3306] ──tunnel SSH──→ [Serveur] ──→ [MySQL :3306]
```

#### Tunnel distant (-R) : exposer un service local sur le serveur

```bash
# Scenario : votre application tourne sur votre PC en localhost:8080
# et vous voulez la rendre accessible depuis le serveur.

ssh -R 9090:localhost:8080 user@serveur

# Schema :
# [Internet] → [Serveur :9090] ──tunnel SSH──→ [Votre PC :8080]
```

#### Tunnel dynamique (-D) : proxy SOCKS

```bash
# Cree un proxy SOCKS sur votre machine locale
ssh -D 1080 user@serveur

# Configurez votre navigateur pour utiliser le proxy SOCKS localhost:1080
# Tout le trafic web passe par le serveur SSH (comme un VPN leger)
```

### Hardening SSH : securiser le serveur SSH

Editez `/etc/ssh/sshd_config` :

```bash
# Desactiver la connexion root
PermitRootLogin no

# Desactiver l'authentification par mot de passe
PasswordAuthentication no

# Autoriser uniquement l'authentification par cle
PubkeyAuthentication yes

# Changer le port par defaut (optionnel mais reduit le bruit)
Port 2222

# Limiter les utilisateurs autorises
AllowUsers deploy alice

# Desactiver les protocoles inutiles
X11Forwarding no
PermitEmptyPasswords no

# Timeouts
ClientAliveInterval 300
ClientAliveCountMax 2
LoginGraceTime 30

# Algorithmes forts uniquement
KexAlgorithms curve25519-sha256@libssh.org
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
```

```bash
# Apres modification, redemarrer SSH
sudo systemctl restart sshd

# ATTENTION : gardez votre session SSH actuelle ouverte
# et testez dans un AUTRE terminal avant de fermer !
```

> [!warning] Ne vous enfermez pas dehors !
> Quand vous durcissez la configuration SSH, **testez toujours** la nouvelle configuration dans un **second terminal** avant de fermer la session actuelle. Si la configuration est cassee, vous risquez de perdre tout acces au serveur.

### fail2ban : protection contre le brute-force

**fail2ban** surveille les logs et **bannit temporairement** les IP qui font trop de tentatives de connexion echouees :

```bash
# Installation
sudo apt install fail2ban

# Configuration pour SSH
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

```ini
# /etc/fail2ban/jail.local
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3          # 3 tentatives echouees
bantime = 3600        # banni pendant 1 heure
findtime = 600        # dans une fenetre de 10 minutes
```

```bash
# Demarrer fail2ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban

# Voir les IP bannies
sudo fail2ban-client status sshd

# Debannir une IP
sudo fail2ban-client set sshd unbanip 203.0.113.50
```

---

## VPN (Virtual Private Network)

Un VPN cree un **tunnel chiffre** entre votre machine et un serveur distant. Tout le trafic reseau passe par ce tunnel, comme si vous etiez physiquement sur le reseau distant.

### Types de VPN

```
┌──────────────────────────────────────────────────────────────┐
│  Site-to-Site VPN                                            │
│                                                              │
│  [Bureau Paris] ═══tunnel chiffre═══ [Bureau New York]       │
│   192.168.1.0/24                      192.168.2.0/24         │
│  Les deux reseaux communiquent comme un seul LAN             │
├──────────────────────────────────────────────────────────────┤
│  Remote Access VPN                                           │
│                                                              │
│  [Employe a domicile] ═══tunnel═══ [Reseau entreprise]       │
│  L'employe accede aux ressources internes depuis chez lui    │
└──────────────────────────────────────────────────────────────┘
```

### WireGuard : le VPN moderne

**WireGuard** est un protocole VPN recent, simple, rapide et securise. Il utilise ~4000 lignes de code (contre ~100 000 pour OpenVPN) :

```bash
# Installation
sudo apt install wireguard

# Generer les cles (sur chaque machine)
wg genkey | tee privatekey | wg pubkey > publickey

# Configuration serveur (/etc/wireguard/wg0.conf)
[Interface]
PrivateKey = <cle_privee_serveur>
Address = 10.0.0.1/24
ListenPort = 51820

[Peer]
PublicKey = <cle_publique_client>
AllowedIPs = 10.0.0.2/32

# Configuration client (/etc/wireguard/wg0.conf)
[Interface]
PrivateKey = <cle_privee_client>
Address = 10.0.0.2/24
DNS = 1.1.1.1

[Peer]
PublicKey = <cle_publique_serveur>
Endpoint = 203.0.113.10:51820
AllowedIPs = 0.0.0.0/0    # Tout le trafic passe par le VPN
PersistentKeepalive = 25

# Demarrer le VPN
sudo wg-quick up wg0

# Verifier le statut
sudo wg show

# Arreter le VPN
sudo wg-quick down wg0
```

### OpenVPN

OpenVPN est le VPN open source le plus deploye. Plus complexe que WireGuard mais tres flexible :

```bash
# Installation
sudo apt install openvpn

# Connexion avec un fichier de configuration
sudo openvpn --config client.ovpn

# Le fichier .ovpn contient :
# - Adresse du serveur
# - Certificats TLS
# - Configuration reseau
# - Options de chiffrement
```

| Aspect        | WireGuard                | OpenVPN                     |
| ------------- | ------------------------ | --------------------------- |
| Code          | ~4 000 lignes            | ~100 000 lignes             |
| Performance   | Excellente (kernel-level)| Bonne (userspace)           |
| Configuration | Simple                   | Complexe                    |
| Chiffrement   | Moderne (Curve25519, ChaCha20) | Configurable (OpenSSL) |
| Protocole     | UDP uniquement           | TCP ou UDP                  |
| Maturite      | Recent (2020)            | Eprouve (2001)              |

> [!tip] Conseil
> Pour un nouveau deploiement VPN, privilegiez **WireGuard** : il est plus simple, plus rapide et sa base de code reduite facilite l'audit de securite. OpenVPN reste pertinent pour les environnements qui necessitent TCP (reseaux tres restrictifs) ou une compatibilite avec des infrastructures existantes.

---

## TLS/SSL et certificats

### Rappel : qu'est-ce que TLS ?

TLS (Transport Layer Security) chiffre les communications entre un client et un serveur. Il est utilise par HTTPS, SMTPS, IMAPS, et bien d'autres protocoles. Voir [[02 - HTTP et Protocoles Web]] pour le handshake TLS detaille.

### Certificats auto-signes vs CA-signes

| Aspect            | Auto-signe                      | CA-signe (Let's Encrypt, etc.)  |
| ----------------- | ------------------------------- | ------------------------------- |
| Cout              | Gratuit                         | Gratuit (Let's Encrypt) ou payant |
| Confiance         | Non (navigateur affiche un warning) | Oui (CA dans le trust store)  |
| Usage             | Dev/test, interne               | Production, public              |
| Renouvellement    | Manuel                          | Automatique (certbot)           |

### Creer un certificat auto-signe (dev/test)

```bash
# Generer une cle privee et un certificat auto-signe
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem \
  -days 365 -nodes -subj "/CN=localhost"

# Resultat :
# key.pem   → Cle privee
# cert.pem  → Certificat (cle publique + metadonnees)
```

### Let's Encrypt avec certbot (production)

```bash
# Installer certbot
sudo apt install certbot python3-certbot-nginx

# Obtenir un certificat (avec verification HTTP)
sudo certbot --nginx -d example.com -d www.example.com

# Renouvellement automatique (cron)
sudo certbot renew

# Verifier le renouvellement automatique
sudo certbot renew --dry-run

# Les certificats sont dans :
# /etc/letsencrypt/live/example.com/fullchain.pem  (certificat + chaine)
# /etc/letsencrypt/live/example.com/privkey.pem    (cle privee)
```

> [!info] Duree de vie des certificats
> Les certificats Let's Encrypt expirent apres **90 jours**. C'est delibere : ca force l'automatisation du renouvellement et limite les degats en cas de compromission d'une cle. Certbot configure automatiquement un **cron** ou un **timer systemd** pour renouveler avant expiration.

---

## Attaques reseau courantes

### Man-in-the-Middle (MITM)

L'attaquant s'interpose entre le client et le serveur pour intercepter ou modifier les communications :

```
Normal :
  [Client] ──────────────────────────→ [Serveur]

MITM :
  [Client] ──→ [Attaquant] ──→ [Serveur]
               (intercepte, lit,
                modifie les donnees)
```

**Protection** : HTTPS/TLS (le client verifie le certificat du serveur, rendant l'interposition detectable).

### DNS Spoofing (empoisonnement DNS)

L'attaquant modifie les reponses DNS pour rediriger un nom de domaine vers une IP malveillante :

```
Requete DNS : "google.com" → quelle IP ?
  Reponse legitime : 142.250.179.110 (vrai Google)
  Reponse falsifiee : 198.51.100.50  (site de phishing)
```

**Protection** : DNSSEC (signatures cryptographiques sur les enregistrements DNS), DNS over HTTPS (DoH), DNS over TLS (DoT).

### ARP Spoofing

Sur un reseau local, l'attaquant envoie de fausses reponses ARP pour associer son adresse MAC a l'adresse IP de la passerelle, interceptant tout le trafic :

```
Table ARP normale :
  192.168.1.1 (gateway) → AA:BB:CC:DD:EE:FF (MAC du routeur)

Apres ARP spoofing :
  192.168.1.1 (gateway) → 11:22:33:44:55:66 (MAC de l'attaquant !)
  → Tout le trafic destinee a la gateway passe par l'attaquant
```

**Protection** : ARP statique, 802.1X (authentification reseau), detection ARP spoofing (arpwatch).

### DDoS (Distributed Denial of Service)

L'attaquant submerge un serveur avec un volume massif de requetes depuis de nombreuses machines (botnet), rendant le service indisponible :

```
  [Bot 1] ──→
  [Bot 2] ──→   Millions de
  [Bot 3] ──→   requetes     ──→ [Serveur] → SURCHARGE → INDISPONIBLE
  [Bot 4] ──→   simultanees
   ...     ──→
  [Bot N] ──→
```

**Protection** : CDN (Cloudflare, Akamai), rate limiting, scrubbing centers, auto-scaling.

### Port Scanning

L'attaquant scanne les ports d'un serveur pour decouvrir les services exposes et identifier des vulnerabilites :

```bash
# Ce que fait un attaquant (et un administrateur pour auditer) :
nmap -sV 203.0.113.10
# Resultat :
# 22/tcp   open  ssh      OpenSSH 8.9
# 80/tcp   open  http     nginx 1.24
# 443/tcp  open  https
# 3306/tcp open  mysql    MySQL 8.0   ← CE PORT NE DEVRAIT PAS ETRE EXPOSE !
```

**Protection** : firewall (n'exposer que les ports necessaires), port knocking, IDS/IPS.

> [!warning] Connaitre les attaques pour mieux se defendre
> Comprendre les techniques d'attaque n'est pas un encouragement a les utiliser de maniere malveillante. C'est **indispensable** pour concevoir des systemes securises. Testez toujours uniquement sur des systemes dont vous avez l'**autorisation explicite**.

---

## Outils de securite reseau

### nmap : scanner de ports

**nmap** est l'outil de reference pour la decouverte reseau et l'audit de securite :

```bash
# Scan basique des ports ouverts
nmap 192.168.1.1

# Scan avec detection de version des services
nmap -sV 192.168.1.1

# Scan de tout un sous-reseau
nmap 192.168.1.0/24

# Scan rapide (top 100 ports)
nmap -F 192.168.1.1

# Scan UDP (plus lent)
nmap -sU 192.168.1.1

# Scan avec detection d'OS
nmap -O 192.168.1.1

# Scan complet (tous les ports TCP)
nmap -p- 192.168.1.1

# Scan furtif (SYN scan, ne complete pas le handshake)
sudo nmap -sS 192.168.1.1

# Scanner un port specifique
nmap -p 22,80,443 192.168.1.1
```

> [!example] Utilisation ethique de nmap
> Scannez uniquement les machines dont vous etes **responsable** ou pour lesquelles vous avez une **autorisation ecrite**. Scanner des machines sans autorisation est **illegal** dans la plupart des pays.

### Wireshark : analyse de paquets

**Wireshark** est un analyseur de paquets graphique qui capture et decode le trafic reseau en temps reel :

```
Wireshark permet de :
- Voir CHAQUE paquet traversant une interface reseau
- Filtrer par protocole, IP, port, contenu
- Suivre une "conversation" TCP complete
- Identifier des anomalies (retransmissions, erreurs)
- Analyser le handshake TLS

Filtres utiles :
  http                     → Tout le trafic HTTP
  tcp.port == 443          → Tout le trafic HTTPS
  ip.addr == 192.168.1.42  → Trafic d'une IP specifique
  dns                      → Requetes DNS
  tcp.flags.syn == 1       → Paquets SYN (debut de connexion)
```

> [!tip] tcpdump comme alternative en ligne de commande
> Si vous n'avez pas d'interface graphique (serveur), utilisez `tcpdump` (voir [[01 - Fondamentaux Reseaux]]) et exportez le fichier `.pcap` pour l'analyser dans Wireshark sur votre machine locale :
> ```bash
> sudo tcpdump -i eth0 -w capture.pcap
> # Transferer le fichier puis ouvrir avec Wireshark
> ```

### fail2ban : protection brute-force

Deja vu dans la section SSH. fail2ban surveille les logs de **n'importe quel** service et bannit les IP suspectes :

```ini
# Protection pour nginx (tentatives de login)
[nginx-http-auth]
enabled = true
port = http,https
filter = nginx-http-auth
logpath = /var/log/nginx/error.log
maxretry = 3
bantime = 3600

# Protection pour un service custom
[mon-app]
enabled = true
port = 8080
filter = mon-app
logpath = /var/log/mon-app/access.log
maxretry = 5
bantime = 600
```

---

## Le modele Zero Trust

Le modele de securite traditionnel est base sur le **perimetre** : on fait confiance a tout ce qui est **a l'interieur** du reseau et on bloque ce qui vient de l'exterieur. Le modele **Zero Trust** rejette cette approche.

### Principes

```
Modele traditionnel (chateau et douves) :
┌─────────────────────────────────────┐
│  RESEAU INTERNE (de confiance)      │  ← "Tout ce qui est dedans est sur"
│  [Serveur A] ←→ [Serveur B]        │
│  [Base DB]   ←→ [Serveur C]        │  ← Pas d'authentification interne
│                                     │
├──── FIREWALL ───────────────────────┤
│  INTERNET (non fiable)              │
└─────────────────────────────────────┘
Probleme : si un attaquant passe le firewall, il a acces a TOUT.

Modele Zero Trust :
┌─────────────────────────────────────┐
│  CHAQUE service verifie chaque      │
│  requete, meme interne.             │
│                                     │
│  [Serveur A] ──auth+chiffre──→ [B] │
│  [Client]    ──auth+chiffre──→ [DB]│
│                                     │
│  "Never trust, always verify"       │
└─────────────────────────────────────┘
```

### Les piliers du Zero Trust

1. **Verifier explicitement** : chaque requete est authentifiee et autorisee, quelle que soit sa provenance
2. **Principe du moindre privilege** : chaque utilisateur/service n'a acces qu'a ce dont il a strictement besoin
3. **Supposer la compromission** : concevoir les systemes comme si l'attaquant etait deja a l'interieur
4. **Microsegmentation** : decouper le reseau en zones minuscules, chacune avec ses propres controles d'acces

> [!info] Zero Trust en pratique
> - **mTLS** (mutual TLS) : les services s'authentifient mutuellement avec des certificats
> - **Service mesh** (Istio, Linkerd) : gere l'authentification et le chiffrement entre microservices
> - **Identity-aware proxy** (BeyondCorp de Google) : chaque acces est verifie par un proxy central
> - **RBAC/ABAC** : controle d'acces base sur les roles ou les attributs

---

## Bonnes pratiques de securite reseau

### 1. Principe du moindre privilege

```
MAUVAIS :
  - Un serveur web a acces root
  - Un developpeur a les droits admin sur la production
  - Un service a acces a toutes les bases de donnees

BON :
  - Le serveur web tourne avec un utilisateur dedie sans privileges
  - Les developpeurs ont des acces en lecture seule sur la production
  - Chaque service n'accede qu'a sa propre base de donnees
```

### 2. Chiffrer tout le trafic

```bash
# HTTPS pour les APIs et sites web
# TLS pour les connexions aux bases de donnees
# SSH pour l'acces distant
# VPN ou mTLS pour la communication inter-services
# Chiffrement au repos pour les donnees sensibles en base

# Verifier la configuration TLS d'un serveur :
nmap --script ssl-enum-ciphers -p 443 example.com
```

### 3. Maintenir les systemes a jour

```bash
# Mises a jour de securite automatiques (Ubuntu)
sudo apt install unattended-upgrades
sudo dpkg-reconfigure unattended-upgrades

# Verifier les mises a jour disponibles
sudo apt update && sudo apt list --upgradable

# Mettre a jour tout
sudo apt upgrade -y
```

### 4. Surveiller et logger

```bash
# Centraliser les logs (syslog, journald)
journalctl -u sshd --since "1 hour ago"

# Surveiller les connexions SSH echouees
grep "Failed password" /var/log/auth.log | tail -20

# Surveiller les ports ouverts regulierement
ss -tlnp

# Monitorer le trafic reseau inhabituel
sudo tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn) != 0'
```

### 5. Checklist securite serveur

> [!example] Checklist de securite pour un nouveau serveur
> - [ ] Mettre a jour le systeme (`apt update && apt upgrade`)
> - [ ] Creer un utilisateur non-root pour l'administration
> - [ ] Configurer l'authentification SSH par cle uniquement
> - [ ] Desactiver la connexion root SSH
> - [ ] Changer le port SSH (optionnel)
> - [ ] Installer et configurer un firewall (ufw)
> - [ ] N'ouvrir que les ports necessaires (22, 80, 443)
> - [ ] Installer fail2ban
> - [ ] Configurer les mises a jour automatiques de securite
> - [ ] Configurer HTTPS avec Let's Encrypt
> - [ ] Configurer des backups reguliers
> - [ ] Mettre en place du monitoring et des alertes

---

## Carte Mentale ASCII

```
                          SECURITE RESEAU
                                │
        ┌───────────┬───────────┼───────────┬──────────────┐
        │           │           │           │              │
    Firewalls     SSH         VPN        Attaques      Principes
        │           │           │           │              │
   ┌────┴────┐  ┌───┴───┐  ┌───┴───┐  ┌───┴───┐    ┌────┴────┐
   │    │    │  │       │  │       │  │       │    │         │
  ufw  ipt  nft Cles  Tunnels WG  OVPN MITM DDoS Zero    Defense
       ables    Ed25519 -L      DNS spoof   Trust  profondeur
               config  -R      ARP spoof   Moindre
               fail2ban -D     Port scan   privilege
               hardening                   Chiffrer
                                           Surveiller
```

---

## Exercices

### Exercice 1 : Securiser un serveur

Vous venez de recevoir l'acces root a un nouveau VPS Ubuntu. Ecrivez un script bash qui automatise les etapes suivantes :
1. Mettre a jour le systeme
2. Creer un utilisateur `deploy` avec des droits sudo
3. Copier votre cle SSH publique pour l'utilisateur `deploy`
4. Configurer SSH (desactiver root, desactiver password auth)
5. Installer et configurer ufw (SSH, HTTP, HTTPS)
6. Installer fail2ban avec une configuration SSH

### Exercice 2 : Tunnels SSH

1. Configurez un tunnel local pour acceder a un service PostgreSQL (port 5432) distant via SSH
2. Configurez un tunnel distant pour exposer votre serveur de dev local (port 3000) sur le port 9000 du serveur
3. Ecrivez l'entree correspondante dans votre `~/.ssh/config` pour ne pas retaper la commande a chaque fois

### Exercice 3 : Audit de securite

Utilisez nmap pour scanner votre propre machine ou un serveur de test :
1. Listez tous les ports ouverts
2. Identifiez les services et leurs versions
3. Y a-t-il des ports qui ne devraient pas etre exposes ?
4. Proposez des regles de firewall pour fermer les ports inutiles

### Exercice 4 : Analyse de trafic

1. Utilisez `tcpdump` pour capturer le trafic HTTP (port 80) pendant 30 secondes
2. Envoyez une requete `curl http://example.com` pendant la capture
3. Analysez la capture : combien de paquets ont ete echanges ? Pouvez-vous identifier le handshake TCP, la requete HTTP et la reponse ?
4. Repetez avec HTTPS (port 443) : pouvez-vous lire le contenu ? Pourquoi ?

---

## Liens

- [[01 - Fondamentaux Reseaux]] -- Modele OSI, TCP/IP, outils reseau de base
- [[02 - HTTP et Protocoles Web]] -- HTTPS, TLS, cookies et authentification
- [[02 - Securite Web OWASP]] -- Vulnerabilites web : XSS, injection SQL, CSRF
