# Fondamentaux Hacking Ethique

Le hacking éthique est la pratique consistant à tester la sécurité d'un système informatique avec l'autorisation explicite de son propriétaire, dans le but d'identifier et de corriger les vulnérabilités avant qu'un acteur malveillant ne les exploite. C'est le fondement de toute la formation Holberton Cybersécurité.

> [!warning] Cadre légal obligatoire
> Toutes les techniques présentées dans ce cours sont réservées à des environnements autorisés : CTF, labos isolés, machines virtuelles, plateformes comme Hack The Box ou TryHackMe. Toute intrusion non autorisée est un délit pénal en France (article 323-1 du Code pénal). **Toujours obtenir une autorisation écrite avant tout test.**

---

## 1. Définitions fondamentales

### Ethical Hacking

Un **hacker éthique** (ou pentesteur, ou red teamer) est un professionnel de la sécurité mandaté pour attaquer un système avec les mêmes techniques qu'un attaquant malveillant, mais dans un cadre légal et contractuel. L'objectif est d'identifier les failles **avant** qu'elles ne soient exploitées.

```
Hackers — Taxonomy classique

White Hat ── Hacker éthique, travaille avec autorisation
│            Exemples : pentesteurs, chercheurs en sécurité, bug bounty hunters
│
Gray Hat ─── Hacker qui teste sans autorisation mais sans intention malveillante
│            Souvent illégal même sans dommages causés
│
Black Hat ── Hacker malveillant, motivations criminelles ou idéologiques
             Exemples : cybercriminels, groupes APT, script kiddies
```

### Pentest vs Audit de sécurité

| Critère | Pentest | Audit de sécurité |
|---------|---------|-------------------|
| **Approche** | Offensif, exploitation réelle | Revue de configuration et conformité |
| **Objectif** | Démontrer l'impact d'une faille | Vérifier la conformité à un standard |
| **Résultat** | Rapport avec preuves d'exploitation | Rapport de conformité ISO/NIST |
| **Durée** | 1 à 4 semaines typiquement | Variable |
| **Standards** | PTES, OWASP | ISO 27001, SOC2, NIST |

### Bug Bounty

Programme lancé par une organisation invitant des chercheurs indépendants à identifier et signaler des vulnérabilités en échange d'une récompense financière. Les plateformes majeures sont **HackerOne**, **Bugcrowd** et **Intigriti**. La récompense (bounty) varie de quelques centaines à plusieurs centaines de milliers de dollars selon la criticité.

```
Exemple de barème bug bounty (approximatif) :

Critique (RCE, SQLi, Auth bypass)  ──── $5,000 – $100,000+
Élevé (SSRF, XXE, IDOR sensible)   ──── $1,000 – $10,000
Moyen (XSS stocké, CSRF sensible)  ──── $200 – $2,000
Faible (XSS réfléchi, info-leak)   ──── $50 – $500
Informatif (non exploitable)        ──── $0 ou rejet
```

### Red Team / Blue Team / Purple Team

```
Red Team ── Attaquants (pentesteurs, red teamers)
│           Simulent des attaques réalistes (APT, ransomware, phishing)
│           Objectif : tester les défenses dans leur ensemble
│
Blue Team ─ Défenseurs (SOC, CERT, analyste sécurité)
│           Détectent, analysent et répondent aux incidents
│           Outils : SIEM, EDR, firewall, IDS/IPS
│
Purple Team Collaboration Red + Blue en temps réel
            Amélioration continue des détections
            Le Red Team partage ses TTPs, le Blue Team améliore ses règles
```

---

## 2. Cadre légal

> [!warning] Loi française — Article 323-1 du Code pénal
> "Le fait d'accéder ou de se maintenir, frauduleusement, dans tout ou partie d'un système de traitement automatisé de données est puni de deux ans d'emprisonnement et de 60 000 euros d'amende."
> L'autorisation écrite du propriétaire est la seule protection légale du pentesteur.

### Loi LCEN (Loi pour la Confiance dans l'Économie Numérique)

La LCEN de 2004 transpose en droit français les directives européennes sur la cybercriminalité. Elle encadre :
- L'accès non autorisé aux systèmes informatiques (délit)
- La détention d'outils de hacking sans justification légitime (délit)
- La mise en place de CSIRT et obligations de notification

### RGPD et données personnelles

Lors d'un pentest, des données personnelles peuvent être accessibles. Le pentesteur a des obligations RGPD :
- Ne jamais exfiltrer de vraies données personnelles sans accord explicite
- Signaler immédiatement toute exposition de données personnelles
- Utiliser des données anonymisées dans les preuves de concept (PoC)

### Règles d'engagement (Rules of Engagement — RoE)

Document contractuel définissant les limites du test. Doit préciser :

