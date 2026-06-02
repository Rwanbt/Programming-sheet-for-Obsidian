# Reverse Engineering et Analyse Binaire

Le reverse engineering binaire est l'art de comprendre le fonctionnement interne d'un programme à partir de son code compilé, sans accès au code source. C'est une compétence fondamentale en cybersécurité offensive et défensive — elle permet d'analyser des malwares, de retrouver des vulnérabilités dans des binaires fermés et de résoudre des challenges CTF. Ce cours couvre les formats binaires PE et ELF, les outils d'analyse statique et dynamique, le désassemblage x86/x64, et les techniques de patching.

> [!warning] Cadre légal obligatoire
> Le reverse engineering de binaires tiers peut être soumis à des restrictions légales (DMCA aux États-Unis, directives européennes). Dans le contexte académique et CTF, toutes les techniques présentées s'appliquent à des logiciels que vous possédez, des crackmes publics, ou des environnements de lab isolés. Ne jamais analyser un logiciel commercial en violation de sa licence EULA.

---

## 1. Formats Binaires — PE et ELF

### 1.1 Vue d'ensemble : qu'est-ce qu'un exécutable ?

Un fichier exécutable n'est pas un simple flux d'instructions machine. C'est un **fichier structuré** que le système d'exploitation sait charger en mémoire de façon précise. Deux formats dominent :

| Format | OS | Extension courante | Architecture |
|--------|----|--------------------|-------------|
| **ELF** (Executable and Linkable Format) | Linux, BSD, Android | aucune, `.so`, `.o` | x86, x64, ARM, MIPS, RISC-V |
| **PE** (Portable Executable) | Windows | `.exe`, `.dll`, `.sys` | x86, x64, ARM64 |
| **Mach-O** | macOS, iOS | aucune, `.dylib` | x86_64, ARM64 |

```
Flux de vie d'un binaire ELF

Source C  ──[gcc -c]──► Fichier objet (.o)
                              │
              [ld / gcc]      │  Éditeur de liens
                              ▼
                     Binaire ELF (exécutable)
                              │
              [OS loader]     │  Chargement en mémoire
                              ▼
                   Process en mémoire virtuelle
                   ┌─────────────────────┐
                   │  0xFFFFFFFF (high)   │
                   │  Stack              │ ← rsp
                   │  (grows down)       │
                   │  ...                │
                   │  Heap               │ ← malloc
                   │  (grows up)         │
                   │  .bss               │
                   │  .data              │
                   │  .text              │ ← rip (code)
                   │  0x00400000 (low)   │
                   └─────────────────────┘
```

### 1.2 Format ELF (Linux)

L'ELF est le format binaire standard sous Linux. Sa spécification est publique et bien documentée. Voici sa structure :

```
Structure d'un fichier ELF

┌──────────────────────────────────┐
│        ELF Header (64 bytes)     │  Magic bytes, arch, type, entry point
├──────────────────────────────────┤
│   Program Header Table (PHT)     │  Segments — vus par l'OS loader
│   (optionnel pour .o)            │  LOAD, DYNAMIC, INTERP, NOTE
├──────────────────────────────────┤
│         Sections                 │
│  .text    — code exécutable      │
│  .data    — données initialisées │
│  .bss     — données non-init     │
│  .rodata  — constantes           │
│  .plt     — PLT (imports)        │
│  .got     — GOT (adresses)       │
│  .symtab  — symboles (debug)     │
│  .strtab  — table des chaînes    │
│  .debug_* — infos DWARF          │
├──────────────────────────────────┤
│   Section Header Table (SHT)     │  Sections — vus par les outils
└──────────────────────────────────┘
```

**Le ELF Header** contient en particulier :
- Les **magic bytes** : `7f 45 4c 46` (`\x7fELF`) — signature reconnaissable
- Le **type** : `ET_EXEC` (exécutable), `ET_DYN` (shared object / PIE), `ET_REL` (relocatable .o)
- L'**architecture** : `EM_X86_64` (62), `EM_ARM` (40), etc.
- L'**entry point** : adresse de `_start` (pas `main`) en mémoire

**Les sections critiques** :

| Section | Contenu | Permissions |
|---------|---------|-------------|
| `.text` | Instructions machine | r-x (lecture + exécution) |
| `.data` | Variables globales initialisées | rw- (lecture + écriture) |
| `.bss` | Variables globales non initialisées | rw- (pas de place dans le fichier) |
| `.rodata` | Chaînes littérales, constantes | r-- (lecture seule) |
| `.plt` | Procedure Linkage Table — stubs pour les imports | r-x |
| `.got` | Global Offset Table — adresses réelles des fonctions | rw- |
| `.got.plt` | Partie GOT pour les appels PLT | rw- |

> [!info] .bss ne prend pas de place sur le disque
> La section `.bss` est déclarée dans les headers avec une taille, mais le fichier ne contient pas ses octets. L'OS les alloue et les met à zéro au chargement. C'est pourquoi un binaire avec de grandes tables statiques non initialisées ne pèse pas plus sur le disque.

**PLT/GOT — la résolution dynamique des imports** :

```
Appel à printf() par liaison dynamique

main() appelle  call   0x401050  <printf@plt>
                        │
               PLT stub │
                        ▼
               jmp    [0x404018]  ←── entrée GOT pour printf
                        │
               Premier appel ──► GOT pointe sur le résolveur ld-linux
               ld-linux trouve printf dans libc.so
               GOT[printf] = adresse réelle printf
               Appels suivants ──► jmp direct vers printf (cache)
```

C'est le mécanisme **lazy binding** : les symboles ne sont résolus qu'au premier appel. La technique **GOT overwrite** exploite le fait que `.got.plt` est writable.

### 1.3 Format PE (Windows)

Le format PE est une évolution du format COFF. Il est plus complexe que l'ELF avec plusieurs niveaux de headers.

```
Structure d'un fichier PE (Portable Executable)

┌──────────────────────────────────┐  Offset 0x0
│    DOS Header (64 bytes)         │  MZ signature (4D 5A), e_lfanew
├──────────────────────────────────┤
│    DOS Stub (petit programme)    │  "This program cannot be run..."
├──────────────────────────────────┤  Offset = e_lfanew
│    PE Signature (4 bytes)        │  "PE\0\0" (50 45 00 00)
├──────────────────────────────────┤
│    COFF File Header (20 bytes)   │  Machine, NbSections, Timestamp
├──────────────────────────────────┤
│    Optional Header (224/240 B)   │  ImageBase, EntryPoint, SizeOfImage
│    ├── Data Directories          │  Import Table, Export Table, TLS...
├──────────────────────────────────┤
│    Section Headers               │  .text, .data, .rdata, .rsrc...
├──────────────────────────────────┤
│    Sections (raw data)           │
│  .text    — code                 │
│  .data    — données RW           │
│  .rdata   — données RO           │
│  .rsrc    — ressources (icônes)  │
│  .reloc   — relocations          │
└──────────────────────────────────┘
```

**Concepts PE spécifiques** :

- **ImageBase** : adresse de chargement préférée (0x400000 pour .exe, 0x10000000 pour .dll). Avec ASLR, elle est randomisée.
- **RVA** (Relative Virtual Address) : offset depuis l'ImageBase. Les headers PE utilisent des RVA, pas des adresses absolues.
- **Import Table (IAT)** : équivalent du GOT Linux. Liste les DLLs et fonctions importées.
- **Export Table** : pour les DLLs — liste les fonctions exportées et leurs RVA.
- **TLS Callbacks** : code exécuté **avant** l'entry point. Utilisé par les packers et anti-debug.

| Concept PE | Équivalent ELF |
|-----------|----------------|
| Import Table (IAT) | GOT / PLT |
| Export Table | `.dynsym` |
| `.rdata` | `.rodata` |
| `.reloc` | `.rel.dyn` |
| TLS Callbacks | `.init_array` |
| ImageBase | Base de chargement |

---

## 2. Analyse Statique — Outils en Ligne de Commande

### 2.1 `file` — identifier un binaire

La commande `file` lit les magic bytes pour identifier le type d'un fichier sans se fier à son extension.

```bash
# Exemples d'utilisation
file /bin/ls
# /bin/ls: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV),
#           dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2,
#           BuildID[sha1]=..., for GNU/Linux 3.2.0, stripped

file crackme.exe
# crackme.exe: PE32+ executable (console) x86-64, for MS Windows

file suspicious.bin
# suspicious.bin: UPX compressed data

file library.so
# library.so: ELF 64-bit LSB shared object, x86-64, dynamically linked, not stripped
```

> [!tip] Flags importants dans la sortie `file`
> - **stripped** vs **not stripped** : un binaire stripped n'a plus ses symboles de debug (`.symtab`). Beaucoup plus dur à analyser.
> - **dynamically linked** vs **statically linked** : statique = tout le code de libc est inclus dans le binaire (plus gros, plus autonome, pas de PLT/GOT).
> - **PIE** (Position Independent Executable) : le binaire peut être chargé à n'importe quelle adresse. Activé par ASLR.

### 2.2 `strings` — extraire les chaînes lisibles

`strings` scanne le binaire à la recherche de séquences de caractères ASCII (ou Unicode) imprimables d'au moins N octets. Souvent la **première étape** d'une analyse rapide.

