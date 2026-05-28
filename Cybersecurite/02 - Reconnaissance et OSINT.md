# Reconnaissance et OSINT

La reconnaissance est la première phase d'un pentest et souvent la plus déterminante. Un attaquant qui connaît parfaitement sa cible avant d'attaquer a un avantage considérable. L'OSINT (Open Source Intelligence) désigne la collecte d'informations à partir de sources publiques et légalement accessibles.

> [!warning] Rappel légal
> La reconnaissance passive (OSINT) est légale car elle n'interagit pas directement avec les systèmes de la cible. La reconnaissance active (scans réseau) nécessite une autorisation écrite. Ne jamais scanner sans autorisation, même un scan nmap simple peut déclencher des alertes et constituer un délit.

---

## 1. Passive vs Active Reconnaissance

```
Reconnaissance — Deux grandes catégories

Passive ──── Aucune interaction directe avec la cible
│            Sources : internet public, DNS, WHOIS, réseaux sociaux
│            Légalité : toujours légal (sources publiques)
│            Détection : indétectable par la cible
│            Exemples : Google Dorks, Shodan, theHarvester
│
Active ────── Interaction directe avec les systèmes cibles
             Sources : réponses des serveurs, bannières de services
             Légalité : nécessite autorisation écrite
             Détection : visible dans les logs de la cible
             Exemples : nmap, nikto, ffuf, gobuster
```

---

## 2. Google Dorks

Les Google Dorks (ou Google Hacking) sont des requêtes de recherche avancées utilisant les opérateurs de Google pour trouver des informations sensibles indexées publiquement.

### Opérateurs fondamentaux

| Opérateur | Usage | Exemple |
|-----------|-------|---------|
| `site:` | Limiter à un domaine | `site:example.com` |
| `filetype:` | Type de fichier | `filetype:pdf confidential` |
| `inurl:` | Terme dans l'URL | `inurl:admin login` |
| `intitle:` | Terme dans le titre | `intitle:"index of"` |
| `intext:` | Terme dans le corps | `intext:"password"` |
| `cache:` | Version en cache Google | `cache:example.com` |
| `link:` | Pages qui pointent vers | `link:example.com` |
| `-` | Exclusion | `site:example.com -www` |
| `"` | Phrase exacte | `"mot de passe oublié"` |
| `OR` | Alternative | `filetype:sql OR filetype:db` |

### Dorks utiles en pentest

```bash
# Trouver des pages d'administration
site:example.com inurl:admin
site:example.com inurl:login
site:example.com intitle:"admin panel"

# Fichiers sensibles exposés
site:example.com filetype:pdf "confidentiel"
site:example.com filetype:xls "password"
site:example.com filetype:sql
site:example.com filetype:env

# Répertoires ouverts (directory listing)
intitle:"index of" site:example.com
intitle:"index of" "parent directory"

# Panels de configuration exposés
intitle:"phpMyAdmin" "Welcome to phpMyAdmin"
intitle:"Kibana" site:example.com

# Caméras et IoT exposés
intitle:"webcam 7" inurl:"/viewer/live/index.html"

# Erreurs révélant des infos système
site:example.com "Warning: mysql_fetch_array()"
site:example.com "Fatal error" "on line"

# Documents avec informations sensibles
site:example.com "not for distribution" filetype:pdf
site:example.com filetype:doc "username" "password"
```

> [!tip] Google Hacking Database
> La base de données GHDB sur exploit-db.com répertorie des milliers de dorks classés par catégorie. URL : https://www.exploit-db.com/google-hacking-database

---

## 3. Shodan

Shodan est le "Google des objets connectés". Il indexe les bannières de services exposés sur internet (HTTP, SSH, FTP, RTSP, etc.) et permet de rechercher des appareils selon de nombreux critères.

### Recherches de base

