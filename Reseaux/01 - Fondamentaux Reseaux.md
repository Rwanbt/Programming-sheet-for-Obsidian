# 01 - Fondamentaux Reseaux

## Pourquoi les reseaux sont essentiels pour un developpeur

Aujourd'hui, quasiment **aucune application** ne fonctionne de maniere isolee. Que ce soit une application web, mobile, un microservice ou meme un script qui appelle une API, tout passe par le **reseau**. Comprendre les fondamentaux des reseaux, c'est comprendre **comment les donnees voyagent** d'un point A a un point B.

> [!info] Ce que vous allez apprendre
> - Le modele OSI et le modele TCP/IP
> - Les adresses IP, les ports et le sous-reseau
> - TCP vs UDP : deux philosophies de communication
> - DNS, DHCP et NAT : les services qui font fonctionner Internet
> - Les outils reseau indispensables pour diagnostiquer et debugger

> [!tip] Analogie
> Imaginez le reseau comme le **systeme postal mondial**. Les adresses IP sont les adresses postales, les ports sont les numeros d'appartement, les routeurs sont les centres de tri, et les protocoles (TCP, UDP) sont les differents modes d'envoi (recommande avec accuse de reception, ou simple carte postale sans garantie).

---

## Le modele OSI (Open Systems Interconnection)

Le modele OSI est un modele **theorique** en **7 couches** qui decrit comment les donnees circulent dans un reseau. Il a ete cree par l'ISO en 1984 pour standardiser les communications reseau.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MODELE OSI - 7 COUCHES                          │
├──────┬──────────────────────┬───────────────────────────────────────┤
│  7   │  Application         │  HTTP, FTP, SMTP, DNS, SSH           │
├──────┼──────────────────────┼───────────────────────────────────────┤
│  6   │  Presentation        │  SSL/TLS, JPEG, ASCII, encryption    │
├──────┼──────────────────────┼───────────────────────────────────────┤
│  5   │  Session             │  NetBIOS, RPC, gestion de sessions   │
├──────┼──────────────────────┼───────────────────────────────────────┤
│  4   │  Transport           │  TCP, UDP (ports, segments)          │
├──────┼──────────────────────┼───────────────────────────────────────┤
│  3   │  Reseau              │  IP, ICMP, routage (paquets)         │
├──────┼──────────────────────┼───────────────────────────────────────┤
│  2   │  Liaison de donnees  │  Ethernet, Wi-Fi, MAC (trames)       │
├──────┼──────────────────────┼───────────────────────────────────────┤
│  1   │  Physique            │  Cables, ondes radio, signaux        │
└──────┴──────────────────────┴───────────────────────────────────────┘
         Donnees descendantes ↓              ↑ Donnees montantes
         (encapsulation)                     (decapsulation)
```

### Couche 7 : Application

C'est la couche la plus proche de l'utilisateur. Elle fournit les **protocoles** que les applications utilisent directement : HTTP pour le web, SMTP pour les emails, FTP pour le transfert de fichiers, DNS pour la resolution de noms.

```
Navigateur web  →  requete HTTP GET /index.html  →  serveur web
Client email    →  commande SMTP MAIL FROM:       →  serveur mail
```

### Couche 6 : Presentation

Elle s'occupe du **format des donnees** : encodage (ASCII, UTF-8), compression, et surtout **chiffrement** (TLS/SSL). C'est cette couche qui garantit que les deux extremites "parlent la meme langue".

### Couche 5 : Session

Elle gere l'**ouverture, le maintien et la fermeture** des sessions de communication entre deux machines. En pratique, cette couche est souvent fusionnee avec les couches 6 et 7 dans les implementations reelles.

### Couche 4 : Transport

C'est la couche de **TCP** et **UDP**. Elle decoupe les donnees en **segments**, assure (ou non) la livraison fiable, et utilise les **ports** pour acheminer les donnees vers la bonne application.

> [!tip] Analogie
> La couche Transport est comme le service postal qui choisit entre un envoi en **recommande avec accuse de reception** (TCP) ou une **simple carte postale** sans suivi (UDP).

### Couche 3 : Reseau

C'est la couche d'**IP** (Internet Protocol). Elle s'occupe de l'**adressage logique** (adresses IP) et du **routage** : trouver le meilleur chemin pour acheminer un **paquet** d'un reseau a un autre.

### Couche 2 : Liaison de donnees

Elle gere la communication entre deux machines **directement connectees** sur le meme reseau local. Elle utilise les **adresses MAC** (adresses physiques uniques de chaque carte reseau) et organise les donnees en **trames** (frames).

### Couche 1 : Physique

C'est le **medium physique** : cables Ethernet (cuivre, fibre optique), ondes Wi-Fi, signaux electriques. Elle definit les tensions, frequences et connecteurs.

### L'encapsulation des donnees

Quand des donnees descendent les couches, chaque couche ajoute son **en-tete** (header). C'est l'**encapsulation** :

```
Couche 7-5 :  [        DONNEES        ]
                          ↓