```bash
# Extraction basique (minimum 4 chars par défaut)
strings crackme

# Minimum 8 chars pour réduire le bruit
strings -n 8 crackme

# Afficher l'offset dans le fichier
strings -o crackme

# Mode Unicode (UTF-16 LE — commun dans les binaires Windows)
strings -e l crackme.exe

# Combiner avec grep pour chercher des patterns
strings crackme | grep -i "password\|flag\|key\|license\|correct\|wrong"

# Chercher des URLs
strings malware.exe | grep -E "https?://"
```

**Ce qu'on trouve typiquement avec strings :**

```
Exemples de strings intéressantes dans un crackme :

"Enter your license key: "     ← prompt utilisateur
"Congratulations!"             ← message de succès
"Wrong key, try again."        ← message d'échec
"HTB{...}"                     ← flag CTF dans le binaire
"AAAABBBBCCCC"                 ← clé hardcodée ?
"http://192.168.1.100/c2"      ← C2 URL dans un malware
"SOFTWARE\Microsoft\Windows"   ← clé de registre
"/bin/sh"                      ← shell spawning
```

> [!warning] Limites de strings
> - Les chaînes chiffrées ou encodées (XOR, base64) ne sont pas lisibles
> - Les packers compressent le binaire : strings ne voit que le loader, pas le vrai code
> - Les chaînes peuvent être construites dynamiquement en stack (une lettre à la fois)

### 2.3 `readelf` — anatomie d'un ELF

`readelf` affiche toutes les structures internes d'un ELF. Indispensable pour comprendre un binaire avant de le désassembler.

```bash
# Header ELF complet
readelf -h binary
# ELF Header:
#   Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
#   Class:                             ELF64
#   Data:                              2's complement, little endian
#   Entry point address:               0x401060
#   Start of program headers:          64 (bytes into file)

# Table des sections
readelf -S binary
# [Nr] Name          Type     Address          Offset  Size   ...
# [ 0]               NULL     0000000000000000 000000  000000
# [14] .text         PROGBITS 0000000000401060 001060  0002a5
# [25] .data         PROGBITS 0000000000404020 003020  000010

# Table des segments (Program Headers) — vue loader
readelf -l binary

# Symboles (si non stripped)
readelf -s binary
# Num: Value              Size Type    Bind   Name
#  65: 0000000000401156   105  FUNC    GLOBAL main

# Imports dynamiques
readelf -d binary | grep NEEDED
# 0x0000000000000001 (NEEDED) Shared library: [libc.so.6]

# Sections de relocation
readelf -r binary

# Tout afficher
readelf -a binary
```

### 2.4 `objdump` — désassemblage rapide

`objdump` désassemble un binaire en ligne de commande. Moins puissant que Ghidra mais toujours disponible sur n'importe quel Linux.

```bash
# Désassembler la section .text (syntaxe AT&T par défaut)
objdump -d binary

# Syntaxe Intel (plus lisible, préférée en RE)
objdump -d -M intel binary

# Désassembler TOUTES les sections exécutables
objdump -D binary

# Afficher le code source entrelacé (si debug info disponible)
objdump -d -S binary

# Lister toutes les sections
objdump -h binary

# Voir le contenu hexadécimal d'une section
objdump -s -j .rodata binary

# Désassembler une fonction précise (avec nm pour trouver l'adresse)
objdump -d binary | grep -A 30 "<main>:"
```

**Exemple de sortie `objdump -d -M intel` :**

```
0000000000401156 <main>:
  401156:       55                      push   rbp
  401157:       48 89 e5                mov    rbp,rsp
  40115a:       48 83 ec 20             sub    rsp,0x20
  40115e:       48 8d 3d 9f 0e 00 00    lea    rdi,[rip+0xe9f]   # "Enter key: "
  401165:       e8 e6 fe ff ff          call   401050 <puts@plt>
  40116a:       48 8b 45 f8             mov    rax,QWORD PTR [rbp-0x8]
```

### 2.5 `nm` — table des symboles

```bash
# Lister tous les symboles
nm binary

# Trier par adresse
nm -n binary

# Seulement les symboles définis (pas les imports)
nm -D binary | grep -v " U "

# Format lisible
nm -C binary  # demangling C++

# Types de symboles importants :
# T = dans .text (code)
# D = dans .data (données initialisées)
# B = dans .bss
# U = undefined (importé)
# t = local (static) dans .text
```

### 2.6 `ldd` — dépendances dynamiques

```bash
# Lister les bibliothèques requises
ldd /bin/ls
#         linux-vdso.so.1 (0x00007ffd1e5f3000)
#         libselinux.so.1 => /lib/x86_64-linux-gnu/libselinux.so.1
#         libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6

# Ne jamais utiliser ldd sur des binaires suspects !
# ldd exécute le binaire avec LD_TRACE_LOADED_OBJECTS — cela peut déclencher du code malveillant
# Utiliser plutôt :
objdump -p malware | grep NEEDED
readelf -d malware | grep NEEDED
```

> [!warning] ldd est dangereux sur des binaires suspects
> `ldd` peut exécuter le binaire pour résoudre les dépendances. Sur un malware, cela peut déclencher l'exécution de code. Préférer `readelf -d` ou `objdump -p` pour les binaires non fiables.

---

## 3. Désassemblage — Syntaxe x86/x64 Assembly

### 3.1 Registres x86-64

Comprendre les registres est indispensable pour lire l'Assembly.

```
Registres généraux x86-64

64-bit   32-bit  16-bit  8-bit high  8-bit low   Usage conventionnel
rax      eax     ax      ah          al          Valeur de retour, accumulateur
rbx      ebx     bx      bh          bl          Callee-saved (préservé par l'appelé)
rcx      ecx     cx      ch          cl          4ème argument, compteur
rdx      edx     dx      dh          dl          3ème argument, multiplicateur
rsi      esi     si      -           sil         2ème argument (Linux)
rdi      edi     di      -           dil         1er argument (Linux)
rsp      esp     sp      -           spl         Stack pointer (pointeur de pile)
rbp      ebp     bp      -           bpl         Base pointer (frame pointer)
r8       r8d     r8w     -           r8b         5ème argument
r9       r9d     r9w     -           r9b         6ème argument
r10-r15  ...                                     Usage général, caller-saved

Registres spéciaux :
rip      -       ip                              Instruction pointer (compteur programme)
rflags   eflags  flags                           Drapeaux (ZF, CF, SF, OF, PF)
```

**Les drapeaux RFLAGS importants** :

| Drapeau | Bit | Signification | Modifié par |
|---------|-----|---------------|------------|
| **ZF** (Zero Flag) | 6 | Résultat == 0 | Toutes les opérations arithmétiques |
| **CF** (Carry Flag) | 0 | Dépassement non signé | add, sub, shift |
| **SF** (Sign Flag) | 7 | Résultat négatif | Opérations arithmétiques |
| **OF** (Overflow Flag) | 11 | Dépassement signé | add, sub |
| **PF** (Parity Flag) | 2 | Parité des bits bas | Arithmétique |

### 3.2 Instructions x86-64 essentielles

```nasm
; ── TRANSFERT DE DONNÉES ──────────────────────────────────

mov  rax, rbx          ; rax = rbx (copie)
mov  rax, [rbx]        ; rax = *rbx (déréférencement)
mov  [rbx], rax        ; *rbx = rax (écriture en mémoire)
mov  rax, 42           ; rax = 42 (immédiat)
mov  rax, [rbx+8]      ; rax = *(rbx + 8) (offset)
mov  rax, [rbx+rcx*4]  ; rax = *(rbx + rcx*4) (SIB addressing)

lea  rax, [rbx+8]      ; rax = rbx + 8 (Load Effective Address — PAS de déréférencement)
                       ; Souvent utilisé pour calculer des adresses sans accéder à la mémoire

xchg rax, rbx          ; swap rax et rbx

; ── PILE (STACK) ───────────────────────────────────────────

push rax               ; rsp -= 8 ; *rsp = rax
pop  rax               ; rax = *rsp ; rsp += 8

; ── ARITHMÉTIQUE ──────────────────────────────────────────

add  rax, rbx          ; rax += rbx
sub  rax, rbx          ; rax -= rbx
mul  rbx               ; rdx:rax = rax * rbx (non signé, résultat 128 bits)
imul rax, rbx          ; rax *= rbx (signé, résultat 64 bits)
div  rbx               ; rax = rdx:rax / rbx, rdx = reste (non signé)
idiv rbx               ; signé
inc  rax               ; rax++
dec  rax               ; rax--
neg  rax               ; rax = -rax

; ── LOGIQUE ET BITS ───────────────────────────────────────

and  rax, rbx          ; rax &= rbx
or   rax, rbx          ; rax |= rbx
xor  rax, rbx          ; rax ^= rbx
xor  rax, rax          ; rax = 0 (idiome courant pour mettre à zéro)
not  rax               ; rax = ~rax
shl  rax, 3            ; rax <<= 3 (shift left = multiply by 8)
shr  rax, 3            ; rax >>= 3 (shift right logique)
sar  rax, 3            ; shift arithmetic right (préserve le signe)

; ── COMPARAISON ET SAUTS ──────────────────────────────────

cmp  rax, rbx          ; calcule rax - rbx, modifie les flags, ne stocke pas le résultat
test rax, rbx          ; calcule rax & rbx, modifie les flags (souvent test rax, rax pour cmp avec 0)

; Sauts conditionnels (après cmp ou test)
jz   label             ; Jump if Zero (ZF=1)
jnz  label             ; Jump if Not Zero (ZF=0)
je   label             ; Jump if Equal (alias jz)
jne  label             ; Jump if Not Equal (alias jnz)
jl   label             ; Jump if Less (signé)
jle  label             ; Jump if Less or Equal (signé)
jg   label             ; Jump if Greater (signé)
jge  label             ; Jump if Greater or Equal (signé)
jb   label             ; Jump if Below (non signé, alias jc)
ja   label             ; Jump if Above (non signé)
jmp  label             ; Saut inconditionnel

; ── APPELS DE FONCTIONS ───────────────────────────────────

call foo               ; push rip ; jmp foo (sauvegarde adresse retour)
ret                    ; pop rip (retour à l'appelant)

; ── DIVERS ────────────────────────────────────────────────

nop                    ; No Operation (0x90) — ne fait rien
syscall                ; Appel système Linux (numéro dans rax)
int  0x80              ; Appel système Linux 32-bit (legacy)
hlt                    ; Halt — arrête le CPU (mode noyau seulement)
```