```
Règles d'Engagement — Points obligatoires

1. Scope ─────── Quels systèmes sont testables (IPs, domaines, URLs)
2. Exclusions ─── Ce qui ne doit PAS être touché (prod critique, serveurs tiers)
3. Horaires ───── Fenêtres de test autorisées (ex: hors heures de bureau)
4. Techniques ─── Attaques autorisées et interdites (ex: DoS interdit)
5. Contacts ───── Qui appeler en cas d'incident ou découverte critique
6. Données ─────── Comment traiter les données sensibles découvertes
7. Rapport ─────── Format, destinataires, délais de livraison
```

---

## 3. Méthodologies

### PTES (Penetration Testing Execution Standard)

Standard de référence pour la conduite de pentests professionnels.

```
PTES — 7 phases

1. Pre-engagement Interactions
   └── Contrat, RoE, définition du scope

2. Intelligence Gathering (Reconnaissance)
   └── OSINT, DNS, réseaux sociaux, scan passif

3. Threat Modeling
   └── Identifier les actifs critiques et les vecteurs d'attaque probables

4. Vulnerability Analysis
   └── Scanner les failles, valider manuellement

5. Exploitation
   └── Exploiter les vulnérabilités confirmées

6. Post-Exploitation
   └── Persistence, lateral movement, data exfiltration

7. Reporting
   └── Documentation, recommandations, présentation client
```

### OWASP Testing Guide (OTG)

Guide méthodologique pour les tests de sécurité des applications web. Couvre plus de 100 types de tests organisés en catégories (OTG-INFO, OTG-CONF, OTG-SESS, etc.). Version actuelle : OTG v4.2.

### MITRE ATT&CK Framework

Base de connaissances des techniques et tactiques utilisées par les attaquants réels, documentées depuis des incidents réels.

```
MITRE ATT&CK — 14 tactiques (Enterprise)

TA0043  Reconnaissance
TA0042  Resource Development
TA0001  Initial Access
TA0002  Execution
TA0003  Persistence
TA0004  Privilege Escalation
TA0005  Defense Evasion
TA0006  Credential Access
TA0007  Discovery
TA0008  Lateral Movement
TA0009  Collection
TA0011  Command and Control (C2)
TA0010  Exfiltration
TA0040  Impact
```

---

## 4. Phases d'un pentest

### Phase 1 — Reconnaissance

Collecte d'informations sur la cible **sans interagir directement** avec elle (passive) ou en la sondant (active).

```bash
# Exemples outils reconnaissance passive
whois example.com
dig example.com ANY
theHarvester -d example.com -b google
```

### Phase 2 — Scanning et Enumération

Identification des services actifs, versions, systèmes d'exploitation.

```bash
# Nmap — scan complet
nmap -sV -sC -O -p- --min-rate 5000 192.168.1.100

# Alternatives rapides
masscan -p1-65535 192.168.1.0/24 --rate=10000
rustscan -a 192.168.1.100 --ulimit 5000
```

### Phase 3 — Exploitation

Exploitation des vulnérabilités identifiées pour obtenir un accès.

```bash
# Via Metasploit
msfconsole
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 192.168.1.100
set LHOST 192.168.1.50
run
```

### Phase 4 — Post-Exploitation

Actions après l'obtention d'un accès : escalade de privilèges, persistence, mouvement latéral, exfiltration.

```bash
# Depuis Meterpreter
getsystem          # escalade de privilèges
hashdump           # dump des hashes NTLM
run post/windows/gather/credentials/credential_collector
```

### Phase 5 — Reporting

Documentation de toutes les découvertes avec preuves, impact et recommandations.

---

## 5. Types de pentest

| Type | Informations reçues | Usage typique |
|------|--------------------|-|
| **Black Box** | Aucune — comme un attaquant externe | Test réaliste, attaque externe |
| **Grey Box** | Partielles — credentials standard, schéma réseau | Test plus efficace, approche insider |
| **White Box** | Complètes — code source, configs, schémas | Audit approfondi, évaluation complète |

> [!tip] En pratique
> Le Grey Box est le plus courant en entreprise : il est plus représentatif qu'un Black Box (un vrai attaquant fait de la reconnaissance étendue) tout en étant plus efficient qu'un Black Box pur.

---

## 6. Environnement de travail

### Kali Linux

Distribution Linux orientée sécurité, maintenue par Offensive Security. Inclut 600+ outils pré-installés.

```bash
# Installation en VM (recommandé pour l'isolation)
# Image disponible sur https://www.kali.org/get-kali/

# Mise à jour
sudo apt update && sudo apt full-upgrade -y

# Outils manquants
sudo apt install -y gobuster ffuf feroxbuster dirsearch

# Installer les métapaquets Kali
sudo apt install -y kali-tools-web kali-tools-exploitation
```