Couche 4 :    [ TCP HDR | DONNEES     ]       → Segment
                          ↓
Couche 3 :    [ IP HDR  | TCP | DONNEES ]     → Paquet
                          ↓
Couche 2 :    [ ETH HDR | IP | TCP | DONNEES | ETH FTR ] → Trame
                          ↓
Couche 1 :    01101001 01110100 ...            → Bits
```

> [!info] Mnemonique pour retenir les 7 couches
> De bas en haut : **P**lease **D**o **N**ot **T**hrow **S**ausage **P**izza **A**way
> (Physique, Donnees, Network, Transport, Session, Presentation, Application)

---

## Le modele TCP/IP (modele Internet)

Le modele TCP/IP est le modele **pratique** utilise sur Internet. Il ne comporte que **4 couches** et correspond a l'implementation reelle des protocoles.

```
┌──────────────────────────────────────────────────────────────┐
│              COMPARAISON OSI vs TCP/IP                        │
├─────────┬──────────────────┬─────────┬───────────────────────┤
│   OSI   │  Nom couche OSI  │  TCP/IP │  Nom couche TCP/IP    │
├─────────┼──────────────────┼─────────┼───────────────────────┤
│    7    │  Application      │         │                       │
│    6    │  Presentation     │    4    │  Application          │
│    5    │  Session          │         │  (HTTP, DNS, SSH...)  │
├─────────┼──────────────────┼─────────┼───────────────────────┤
│    4    │  Transport        │    3    │  Transport            │
│         │                  │         │  (TCP, UDP)           │
├─────────┼──────────────────┼─────────┼───────────────────────┤
│    3    │  Reseau           │    2    │  Internet             │
│         │                  │         │  (IP, ICMP)           │
├─────────┼──────────────────┼─────────┼───────────────────────┤
│    2    │  Liaison          │    1    │  Acces reseau         │
│    1    │  Physique         │         │  (Ethernet, Wi-Fi)    │
└─────────┴──────────────────┴─────────┴───────────────────────┘
```

> [!info] OSI vs TCP/IP : lequel retenir ?
> - Le modele **OSI** est un modele **de reference** : il sert a **comprendre** et **enseigner** les concepts reseau
> - Le modele **TCP/IP** est le modele **d'implementation** : c'est celui qui est reellement utilise sur Internet
> - En pratique, on utilise les termes des deux modeles de maniere interchangeable

---

## Adresses IP

Une adresse IP est un **identifiant unique** attribue a chaque appareil connecte a un reseau. C'est l'equivalent d'une **adresse postale** dans le monde numerique.

### IPv4

Une adresse IPv4 est composee de **4 octets** (32 bits), separes par des points :

```
Exemple :  192.168.1.42

En binaire :  11000000.10101000.00000001.00101010
              |---8b---|---8b---|---8b---|---8b---|
                       Total : 32 bits

Plage totale :  0.0.0.0  →  255.255.255.255
Nombre d'adresses :  2^32 = 4 294 967 296 (~4.3 milliards)
```

### Plages d'adresses privees (RFC 1918)

Certaines plages sont reservees aux **reseaux locaux** et ne sont **pas routables** sur Internet :

| Classe | Plage                       | Masque par defaut | Nombre d'adresses |
| ------ | --------------------------- | ----------------- | ----------------- |
| A      | `10.0.0.0` - `10.255.255.255`     | `/8` (255.0.0.0)      | ~16.7 millions    |
| B      | `172.16.0.0` - `172.31.255.255`   | `/12` (255.240.0.0)   | ~1 million        |
| C      | `192.168.0.0` - `192.168.255.255` | `/16` (255.255.0.0)   | ~65 000           |

> [!tip] Analogie
> Les adresses privees sont comme les numeros de chambre **a l'interieur d'un hotel**. De l'exterieur, on connait l'adresse de l'hotel (IP publique), mais les numeros de chambre (IP privees) ne sont visibles que de l'interieur.

### Adresses speciales

| Adresse           | Role                                                    |
| ----------------- | ------------------------------------------------------- |
| `127.0.0.1`       | **Loopback** : la machine se parle a elle-meme (localhost) |
| `0.0.0.0`         | "Toutes les interfaces" (ecouter sur tout)              |
| `255.255.255.255` | **Broadcast** : envoyer a tous sur le reseau local      |
| `169.254.x.x`     | **Link-local** : auto-attribuee si pas de DHCP          |

### IPv6

IPv4 a un probleme : seulement ~4.3 milliards d'adresses, ce qui est **insuffisant** pour tous les appareils connectes. IPv6 utilise **128 bits** :

```
IPv4 :  192.168.1.42                            (32 bits)
IPv6 :  2001:0db8:85a3:0000:0000:8a2e:0370:7334 (128 bits)