### 3.3 Calling Conventions

Une **calling convention** définit comment les arguments sont passés, qui nettoie la pile, et quels registres sont préservés entre appels.

**System V AMD64 ABI (Linux, macOS x64) :**

```
Arguments entiers/pointeurs : rdi, rsi, rdx, rcx, r8, r9 (dans l'ordre)
Arguments flottants          : xmm0 - xmm7
Arguments supplémentaires    : sur la pile (de droite à gauche)
Valeur de retour entier      : rax (+ rdx pour 128 bits)
Valeur de retour flottant    : xmm0

Registres caller-saved (l'appelant doit sauvegarder si besoin) :
  rax, rcx, rdx, rsi, rdi, r8, r9, r10, r11

Registres callee-saved (l'appelé DOIT restaurer) :
  rbx, rbp, r12, r13, r14, r15
```

**Exemple concret : appel de fonction C en Assembly**

```c
// C source
int add(int a, int b, int c) {
    return a + b + c;
}

int main() {
    int result = add(10, 20, 30);
}
```

```nasm
; Assembly généré (System V AMD64)
add:
    push   rbp
    mov    rbp, rsp
    ; rdi = a (10), rsi = b (20), rdx = c (30)
    mov    eax, edi        ; eax = a
    add    eax, esi        ; eax += b
    add    eax, edx        ; eax += c
    pop    rbp
    ret                    ; retourne eax = 60

main:
    push   rbp
    mov    rbp, rsp
    sub    rsp, 16         ; alloue espace stack frame
    mov    edx, 30         ; 3ème argument
    mov    esi, 20         ; 2ème argument
    mov    edi, 10         ; 1er argument
    call   add
    ; rax contient maintenant 60
    mov    DWORD PTR [rbp-4], eax  ; stocke result
```

**Microsoft x64 ABI (Windows) :**

```
Arguments                    : rcx, rdx, r8, r9 (puis pile)
Shadow space obligatoire     : 32 bytes réservés sur la pile avant tout call
Valeur de retour             : rax
Callee-saved                 : rbx, rbp, rdi, rsi, r12-r15, xmm6-xmm15
```

> [!info] Différence critique Linux vs Windows
> Sur Linux, les 6 premiers args passent par rdi/rsi/rdx/rcx/r8/r9.
> Sur Windows, seulement 4 (rcx/rdx/r8/r9) et il faut un "shadow space" de 32 bytes.
> Cette différence est cruciale quand on analyse des shellcodes ou des exploits portés d'un OS à l'autre.

### 3.4 Lire la pile (Stack Frame)

```
Stack frame typique d'une fonction (System V AMD64)

Adresses croissantes vers le bas (haute → basse)

rsp avant call  ──► [rip de retour]     ← poussé par CALL
                    [rbp sauvegardé]    ← push rbp
rbp             ──► [local var -8]      ← variables locales
                    [local var -16]
                    [local var -24]
rsp             ──► [local var -N]      ← sub rsp, N

Accès :
  [rbp-8]   = première variable locale
  [rbp+8]   = adresse de retour
  [rbp+16]  = 7ème argument (si > 6 args)
```

**Exemple : lire une stack frame dans Ghidra/GDB**

```nasm
; Fonction avec variables locales
check_password:
    push   rbp
    mov    rbp, rsp
    sub    rsp, 0x30          ; 48 bytes de variables locales
    mov    [rbp-0x28], rdi    ; sauvegarde du pointeur de chaîne entré
    ; ...
    mov    rdi, [rbp-0x28]    ; récupère le pointeur
    call   strlen
    cmp    eax, 0x10          ; la longueur doit être 16
    jne    .fail
```

---

## 4. Ghidra — Analyse Statique Avancée

### 4.1 Installation et configuration

Ghidra est un outil de reverse engineering développé et open-sourcé par la NSA. Il offre décompilation, désassemblage et scripting. Il est gratuit et multiplateforme.

```bash
# Prérequis : Java 17+
java --version  # doit afficher 17 ou plus

# Téléchargement
# https://ghidra-sre.org/ → dernière release
wget https://github.com/NationalSecurityAgency/ghidra/releases/download/Ghidra_11.x_build/ghidra_11.x_PUBLIC_YYYYMMDD.zip

# Extraction
unzip ghidra_11.x_PUBLIC_YYYYMMDD.zip
cd ghidra_11.x_PUBLIC/

# Lancement
./ghidraRun          # Linux/macOS
ghidraRun.bat        # Windows
```

**Configuration recommandée pour CTF/crackmes :**

```
Edit → Tool Options → Listing Fields
  → Activer "Pre Comment", "EOL Comment", "Plate Comment"

Edit → Tool Options → Decompiler
  → Simplify Predefined → True
  → Prototype Evaluation → Spec from Program

Window → Script Manager → ajouter des scripts supplémentaires
```

### 4.2 Workflow complet d'analyse avec Ghidra

**Étape 1 : Créer un projet et importer le binaire**

```
1. File → New Project → Non-Shared Project
2. Choisir un dossier de projet
3. File → Import File → sélectionner le binaire
4. Ghidra détecte automatiquement : format, architecture, endianness
5. Double-clic sur le fichier importé → ouvre CodeBrowser
6. "Analyze Now?" → Yes → lancer l'auto-analyse (30s à plusieurs minutes)
```

**Étape 2 : Navigation dans CodeBrowser**

```
Interface principale :
┌─────────────────────────────────────────────────────────┐
│ Program Trees │        Listing (ASM)        │ Decompiler│
│               │ 00401156  push  rbp          │ void main │
│  [.text]      │ 00401157  mov   rbp, rsp     │ {         │
│  [.data]      │ 00401160  sub   rsp, 0x20    │   puts(   │
│  [.rodata]    │ ...                          │     ...); │
├───────────────┤                              │ }         │
│ Symbol Tree   │   References (Xrefs)         │           │
│ Functions     │                              │           │
│ Labels        │                              │           │
└─────────────────────────────────────────────────────────┘
```

**Raccourcis clavier essentiels :**