```
Shodan — Opérateurs courants

hostname:example.com      Devices liés à un hostname
org:"Example Corp"        Devices d'une organisation
net:192.168.0.0/24        Plage d'adresses IP
port:22                   Devices avec SSH exposé
product:nginx             Version de produit
os:"Windows Server 2019"  Système d'exploitation
country:FR                Devices en France
city:Paris                Devices dans une ville
ssl:"example.com"         Certificat SSL contenant
before/after:2024-01-01   Filtrage par date de découverte
```

### Exemples de recherches en pentest

```bash
# Via l'API Shodan (nécessite un compte)
pip install shodan

# Script Python d'interrogation Shodan
python3 << 'EOF'
import shodan

api = shodan.Shodan("VOTRE_API_KEY")

# Rechercher les serveurs web d'une organisation
results = api.search("org:'Example Corporation' port:80,443")
print(f"Résultats : {results['total']}")

for result in results['matches']:
    print(f"IP: {result['ip_str']}")
    print(f"Port: {result['port']}")
    print(f"Banner: {result['data'][:100]}")
    print("---")
EOF

# Via l'outil CLI shodan
shodan search "hostname:example.com" --fields ip_str,port,product
shodan host 192.168.1.100         # Infos sur une IP spécifique
shodan count "apache country:FR"  # Compter les résultats
```

### Filtres avancés Shodan

```
Vulnérabilités connues :
vuln:CVE-2021-44228           Devices vulnérables à Log4Shell
vuln:CVE-2019-0708            BlueKeep (RDP)

Services critiques exposés :
port:3389 os:"Windows"        RDP Windows
port:5900 authentication:     VNC sans auth
port:27017 MongoDB            MongoDB sans auth
port:6379 Redis               Redis sans auth
port:9200 elasticsearch       Elasticsearch exposé

Industriel (SCADA/ICS) :
port:502 Modbus               Protocole industriel Modbus
port:102 s7comm               Siemens S7 PLC
```

---

## 4. theHarvester

Outil OSINT qui agrège des informations depuis de multiples sources publiques (moteurs de recherche, DNS, PGP, etc.).

```bash
# Syntaxe de base
theHarvester -d example.com -b google

# Options principales
# -d : domaine cible
# -b : source (google, bing, yahoo, dnsdumpster, crtsh, linkedin, github...)
# -l : limite de résultats
# -f : fichier de sortie (HTML/XML)

# Recherche sur plusieurs sources
theHarvester -d example.com -b google,bing,linkedin,crtsh -l 500

# Sources disponibles
theHarvester -h | grep "data sources"
# google, bing, yahoo, duckduckgo, baidu
# linkedin, twitter, github
# crtsh (certificats SSL → sous-domaines)
# dnsdumpster, bevigil, zoomeye
# shodan (avec API key)

# Export des résultats
theHarvester -d example.com -b all -f rapport_harvest.html
```