Nombre d'adresses IPv6 : 2^128 = 3.4 × 10^38
(assez pour donner une adresse a chaque grain de sable sur Terre)
```

Les regles de notation IPv6 :
- On peut supprimer les zeros en tete : `0db8` → `db8`
- On peut remplacer un groupe consecutif de `0000` par `::` (une seule fois)
- Exemple : `2001:db8::8a2e:370:7334`

> [!info] IPv6 en pratique
> L'adoption d'IPv6 est lente mais progresse. Google rapporte environ 45% de trafic IPv6 en 2025. En tant que developpeur, il est important de supporter les deux versions dans vos applications (double stack).

---

## Les ports

Un port est un **numero** (de 0 a 65535) qui identifie une **application** ou un **service** sur une machine. Si l'adresse IP est l'adresse d'un immeuble, le port est le **numero d'appartement**.

```
Requete HTTP :   192.168.1.42:80
                 └─────┬─────┘ └┬┘
                  Adresse IP   Port

Connexion SSH :  10.0.0.5:22
Base MySQL :     10.0.0.5:3306
```

### Ports bien connus (0 - 1023)

| Port  | Protocole | Service                              |
| ----- | --------- | ------------------------------------ |
| 20/21 | FTP       | Transfert de fichiers                |
| 22    | SSH       | Shell securise / acces distant       |
| 25    | SMTP      | Envoi d'emails                       |
| 53    | DNS       | Resolution de noms de domaine        |
| 80    | HTTP      | Web non chiffre                      |
| 443   | HTTPS     | Web chiffre (TLS)                    |
| 3306  | MySQL     | Base de donnees MySQL/MariaDB        |
| 5432  | PostgreSQL| Base de donnees PostgreSQL           |
| 6379  | Redis     | Cache / base cle-valeur en memoire   |
| 8080  | HTTP Alt  | Serveur de dev / proxy               |
| 27017 | MongoDB   | Base de donnees MongoDB              |

### Ports enregistres (1024 - 49151)

Ports utilises par des applications specifiques mais sans privilege root.

### Ports ephemeres (49152 - 65535)

Ce sont les ports **attribues dynamiquement** par le systeme d'exploitation pour les connexions sortantes. Quand votre navigateur se connecte a un serveur web, il utilise un port ephemere aleatoire comme port **source**.

```
Votre navigateur              Serveur web
  Port 52847 (ephemere) ────→ Port 443 (HTTPS)
  Port 52848 (ephemere) ────→ Port 443 (HTTPS)
```

> [!warning] Securite des ports
> Sur un serveur, ne laissez **jamais** des ports ouverts inutilement. Chaque port ouvert est une porte d'entree potentielle pour un attaquant. Utilisez un pare-feu pour n'exposer que les ports necessaires.

---

## Sous-reseau (Subnetting)

Le sous-reseau permet de **decouper** un reseau IP en sous-reseaux plus petits. C'est essentiel pour organiser les reseaux, limiter le trafic broadcast et ameliorer la securite.

### Notation CIDR

La notation **CIDR** (Classless Inter-Domain Routing) indique combien de bits sont reserves a la **partie reseau** de l'adresse :

```
192.168.1.0/24
└────┬─────┘ └┬┘
  Adresse    Nombre de bits reseau (masque)

/24 signifie : les 24 premiers bits = partie reseau
               les 8 bits restants  = partie hote
```

### Masques de sous-reseau

| CIDR | Masque            | Bits hotes | Nb hotes utilisables |
| ---- | ----------------- | ---------- | -------------------- |
| /8   | 255.0.0.0         | 24         | 16 777 214           |
| /16  | 255.255.0.0       | 16         | 65 534               |
| /24  | 255.255.255.0     | 8          | 254                  |
| /25  | 255.255.255.128   | 7          | 126                  |
| /26  | 255.255.255.192   | 6          | 62                   |
| /27  | 255.255.255.224   | 5          | 30                   |
| /28  | 255.255.255.240   | 4          | 14                   |
| /30  | 255.255.255.252   | 2          | 2                    |
| /32  | 255.255.255.255   | 0          | 1 (une seule machine)|

### Calculer adresse reseau, broadcast et hotes

Pour le reseau `192.168.1.100/26` :

```
Etape 1 : Ecrire l'adresse en binaire
  192.168.1.100 = 11000000.10101000.00000001.01100100