| Raccourci | Action |
|-----------|--------|
| `G` | Aller à une adresse |
| `L` | Renommer label/variable |
| `T` | Changer le type d'une variable |
| `F` | Recherche de texte |
| `Ctrl+L` | Rechercher un symbole |
| `X` | Voir toutes les références à un symbole (xrefs) |
| `;` | Ajouter un commentaire |
| `D` | Désassembler à cette adresse |
| `U` | Undefine (effacer l'analyse) |
| `C` | Créer une fonction |
| `P` | Créer une procédure (fonction) |

**Étape 3 : Trouver le main() et naviguer**

```
1. Symbol Tree → Functions → main (ou "_main" sur Windows)
2. Double-clic → Listing saute à l'adresse
3. Fenêtre Decompiler affiche le C pseudo-code correspondant

Si le binaire est stripped (pas de symboles) :
→ Chercher "entry" dans Symbol Tree
→ entry appelle __libc_start_main(main, ...) → le premier arg est main
→ Ou chercher "puts", "printf" dans les xrefs (menu Search → For Strings)
```

**Étape 4 : Renommer les fonctions et variables**

Le renommage est l'action la plus importante en RE — il transforme du code illisible en code compréhensible.

```
Workflow de renommage :

1. Identifier une fonction inconnue (sub_401234)
2. Lire le Decompiler :
   - Quels arguments reçoit-elle ?
   - Quelles fonctions appelle-t-elle ?
   - Que retourne-t-elle ?
3. Appuyer sur L → saisir un nom descriptif (ex: validate_license)
4. Les xrefs se mettent à jour automatiquement partout dans le projet
5. Renommer aussi les paramètres et variables locales (L dans le Decompiler)
```

**Exemple avant/après renommage :**

```c
// Avant renommage (généré par Ghidra)
undefined8 sub_401156(char *param_1) {
    int iVar1;
    char local_28[32];
    
    iVar1 = strlen(param_1);
    if (iVar1 != 0x10) {
        return 0;
    }
    sub_401000(local_28, param_1, 0x10);  // mystérieux
    iVar1 = sub_401080(local_28, &DAT_402010, 0x10);
    return (undefined8)(iVar1 == 0);
}

// Après renommage
bool validate_license(char *user_input) {
    int input_len;
    char xor_result[32];
    
    input_len = strlen(user_input);
    if (input_len != 16) {
        return false;
    }
    xor_encrypt(xor_result, user_input, 16);   // renommée après analyse
    input_len = memcmp(xor_result, &EXPECTED_HASH, 16);
    return (input_len == 0);
}
```

**Étape 5 : Analyser les types et structures**

```
Si une variable est traitée comme un pointeur vers une structure :
1. Sélectionner la variable dans le Decompiler
2. Clic droit → Auto Create Structure (Ghidra infère les offsets)
3. Ou : Data Type Manager → Create Structure → ajouter les champs manuellement

Exemple : 
  iVar1 = (int)(*(char *)(uVar2 + 4));
→ Probablement un champ "type" à l'offset 4 d'une structure

Créer : struct Header { int magic; int type; int size; ... };
Appliquer le type → le Decompiler devient lisible
```

### 4.3 Scripting Ghidra

Ghidra supporte les scripts Java et Python (Jython).

```python
# Script Python Ghidra — trouver toutes les chaînes passées à puts()
# Window → Script Manager → New Script → GhidraScript

from ghidra.program.model.symbol import RefType

listing = currentProgram.getListing()
fm = currentProgram.getFunctionManager()

# Trouver la fonction puts
puts_sym = getSymbol("puts", None)
if puts_sym is None:
    print("puts non trouvé")

puts_addr = puts_sym.getAddress()

# Trouver toutes les références à puts
refs = getReferencesTo(puts_addr)
for ref in refs:
    call_addr = ref.getFromAddress()
    # Regarder l'instruction précédente (mov rdi, ...)
    prev_instr = listing.getInstructionBefore(call_addr)
    print(f"Call à puts depuis {call_addr} : {prev_instr}")
```

---

## 5. IDA Free — Interface et CFG

### 5.1 IDA Free vs IDA Pro

| Fonctionnalité | IDA Free | IDA Pro |
|---------------|----------|---------|
| x86/x64 | Oui | Oui |
| ARM, MIPS, autres | Non | Oui |
| Décompilation Hex-Rays | Non | Oui (payant) |
| Scripting IDAPython | Limité | Complet |
| FLIRT signatures | Oui | Oui |
| Prix | Gratuit | $3,000+ |

IDA Free reste utile pour le CFG (Control Flow Graph) et les signatures FLIRT.

### 5.2 Graphe de Flux de Contrôle (CFG)

Le CFG est l'une des fonctionnalités les plus puissantes d'IDA. Il représente visuellement les blocs de code et leurs transitions.

```
Vue CFG d'IDA (Space dans la vue Listing)

     ┌─────────────────┐
     │   Entry Block   │
     │  push rbp       │
     │  mov rbp, rsp   │
     │  cmp eax, 16    │
     └────────┬────────┘
              │
       ┌──────┴──────┐
      JNE           JE
       │              │
┌──────▼───────┐  ┌───▼──────────┐
│  FAIL Block  │  │  SUCCESS     │
│ mov eax, 0   │  │ mov eax, 1   │
│ jmp epilogue │  │ jmp epilogue │
└──────┬───────┘  └───┬──────────┘
       └──────┬────────┘
              │
     ┌────────▼────────┐
     │   Epilogue      │
     │   pop rbp       │
     │   ret           │
     └─────────────────┘
```

Les blocs en **rouge** = chemin d'échec (sauts pris vers un mauvais chemin).
Les blocs en **vert** = chemin de succès.
Cette visualisation rend immédiatement évident **quel saut bypasser** dans un crackme.

### 5.3 FLIRT Signatures

FLIRT (Fast Library Identification and Recognition Technology) permet à IDA de reconnaître des fonctions de bibliothèques standard même dans des binaires statiquement liés ou stripped.

```
Sans FLIRT : un binaire statique a des centaines de sub_XXXXXX inconnues
Avec FLIRT  : IDA reconnaît "sub_401234" = "strcmp" de glibc 2.31

Application :
1. Options → Load File → FLIRT Signature File
2. Sélectionner les .sig correspondant à la version de libc suspectée
3. IDA renomme automatiquement les fonctions reconnues
```

**Créer ses propres signatures FLIRT (cas avancé) :**

```bash
# Avec pcf (Pattern Compiler) d'IDA
pcf libc.a libc.pat
sigmake libc.pat libc.sig
# Copier libc.sig dans le dossier sig/ d'IDA
```

---

## 6. Analyse Dynamique — GDB et pwndbg

### 6.1 GDB de base

GDB (GNU Debugger) permet d'exécuter un programme en le contrôlant pas-à-pas, d'inspecter la mémoire et les registres, et de modifier l'état du programme.

```bash
# Lancer gdb sur un binaire
gdb ./crackme

# Avec arguments
gdb --args ./crackme arg1 arg2

# Avec un fichier core dump
gdb ./crackme core

# Interface TUI (texte graphique)
gdb -tui ./crackme
```

**Commandes GDB essentielles :**

```bash
# ── BREAKPOINTS ────────────────────────────────────────────

break main              # breakpoint sur la fonction main
b *0x401156             # breakpoint sur une adresse précise
b main.c:42             # breakpoint sur une ligne source
info breakpoints        # lister les breakpoints
delete 2                # supprimer le breakpoint n°2
disable 1               # désactiver sans supprimer
enable 1                # réactiver

# Breakpoints conditionnels
b *0x401156 if $eax == 0x42

# Watchpoints (arrêt quand une variable est modifiée)
watch *(int *)0x404020  # surveille une adresse mémoire

# ── EXÉCUTION ─────────────────────────────────────────────

run                     # lancer l'exécution (ou r)
run arg1 arg2           # avec arguments
run < input.txt         # avec stdin depuis un fichier
continue               # continuer jusqu'au prochain breakpoint (ou c)
step                    # step into — entre dans les fonctions (ou s)
next                    # step over — passe par dessus les appels (ou n)
finish                  # exécuter jusqu'à la fin de la fonction courante
until 0x401200          # exécuter jusqu'à cette adresse
jump *0x401156          # forcer le saut à une adresse (dangereux)

# ── INSPECTION ─────────────────────────────────────────────

info registers          # tous les registres
print $rax              # valeur de rax
print/x $rax            # en hexadécimal
print/d $rax            # en décimal
print/s $rdi            # comme chaîne

# Examiner la mémoire (x = examine)
x/10gx $rsp             # 10 quadwords (8 bytes) en hex depuis rsp
x/20wx 0x404000         # 20 words (4 bytes) en hex
x/s 0x402010            # comme une chaîne
x/10i $rip              # 10 instructions depuis rip (désassemblage)
x/50bx $rsp             # 50 bytes en hex

# Backtrace
bt                      # afficher la pile d'appels
frame 2                 # se placer dans le frame n°2
info locals             # variables locales du frame courant
info args               # arguments de la fonction courante

# ── MODIFICATION ───────────────────────────────────────────

set $rax = 0x1          # modifier un registre
set *(int *)0x404020 = 42  # modifier une valeur mémoire
set $eflags = $eflags ^ 0x40  # toggler le Zero Flag (ZF)
```

### 6.2 pwndbg — GDB amélioré

pwndbg est un plugin GDB qui affiche automatiquement les registres, la pile et le désassemblage à chaque step. Indispensable pour le CTF.

```bash
# Installation
git clone https://github.com/pwndbg/pwndbg
cd pwndbg
./setup.sh

# Ou via pip
pip install pwndbg
```

**Commandes supplémentaires pwndbg :**

```bash
context          # affiche registres + stack + code — fait automatiquement à chaque stop
nearpc 20        # 20 instructions autour de rip
stack 20         # 20 entrées de la stack
telescope $rsp   # affiche la stack avec déréférencement automatique
vmmap            # carte mémoire du process (permissions, mappings)
got              # afficher toutes les entrées de la GOT
plt              # afficher les entrées PLT
checksec         # vérifier les protections du binaire (NX, PIE, Canary, RELRO)
cyclic 100       # générer un pattern cyclique de 100 bytes (pour trouver offset overflow)
cyclic -l 0x6161616161616166  # trouver l'offset d'un pattern
heap             # afficher l'état du heap (chunks)
```

**Exemple de session GDB/pwndbg typique sur un crackme :**

```bash
$ gdb ./crackme
pwndbg> checksec
[*] '/tmp/crackme'
    Arch:     amd64-64-little
    RELRO:    Full RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)

pwndbg> info functions
0x0000000000401156  main
0x0000000000401090  validate_key
0x0000000000401000  _init

pwndbg> b validate_key
Breakpoint 1 at 0x401090

pwndbg> run
Enter license key: AAAAAAAAAAAAAAAA
Breakpoint 1, 0x0000000000401090 in validate_key ()

pwndbg> context
 ► 0x401090 <validate_key>    push   rbp
   0x401091 <validate_key+1>  mov    rbp, rsp

REGISTERS
 RDI  0x7fffffffde40 ◂— 'AAAAAAAAAAAAAAAA'   ← notre input
 RSI  0x402010 ◂— 0x4839...                   ← clé attendue ?

pwndbg> x/s 0x402010
0x402010: "SecretKey123456"
```

### 6.3 Techniques GDB avancées

**Changer le ZF pour bypasser un check :**

```bash
# Le programme va faire jne (saute si != donc si mauvais mdp)
# On est stoppé sur le je/jne :
pwndbg> info registers eflags
eflags         0x202  [ IF ]
# ZF = 0 → va sauter vers FAIL
# On veut ZF = 1 → aller vers SUCCESS

pwndbg> set $eflags = $eflags | 0x40   # mettre ZF à 1
pwndbg> continue
# → SUCCESS même avec le mauvais mot de passe
```

**Patcher un saut directement :**

```bash
# Méthode 2 : forcer le saut avec jump
pwndbg> disassemble validate_key
   0x4010c8  jne    0x4010d4 <fail_block>
   0x4010ca  jmp    0x4010e0 <success_block>

pwndbg> set $rip = 0x4010ca  # forcer l'exécution depuis jmp success
pwndbg> continue
```

**GDB scripting pour automatiser l'analyse :**

```python
# Fichier script.gdb — lancer avec : gdb -x script.gdb ./crackme
set pagination off
break validate_key
run <<< "AAAAAAAAAAAAAAAA"
# Afficher les arguments quand on atteint validate_key
define hook-stop
  info registers rdi rsi
end
continue
quit
```

---

## 7. x64dbg — Débogueur Windows

### 7.1 Interface x64dbg

x64dbg est le débogueur open-source de référence pour Windows. Il combine désassemblage, debugging et patching.

```
Interface x64dbg

┌─────────────────────────────────────────────────────────┐
│  CPU View (Assembleur)     │  Registers                 │
│  00401156  push ebp        │  EAX: 00000001             │
│  00401157  mov  ebp,esp    │  EBX: 00000000             │
│  [→] 0040115A cmp eax,10   │  ECX: 0040201A             │
│  0040115D  jne  00401200   │  EIP: 0040115A ◄── actuel  │
├─────────────────────────────┤  ESP: 0019FF50             │
│  Stack View                │  EFLAGS: 00000202          │
│  0019FF50  00401162 (ret) │                              │
│  0019FF54  00000001       │  ┌─────────────────────┐    │
│  0019FF58  0019FF78       │  │   Memory Map        │    │
├────────────────────────────┴──┤   Breakpoints       │    │
│  Hex Dump                     │   Log               │    │
│  00402010: 53 65 63 72 65 74  │                     │    │
└───────────────────────────────┴─────────────────────┘────┘
```

**Raccourcis x64dbg :**

| Touche | Action |
|--------|--------|
| `F2` | Toggle breakpoint sur l'instruction sélectionnée |
| `F7` | Step Into (équivalent gdb `step`) |
| `F8` | Step Over (équivalent gdb `next`) |
| `F9` | Run (continuer jusqu'au prochain BP) |
| `Ctrl+G` | Aller à une adresse |
| `Ctrl+F` | Rechercher dans les instructions |
| `Espace` | Patcher une instruction (éditeur d'assembleur intégré) |
| `Alt+E` | Afficher les modules chargés |
| `Ctrl+B` | Recherche de pattern hexadécimal en mémoire |

### 7.2 Patching de binaire avec x64dbg

x64dbg permet de modifier directement les instructions en mémoire et d'appliquer le patch au fichier.

```
Workflow de patch :

1. Trouver le saut conditionnel à bypasser
   Exemple : JNE 00401200 (saute vers FAIL si mauvais mot de passe)

2. Clic droit sur l'instruction → "Assemble" (ou Espace)
   Remplacer par : JMP 00401100 (saute vers SUCCESS toujours)
   Ou : NOP (no-operation, 0x90) pour neutraliser le saut

3. Clic droit sur la zone modifiée → Patches → "Patch File"
   → Sauvegarde le binaire patché sur le disque

4. Tester le binaire patché
```

**Patching en hexadécimal :**

```
Adresse  | Original    | Patché
---------|-------------|--------
0040115D | 75 41       | 90 90    (JNE → NOP NOP)
         | jne +0x41   | nop ; nop

Encodage des sauts :
  EB xx = JMP court (xx = offset signé 1 byte)
  74 xx = JE (xx = offset)
  75 xx = JNE (xx = offset)
  0F 84 xx xx xx xx = JE long (offset 4 bytes)
  0F 85 xx xx xx xx = JNE long
```

### 7.3 Breakpoints conditionnels et hardware

```
x64dbg → Breakpoints → Software BP (INT3, 0xCC)
                     → Hardware BP (registres DR0-DR3 du CPU)
                     → Memory BP (surveille accès mémoire)

Breakpoint conditionnel :
  Clic droit sur le BP → Edit Breakpoint
  Condition : [eax] == 0x42
  (x64dbg utilise une expression engine propre)

Hardware BP (utile quand le code est automodifiant) :
  Debug → Set Hardware Breakpoint → Enter Address
  Type : Execute / Read / Write
  4 hardware BP maximum simultanément (limitation CPU)
```

---

## 8. Anti-Debugging et Protections

### 8.1 Détection de debugger — Windows

Les logiciels commerciaux et les malwares utilisent des techniques pour détecter si un debugger est attaché.

**IsDebuggerPresent :**

```c
// Technique la plus simple (WinAPI)
#include <windows.h>

int main() {
    if (IsDebuggerPresent()) {
        MessageBox(NULL, "Debugger detected!", "Error", 0);
        ExitProcess(1);
    }
    // Code protégé ici
}
```

```nasm
; En assembleur : IsDebuggerPresent lit le champ BeingDebugged
; dans le TEB (Thread Environment Block) via le PEB
; PEB → offset 0x2 = BeingDebugged

mov eax, dword ptr fs:[0x30]  ; PEB (sur 32-bit, gs:[0x60] sur 64-bit)
movzx eax, byte ptr [eax+2]   ; BeingDebugged
test eax, eax
jnz debugger_found
```

**CheckRemoteDebuggerPresent :**

```c
BOOL isDebuggerPresent = FALSE;
CheckRemoteDebuggerPresent(GetCurrentProcess(), &isDebuggerPresent);
if (isDebuggerPresent) { exit(1); }
```

**NtQueryInformationProcess :**

```c
// Méthode plus discrète via l'API native NT
typedef NTSTATUS(WINAPI *NtQueryInformationProcess_t)(
    HANDLE, UINT, PVOID, ULONG, PULONG);

NtQueryInformationProcess_t NtQIP = (NtQueryInformationProcess_t)
    GetProcAddress(GetModuleHandle("ntdll.dll"), "NtQueryInformationProcess");

DWORD debugPort = 0;
NtQIP(GetCurrentProcess(), 7 /*ProcessDebugPort*/, &debugPort, 4, NULL);
if (debugPort != 0) { exit(1); }
```

**Timing attacks (détection par la lenteur du debug) :**

```c
// Un debugger ralentit l'exécution
LARGE_INTEGER start, end;
QueryPerformanceCounter(&start);
// Code quelconque
for (int i = 0; i < 10000; i++) { volatile int x = i * i; }
QueryPerformanceCounter(&end);

if ((end.QuadPart - start.QuadPart) > THRESHOLD) {
    // Probablement en train d'être débogué
    exit(1);
}
```

### 8.2 Détection de debugger — Linux

```c
// ptrace self-check : un process ne peut être ptracé qu'une fois
// Si ptrace échoue → déjà débogué
#include <sys/ptrace.h>

if (ptrace(PTRACE_TRACEME, 0, 0, 0) < 0) {
    printf("Debugger detected!\n");
    exit(1);
}
```

```c
// Vérifier /proc/self/status → TracerPid
FILE *f = fopen("/proc/self/status", "r");
char line[256];
while (fgets(line, sizeof(line), f)) {
    if (strncmp(line, "TracerPid:", 10) == 0) {
        int tracerPid = atoi(line + 10);
        if (tracerPid != 0) {
            printf("TracerPid: %d — debugger détecté\n", tracerPid);
            exit(1);
        }
    }
}
```

### 8.3 Bypass des anti-debug

**Sous GDB/pwndbg :**

```bash
# Bypass IsDebuggerPresent via le PEB
# Trouver l'adresse du PEB
pwndbg> vmmap

# Dans Ghidra/x64dbg : patcher l'appel à IsDebuggerPresent
# Trouver : call IsDebuggerPresent → remplacer par xor eax,eax
# (xor eax,eax = retourner 0 = "pas de debugger")

# GDB : hooker la fonction avec un script
define hook_isdebuggerpresent
    set $rax = 0
    return
end
```

**Bypass ptrace :**

```bash
# Méthode 1 : patch le saut conditionnel après ptrace()
# Méthode 2 : forcer la valeur de retour de ptrace à 0
# Dans GDB, sur Linux :
catch syscall ptrace        # intercepte tous les appels ptrace
commands                    # quand intercepté :
    set $rax = 0            # forcer le retour à succès
    continue
end
```

### 8.4 Obfuscation et Packing

**Obfuscation de code :**

```
Techniques d'obfuscation courantes :

1. Garbage instructions — insertions de NOP/PUSH+POP sans effet
2. Dead code — branches jamais atteintes mais qui trompent le désassembleur
3. String encryption — XOR, ROT, AES des chaînes littérales
4. Control flow flattening — toutes les branches passent par un dispatcher
5. VM Protector — le code est transformé en bytecode d'une VM custom
6. Code mutation — le code se réécrit à chaque exécution (polymorphisme)
```

**Packers :**

Un packer compresse/chiffre le binaire original, qui est "décompressé" au runtime par un stub. Résultat : `file` indique "UPX compressed data" ou "packed executable".

```bash
# Détecter un packer
strings binary | grep -i "upx\|packed\|stub"
file binary        # "UPX compressed" ou similaire
binwalk binary     # détecte les sections avec haute entropie

# Mesurer l'entropie des sections (haute entropie = chiffré/compressé)
# avec rabin2 (radare2)
rabin2 -S binary

# Unpacker UPX (le plus commun en CTF)
upx -d packed_binary -o unpacked_binary

# Pour les packers custom → analyse dynamique
# 1. Lancer sous x64dbg/gdb
# 2. Poser un breakpoint sur l'entry point après décompression
#    (chercher le OEP — Original Entry Point)
# 3. Dumper le process depuis la mémoire (plugin OllyDumpEx ou pe-sieve)
```

**Repérer l'OEP (Original Entry Point) :**

```
Méthode classique "ESP trick" pour UPX :
1. Lancer le binaire packé sous x64dbg
2. Poser un hardware breakpoint sur esp (write) au tout début du stub
3. UPX sauvegarde esp → décompresse → restaure esp → jmp OEP
4. Le hardware BP se déclenche sur la restauration de esp
5. F8 quelques fois → on arrive sur l'OEP (jmp eax ou jmp [table])
```

---

## 9. Crackmes — Méthodologie et Exemples

### 9.1 Méthodologie générale

Un crackme est un programme conçu pour être "cracké" — il vérifie un mot de passe ou une clé de licence et le but est de trouver la bonne valeur ou de bypasser la vérification.

```
Méthodologie crackme en 6 étapes

1. RECONNAISSANCE STATIQUE RAPIDE
   ├── file binary          (type, architecture, stripped?)
   ├── strings binary       (messages, clés hardcodées, format attendu?)
   ├── checksec binary      (protections : PIE, NX, canary, RELRO)
   └── readelf -h binary    (sections, entry point)

2. ANALYSE STATIQUE APPROFONDIE (Ghidra)
   ├── Ouvrir dans Ghidra → analyser
   ├── Trouver main() et la fonction de validation
   ├── Comprendre le flux de contrôle (CFG)
   └── Renommer fonctions et variables

3. IDENTIFICATION DU TYPE DE VÉRIFICATION
   ├── Comparaison directe (strcmp, memcmp)
   ├── Hash (MD5, SHA, custom)
   ├── Chiffrement (XOR, rotation, base64)
   ├── Algorithme mathématique (CRC, checksum)
   └── VM custom (bytecode interprété)

4. STRATÉGIE DE RÉSOLUTION
   ├── Trouver la clé statiquement (reverse de l'algo)
   ├── Brute force (si espace de recherche petit)
   ├── Symbolic execution (angr)
   └── Patch du saut (bypass)

5. VALIDATION DYNAMIQUE
   ├── Confirmer avec gdb/x64dbg
   └── Tester le binaire patché

6. DOCUMENTATION
   └── Rédiger un writeup pour mémoriser la technique
```

### 9.2 Exemple 1 — Comparaison directe (strcmp)

```c
// Code source du crackme (pour comprendre)
#include <stdio.h>
#include <string.h>

int main(int argc, char *argv[]) {
    if (argc < 2) {
        printf("Usage: %s <password>\n", argv[0]);
        return 1;
    }
    
    char expected[] = "H0lb3rt0n!";
    
    if (strcmp(argv[1], expected) == 0) {
        printf("Correct! Flag: HTB{y0u_g0t_1t}\n");
    } else {
        printf("Wrong password!\n");
    }
    return 0;
}
```

**Analyse avec strings :**

```bash
$ strings crackme1
...
H0lb3rt0n!
Correct! Flag: HTB{y0u_g0t_1t}
Wrong password!
...
# → La clé est directement visible dans les strings !
```

**Analyse avec Ghidra :**

```c
// Pseudo-code Ghidra après analyse
int main(int argc, char **argv) {
    if (argc < 2) {
        printf("Usage: %s <password>\n", *argv);
        return 1;
    }
    int result = strcmp(argv[1], "H0lb3rt0n!");
    if (result == 0) {
        puts("Correct! Flag: HTB{y0u_g0t_1t}");
    } else {
        puts("Wrong password!");
    }
    return 0;
}
// → strcmp avec "H0lb3rt0n!" — clé évidente
```

### 9.3 Exemple 2 — Algorithme XOR

```c
// Vérification avec XOR
// L'input est XORé avec une clé, puis comparé au résultat attendu

char key[]      = {0x41, 0x42, 0x43, 0x44};          // "ABCD"
char expected[] = {0x18, 0x2e, 0x26, 0x68,            // résultat XOR attendu
                   0x01, 0x27, 0x27, 0x27};

void check(char *input) {
    char result[8];
    for (int i = 0; i < 8; i++) {
        result[i] = input[i] ^ key[i % 4];
    }
    if (memcmp(result, expected, 8) == 0) {
        puts("Correct!");
    } else {
        puts("Wrong!");
    }
}
```

**Reverse de l'algorithme XOR :**

```python
# XOR est sa propre inverse : si result = input XOR key
# alors input = result XOR key

key      = [0x41, 0x42, 0x43, 0x44]  # "ABCD"
expected = [0x18, 0x2e, 0x26, 0x68, 0x01, 0x27, 0x27, 0x27]

password = ""
for i in range(len(expected)):
    password += chr(expected[i] ^ key[i % 4])

print(f"Password: {password}")
# Password: YlieHole
```

### 9.4 Exemple 3 — Serial Key avec contraintes

```c
// Vérification d'une clé de licence au format XXXX-XXXX-XXXX
bool check_license(const char *key) {
    // Format : 4 groupes de 4 chiffres séparés par '-'
    if (strlen(key) != 19) return false;
    if (key[4] != '-' || key[9] != '-' || key[14] != '-') return false;
    
    // Contrainte 1 : somme du groupe 1 = 10
    int sum1 = 0;
    for (int i = 0; i < 4; i++) {
        if (!isdigit(key[i])) return false;
        sum1 += key[i] - '0';
    }
    if (sum1 != 10) return false;
    
    // Contrainte 2 : groupe 2 * 2 = groupe 3 (valeur numérique)
    int g2 = atoi_n(key + 5, 4);
    int g3 = atoi_n(key + 10, 4);
    if (g2 * 2 != g3) return false;
    
    // Contrainte 3 : groupe 4 = "1337"
    if (strncmp(key + 15, "1337", 4) != 0) return false;
    
    return true;
}
```

**Résolution des contraintes :**

```python
# Contrainte 1 : somme groupe 1 = 10 → exemple : 1234 (1+2+3+4=10)
# Contrainte 2 : g2 * 2 = g3 → exemple : g2=0100, g3=0200
# Contrainte 3 : groupe 4 = "1337"

license = "1234-0100-0200-1337"
print(f"License key: {license}")

# Vérification
parts = license.split('-')
assert sum(int(c) for c in parts[0]) == 10
assert int(parts[1]) * 2 == int(parts[2])
assert parts[3] == "1337"
print("Valide!")
```

---

## 10. Patching de Binaire

### 10.1 NOP slides et jump patches

**Principes fondamentaux du patching :**

```
Objectif : modifier le comportement d'un binaire sans son code source

Règle d'or du patching :
  → On ne peut PAS agrandir une section (les offsets seraient décalés)
  → On peut seulement REMPLACER des bytes par des bytes de même taille
  → NOP (0x90) permet de "gommer" des instructions sans casser l'alignement
```

**NOP slide — neutraliser une vérification :**

```nasm
; AVANT : saut vers FAIL si la vérification échoue
; Adresse  Hex       Mnémonique
; 0040115D 75 41     jne   004011A0   (jump to fail)
; 0040115F 8B 45 F8  mov   eax, [ebp-8]

; APRÈS : deux NOPs remplacent le JNE
; 0040115D 90        nop
; 0040115E 90        nop
; 0040115F 8B 45 F8  mov   eax, [ebp-8]

; La vérification est annulée — le programme continue toujours
```

**Jump patch — rediriger vers le succès :**

```nasm
; AVANT
; 0040115D 75 41     jne   004011A0   (jump to fail si ZF=0)

; APRÈS : forcer le saut vers SUCCESS (adresse 0x401162)
; On calcule l'offset : 0x401162 - 0x40115F = 3
; jmp court : EB xx = EB 03
; 0040115D EB 03     jmp   00401162   (toujours vers success)
; 0040115F 90        nop              (aligner)
```

### 10.2 Patching avec xxd et dd

```bash
# Trouver l'offset du byte à modifier
objdump -d -M intel binary | grep "jne" | head -5
# 40115d:  75 41    jne 4011a0

# Calculer l'offset dans le fichier (pas en mémoire)
# Adresse mémoire : 0x40115d
# ImageBase (readelf -h) : 0x400000
# Offset dans le fichier : 0x40115d - 0x400000 + offset_section_text
readelf -S binary | grep .text
# [14] .text  PROGBITS  0000000000401060  00001060  ...
# Offset fichier = 0x40115d - 0x401060 + 0x1060 = 0x115d

# Patch avec dd (remplacer les 2 bytes "75 41" par "90 90")
cp binary binary_patched
printf '\x90\x90' | dd of=binary_patched bs=1 seek=$((16#115d)) conv=notrunc

# Vérifier
xxd binary_patched | grep -A1 "1150:"
```

**Patching avec Python :**

```python
#!/usr/bin/env python3
# Patch un binaire : remplace JNE par NOP NOP

import sys

PATCH_OFFSET = 0x115d  # offset dans le fichier (calculé)
ORIGINAL = b'\x75\x41'  # JNE opcode + offset
PATCHED  = b'\x90\x90'  # NOP NOP

with open("crackme", "rb") as f:
    data = bytearray(f.read())

# Vérification que c'est bien ce qu'on attend
assert data[PATCH_OFFSET:PATCH_OFFSET+2] == ORIGINAL, \
    f"Byte inattendu : {data[PATCH_OFFSET:PATCH_OFFSET+2].hex()}"

# Application du patch
data[PATCH_OFFSET:PATCH_OFFSET+2] = PATCHED

with open("crackme_patched", "wb") as f:
    f.write(data)

print(f"Patch appliqué à l'offset 0x{PATCH_OFFSET:x}")
print("Binaire patché : crackme_patched")
```

### 10.3 Protections contre le patching

```
Protections courantes :

1. Checksum de l'exécutable lui-même
   → Le programme hash ses propres sections .text au démarrage
   → Tout patch invalide le checksum → crash ou fausse clé

2. Code signing (Windows)
   → Signature cryptographique du binaire
   → Vérification par l'OS avant exécution

3. Anti-tamper commercial
   → VMProtect, Themida — sections chiffrées déchiffrées au runtime
   → Vérifications périodiques du code en mémoire

Contournement du checksum :
   → Analyser le code de vérification avec Ghidra
   → Trouver la comparaison du checksum
   → NOP la comparaison OU corriger le checksum stocké
     (qui est lui-même dans le binaire, souvent en .rodata)
```

---

## 11. CTF RE — Méthodologie et Outils Complémentaires

### 11.1 Méthodologie CTF Reverse Engineering

```
Checklist CTF RE — à faire dans l'ordre

□ file binary
□ strings -n 8 binary | grep -i "flag\|htb\|ctf\|key\|pass"
□ checksec binary (ou security-check)
□ Taille du binaire (petit = simple, gros = obfusqué ou statique)
□ ltrace ./binary <<< "test"   (appels de bibliothèque)
□ strace ./binary <<< "test"   (appels système)
□ Ghidra : importer, analyser, chercher main()
□ IDA Free : CFG pour visualiser les branchements
□ Identifier le type de vérification (strcmp, memcmp, hash, algo)
□ Résoudre statiquement OU patcher OU angr
□ Valider et soumettre le flag
```

### 11.2 ltrace et strace

```bash
# ltrace : intercepte les appels aux bibliothèques partagées
ltrace ./crackme AAAA
# strcmp("AAAA", "SecretKey")  = -50  ← la bonne clé est visible !
# puts("Wrong password!")       = 17

# strace : intercepte les appels système kernel
strace ./crackme AAAA 2>&1 | head -30
# openat(AT_FDCWD, "/proc/self/maps", O_RDONLY)  ← vérifie les mappings (anti-debug ?)
# read(5, "AAAA\n", 128)                          ← lit l'input
# write(1, "Wrong\n", 6)                          ← affiche Wrong

# Filtrer par famille de syscalls
strace -e trace=read,write,open ./crackme AAAA

# ltrace avec -S pour voir aussi les syscalls
ltrace -S ./crackme AAAA
```

> [!tip] ltrace révèle souvent la solution immédiatement
> Si le crackme utilise `strcmp`, `memcmp`, `strncmp` ou même des fonctions custom qui appellent ces dernières, `ltrace` affichera les deux chaînes comparées — incluant la bonne clé. C'est la technique la plus rapide à essayer en CTF.

### 11.3 angr — Exécution Symbolique

angr est un framework Python d'analyse de programme qui utilise l'exécution symbolique pour explorer automatiquement tous les chemins d'exécution d'un binaire.

```bash
# Installation
pip install angr
```

**Principe de l'exécution symbolique :**

```
Exécution normale :
  Input = "AAAA" → compare avec "BBBB" → False

Exécution symbolique :
  Input = symbole S (valeur inconnue)
  Le moteur SMT (Z3) résout les contraintes pour que le résultat soit True
  → S = "BBBB" (solution trouvée automatiquement)
```

**Script angr de base pour un crackme :**

```python
#!/usr/bin/env python3
# Résoudre un crackme avec angr
import angr
import claripy

# Charger le binaire
proj = angr.Project('./crackme', auto_load_libs=False)

# Créer un symbole pour l'input (16 bytes)
# BVS = BitVectrSymbol — variable symbolique
input_chars = [claripy.BVS(f'c_{i}', 8) for i in range(16)]
password = claripy.Concat(*input_chars)

# Contrainte : les chars doivent être imprimables
# (optimisation — réduit l'espace de recherche)
constraints = []
for c in input_chars:
    constraints.append(c >= 0x20)  # ' '
    constraints.append(c <= 0x7e)  # '~'

# État initial : appeler main avec notre symbole
state = proj.factory.entry_state(
    args=['./crackme'],
    stdin=claripy.Concat(password, claripy.BVV(b'\n'))
)
for c in constraints:
    state.solver.add(c)

# Simulation
simgr = proj.factory.simulation_manager(state)

# Trouver l'adresse de "Correct!" et éviter "Wrong!"
CORRECT_ADDR = 0x401190  # adresse de la branche success (trouver avec Ghidra)
WRONG_ADDR   = 0x4011b0  # adresse de la branche fail

simgr.explore(find=CORRECT_ADDR, avoid=WRONG_ADDR)

if simgr.found:
    found_state = simgr.found[0]
    # Extraire la solution
    solution = found_state.solver.eval(password, cast_to=bytes)
    print(f"Solution: {solution.decode('latin-1', errors='replace')}")
else:
    print("Aucune solution trouvée")
```

**angr avec stdin :**

```python
#!/usr/bin/env python3
# Version plus robuste — cherche automatiquement les adresses
import angr

proj = angr.Project('./crackme', auto_load_libs=False)

# Chercher "Correct" dans les chaînes du binaire
correct_addr = None
for addr, obj in proj.loader.main_object.memory.backers():
    # Cherche la chaîne dans le binaire
    pass

# Méthode alternative : utiliser le stdout
state = proj.factory.entry_state()
simgr = proj.factory.simulation_manager(state)

# Explorer avec un timeout
simgr.run(until=lambda sm: len(sm.deadended) > 0)

# Parmi les états terminés, trouver ceux qui ont écrit "Correct"
for s in simgr.deadended:
    output = s.posix.dumps(1)  # stdout
    if b'Correct' in output or b'correct' in output:
        flag = s.posix.dumps(0)  # stdin = notre input
        print(f"Flag: {flag}")
        break
```

### 11.4 Radare2 — Alternative en ligne de commande

```bash
# Radare2 — analyse complète en ligne de commande
r2 ./crackme

# Commandes essentielles dans r2
aaa          # analyser complètement le binaire
afl          # lister toutes les fonctions
pdf @main    # print disassembly of function main
pdf @sym.validate_key
s 0x401156   # seek à une adresse
px 32        # print hex 32 bytes
pi 10        # print instructions (désassembler 10 instr)
VV           # visual mode — CFG graphique dans le terminal !
q            # quitter

# Afficher la CFG en ASCII dans le terminal
r2 -A ./crackme
[0x00401060]> VV @ main
```

### 11.5 Outils complémentaires et ressources

```
Analyse rapide :
  binwalk -E binary    → entropie section par section (détecte packing)
  entropy.py binary    → visualiser l'entropie graphiquement
  pescanner binary     → analyse PE (anomalies, imports suspects)
  die binary           → Detect-It-Easy (identifies packer/compiler)

Déchiffrement :
  cyberchef            → https://gchq.github.io/CyberChef/ (XOR, base64, etc.)
  xortool binary       → trouver la clé XOR et décrypter

Fuzzing d'input :
  radamsa              → générateur de cas de test (fuzzer de mutation)

Ressources CTF RE :
  crackmes.one         → crackmes classés par difficulté
  reversing.kr         → challenges de RE coréens
  Hack The Box RE      → https://www.hackthebox.com/

Communauté :
  r/ReverseEngineering → Reddit
  OpenRCE              → openrce.org
  Malware Traffic Analysis → malware-traffic-analysis.net
```

---

## 12. Exercices Pratiques

### Exercice 1 — Analyse statique de base

```
Niveau : Débutant
Outils : file, strings, readelf, objdump

Créer le binaire de test :
```

```c
// crackme_ex1.c
#include <stdio.h>
#include <string.h>

int main(int argc, char *argv[]) {
    if (argc < 2) {
        printf("Usage: %s <flag>\n", argv[0]);
        return 1;
    }
    
    // Clé encodée en XOR avec 0x42
    unsigned char encoded[] = {
        0x2a, 0x26, 0x20, 0x7e, 0x24, 0x22, 0x7b, 0x4b,
        0x27, 0x26, 0x2e, 0x26, 0x22, 0x24, 0x69
    };
    
    char decoded[16];
    for (int i = 0; i < 15; i++) {
        decoded[i] = encoded[i] ^ 0x42;
    }
    decoded[15] = '\0';
    
    if (strcmp(argv[1], decoded) == 0) {
        printf("Correct! You found the flag.\n");
    } else {
        printf("Wrong!\n");
    }
    return 0;
}
```

```bash
# Compilation et analyse
gcc -o crackme_ex1 crackme_ex1.c -s  # -s = strip les symboles

# Étape 1 : identification
file crackme_ex1

# Étape 2 : strings (indice : la clé XOR est peut-être visible ?)
strings crackme_ex1

# Étape 3 : désassembler et trouver la boucle XOR
objdump -d -M intel crackme_ex1 | grep -A 50 "<main>"

# Étape 4 : reverse de l'algo XOR en Python
python3 -c "
encoded = [0x2a, 0x26, 0x20, 0x7e, 0x24, 0x22, 0x7b, 0x4b,
           0x27, 0x26, 0x2e, 0x26, 0x22, 0x24, 0x69]
decoded = ''.join(chr(b ^ 0x42) for b in encoded)
print(f'Flag: {decoded}')
"
```

### Exercice 2 — GDB et Ghidra

```
Niveau : Intermédiaire
Objectif : Trouver le mot de passe sans voir le code source
```

```c
// crackme_ex2.c — COMPILER SANS VOIR CE CODE
#include <stdio.h>
#include <string.h>

int check(const char *input) {
    // Vérification multi-étapes
    if (strlen(input) != 8) return 0;
    
    int sum = 0;
    for (int i = 0; i < 8; i++) sum += input[i];
    if (sum != 0x310) return 0;  // sum doit valoir 784
    
    if (input[0] != 'H') return 0;
    if (input[7] != '!') return 0;
    
    return 1;
}

int main() {
    char buf[32];
    printf("Password: ");
    fgets(buf, 32, stdin);
    buf[strcspn(buf, "\n")] = 0;
    
    if (check(buf)) {
        printf("Access granted!\n");
    } else {
        printf("Access denied.\n");
    }
    return 0;
}
```

```
Tâches :
1. Importer dans Ghidra, analyser et renommer les fonctions
2. Identifier les contraintes (longueur, somme, premier/dernier char)
3. Écrire un script Python pour trouver un mot de passe valide
4. Vérifier avec GDB en inspectant les registres sur le cmp
```

**Solution script Python :**

```python
#!/usr/bin/env python3
# Trouver un mot de passe satisfaisant toutes les contraintes

# Contraintes identifiées dans Ghidra :
# - Longueur == 8
# - Somme des ASCII == 0x310 (784)
# - input[0] == 'H' (0x48)
# - input[7] == '!' (0x21)

# Calcul du reste : 784 - 0x48 - 0x21 = 784 - 72 - 33 = 679
# À distribuer sur 6 caractères (indices 1 à 6)
# 679 / 6 ≈ 113.2 → utiliser des chars ASCII autour de 113 ('q' = 113)

# Approche greedy
TARGET = 0x310  # 784
first = ord('H')   # 72
last  = ord('!')   # 33
remaining = TARGET - first - last  # 679

# Construire les 6 chars du milieu
middle_chars = []
r = remaining
for i in range(6):
    # Dernier char : prend le reste
    if i == 5:
        c = r
    else:
        c = min(r - (5-i) * 33, 126)  # ne pas dépasser 126 ('~')
        c = max(c, 33)                 # ne pas descendre sous 33 ('!')
    middle_chars.append(c)
    r -= c

password = chr(first) + ''.join(chr(c) for c in middle_chars) + chr(last)
print(f"Password: '{password}'")
print(f"Longueur : {len(password)}")
print(f"Somme : {sum(ord(c) for c in password)} (attendu : {TARGET})")
print(f"Premier : {password[0]} (attendu : H)")
print(f"Dernier : {password[7]} (attendu : !)")
```

### Exercice 3 — Challenge CTF complet

```
Niveau : Avancé
Scénario : Vous avez capturé un binaire qui vérifie un flag CTF.
           Le flag suit le format HTB{...}.
           Trouver le flag sans exécuter le binaire.

Workflow attendu :
1. file + strings → informations initiales
2. Ghidra → décompiler main() et find_flag()
3. Identifier l'algorithme (hint: rotation de bits)
4. Reverse en Python ou avec angr
5. Valider

Données pour l'exercice :
```

```python
# Script pour générer le binaire de l'exercice
# Algorithme : chaque char du flag est rotationné de 3 bits à gauche
# puis XOR avec son index

def rot_left(val, n, size=8):
    """Rotation de bits à gauche sur size bits"""
    n = n % size
    return ((val << n) | (val >> (size - n))) & ((1 << size) - 1)

flag = "HTB{r3v3rs3_m3_1f_y0u_c4n}"
encoded = []
for i, c in enumerate(flag):
    enc = rot_left(ord(c), 3) ^ i
    encoded.append(enc)

print("Encoded:", [hex(b) for b in encoded])

# Solution :
def rot_right(val, n, size=8):
    n = n % size
    return ((val >> n) | (val << (size - n))) & ((1 << size) - 1)

decoded = ""
for i, b in enumerate(encoded):
    decoded += chr(rot_right(b ^ i, 3))
print(f"Decoded: {decoded}")
```

---

## 13. Tableau Récapitulatif — Outils et Usages

| Outil | OS | Type | Usage principal | Commande clé |
|-------|----|----|-----------------|-------------|
| `file` | Linux/Win | Statique | Identifier le format | `file binary` |
| `strings` | Linux/Win | Statique | Extraire les chaînes | `strings -n 8 binary` |
| `readelf` | Linux | Statique | Anatomie ELF | `readelf -a binary` |
| `objdump` | Linux | Statique | Désassemblage rapide | `objdump -d -M intel binary` |
| `nm` | Linux | Statique | Symboles | `nm -n binary` |
| `ldd` | Linux | Statique | Dépendances | `ldd binary` (pas sur malware) |
| `ltrace` | Linux | Dynamique | Appels de lib | `ltrace ./binary <<< "test"` |
| `strace` | Linux | Dynamique | Syscalls | `strace ./binary` |
| `GDB` | Linux | Dynamique | Debug pas-à-pas | `gdb ./binary` |
| `pwndbg` | Linux | Dynamique | GDB amélioré CTF | `b main ; run` |
| `Ghidra` | Multi | Statique | Décompilation complète | GUI |
| `IDA Free` | Multi | Statique | CFG, FLIRT | GUI |
| `radare2` | Multi | Statique+Dyn | Tout-en-un CLI | `r2 -A binary` |
| `x64dbg` | Windows | Dynamique | Debug + patch Win | GUI |
| `angr` | Multi | Symbolique | Résolution auto | `simgr.explore()` |
| `binwalk` | Linux | Statique | Entropie, packing | `binwalk -E binary` |
| `upx` | Multi | Statique | Unpack UPX | `upx -d binary` |

---

## 14. Bonnes Pratiques et Pièges Courants

### 14.1 Erreurs à éviter

> [!warning] Ne jamais exécuter un binaire suspect directement sur votre machine
> Toujours analyser dans une VM isolée (VirtualBox, VMware) avec les réseaux coupés. Un crackme de CTF peut être malveillant. Utiliser des snapshots pour revenir à un état propre.

> [!warning] ldd sur des binaires inconnus
> `ldd` peut exécuter du code arbitraire. Utiliser `readelf -d binary | grep NEEDED` à la place.

> [!warning] Les outils de décompilation ne sont pas parfaits
> Ghidra et IDA produisent du pseudo-code, pas du vrai C. Les boucles, les types et les structures peuvent être incorrectement représentés. Toujours cross-référencer avec l'Assembly brut.

### 14.2 Bonnes pratiques RE

> [!tip] Commencer par les strings et ltrace
> En CTF, 30% des crackmes se résolvent en moins de 2 minutes avec `strings` + `ltrace`. Ne pas passer directement à Ghidra sans avoir fait l'analyse rapide.

> [!tip] Renommer immédiatement dans Ghidra
> La qualité de l'analyse dans Ghidra est proportionnelle à la qualité du renommage. Chaque fonction comprise → renommée immédiatement. Ne jamais laisser 10 `sub_XXXXXX` sans labels.

> [!tip] CFG avant de lire l'Assembly
> La vue CFG d'IDA ou Ghidra donne la structure logique en 5 secondes. Identifier les blocs "success" et "fail" avant de lire les instructions détaillées.

> [!tip] Documenter pendant l'analyse
> Prendre des notes sur les adresses importantes, les algorithmes identifiés, les strings trouvées. En CTF avec chrono, il est facile d'oublier ce qu'on avait trouvé 20 minutes plus tôt.

### 14.3 Ordre de priorité des techniques

```
Arbre de décision — Quel outil utiliser ?

                  ┌─────────────────┐
                  │ Nouveau binaire  │
                  └────────┬────────┘
                           │
                    file + strings
                    ltrace + strace
                           │
              ┌────────────┴────────────┐
              │                         │
         Flag visible?             Rien de visible
              │                         │
            DONE              ┌─────────┴─────────┐
                              │                   │
                        Binaire simple?      Binaire complexe?
                        (< 10 fonctions)     (> 50 fonctions)
                              │                   │
                          Ghidra              Ghidra + angr
                          + GDB               + Scripting
                              │                   │
                   Reverse algo manuellement  Exécution symbolique
                   + Python script            automatique
```

---

## 15. Glossaire

| Terme | Définition |
|-------|-----------|
| **Binary** | Fichier exécutable compilé — PE (Windows) ou ELF (Linux) |
| **CFG** | Control Flow Graph — représentation visuelle des blocs de code |
| **Decompiler** | Outil qui transforme l'Assembly en pseudo-code C |
| **Disassembler** | Outil qui transforme le bytecode machine en mnémoniques Assembly |
| **Entry Point** | Première instruction exécutée par l'OS (souvent `_start`, pas `main`) |
| **ELF** | Executable and Linkable Format — format binaire Linux |
| **FLIRT** | Fast Library Identification and Recognition Technology (IDA) |
| **GOT** | Global Offset Table — table des adresses de fonctions importées |
| **OEP** | Original Entry Point — entry point du binaire avant unpacking |
| **PE** | Portable Executable — format binaire Windows |
| **PIE** | Position Independent Executable — charge à des adresses ASLR |
| **PLT** | Procedure Linkage Table — stubs d'appel pour les imports dynamiques |
| **RVA** | Relative Virtual Address — offset depuis l'ImageBase (PE) |
| **Stripped** | Binaire sans symboles de debug (`.symtab` supprimée) |
| **Symbolic Execution** | Analyse qui explore tous les chemins d'exécution (angr, KLEE) |
| **Xref** | Cross-reference — liste de tous les endroits qui référencent un symbole |