### Parrot OS Security

Alternative à Kali, plus légère, basée sur Debian. Recommandée pour les machines peu puissantes.

### VMs isolées — bonne pratique

```
Architecture recommandée pour le lab

Host OS (Windows/macOS)
│
├── VM Attaquant ── Kali Linux (NAT ou Host-only)
│   └── Outils d'attaque, Burp Suite, Metasploit
│
├── VM Cible ─────── Windows Server 2019 / Ubuntu vulnérable
│   └── Réseau isolé, pas d'accès internet direct
│
└── VM CTF ──────── Réseau VPN vers HTB/THM
    └── OpenVPN vers la plateforme
```

```bash
# Connexion VPN HTB/THM
sudo openvpn ~/Downloads/lab_username.ovpn

# Vérifier la connexion
ip a show tun0
ping 10.10.10.1  # gateway HTB
```

---

## 7. Rapport de pentest

### Structure d'un rapport professionnel

```
Rapport de Pentest — Structure standard

1. Page de garde
   ├── Titre, date, version
   ├── Client et prestataire
   └── Classificaton (CONFIDENTIEL)

2. Résumé exécutif (1-2 pages)
   ├── Objectifs du test
   ├── Résumé des découvertes (graphique)
   └── Recommandations prioritaires (top 3)

3. Périmètre et méthodologie
   ├── Scope testé
   ├── Dates et conditions du test
   └── Méthodologie utilisée (PTES, OWASP)

4. Findings détaillés (pour chaque vulnérabilité)
   ├── Titre et identifiant
   ├── Sévérité (CVSS score)
   ├── Description technique
   ├── Preuve (screenshot, output)
   ├── Impact business
   └── Recommandation de correction

5. Conclusion et roadmap de remédiation

Annexes
   ├── Sorties d'outils brutes
   └── Glossaire
```

### CVSS — Common Vulnerability Scoring System

Système standardisé de score de criticité (0.0 à 10.0).

| Score | Sévérité | Couleur |
|-------|----------|---------|
| 0.0 | None | Gris |
| 0.1 – 3.9 | Low | Vert |
| 4.0 – 6.9 | Medium | Jaune |
| 7.0 – 8.9 | High | Orange |
| 9.0 – 10.0 | Critical | Rouge |

```
CVSS v3.1 — Métriques de base

Attack Vector (AV)      : N(etwork) / A(djacent) / L(ocal) / P(hysical)
Attack Complexity (AC)  : L(ow) / H(igh)
Privileges Required (PR): N(one) / L(ow) / H(igh)
User Interaction (UI)   : N(one) / R(equired)
Scope (S)               : U(nchanged) / C(hanged)
Confidentiality (C)     : N(one) / L(ow) / H(igh)
Integrity (I)           : N(one) / L(ow) / H(igh)
Availability (A)        : N(one) / L(ow) / H(igh)

Exemple — Log4Shell (CVE-2021-44228)
AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H → CVSS 10.0 CRITICAL
```

---

## 8. Certifications du domaine

| Certification | Organisme | Niveau | Particularité |
|--------------|-----------|--------|---------------|
| **eJPT** | eLearnSecurity / INE | Débutant | 100% pratique, lab en ligne |
| **CEH** | EC-Council | Intermédiaire | Reconnu corporate, QCM + pratique |
| **OSCP** | Offensive Security | Intermédiaire | Examen 24h, 100% pratique, référence industrie |
| **OSEP** | Offensive Security | Avancé | Evasion, Active Directory avancé |
| **CRTE** | Altered Security | Avancé | Active Directory Red Team |
| **CPTS** | HackTheBox | Intermédiaire | Nouveau, très pratique, très complet |
| **PNPT** | TCM Security | Intermédiaire | Rapport inclus, très pratique |

> [!tip] Parcours recommandé Holberton
> eJPT → PNPT ou CPTS → OSCP. L'OSCP reste la référence que les recruteurs reconnaissent immédiatement.

---

## 9. Introduction aux CTF

### Qu'est-ce qu'un CTF ?

Un **Capture The Flag** est une compétition de cybersécurité où les participants doivent trouver des chaînes cachées (flags) dans des systèmes ou fichiers délibérément vulnérables. Format typique du flag : `HTB{s0m3_s3cr3t_str1ng}` ou `flag{...}`.

### Plateformes

| Plateforme | Type | Niveau | Spécialité |
|-----------|------|--------|------------|
| **Hack The Box (HTB)** | Machines + Labs | Intermédiaire–Expert | Linux/Windows pentest |
| **TryHackMe (THM)** | Guidé + Rooms | Débutant–Intermédiaire | Apprentissage structuré |
| **PicoCTF** | Compétition annuelle | Débutant–Intermédiaire | Toutes catégories |
| **Root-Me** | Challenges | Débutant–Expert | Très large catalogue |
| **CTFtime.org** | Agenda CTF | Variable | Calendrier des compétitions |
| **PortSwigger Web Academy** | Labs web | Débutant–Expert | Web uniquement, gratuit |