Etape 2 : Le masque /26 signifie 26 bits reseau, 6 bits hotes
  Masque = 11111111.11111111.11111111.11000000 = 255.255.255.192

Etape 3 : Adresse reseau (ET logique entre adresse et masque)
  11000000.10101000.00000001.01100100
  AND
  11111111.11111111.11111111.11000000
  = 11000000.10101000.00000001.01000000
  = 192.168.1.64                            ← Adresse reseau

Etape 4 : Adresse broadcast (bits hotes tous a 1)
  11000000.10101000.00000001.01111111
  = 192.168.1.127                           ← Adresse broadcast

Etape 5 : Plage d'hotes utilisables
  Premier hote : 192.168.1.65
  Dernier hote : 192.168.1.126
  Nombre d'hotes : 2^6 - 2 = 62            (on retire reseau + broadcast)
```

> [!info] Le lien avec les operations binaires
> Le subnetting repose entierement sur les **operations binaires** : le ET logique (AND) pour calculer l'adresse reseau, le OU logique (OR) pour le broadcast. Si ces operations ne sont pas claires, consultez la note sur les bases numeriques : [[02 - Bases Numeriques]].

---

## TCP vs UDP

Ce sont les deux protocoles de la **couche Transport**. Ils ont des philosophies fondamentalement differentes.

### TCP (Transmission Control Protocol)

TCP est un protocole **oriente connexion** qui garantit la **livraison fiable** et **ordonnee** des donnees. Avant tout echange, les deux parties etablissent une connexion via le **three-way handshake** :

```
    Client                          Serveur
      │                               │
      │──── SYN (seq=100) ──────────→│  1. Le client demande une connexion
      │                               │
      │←── SYN-ACK (seq=300,ack=101)──│  2. Le serveur accepte et repond
      │                               │
      │──── ACK (ack=301) ──────────→│  3. Le client confirme
      │                               │
      │     CONNEXION ETABLIE         │
      │                               │
      │←──── Echange de donnees ────→│
      │                               │
      │──── FIN ────────────────────→│  Fermeture de connexion
      │←── FIN-ACK ─────────────────│
      │──── ACK ────────────────────→│
      │                               │
```

Mecanismes de fiabilite de TCP :
- **Numeros de sequence** : chaque octet est numerote pour garantir l'ordre
- **Acquittements (ACK)** : le recepteur confirme la reception
- **Retransmission** : les segments perdus sont renvoyes apres un timeout
- **Controle de flux** : le recepteur peut ralentir l'emetteur (fenetre glissante)
- **Controle de congestion** : TCP adapte son debit en fonction de l'etat du reseau

### UDP (User Datagram Protocol)

UDP est un protocole **sans connexion** et **sans garantie**. Il envoie des **datagrammes** sans se soucier de savoir s'ils arrivent, dans quel ordre, ou s'ils sont dupliques.

```
    Client                          Serveur
      │                               │
      │──── Datagramme 1 ───────────→│  Envoye, pas de confirmation
      │──── Datagramme 2 ───────────→│  Envoye, pas de confirmation
      │──── Datagramme 3 ──── X      │  Perdu ! Pas de retransmission
      │──── Datagramme 4 ───────────→│  Continue sans attendre
      │                               │