**Ce que theHarvester récupère :**
- Adresses email (format prénom.nom@example.com → déduit le schéma d'email)
- Sous-domaines (mail.example.com, dev.example.com, vpn.example.com...)
- Adresses IP associées
- Noms de personnes (depuis LinkedIn)

---

## 5. WHOIS et DNS

### WHOIS — Informations sur les domaines

```bash
# Informations d'enregistrement du domaine
whois example.com

# Informations typiques : registrar, dates, nameservers, contacts (souvent cachés par privacy shield)
# Points d'intérêt : nameservers, dates de création/expiration

# whois sur une IP (informations ASN et organisation)
whois 93.184.216.34
```

### dig — Résolution DNS avancée

```bash
# Enregistrement A (IPv4)
dig example.com A

# Enregistrement AAAA (IPv6)
dig example.com AAAA

# Enregistrement MX (serveurs mail)
dig example.com MX
# Révèle souvent le provider mail (Google, Microsoft, Proofpoint...)

# Enregistrement NS (nameservers)
dig example.com NS

# Enregistrement TXT (SPF, DKIM, vérifications)
dig example.com TXT
# SPF révèle quels services envoient des mails (Mailchimp, AWS SES, Sendgrid...)

# Enregistrement CNAME (alias)
dig www.example.com CNAME

# SOA (informations sur la zone DNS)
dig example.com SOA

# Tous les enregistrements
dig example.com ANY

# Zone transfer (si mal configuré, révèle TOUS les sous-domaines)
dig @ns1.example.com example.com AXFR

# Utiliser un DNS spécifique
dig @8.8.8.8 example.com A      # Google DNS
dig @1.1.1.1 example.com A      # Cloudflare DNS

# Reverse DNS (IP → hostname)
dig -x 93.184.216.34
```

> [!tip] Zone Transfer
> Un Zone Transfer (AXFR) mal configuré révèle **tous** les sous-domaines d'un domaine. C'est une misconfiguration classique qui donne instantanément une cartographie complète de l'infrastructure.

---

## 6. Énumération de sous-domaines

### amass — Le standard actuel

```bash
# Installation
sudo apt install amass

# Enumération passive (OSINT uniquement)
amass enum -passive -d example.com

# Enumération active (résolution DNS)
amass enum -active -d example.com

# Mode agressif avec bruteforce
amass enum -brute -d example.com -w /usr/share/wordlists/amass/subdomains-top1mil-20000.txt

# Export
amass enum -d example.com -o sous-domaines.txt

# Visualisation du graphe
amass viz -d3 -d example.com -o graph.html
```

### subfinder — Rapide et passif

```bash
# Installation
go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest

# Utilisation
subfinder -d example.com
subfinder -d example.com -o subdomains.txt

# Avec toutes les sources (nécessite des API keys)
subfinder -d example.com -all

# Pipeline avec httpx pour trouver les sous-domaines actifs
subfinder -d example.com -silent | httpx -silent -status-code
```

### Wordlists pour bruteforce DNS

```bash
# Wordlists populaires
ls /usr/share/wordlists/
# SecLists (à installer)
sudo apt install seclists
ls /usr/share/seclists/Discovery/DNS/

# Bruteforce DNS avec gobuster
gobuster dns -d example.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# Bruteforce DNS avec ffuf
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
     -u http://FUZZ.example.com \
     -mc 200,301,302
```

---

## 7. Wayback Machine et endpoints historiques

```bash
# Wayback Machine — URLs historiques
# Site web : https://web.archive.org

# Via l'API Wayback Machine
curl "http://web.archive.org/cdx/search/cdx?url=*.example.com/*&output=text&fl=original&collapse=urlkey"

# Outil gau (Get All URLs) — toutes les URLs d'un domaine
go install github.com/lc/gau/v2/cmd/gau@latest
gau example.com | tee urls-historiques.txt

# Filtrer les endpoints intéressants
cat urls-historiques.txt | grep -E "\.(php|asp|aspx|do|action|jsp|cgi|env|config|backup|bak|sql|log)"

# waybackurls — similaire
go install github.com/tomnomnom/waybackurls@latest
waybackurls example.com | tee wayback-output.txt
```

**Ce qu'on cherche dans les URLs historiques :**
- Anciens endpoints d'API supprimés mais potentiellement toujours actifs
- Paramètres de requête révélant la structure de l'application
- Fichiers de configuration/backup exposés dans le passé
- Anciens sous-domaines

---

## 8. GitHub Dorking

Les dépôts GitHub publics contiennent régulièrement des secrets accidentellement commités (clés API, mots de passe, tokens, fichiers .env).

### Dorks GitHub

```
Recherche sur github.com — barre de recherche

# Clés et secrets
"example.com" password
"example.com" secret
"example.com" api_key
"example.com" token
org:ExampleOrg filename:.env
org:ExampleOrg DB_PASSWORD
org:ExampleOrg "BEGIN RSA PRIVATE KEY"

# Fichiers de configuration
filename:wp-config.php "DB_PASSWORD"
filename:.htpasswd
filename:config.yml password
filename:database.yml password
filename:settings.py SECRET_KEY
filename:.bash_history
filename:id_rsa
filename:id_rsa.pub
filename:known_hosts

# Cloud credentials
"AKIA" aws_access_key       # AWS Access Key ID
"AIza" google_api_key       # Google API Key
```

### truffleHog — Scan automatisé de secrets

```bash
# Installation
pip install trufflehog3

# Scanner un repo
trufflehog git https://github.com/example-org/example-repo

# Scanner une organisation entière
trufflehog github --org=ExampleOrg

# Scanner localement
trufflehog filesystem /chemin/vers/repo
```

### gitrob — Cartographie des secrets dans une org

```bash
# Analyser les repos publics d'une organisation GitHub
gitrob analyze ExampleOrg
```

---

## 9. LinkedIn et ingénierie sociale

> [!info] Contexte légal
> La collecte d'informations publiques sur LinkedIn est légale. L'ingénierie sociale (manipulation de personnes) dans le cadre d'un pentest nécessite une autorisation explicite dans le scope.

### Ce qu'on cherche sur LinkedIn

```
Objectifs OSINT LinkedIn

1. Organigramme ──── Qui travaille où, quels départements existent
2. Stack technique ─ Les profils mentionnent souvent les technos utilisées
3. Offres d'emploi ─ "Cherche dev Java Spring + Oracle DB" → révèle la stack
4. Emails ──────────  Prénom.Nom@company.com → valider le format d'email
5. Anciens employés ─ Peuvent avoir encore des accès, informations internes
```

### Outils de collecte LinkedIn

```bash
# linkedin2username — génère des listes d'usernames depuis LinkedIn
python3 linkedin2username.py -u votre@email.com -c example-company-name

# Hunter.io — trouver des emails professionnels (version web)
# https://hunter.io/domain-search

# PhoneBook.cz — OSINT emails/domaines
# https://phonebook.cz
```

---

## 10. Recon-ng Framework

Framework de reconnaissance modulaire similaire à Metasploit mais pour l'OSINT.

```bash
# Lancer recon-ng
recon-ng

# Interface interactive
[recon-ng][default] > help
[recon-ng][default] > marketplace install all
[recon-ng][default] > workspaces create example_pentest

# Ajouter des domaines à analyser
[recon-ng][example_pentest] > db insert domains
domain > example.com

# Utiliser des modules
[recon-ng][example_pentest] > modules load recon/domains-hosts/hackertarget
[recon-ng][example_pentest][hackertarget] > run

# Voir les résultats
[recon-ng][example_pentest] > show hosts
[recon-ng][example_pentest] > show contacts

# Rapport HTML
[recon-ng][example_pentest] > modules load reporting/html
[recon-ng][example_pentest][html] > set FILENAME /tmp/rapport.html
[recon-ng][example_pentest][html] > run
```

---

## 11. Scanning actif avec nmap

> [!warning] Autorisation requise
> Le scanning actif (nmap) génère du trafic vers la cible et est visible dans ses logs. Ne jamais scanner sans autorisation écrite.

### Scans essentiels

```bash
# Découverte de hosts actifs (ping scan)
nmap -sn 192.168.1.0/24

# Scan rapide des ports courants
nmap -F 192.168.1.100

# Scan complet (tous les ports) + versions + scripts
nmap -sV -sC -p- --min-rate 5000 192.168.1.100 -oN scan_complet.txt

# Options détaillées
nmap \
  -sV \           # Version detection
  -sC \           # Scripts par défaut (NSE)
  -O \            # OS detection
  -A \            # All (sV + sC + O + traceroute)
  -p- \           # Tous les 65535 ports
  --min-rate 5000 \ # Vitesse minimale (agressif)
  -T4 \           # Template de timing (0=paranoid, 5=insane)
  192.168.1.100 \
  -oN scan.txt \  # Output normal
  -oX scan.xml \  # Output XML
  -oG scan.gnmap  # Output greppable

# Scan UDP (plus lent, important pour DNS, SNMP, DHCP)
nmap -sU -p 53,67,68,69,123,161,162,500 192.168.1.100

# Scan discret (SYN scan par défaut avec root)
sudo nmap -sS 192.168.1.100

# Scan avec scripts NSE spécifiques
nmap --script=http-title,http-auth-finder 192.168.1.100
nmap --script=smb-vuln-* 192.168.1.100
nmap --script=ssl-cert,ssl-enum-ciphers 192.168.1.100 -p 443
```

### Templates de timing nmap

| Template | Nom | Usage |
|----------|-----|-------|
| `-T0` | Paranoid | IDS evasion, très lent |
| `-T1` | Sneaky | IDS evasion |
| `-T2` | Polite | Peu intrusif |
| `-T3` | Normal | Par défaut |
| `-T4` | Aggressive | Réseau rapide |
| `-T5` | Insane | Très rapide, peut manquer des ports |

---

## 12. Nikto et fuzzing web

### Nikto — Scanner de vulnérabilités web

```bash
# Scan de base
nikto -h http://192.168.1.100

# Avec SSL
nikto -h https://example.com -ssl

# Options utiles
nikto -h http://192.168.1.100 \
  -port 8080 \          # Port non standard
  -Tuning 1234 \        # Types de checks (1=files, 2=misconfig, 3=info, 4=injection)
  -output nikto.html \  # Export HTML
  -Format htm

# Ce que Nikto cherche :
# - Fichiers par défaut dangereux (/phpmyadmin, /admin, /.git)
# - Headers de sécurité manquants
# - Versions vulnérables
# - Configuration incorrecte
```

### ffuf — Fuzzer web rapide

```bash
# Fuzzing de répertoires
ffuf -w /usr/share/wordlists/dirb/common.txt -u http://192.168.1.100/FUZZ

# Fuzzing d'extensions
ffuf -w /usr/share/wordlists/dirb/common.txt \
     -u http://192.168.1.100/FUZZ \
     -e .php,.html,.txt,.bak,.old,.zip

# Fuzzing de paramètres GET
ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt \
     -u http://example.com/search?FUZZ=test \
     -mc 200,301

# Fuzzing de sous-domaines (vhost)
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
     -H "Host: FUZZ.example.com" \
     -u http://192.168.1.100 \
     -mc 200,301,302

# Options de filtrage (éviter le bruit)
ffuf ... -fc 404         # Filtrer les codes 404
ffuf ... -fs 1234        # Filtrer les réponses de taille 1234 bytes
ffuf ... -fw 10          # Filtrer par nombre de mots
ffuf ... -fl 15          # Filtrer par nombre de lignes
```

### gobuster — Alternative à ffuf

```bash
# Mode dir (répertoires)
gobuster dir -u http://192.168.1.100 \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -x php,html,txt \
  -t 50 \          # Threads
  -o gobuster.txt  # Output

# Mode dns (sous-domaines)
gobuster dns -d example.com \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
  -t 50

# Mode vhost (virtual hosts)
gobuster vhost -u http://example.com \
  -w /usr/share/seclists/Discovery/Web-Content/big.txt \
  --append-domain
```

---

## Exercices pratiques (CTF style)

> [!info] Environnement requis
> Compte TryHackMe (gratuit), Kali Linux, connexion internet.

### Exercice 1 — OSINT sur un domaine fictif (legal)

```bash
# Cible d'entraînement : megacorpone.com (domaine de formation Offensive Security)
# 1. Collecter les informations WHOIS
whois megacorpone.com

# 2. Énumérer les enregistrements DNS
dig megacorpone.com ANY
dig megacorpone.com MX
dig megacorpone.com TXT

# 3. Tenter un zone transfer
dig @ns1.megacorpone.com megacorpone.com AXFR
dig @ns2.megacorpone.com megacorpone.com AXFR

# 4. Collecter des emails
theHarvester -d megacorpone.com -b google,bing -l 200

# Questions :
# - Combien de sous-domaines avez-vous trouvé ?
# - Le zone transfer a-t-il fonctionné ? Pourquoi ?
# - Quel est le format des emails ?
```

### Exercice 2 — Google Dorks pratique

```bash
# 1. Trouver des fichiers PDF potentiellement sensibles sur un site gouvernemental public
# (adapter le site à un domaine de test)
site:ac-paris.fr filetype:pdf "confidentiel"

# 2. Trouver des pages d'index ouvertes
intitle:"index of" site:example-target.com

# 3. Chercher des erreurs SQL révélatrices
site:testphp.vulnweb.com "Warning: mysql"

# Documenter chaque dork :
# - URL complète de la recherche
# - Ce que vous avez trouvé
# - Impact potentiel
```

### Exercice 3 — Nmap et analyse d'un lab

```bash
# Room TryHackMe : "Nmap" (gratuit)
# https://tryhackme.com/room/furthernmap

# Après connexion VPN THM, scanner la machine cible
sudo nmap -sV -sC -p- --min-rate 3000 <IP_CIBLE> -oN scan_exercice.txt

# Analyser et répondre :
# 1. Combien de ports ouverts ?
# 2. Quel OS probable ?
# 3. Quels services vulnérables identifiés ?
# 4. Quel vecteur d'attaque semble le plus prometteur ?
```

### Exercice 4 — Script Python d'automatisation OSINT

```python
#!/usr/bin/env python3
"""
Script OSINT basique — Énumération DNS automatisée
Usage : python3 osint_dns.py example.com
"""

import subprocess
import sys

def run_dig(domaine, record_type):
    """Exécuter dig et retourner la sortie."""
    result = subprocess.run(
        ["dig", domaine, record_type, "+short"],
        capture_output=True,
        text=True
    )
    return result.stdout.strip()

def enumerate_dns(domaine):
    print(f"\n=== Énumération DNS : {domaine} ===\n")
    
    types = ["A", "AAAA", "MX", "NS", "TXT", "CNAME", "SOA"]
    
    for record_type in types:
        resultat = run_dig(domaine, record_type)
        if resultat:
            print(f"[{record_type}]")
            for ligne in resultat.split('\n'):
                print(f"  {ligne}")
            print()

def zone_transfer(domaine):
    """Tenter un zone transfer via tous les NS."""
    print(f"\n=== Zone Transfer : {domaine} ===")
    
    ns_result = subprocess.run(
        ["dig", domaine, "NS", "+short"],
        capture_output=True, text=True
    )
    
    for ns in ns_result.stdout.strip().split('\n'):
        if ns:
            print(f"\nTentative via {ns}...")
            axfr = subprocess.run(
                ["dig", f"@{ns}", domaine, "AXFR"],
                capture_output=True, text=True
            )
            if "Transfer failed" not in axfr.stdout:
                print(f"[!] Zone transfer RÉUSSI via {ns} !")
                print(axfr.stdout)
            else:
                print(f"  Transfer refusé.")

if __name__ == "__main__":
    if len(sys.argv) != 2:
        print(f"Usage: {sys.argv[0]} <domaine>")
        sys.exit(1)
    
    domaine = sys.argv[1]
    enumerate_dns(domaine)
    zone_transfer(domaine)
```

> [!tip] Liens vers les autres notes
> - [[01 - Fondamentaux Hacking Ethique]] — méthodologies et cadre légal
> - [[03 - Exploitation Web et Systeme]] — exploitation des vulnérabilités découvertes
> - [[02 - Securite Web OWASP]] — Top 10 OWASP et contre-mesures