### Workflow CTF — Machine HTB

```bash
# 1. Connexion VPN
sudo openvpn ~/htb.ovpn

# 2. Reconnaissance initiale
export IP=10.10.10.100
nmap -sV -sC -p- --min-rate 5000 $IP -oN scan_initial.txt

# 3. Enumération web (si port 80/443)
gobuster dir -u http://$IP -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html

# 4. Exploitation → accès initial
# (dépend de la vulnérabilité trouvée)

# 5. Récupérer les flags
cat /home/user/user.txt    # user flag
cat /root/root.txt          # root flag
```

---

## 10. Outils essentiels — vue d'ensemble

```
Kali Linux — Catégories d'outils

Information Gathering
├── nmap, masscan, rustscan          (scan réseau)
├── theHarvester, maltego            (OSINT)
├── amass, subfinder, assetfinder    (sous-domaines)
└── shodan, censys                   (moteurs de recherche IoT/IP)

Web Application Analysis
├── burpsuite                         (proxy intercept)
├── nikto, nuclei                     (scanners web)
├── sqlmap                            (SQLi automatisé)
├── ffuf, gobuster, feroxbuster       (fuzzing)
└── wfuzz                             (fuzzing avancé)

Exploitation
├── metasploit, msfvenom              (framework exploit + payloads)
├── searchsploit                      (base exploit-db locale)
└── pwntools                          (pwn/binary exploitation)

Password Attacks
├── hashcat, john                     (cracking hashes)
├── hydra, medusa                     (brute force services)
└── crunch, cewl                      (génération wordlists)

Post-Exploitation
├── mimikatz, impacket                (Windows/AD)
├── linpeas, winpeas                  (escalade de privilèges)
├── bloodhound, sharphound            (cartographie AD)
└── empire, covenant                  (C2 frameworks)

Forensics / Blue Team
├── volatility                        (analyse mémoire)
├── autopsy, sleuth kit               (analyse disque)
├── wireshark, tcpdump                (analyse réseau)
└── yara, suricata                    (détection malware)
```

---

## Exercices pratiques (CTF style)

> [!info] Environnement requis
> Kali Linux ou Parrot OS, connexion VPN vers TryHackMe ou HTB, terminal et navigateur.

### Exercice 1 — Setup de l'environnement

```bash
# Étape 1 : Installer Kali en VM (VirtualBox ou VMware)
# Étape 2 : Créer un compte TryHackMe (gratuit)
# Étape 3 : Télécharger le fichier VPN depuis le profil THM
sudo openvpn ~/Downloads/tryhackme.ovpn &

# Étape 4 : Rejoindre la room "Pre-Security" sur THM
# URL : https://tryhackme.com/path/outline/presecurity
```

### Exercice 2 — Premier scan nmap

```bash
# Machine cible : votre propre VM Kali ou une cible THM
# Scanner les services
nmap -sV -sC 10.10.X.X -oN premier_scan.txt

# Analyser la sortie :
# - Quels ports sont ouverts ?
# - Quelles versions de services tournent ?
# - Y a-t-il des scripts NSE qui ont révélé des infos ?
cat premier_scan.txt
```

### Exercice 3 — Room TryHackMe recommandées

```
Parcours débutant recommandé sur TryHackMe :

1. "Introduction to Cybersecurity" (gratuit)
2. "Pre-Security" path (gratuit)
3. "Jr Penetration Tester" path (abonnement)
   ├── Linux Fundamentals 1/2/3
   ├── Network Services
   ├── Web Fundamentals
   └── Metasploit
```

### Exercice 4 — Premier rapport de vulnérabilité

Après avoir terminé une room THM ou HTB, rédiger un rapport de vulnérabilité avec :
- **Titre** : "Rapport de test — [Nom de la machine]"
- **Sévérité CVSS** : calculer manuellement via https://www.first.org/cvss/calculator/3.1
- **Description** : expliquer la faille en 3 phrases
- **Preuve** : coller le screenshot ou output terminal
- **Recommandation** : comment corriger en 2-3 phrases

> [!tip] Liens vers les autres notes
> - [[02 - Reconnaissance et OSINT]] — techniques de collecte d'informations
> - [[03 - Exploitation Web et Systeme]] — SQLi, XSS, Metasploit
> - [[04 - Active Directory et Post-Exploitation]] — AD, mimikatz, BloodHound
> - [[05 - CTF Methodologie et Red Team]] — méthodes CTF et Red Team