```

### Comparaison TCP vs UDP

| Caracteristique       | TCP                          | UDP                        |
| --------------------- | ---------------------------- | -------------------------- |
| Connexion             | Oriente connexion (handshake)| Sans connexion             |
| Fiabilite             | Livraison garantie           | Aucune garantie            |
| Ordre                 | Donnees ordonnees            | Pas d'ordre garanti        |
| Controle de flux      | Oui (fenetre glissante)      | Non                        |
| Overhead              | Eleve (en-tete 20+ octets)  | Faible (en-tete 8 octets) |
| Vitesse               | Plus lent (fiabilite couteuse)| Plus rapide               |
| Cas d'usage           | Web, email, fichiers, SSH    | Streaming, jeux, DNS, VoIP|

> [!example] Quand utiliser quoi ?
> - **TCP** : tout ce qui necessite une livraison complete et ordonnee
>   - Navigation web (HTTP/HTTPS)
>   - Emails (SMTP, IMAP)
>   - Transfert de fichiers (FTP, SCP)
>   - Connexion SSH
>   - Bases de donnees
> - **UDP** : tout ce qui privilegien la **vitesse** sur la fiabilite
>   - Streaming video/audio (il vaut mieux perdre une image que tout bloquer)
>   - Jeux en ligne (la position du joueur il y a 2 secondes est obsolete)
>   - DNS (requetes courtes, on peut redemander si pas de reponse)
>   - VoIP (appels telephoniques sur IP)

---

## DNS (Domain Name System)

Le DNS est le **systeme de noms de domaine**. Il traduit les noms lisibles par les humains (comme `google.com`) en adresses IP (comme `142.250.179.110`). C'est l'**annuaire telephonique** d'Internet.

### Resolution DNS etape par etape

Quand vous tapez `www.example.com` dans votre navigateur :

```
┌──────────┐     1. www.example.com ?     ┌──────────────┐
│          │ ──────────────────────────→  │   Resolveur   │
│  Votre   │                              │   DNS local   │
│ navigateur│ ←──────────────────────────  │  (FAI/cache)  │
│          │     8. 93.184.216.34         │              │
└──────────┘                              └──────┬───────┘
                                                 │
        ┌────────────────────────────────────────┤
        │                                        │
        │  2. Qui gere .com ?                    │
        ↓                                        │
┌──────────────┐                                 │
│  Serveur DNS │  3. Demande a a.gtld-servers.net│
│   RACINE     │ ────────────────────────────→   │
│  (root)      │                                 │
└──────────────┘                                 │
                                                 │
        │  4. Qui gere example.com ?             │
        ↓                                        │
┌──────────────┐                                 │
│  Serveur TLD │  5. Demande aux NS de example.com
│  (.com)      │ ────────────────────────────→   │
└──────────────┘                                 │
                                                 │
        │  6. Quelle est l'IP de www.example.com?│
        ↓                                        │
┌──────────────┐                                 │
│  Serveur     │  7. Reponse : 93.184.216.34     │
│  autoritaire │ ────────────────────────────→   │
│ (example.com)│                                 │
└──────────────┘                                 │
```

### Types d'enregistrements DNS

| Type   | Role                                    | Exemple                           |
| ------ | --------------------------------------- | --------------------------------- |
| A      | Nom → adresse IPv4                      | `example.com → 93.184.216.34`    |
| AAAA   | Nom → adresse IPv6                      | `example.com → 2606:2800:...`    |
| CNAME  | Alias vers un autre nom                 | `www.example.com → example.com`  |
| MX     | Serveur de messagerie                   | `example.com → mail.example.com` |
| NS     | Serveur DNS autoritaire pour la zone    | `example.com → ns1.example.com`  |
| TXT    | Texte libre (SPF, DKIM, verification)   | `"v=spf1 include:..." `          |
| SOA    | Informations sur la zone DNS            | Serial, refresh, TTL...          |
| PTR    | Resolution inverse (IP → nom)           | `34.216.184.93 → example.com`    |

### Le fichier /etc/hosts

Le fichier `/etc/hosts` (ou `C:\Windows\System32\drivers\etc\hosts` sur Windows) permet de definir des associations nom/IP **locales**, sans passer par le DNS :

```bash
# Format : IP       nom_de_domaine
127.0.0.1       localhost
127.0.0.1       mon-projet.local
192.168.1.100   serveur-interne.local
```

> [!tip] Utilisation en developpement
> Le fichier hosts est tres utile pour le developpement local : vous pouvez faire pointer un nom de domaine vers `127.0.0.1` pour tester une application web avec un "vrai" nom de domaine, ou pour bloquer des domaines publicitaires.

### Outils DNS

```bash
# nslookup : interroger le DNS (simple)
nslookup google.com
nslookup -type=MX google.com

# dig : interroger le DNS (detaille, Linux/macOS)
dig google.com
dig google.com MX
dig +trace google.com    # Voir toute la chaine de resolution
dig +short google.com    # Reponse courte (juste l'IP)

# Vider le cache DNS local
# Linux :
sudo systemd-resolve --flush-caches
# macOS :
sudo dscacheutil -flushcache
# Windows :
ipconfig /flushdns
```

---

## DHCP (Dynamic Host Configuration Protocol)

Le DHCP est le protocole qui attribue **automatiquement** une adresse IP a un appareil quand il se connecte a un reseau. Sans DHCP, il faudrait configurer manuellement l'IP de chaque appareil.

### Le processus DORA

L'attribution d'une adresse IP par DHCP suit 4 etapes, appelees **DORA** :

```
    Client (nouveau)                      Serveur DHCP
         │                                     │
         │──── 1. DISCOVER (broadcast) ──────→│  "Je cherche un serveur DHCP"
         │     (src: 0.0.0.0, dst: 255.x)     │
         │                                     │
         │←─── 2. OFFER ─────────────────────│  "Voici une IP disponible"
         │     (IP proposee: 192.168.1.42)     │
         │                                     │
         │──── 3. REQUEST ──────────────────→│  "J'accepte cette IP"
         │     (192.168.1.42 s'il vous plait)  │
         │                                     │
         │←─── 4. ACK ──────────────────────│  "C'est confirme, elle est a toi"
         │     (bail de 24h, masque, gateway)   │
         │                                     │
```

Le serveur DHCP fournit non seulement l'**adresse IP**, mais aussi :
- Le **masque de sous-reseau** (ex : 255.255.255.0)
- La **passerelle par defaut** (gateway, ex : 192.168.1.1)
- Les **serveurs DNS** a utiliser
- La **duree du bail** (lease time)

> [!info] Bail DHCP (lease)
> L'adresse IP n'est pas attribuee indefiniment. Le bail a une duree (souvent 24h). A mi-bail, le client tente de **renouveler** aupres du meme serveur. Si le bail expire sans renouvellement, l'adresse est liberee et peut etre reutilisee.

---

## NAT (Network Address Translation)

Le NAT est un mecanisme qui **traduit** les adresses IP privees en adresses IP publiques (et inversement). C'est ce qui permet a des dizaines d'appareils chez vous de partager **une seule** adresse IP publique.

```
Reseau local (prive)                    Internet (public)
┌─────────────────────┐
│ PC     192.168.1.10 │
│ Tel    192.168.1.11 │──→ Routeur NAT ──→  IP publique: 86.42.15.200
│ Tablet 192.168.1.12 │   192.168.1.1
└─────────────────────┘

Quand le PC envoie une requete :
  Source : 192.168.1.10:52000  → Routeur →  Source : 86.42.15.200:40001
  Le routeur traduit et garde une table de correspondance :

  ┌─────────────────────────┬────────────────────────────┐
  │  Interne                │  Externe                   │
  ├─────────────────────────┼────────────────────────────┤
  │  192.168.1.10:52000     │  86.42.15.200:40001        │
  │  192.168.1.11:48300     │  86.42.15.200:40002        │
  │  192.168.1.12:51200     │  86.42.15.200:40003        │
  └─────────────────────────┴────────────────────────────┘
```

> [!warning] Limitation du NAT
> Le NAT fonctionne bien pour les connexions **sortantes** (initiees depuis le reseau local), mais il complique les connexions **entrantes**. C'est pourquoi il faut configurer une **redirection de port** (port forwarding) pour heberger un serveur derriere un NAT.

Le NAT a aussi ete une "solution" temporaire a la penurie d'adresses IPv4 : au lieu d'une IP par appareil, une seule IP publique suffit pour tout un foyer ou une entreprise. IPv6, avec son espace d'adressage immense, rendrait le NAT theoriquement inutile.

---

## ARP (Address Resolution Protocol)

ARP est le protocole qui fait le lien entre les adresses **IP** (couche 3) et les adresses **MAC** (couche 2). Quand une machine veut communiquer avec une autre sur le meme reseau local, elle a besoin de connaitre l'adresse MAC de la destination.

### Fonctionnement

```
Machine A (192.168.1.10) veut envoyer un paquet a Machine B (192.168.1.20)

1. A cherche dans sa table ARP : "ai-je deja l'adresse MAC de 192.168.1.20 ?"
   → Non (pas dans le cache)

2. A envoie un ARP Request en BROADCAST :
   "Qui est 192.168.1.20 ? Repondez a AA:BB:CC:11:22:33 (mon MAC)"
   → Toutes les machines du reseau local recoivent ce message

3. Machine B reconnait son IP et repond en UNICAST :
   "Je suis 192.168.1.20, mon MAC est DD:EE:FF:44:55:66"

4. A met a jour sa table ARP et peut maintenant envoyer la trame Ethernet
```

```bash
# Voir la table ARP de votre machine
arp -a              # Windows / ancien Linux
ip neigh show       # Linux moderne

# Exemple de sortie :
# 192.168.1.1     dev eth0 lladdr aa:bb:cc:dd:ee:ff REACHABLE
# 192.168.1.20    dev eth0 lladdr dd:ee:ff:44:55:66 STALE
```

> [!info] Cache ARP
> Les entrees ARP sont mises en cache pendant quelques minutes. Cela evite d'envoyer une requete broadcast a chaque paquet. Si une entree devient "STALE" (perimee), une nouvelle requete ARP sera envoyee au prochain besoin.

---

## Le modele client-serveur vs peer-to-peer

### Client-Serveur

Le modele le plus courant sur Internet. Un **serveur** centralise fournit des services, et les **clients** s'y connectent pour les consommer :

```
     [Client 1] ──→
     [Client 2] ──→  [SERVEUR]
     [Client 3] ──→
     [Client N] ──→

Exemples : Web (navigateur → serveur HTTP), Email, bases de donnees
```

**Avantages** : gestion centralisee, securite plus facile a controler, donnees coherentes.
**Inconvenients** : point unique de defaillance, cout du serveur, scalabilite limitee.

### Peer-to-Peer (P2P)

Chaque machine est a la fois **client et serveur**. Les ressources sont partagees entre les pairs sans serveur central :

```
     [Pair A] ←──→ [Pair B]
        ↕               ↕
     [Pair C] ←──→ [Pair D]

Exemples : BitTorrent, blockchain, certains jeux en LAN
```

**Avantages** : pas de point unique de defaillance, scalabilite naturelle, pas de cout serveur.
**Inconvenients** : securite plus complexe, coherence des donnees difficile, performances variables.

> [!tip] En pratique
> La grande majorite des applications que vous developperez utilisent le modele client-serveur. Le P2P est utilise dans des cas specifiques : partage de fichiers (BitTorrent), crypto-monnaies (blockchain), communications decentralisees (WebRTC pour la video pair-a-pair).

---

## Outils reseau indispensables

### ping : tester la connectivite

`ping` envoie des paquets **ICMP Echo Request** et attend des **Echo Reply**. C'est le premier reflexe pour verifier si une machine est joignable.

```bash
# Tester la connectivite vers une machine
ping google.com
ping 192.168.1.1
ping -c 4 google.com    # Limiter a 4 paquets (Linux/macOS)

# Resultat typique :
# PING google.com (142.250.179.110): 56 data bytes
# 64 bytes from 142.250.179.110: icmp_seq=0 ttl=118 time=12.3 ms
# 64 bytes from 142.250.179.110: icmp_seq=1 ttl=118 time=11.8 ms
```

> [!info] TTL (Time To Live)
> Le TTL indique le nombre de routeurs que le paquet peut encore traverser. Chaque routeur decremente le TTL de 1. Si le TTL atteint 0, le paquet est detruit (evite les boucles infinies).

### traceroute / tracert : tracer l'itineraire

`traceroute` montre **tous les routeurs** traverses entre votre machine et la destination :

```bash
# Linux/macOS :
traceroute google.com

# Windows :
tracert google.com

# Resultat typique :
#  1  192.168.1.1      1.2 ms    (votre routeur)
#  2  10.0.0.1         5.4 ms    (routeur FAI)
#  3  72.14.215.65     8.1 ms    (backbone)
#  4  142.250.179.110  12.3 ms   (destination)
```

### netstat / ss : voir les connexions actives

```bash
# Lister toutes les connexions TCP actives
netstat -tn           # Linux ancien
ss -tn                # Linux moderne (plus rapide)

# Voir quel processus ecoute sur quel port
netstat -tlnp         # Linux
ss -tlnp              # Linux
netstat -ano          # Windows

# Exemple de sortie :
# Proto  Local Address      Foreign Address    State
# tcp    0.0.0.0:80         0.0.0.0:*          LISTEN
# tcp    192.168.1.10:52847 142.250.179.110:443 ESTABLISHED
```

### ifconfig / ip : configuration reseau

```bash
# Voir la configuration reseau
ifconfig              # ancien (deprecie mais encore utilise)
ip addr show          # moderne (Linux)
ip a                  # raccourci

# Voir la table de routage
ip route show
route -n              # ancien

# Voir les voisins ARP
ip neigh show
arp -a                # ancien
```

### curl : effectuer des requetes HTTP

```bash
# Requete GET simple
curl https://api.github.com

# Voir les en-tetes de reponse
curl -I https://example.com

# Requete POST avec JSON
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Alice", "email": "alice@example.com"}'

# Verbose (voir les details de la connexion, TLS, headers)
curl -v https://example.com

# Suivre les redirections
curl -L https://example.com

# Telecharger un fichier
curl -O https://example.com/fichier.zip
```

### nc (netcat) : le couteau suisse du reseau

```bash
# Scanner un port
nc -zv 192.168.1.1 80

# Ecouter sur un port (serveur simple)
nc -l 8080

# Se connecter a un port (client)
nc 192.168.1.1 8080

# Transferer un fichier
# Sur le recepteur :
nc -l 9000 > fichier_recu.txt
# Sur l'emetteur :
nc 192.168.1.100 9000 < fichier_a_envoyer.txt

# Tester si un port est ouvert
nc -zv google.com 80 443 22
```

### tcpdump : capturer le trafic reseau

```bash
# Capturer tout le trafic sur une interface
sudo tcpdump -i eth0

# Filtrer par hote
sudo tcpdump host 192.168.1.42

# Filtrer par port
sudo tcpdump port 80

# Filtrer par protocole
sudo tcpdump tcp
sudo tcpdump udp

# Sauvegarder dans un fichier (pour Wireshark)
sudo tcpdump -i eth0 -w capture.pcap

# Capturer avec details hexadecimaux
sudo tcpdump -X -i eth0 port 80
```

> [!warning] Permissions
> `tcpdump` necessite les droits **root** (sudo) car il accede directement aux interfaces reseau pour capturer les paquets bruts.

---

## Travaux pratiques

### TP 1 : Verifier votre configuration reseau

```bash
# 1. Afficher votre adresse IP locale
ip addr show        # Linux
ipconfig            # Windows

# 2. Trouver votre passerelle par defaut
ip route show       # Linux
route print         # Windows

# 3. Trouver vos serveurs DNS
cat /etc/resolv.conf         # Linux
ipconfig /all                # Windows

# 4. Verifier votre IP publique
curl ifconfig.me
curl ipinfo.io/ip
```

### TP 2 : Tracer la route vers un serveur

```bash
# 1. Pinger un serveur pour verifier la connectivite
ping -c 4 google.com

# 2. Tracer la route
traceroute google.com

# 3. Resoudre le DNS
dig google.com
nslookup google.com

# 4. Questions a se poser :
# - Combien de sauts (hops) entre vous et le serveur ?
# - Quel est le TTL ?
# - Quelle est la latence (temps de reponse) ?
# - Y a-t-il des sauts avec un temps eleve ? Pourquoi ?
```

---

## Carte Mentale ASCII

```
                         FONDAMENTAUX RESEAUX
                                │
        ┌───────────┬───────────┼───────────┬──────────────┐
        │           │           │           │              │
     Modeles     Adressage   Protocoles   Services      Outils
        │           │           │           │              │
   ┌────┴────┐   ┌──┴──┐    ┌──┴──┐    ┌───┴───┐    ┌────┴────┐
   │         │   │     │    │     │    │       │    │         │
  OSI     TCP/IP IPv4 Ports TCP  UDP  DNS    DHCP  ping    curl
  7 couches      IPv6       │     │   CNAME  DORA  tracert  nc
  4 couches      CIDR       3-way │   MX     NAT   ss     tcpdump
                 Masques    hand. │   NS           netstat  dig
                 Privees    Fiable│   A/AAAA
                            Ordre Rapide
                                  Sans connexion
```

---

## Exercices

### Exercice 1 : Calcul de sous-reseau

Donnee : le reseau `10.42.128.0/20`

1. Quel est le masque de sous-reseau en notation decimale ?
2. Combien d'adresses hotes utilisables ?
3. Quelle est l'adresse de broadcast ?
4. Un hote avec l'IP `10.42.140.50` fait-il partie de ce sous-reseau ?

### Exercice 2 : Identifier les couches OSI

Pour chaque situation, identifiez la couche OSI principale impliquee :
1. Un cable Ethernet est debranche
2. Un paquet est route d'un reseau a un autre
3. Votre navigateur envoie une requete HTTP
4. TCP retransmet un segment perdu
5. Un certificat TLS est verifie
6. Un switch utilise une adresse MAC pour transmettre une trame

### Exercice 3 : Diagnostic reseau

Votre collegue n'arrive pas a acceder a `https://api.example.com`. Decrivez les etapes de diagnostic dans l'ordre, en utilisant les outils reseau vus dans ce cours.

### Exercice 4 : DNS et resolution

1. Utilisez `dig` ou `nslookup` pour trouver l'adresse IP de `github.com`
2. Trouvez les enregistrements MX de `gmail.com`
3. Utilisez `dig +trace` pour suivre la resolution complete de `wikipedia.org`
4. Modifiez votre fichier `/etc/hosts` pour faire pointer `test.local` vers `127.0.0.1`, puis testez avec `ping test.local`

---

## Liens

- [[02 - HTTP et Protocoles Web]] -- Protocoles applicatifs, HTTP, REST, WebSocket
- [[03 - Securite Reseau]] -- Firewalls, SSH, VPN, TLS et securite reseau
- [[02 - Bases Numeriques]] -- Operations binaires necessaires au subnetting
