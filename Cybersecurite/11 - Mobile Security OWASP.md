# Sécurité Mobile — OWASP Mobile Top 10

La sécurité des applications mobiles est devenue un enjeu critique : avec plus de 6 milliards de smartphones actifs en 2024, ces appareils concentrent des données bancaires, médicales et personnelles d'une valeur considérable. Ce cours couvre l'intégralité du cycle d'attaque et de défense des applications Android et iOS, depuis l'architecture interne jusqu'aux techniques de reverse engineering avancées. À l'issue de ce module, tu seras capable d'auditer une application mobile, d'identifier les vulnérabilités du Top 10 OWASP et de mettre en place les contre-mesures appropriées.

> [!warning] Cadre légal et éthique OBLIGATOIRE
> Toutes les techniques présentées dans ce cours sont à utiliser **exclusivement** sur des applications que tu possèdes, des environnements de lab dédiés (émulateurs, apps CTF), ou dans le cadre d'un **Bug Bounty Program** avec autorisation écrite explicite.
>
> En France, l'article **323-1 du Code pénal** punit l'accès frauduleux à un système informatique de 2 ans d'emprisonnement et 60 000 € d'amende. L'article **323-3** punit la modification de données de 5 ans et 150 000 €.
>
> **Ne jamais** utiliser ces outils sur des applications en production sans autorisation. Holberton School décline toute responsabilité en cas d'usage malveillant.

---

## 1. Architecture Mobile — Comprendre avant d'attaquer

> [!info] Pourquoi l'architecture ?
> Un pentesteur qui ne comprend pas comment fonctionne la plateforme cible va rater 80 % des vulnérabilités. L'architecture définit la surface d'attaque.

### 1.1 Android — de l'APK au processus

#### Le stack Android

```
┌─────────────────────────────────────────┐
│          Applications (APK)             │  ← ta cible
├─────────────────────────────────────────┤
│        Application Framework            │  ActivityManager, PackageManager...
├─────────────────────────────────────────┤
│    Libraries C/C++  │  Android Runtime  │  ART, Bionic libc
├─────────────────────────────────────────┤
│           Linux Kernel                  │  Drivers, SELinux, seccomp
└─────────────────────────────────────────┘
```

#### Le format APK

Un APK est simplement une **archive ZIP** renommée. Tu peux l'ouvrir avec `unzip` :

```bash
# Lister le contenu d'un APK sans décompresser
unzip -l target.apk

# Résultat typique :
#  classes.dex        ← bytecode Dalvik/ART (le code Java compilé)
#  classes2.dex       ← multidex si l'app est grande
#  AndroidManifest.xml ← binaire, pas lisible directement
#  resources.arsc     ← ressources compilées
#  res/               ← images, layouts XML
#  assets/            ← fichiers bruts (bases de données, configs)
#  lib/               ← bibliothèques natives .so (armeabi-v7a, arm64-v8a, x86)
#  META-INF/          ← signature APK (CERT.RSA, CERT.SF, MANIFEST.MF)
```

#### Dalvik vs ART

| Aspect | Dalvik (avant Android 5.0) | ART (Android 5.0+) |
|--------|---------------------------|---------------------|
| Compilation | JIT (Just-In-Time) au runtime | AOT (Ahead-Of-Time) à l'installation |
| Fichier | `.dex` (Dalvik Executable) | `.oat` (Optimized AT) |
| Performance | Plus lente au démarrage | Meilleure performance globale |
| Décompilation | Directe depuis .dex | Depuis .dex embarqué dans l'APK |
| Impact pentest | Même surface d'attaque | Même surface d'attaque |

> [!info] En pratique pour le pentest
> La distinction Dalvik/ART change peu pour nous : les APK contiennent toujours les fichiers `.dex` originaux. C'est ces `.dex` que jadx et apktool décompilent.

#### Le fichier AndroidManifest.xml — la carte de l'app

C'est le **fichier le plus important** pour un auditeur. Il déclare tout :

```xml
<!-- Exemple de manifest annoté -->
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.vulnerableapp"
    android:versionCode="1">

    <!-- Permissions demandées — SURFACE D'ATTAQUE #1 -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.READ_CONTACTS" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.CAMERA" />

    <application
        android:allowBackup="true"          <!-- ← DANGER : backup ADB possible -->
        android:debuggable="true"           <!-- ← DANGER : app déboguable en prod -->
        android:networkSecurityConfig="@xml/network_security_config"
        android:usesCleartextTraffic="true"> <!-- ← DANGER : HTTP autorisé -->

        <!-- Activity exportée sans protection — SURFACE D'ATTAQUE #2 -->
        <activity
            android:name=".LoginActivity"
            android:exported="true">  <!-- accessible par d'autres apps -->
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <!-- Provider exporté — SURFACE D'ATTAQUE #3 -->
        <provider
            android:name=".UserContentProvider"
            android:authorities="com.example.vulnerableapp.provider"
            android:exported="true"     <!-- ← toute app peut lire les données -->
            android:grantUriPermissions="true" />

        <!-- Service exporté — SURFACE D'ATTAQUE #4 -->
        <service
            android:name=".DataSyncService"
            android:exported="true" />  <!-- ← démarrable par n'importe quelle app -->

        <!-- BroadcastReceiver — SURFACE D'ATTAQUE #5 -->
        <receiver
            android:name=".TokenReceiver"
            android:exported="true">
            <intent-filter>
                <action android:name="com.example.ACTION_TOKEN_READY" />
            </intent-filter>
        </receiver>

    </application>
</manifest>
```

#### Le modèle de permissions Android

```
Permissions normales (auto-accordées à l'install)
├── INTERNET
├── ACCESS_NETWORK_STATE
└── VIBRATE

Permissions dangereuses (demandent confirmation utilisateur)
├── READ_CONTACTS / WRITE_CONTACTS
├── ACCESS_FINE_LOCATION
├── READ_EXTERNAL_STORAGE / WRITE_EXTERNAL_STORAGE
├── CAMERA
├── READ_PHONE_STATE
└── READ_SMS

Permissions signature (réservées aux apps système)
├── INSTALL_PACKAGES
└── DELETE_PACKAGES

Permissions special access (Settings > Apps > Special access)
├── SYSTEM_ALERT_WINDOW (overlay)
└── WRITE_SETTINGS
```

#### IPC — Inter-Process Communication Android

Android repose sur **Binder**, un mécanisme IPC kernel. Les composants communiquent via des **Intents** :

```java
// Intent explicite — cible une app/composant précis
Intent explicitIntent = new Intent(this, TargetActivity.class);
explicitIntent.putExtra("token", "secret_value");
startActivity(explicitIntent);

// Intent implicite — broadcast à toutes les apps qui écoutent
// VULNÉRABILITÉ : une app malveillante peut intercepter !
Intent implicitIntent = new Intent("com.example.ACTION_TOKEN_READY");
implicitIntent.putExtra("auth_token", userToken);  // ← token exposé à toutes les apps
sendBroadcast(implicitIntent);

// Intent implicite sécurisé (permission protégée)
sendBroadcast(implicitIntent, "com.example.RECEIVE_TOKEN");
```

### 1.2 iOS — Sandbox, Entitlements et IPA

#### L'architecture iOS

```
┌──────────────────────────────────────────┐
│            Applications (.ipa)           │  ← ta cible
├──────────────────────────────────────────┤
│              Cocoa Touch                 │  UIKit, Foundation, CoreData
├──────────────────────────────────────────┤
│                 Media                    │  CoreAudio, CoreGraphics
├──────────────────────────────────────────┤
│             Core Services               │  Security, CoreLocation, CloudKit
├──────────────────────────────────────────┤
│               Core OS                   │  Darwin (XNU Kernel), libSystem
└──────────────────────────────────────────┘
```

#### Le format IPA

Comme l'APK, un IPA est une archive ZIP :

```bash
# Décompresser un IPA
unzip target.ipa -d extracted_ipa

# Structure :
# Payload/
#   AppName.app/
#     AppName          ← binaire Mach-O (ARM64)
#     Info.plist       ← équivalent du AndroidManifest
#     embedded.mobileprovision ← profil de provisioning
#     _CodeSignature/  ← signature de tous les fichiers
#     Frameworks/      ← frameworks embarqués
#     Assets.car       ← ressources compilées
```

#### Entitlements — les permissions iOS

Les entitlements sont des paires clé/valeur signées qui définissent ce qu'une app peut faire :

```xml
<!-- Extrait d'entitlements typiques -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN">
<plist version="1.0">
<dict>
    <!-- Accès au keychain partagé entre apps du même développeur -->
    <key>keychain-access-groups</key>
    <array>
        <string>com.company.shared</string>
    </array>

    <!-- App Groups — espace de données partagé -->
    <key>com.apple.security.application-groups</key>
    <array>
        <string>group.com.company.app</string>
    </array>

    <!-- Permettre l'exécution de code JIT (rare, très puissant) -->
    <key>dynamic-codesigning</key>
    <true/>

    <!-- Push notifications -->
    <key>aps-environment</key>
    <string>production</string>
</dict>
</plist>
```

#### La sandbox iOS

Chaque app iOS vit dans sa propre sandbox. Structure du container :

```
/var/mobile/Containers/Data/Application/<UUID>/
├── Documents/      ← données utilisateur, sauvegardées par iCloud
├── Library/
│   ├── Preferences/  ← équivalent SharedPreferences (NSUserDefaults)
│   ├── Caches/       ← données temporaires, pas sauvegardées
│   └── Application Support/  ← données persistantes app
├── tmp/            ← temporaire, vidé par iOS
└── SystemData/     ← données système
```

> [!tip] Pour les CTF iOS
> Sur un device jailbreaké (Checkra1n, Palera1n), tu peux accéder directement à ces répertoires via SSH. Cherche des tokens, des bases SQLite, des fichiers de config non chiffrés dans `Documents/`, `Library/Application Support/` et `Library/Caches/`.

---

## 2. OWASP Mobile Top 10 — 2024 Edition

> [!info] Qu'est-ce que l'OWASP Mobile Top 10 ?
> L'OWASP (Open Web Application Security Project) publie une liste des 10 risques les plus critiques pour les applications mobiles. La version 2024 a été significativement remaniée par rapport à 2016 pour refléter les menaces actuelles. Chaque vulnérabilité est documentée avec un score de risque basé sur la probabilité × l'impact.

### 2.1 M1 — Improper Credential Usage (Mauvaise gestion des identifiants)

**Définition** : Stocker, transmettre ou utiliser des identifiants (mots de passe, tokens, clés API) de façon non sécurisée dans le code ou les ressources de l'application.

#### Vecteurs d'attaque courants

```java
// ❌ VULNÉRABLE — Hardcoded credentials dans le code
public class ApiClient {
    private static final String API_KEY = "sk-prod-xK9mN2pQ8rL5wJ3vH7";
    private static final String DB_PASSWORD = "MySuperSecretPassword123!";
    private static final String AWS_SECRET = "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY";

    public void callApi() {
        // Ce code va dans le .dex → extractable par jadx en 30 secondes
        Request request = new Request.Builder()
            .url("https://api.example.com/data")
            .header("Authorization", "Bearer " + API_KEY)
            .build();
    }
}
```

```bash
# Extraction via jadx — trouver des credentials hardcodés
jadx -d output_dir/ target.apk
grep -r "password\|secret\|api_key\|token\|private" output_dir/sources/ -i
grep -r "sk-\|Bearer \|Basic \|AWS" output_dir/sources/
```

```bash
# Extraction depuis les ressources (strings.xml, config.xml)
unzip target.apk -d extracted/
strings extracted/resources.arsc | grep -i "key\|secret\|token\|password"
```

#### Dans les logs

```java
// ❌ VULNÉRABLE — Token dans les logs (accessible via adb logcat)
Log.d("Auth", "User token: " + authToken);
Log.i("Payment", "Processing card: " + cardNumber);
```

```bash
# Récupérer les logs en temps réel
adb logcat | grep -i "token\|password\|secret\|key\|credential"

# Logs historiques
adb logcat -d | grep "com.target.app"
```

#### Contre-mesures

```java
// ✅ SÉCURISÉ — Utiliser le Android Keystore
KeyStore keyStore = KeyStore.getInstance("AndroidKeyStore");
keyStore.load(null);

// Générer une clé dans le Keystore matériel
KeyGenerator keyGenerator = KeyGenerator.getInstance(
    KeyProperties.KEY_ALGORITHM_AES, "AndroidKeyStore");
keyGenerator.init(new KeyGenParameterSpec.Builder(
    "MySecretKey",
    KeyProperties.PURPOSE_ENCRYPT | KeyProperties.PURPOSE_DECRYPT)
    .setBlockModes(KeyProperties.BLOCK_MODE_GCM)
    .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_NONE)
    .setUserAuthenticationRequired(true)  // ← requiert biométrie/PIN
    .build());
SecretKey secretKey = keyGenerator.generateKey();
```

### 2.2 M2 — Inadequate Supply Chain Security

**Définition** : Utilisation de bibliothèques tierces, SDKs publicitaires, ou composants open source contenant des vulnérabilités, du malware, ou des backdoors.

#### Analyse des dépendances

```bash
# Inspecter le build.gradle d'une app décompilée
cat extracted/apktool_output/smali/... # via apktool
# Ou chercher les références de libs dans le code décompilé
grep -r "import com\.\|import io\.\|import org\." jadx_output/sources/ | \
  awk -F'import ' '{print $2}' | sort -u

# Vérifier les versions connues vulnérables
# Exemple : Log4Shell dans une app Android utilisant Log4j
grep -r "log4j\|log4core" jadx_output/ -i
```

```python
#!/usr/bin/env python3
# Script simple pour extraire les noms de librairies des .so embarqués
import subprocess
import zipfile
import os

def analyze_native_libs(apk_path):
    with zipfile.ZipFile(apk_path, 'r') as apk:
        libs = [f for f in apk.namelist() if f.endswith('.so')]
        for lib in libs:
            print(f"[+] Lib trouvée: {lib}")
            # Extraire et analyser avec strings
            apk.extract(lib, '/tmp/apk_analysis/')
            result = subprocess.run(
                ['strings', f'/tmp/apk_analysis/{lib}'],
                capture_output=True, text=True
            )
            # Chercher des URLs, clés, noms de fonctions suspects
            for line in result.stdout.split('\n'):
                if any(k in line.lower() for k in ['http', 'api', 'key', 'secret']):
                    print(f"  └─ Suspect: {line}")

analyze_native_libs('target.apk')
```

### 2.3 M3 — Insecure Authentication/Authorization

**Définition** : Mécanismes d'authentification côté client facilement bypassables, tokens JWT mal validés, autorisation basée sur l'identité de l'app plutôt que de l'utilisateur.

#### Bypass d'authentification côté client

```java
// ❌ VULNÉRABLE — Validation d'authentification côté client
public class LoginActivity extends AppCompatActivity {
    public void checkLogin(String username, String password) {
        // La vérification se fait LOCALEMENT — trivial à bypasser avec Frida
        if (username.equals("admin") && password.equals("1234")) {
            isAuthenticated = true;
            startMainActivity();
        } else {
            showError("Invalid credentials");
        }
    }
}
```

```javascript
// Bypass avec Frida — hook sur la méthode checkLogin
Java.perform(function() {
    var LoginActivity = Java.use("com.target.app.LoginActivity");

    LoginActivity.checkLogin.overload('java.lang.String', 'java.lang.String')
        .implementation = function(username, password) {
            console.log("[*] checkLogin intercepté !");
            console.log("[*] Username: " + username);
            console.log("[*] Password: " + password);

            // Forcer isAuthenticated = true peu importe les credentials
            this.isAuthenticated.value = true;
            this.startMainActivity();
            return; // ne pas appeler la vraie méthode
        };
});
```

#### JWT mal implémenté

```bash
# Décoder un JWT sans vérification de signature
echo "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoiMTIzIiwicm9sZSI6InVzZXIiLCJleHAiOjE3MDAwMDAwMDB9.xxxxx" | \
  cut -d'.' -f2 | base64 -d 2>/dev/null | python3 -m json.tool

# Résultat :
# {
#   "user_id": "123",
#   "role": "user",
#   "exp": 1700000000
# }

# Forger un token avec "none" algorithm (si le serveur l'accepte !)
# Header: {"alg":"none","typ":"JWT"}
# Payload: {"user_id":"1","role":"admin","exp":9999999999}
echo -n '{"alg":"none","typ":"JWT"}' | base64 | tr '+/' '-_' | tr -d '='
echo -n '{"user_id":"1","role":"admin","exp":9999999999}' | base64 | tr '+/' '-_' | tr -d '='
# Token forgé : <header>.<payload>. (signature vide)
```

### 2.4 M4 — Insufficient Input/Output Validation

**Définition** : Absence de validation des données provenant de sources non fiables (intents, deep links, QR codes), menant à des injections SQL, XSS dans les WebViews, ou exécution de commandes.

#### SQL Injection sur SQLite local

```java
// ❌ VULNÉRABLE — Concaténation directe dans une requête SQLite
public Cursor searchUser(String username) {
    SQLiteDatabase db = this.getReadableDatabase();
    // Si username = "' OR '1'='1" → retourne tous les utilisateurs !
    String query = "SELECT * FROM users WHERE username = '" + username + "'";
    return db.rawQuery(query, null);
}
```

```java
// ✅ SÉCURISÉ — Paramètres préparés
public Cursor searchUser(String username) {
    SQLiteDatabase db = this.getReadableDatabase();
    String query = "SELECT * FROM users WHERE username = ?";
    return db.rawQuery(query, new String[]{username});
}
```

#### XSS dans une WebView

```java
// ❌ VULNÉRABLE — WebView avec JavaScript activé chargeant du contenu externe
WebView webView = findViewById(R.id.webview);
webView.getSettings().setJavaScriptEnabled(true);
webView.addJavascriptInterface(new WebAppInterface(this), "Android");
// Si l'URL est contrôlée par l'attaquant → XSS → accès aux méthodes Android
webView.loadUrl(getIntent().getStringExtra("url"));
```

```bash
# Tester un deep link malveillant
adb shell am start -W -a android.intent.action.VIEW \
  -d "myapp://webview?url=javascript:Android.readFile('/data/data/com.target.app/files/secret.txt')"
```

#### Path Traversal via Content Provider

```bash
# Tester un Content Provider vulnérable au path traversal
adb shell content query \
  --uri "content://com.target.app.provider/../../../data/data/com.target.app/databases/users.db"
```

### 2.5 M5 — Insecure Communication

**Définition** : Transmission de données sensibles en clair (HTTP), validation incorrecte des certificats TLS, SSL Pinning absent ou contournable.

#### Détecter le trafic non chiffré

```bash
# Sur l'émulateur — vérifier network_security_config.xml
cat extracted/apktool_output/res/xml/network_security_config.xml
```

```xml
<!-- ❌ Configuration dangereuse qui autorise HTTP et certificats non fiables -->
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config cleartextTrafficPermitted="true">
        <trust-anchors>
            <!-- Fait confiance à TOUS les certificats utilisateur — ← PROXY BURP POSSIBLE -->
            <certificates src="user" />
            <certificates src="system" />
        </trust-anchors>
    </base-config>
</network-security-config>
```

```xml
<!-- ✅ Configuration sécurisée — certificats système uniquement, pas de HTTP -->
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config cleartextTrafficPermitted="false">
        <trust-anchors>
            <certificates src="system" />
        </trust-anchors>
    </base-config>
    <!-- Autoriser HTTP uniquement sur le domaine de debug local -->
    <domain-config cleartextTrafficPermitted="true">
        <domain includeSubdomains="false">localhost</domain>
    </domain-config>
</network-security-config>
```

### 2.6 M6 — Inadequate Privacy Controls

**Définition** : Collecte excessive de données personnelles, partage avec des SDKs tiers non déclarés (analytics, publicité), absence de consentement.

```bash
# Chercher les SDKs analytics/tracking embarqués
grep -r "Firebase\|Amplitude\|Mixpanel\|Adjust\|AppsFlyer\|Segment" \
  jadx_output/sources/ -l

# Chercher la collecte d'identifiants matériels (souvent illégal RGPD)
grep -r "ANDROID_ID\|getImei\|getDeviceId\|getMacAddress\|getSerialNumber" \
  jadx_output/sources/
```

### 2.7 M7 — Insufficient Binary Protections

**Définition** : Absence d'obfuscation, détection de root/émulateur facilement contournable, code facilement décompilable sans contre-mesures.

#### Détecter la détection d'émulateur

```java
// Code de détection d'émulateur typique (chercher dans jadx)
public static boolean isEmulator() {
    return Build.FINGERPRINT.startsWith("generic")
        || Build.FINGERPRINT.startsWith("unknown")
        || Build.MODEL.contains("google_sdk")
        || Build.MODEL.contains("Emulator")
        || Build.MODEL.contains("Android SDK built for x86")
        || Build.MANUFACTURER.contains("Genymotion")
        || (Build.BRAND.startsWith("generic") && Build.DEVICE.startsWith("generic"))
        || "google_sdk".equals(Build.PRODUCT);
}
```

```javascript
// Bypass de la détection d'émulateur avec Frida
Java.perform(function() {
    var Build = Java.use("android.os.Build");

    // Remplacer les valeurs Build par des valeurs d'un vrai téléphone
    Build.FINGERPRINT.value = "samsung/dreamltexx/dreamlte:9/PPR1.180610.011/G950FXXS5DSH2:user/release-keys";
    Build.MODEL.value = "SM-G950F";
    Build.MANUFACTURER.value = "samsung";
    Build.BRAND.value = "samsung";
    Build.DEVICE.value = "dreamlte";
    Build.PRODUCT.value = "dreamltexx";

    console.log("[*] Build properties spoofées !");
});
```

### 2.8 M8 — Security Misconfiguration

**Définition** : Composants Android exportés sans protection, mode debug actif en production, permissions inutilement larges, backups activés.

```bash
# Détecter les misconfigurations avec apktool + analyse du manifest
apktool d target.apk -o decompiled/

python3 << 'EOF'
import xml.etree.ElementTree as ET

tree = ET.parse('decompiled/AndroidManifest.xml')
root = tree.getroot()

ns = {'android': 'http://schemas.android.com/apk/res/android'}

# Vérifier debuggable
app = root.find('application')
debuggable = app.get('{http://schemas.android.com/apk/res/android}debuggable')
backup = app.get('{http://schemas.android.com/apk/res/android}allowBackup')
cleartext = app.get('{http://schemas.android.com/apk/res/android}usesCleartextTraffic')

print(f"[{'DANGER' if debuggable == 'true' else 'OK'}] debuggable: {debuggable}")
print(f"[{'WARN' if backup == 'true' else 'OK'}] allowBackup: {backup}")
print(f"[{'DANGER' if cleartext == 'true' else 'OK'}] cleartextTraffic: {cleartext}")

# Trouver les composants exportés
for tag in ['activity', 'service', 'receiver', 'provider']:
    for comp in root.iter(tag):
        exported = comp.get('{http://schemas.android.com/apk/res/android}exported')
        name = comp.get('{http://schemas.android.com/apk/res/android}name')
        if exported == 'true':
            print(f"[EXPORTED] {tag}: {name}")
EOF
```

### 2.9 M9 — Insecure Data Storage

**Définition** : Données sensibles stockées en clair sur le filesystem, dans SharedPreferences non chiffrées, bases de données sans chiffrement, ou sur stockage externe.

> Voir la section 7 pour un traitement détaillé de ce sujet.

### 2.10 M10 — Insufficient Cryptography

**Définition** : Algorithmes cryptographiques obsolètes (MD5, SHA1, DES, RC4), clés trop courtes, IV réutilisé, implémentations custom de la cryptographie.

```bash
# Détecter l'usage de crypto faible
grep -r "MD5\|SHA1\|DES\|RC4\|ECB\|AES/ECB" jadx_output/sources/ -i

# MD5 pour les mots de passe — trivial à casser
grep -r "MessageDigest.getInstance(\"MD5\")" jadx_output/sources/
```

```java
// ❌ VULNÉRABLE — AES en mode ECB (révèle les patterns)
Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");

// ❌ VULNÉRABLE — IV constant (rend le chiffrement déterministe)
byte[] iv = "1234567890123456".getBytes();
IvParameterSpec ivSpec = new IvParameterSpec(iv);
Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
cipher.init(Cipher.ENCRYPT_MODE, secretKey, ivSpec);

// ✅ SÉCURISÉ — AES-GCM avec IV aléatoire
byte[] iv = new byte[12]; // GCM recommande 12 bytes
SecureRandom random = new SecureRandom();
random.nextBytes(iv);
GCMParameterSpec gcmSpec = new GCMParameterSpec(128, iv);
Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
cipher.init(Cipher.ENCRYPT_MODE, secretKey, gcmSpec);
```

---

## 3. Setup des Outils

### 3.1 ADB — Android Debug Bridge

ADB est le couteau suisse de l'audit Android. Il permet de communiquer avec un émulateur ou un device physique.

```bash
# Installation
sudo apt install adb           # Linux
brew install android-platform-tools  # macOS

# Vérifier les appareils connectés
adb devices
# List of devices attached
# emulator-5554   device
# R5CN703XXXX     device  ← device physique

# Si plusieurs appareils — cibler spécifiquement
adb -s emulator-5554 shell

# Commandes de base
adb shell          # shell interactif sur l'appareil
adb push local.txt /sdcard/  # envoyer un fichier
adb pull /sdcard/data.txt .  # récupérer un fichier
adb logcat         # afficher les logs en temps réel
adb install target.apk  # installer une APK
adb uninstall com.target.app  # désinstaller

# Port forwarding — pour Burp Suite
adb reverse tcp:8080 tcp:8080  # l'appareil accède à Burp sur le PC

# Dump d'une app (backup)
adb backup -f backup.ab -apk com.target.app
# Convertir le backup en tar lisible
dd if=backup.ab bs=1 skip=24 | python3 -c "import zlib,sys; sys.stdout.buffer.write(zlib.decompress(sys.stdin.buffer.read()))" | tar xf -
```

### 3.2 Android Studio Emulator

```bash
# Créer un AVD (Android Virtual Device) via la ligne de commande
# Lister les images système disponibles
sdkmanager --list | grep "system-images"

# Télécharger une image API 30 (Android 11) sans Google Play (plus facile à rooter)
sdkmanager "system-images;android-30;google_apis;x86_64"

# Créer l'AVD
avdmanager create avd \
  --name "PentestAPI30" \
  --package "system-images;android-30;google_apis;x86_64" \
  --device "pixel_4"

# Démarrer l'émulateur avec accès root writable
emulator -avd PentestAPI30 -writable-system &

# Root de l'émulateur
adb root
adb remount  # remonte /system en lecture-écriture

# Installer le certificat Burp dans le système (Android < 10)
adb push burp_cert.der /sdcard/
adb shell "mv /sdcard/burp_cert.der /system/etc/security/cacerts/9a5ba575.0"
adb shell "chmod 644 /system/etc/security/cacerts/9a5ba575.0"
adb reboot
```

### 3.3 Genymotion

Genymotion est un émulateur Android commercial (gratuit pour usage personnel) avec de meilleures performances.

```bash
# Après installation, créer un device via l'interface
# Puis se connecter en ADB
adb connect 192.168.56.101:5555

# Genymotion inclut ARM Translation pour les apps ARM sur x86
# Installer OpenGApps pour les services Google (optionnel pour pentest)

# Script d'initialisation pentest pour Genymotion
cat > setup_genymotion_pentest.sh << 'EOF'
#!/bin/bash
echo "[*] Setup Genymotion pour pentest..."

# Root déjà actif sur Genymotion
adb shell "su -c 'id'"

# Installer Frida server
FRIDA_VERSION=$(frida --version)
ARCH=$(adb shell getprop ro.product.cpu.abi | tr -d '\r')
echo "[*] Arch: $ARCH, Frida: $FRIDA_VERSION"

wget "https://github.com/frida/frida/releases/download/${FRIDA_VERSION}/frida-server-${FRIDA_VERSION}-android-${ARCH}.xz"
unxz frida-server-*.xz
adb push frida-server /data/local/tmp/frida-server
adb shell "chmod 755 /data/local/tmp/frida-server"
adb shell "su -c '/data/local/tmp/frida-server &'"
echo "[*] Frida server démarré !"

# Installer certificat Burp
CERT_PATH="$HOME/.burp_cert.der"
if [ -f "$CERT_PATH" ]; then
    CERT_HASH=$(openssl x509 -inform DER -subject_hash_old -in "$CERT_PATH" | head -1)
    adb push "$CERT_PATH" "/sdcard/${CERT_HASH}.0"
    adb shell "su -c 'cp /sdcard/${CERT_HASH}.0 /system/etc/security/cacerts/'"
    adb shell "su -c 'chmod 644 /system/etc/security/cacerts/${CERT_HASH}.0'"
    echo "[*] Certificat Burp installé : ${CERT_HASH}.0"
fi
EOF
chmod +x setup_genymotion_pentest.sh
```

---

## 4. Analyse Statique Mobile

> [!info] Analyse statique vs dynamique
> **Statique** : analyser l'application sans l'exécuter. Rapide, pas besoin d'un device. Limité par l'obfuscation.
> **Dynamique** : analyser l'application pendant son exécution. Contourne l'obfuscation, voit l'état réel. Nécessite un device/émulateur.

### 4.1 apktool — Décompiler un APK en Smali

Apktool décompile les fichiers `.dex` en **Smali**, un langage assembleur pour la VM Dalvik.

```bash
# Installation
wget https://raw.githubusercontent.com/iBotPeaches/Apktool/master/scripts/linux/apktool
wget https://bitbucket.org/iBotPeaches/apktool/downloads/apktool_2.9.3.jar
chmod +x apktool
sudo mv apktool /usr/local/bin/
sudo mv apktool_2.9.3.jar /usr/local/bin/apktool.jar

# Décompiler
apktool d target.apk -o decompiled_target/

# Structure de sortie :
# decompiled_target/
# ├── AndroidManifest.xml  ← LISIBLE maintenant !
# ├── apktool.yml          ← métadonnées de décompilation
# ├── res/                 ← ressources XML lisibles
# ├── smali/               ← bytecode Dalvik en Smali
# │   └── com/target/app/
# │       ├── MainActivity.smali
# │       └── LoginActivity.smali
# └── original/
#     └── META-INF/        ← signature originale

# Lire du code Smali (exemple)
cat decompiled_target/smali/com/target/app/LoginActivity.smali
```

```smali
# Extrait de code Smali — connexion à une DB
.method public checkPassword(Ljava/lang/String;)Z
    .locals 3

    # Équivalent Java : String query = "SELECT * FROM users WHERE pass='" + password + "'"
    const-string v0, "SELECT * FROM users WHERE pass='"
    invoke-virtual {v0, p1}, Ljava/lang/String;->concat(Ljava/lang/String;)Ljava/lang/String;
    move-result-object v0

    # Vulnérabilité : concaténation directe = SQL injection !
    const-string v1, "'"
    invoke-virtual {v0, v1}, Ljava/lang/String;->concat(Ljava/lang/String;)Ljava/lang/String;
```

```bash
# Recompiler et signer (pour patcher l'APK)
apktool b decompiled_target/ -o target_patched.apk

# Signer l'APK recompilé (obligatoire pour l'installer)
keytool -genkey -v -keystore pentest.keystore -alias pentest \
  -keyalg RSA -keysize 2048 -validity 365

jarsigner -verbose -sigalg SHA256withRSA -digestalg SHA-256 \
  -keystore pentest.keystore target_patched.apk pentest

# Zipalign (optimisation obligatoire)
zipalign -v 4 target_patched.apk target_final.apk

# Installer
adb install target_final.apk
```

### 4.2 jadx — Décompilation en Java lisible

jadx produit du code Java (ou Kotlin) lisible, bien plus utile que le Smali.

```bash
# Installation
wget https://github.com/skylot/jadx/releases/download/v1.4.7/jadx-1.4.7.zip
unzip jadx-1.4.7.zip -d jadx/
sudo ln -s $(pwd)/jadx/bin/jadx /usr/local/bin/jadx
sudo ln -s $(pwd)/jadx/bin/jadx-gui /usr/local/bin/jadx-gui

# Décompiler en ligne de commande
jadx -d jadx_output/ target.apk

# Interface graphique (recommandée pour l'audit)
jadx-gui target.apk &
```

```bash
# Workflow d'audit avec jadx — recherche systématique

# 1. Credentials hardcodés
grep -r "password\|passwd\|secret\|api_key\|apikey\|token\|auth" \
  jadx_output/sources/ -i --include="*.java" | grep -v "//.*"

# 2. URLs et endpoints
grep -r "https\?://" jadx_output/sources/ -h | \
  grep -oP 'https?://[^\s"]+' | sort -u

# 3. Fichiers SQLite et requêtes
grep -r "SQLiteDatabase\|rawQuery\|execSQL\|openDatabase" \
  jadx_output/sources/ -l

# 4. Stockage non sécurisé
grep -r "SharedPreferences\|getExternalStorage\|getExternalFilesDir" \
  jadx_output/sources/ -l

# 5. Chiffrement faible
grep -r "MD5\|DES\|AES/ECB\|RC4\|SHA1withRSA" \
  jadx_output/sources/ -i

# 6. WebViews dangereuses
grep -r "setJavaScriptEnabled(true)\|addJavascriptInterface\|setAllowFileAccess" \
  jadx_output/sources/

# 7. Logs sensibles
grep -r "Log\.d\|Log\.i\|Log\.e\|System\.out\.print" \
  jadx_output/sources/ | grep -i "pass\|token\|key\|secret"

# 8. Reflection (souvent utilisé pour obfusquer)
grep -r "getDeclaredMethod\|invoke\|forName" jadx_output/sources/
```

### 4.3 MobSF — Analyse Automatisée

MobSF (Mobile Security Framework) automatise l'analyse statique et dynamique.

```bash
# Installation avec Docker (recommandé)
docker pull opensecurity/mobile-security-framework-mobsf:latest

# Lancer MobSF
docker run -it --rm -p 8000:8000 \
  -v $(pwd)/uploads:/home/mobsf/.MobSF \
  opensecurity/mobile-security-framework-mobsf:latest

# Accéder à l'interface web
# http://localhost:8000
# Login: mobsf / mobsf

# API MobSF pour l'automatisation
API_KEY="votre_clé_api_mobsf"

# Upload d'un APK
curl -F "file=@target.apk" \
  -H "Authorization: $API_KEY" \
  http://localhost:8000/api/v1/upload

# Lancer l'analyse
curl -X POST \
  -d '{"file_name":"target.apk","hash":"<hash_retourné>"}' \
  -H "Authorization: $API_KEY" \
  -H "Content-Type: application/json" \
  http://localhost:8000/api/v1/scan

# Récupérer le rapport JSON
curl -X POST \
  -d '{"hash":"<hash>"}' \
  -H "Authorization: $API_KEY" \
  http://localhost:8000/api/v1/report_json \
  -o rapport_mobsf.json

python3 -c "
import json
with open('rapport_mobsf.json') as f:
    data = json.load(f)
    # Afficher le score de sécurité
    print(f'Score: {data[\"appsec\"][\"security_score\"]}/100')
    # Afficher les findings critiques
    for finding in data.get('findings', []):
        if finding.get('severity') == 'high':
            print(f'[HIGH] {finding[\"title\"]}')
"
```

---

## 5. Analyse Dynamique — Frida et Objection

> [!warning] Prérequis
> Pour l'analyse dynamique, tu as besoin d'un device rooté ou d'un émulateur. Sur un device physique non rooté, certaines techniques (Frida gadget) fonctionnent mais sont plus complexes à mettre en place.

### 5.1 Frida — Instrumentation Dynamique

Frida est un framework d'instrumentation dynamique. Il injecte un agent JavaScript dans le processus cible et permet de hooker des fonctions en temps réel.

#### Architecture Frida

```
┌─────────────────────────────────────────────────┐
│  Machine de pentest (PC)                        │
│  frida CLI / frida-tools / scripts Python       │
│           │                                     │
│           │ USB/TCP/WiFi                        │
└───────────┼─────────────────────────────────────┘
            │
┌───────────┼─────────────────────────────────────┐
│  Device Android                                 │
│           │                                     │
│   frida-server  ←── tourne en root              │
│           │ injecte dans                        │
│   com.target.app (processus)                    │
│           │                                     │
│   frida-agent.so (JIT compilé depuis JS)        │
└─────────────────────────────────────────────────┘
```

#### Installation

```bash
# Sur le PC
pip3 install frida-tools

# Sur le device (via ADB)
# Télécharger frida-server correspondant à l'architecture et la version
FRIDA_VERSION=$(frida --version)
# Pour un émulateur x86_64
wget "https://github.com/frida/frida/releases/download/${FRIDA_VERSION}/frida-server-${FRIDA_VERSION}-android-x86_64.xz"
unxz frida-server-*.xz
mv frida-server-* frida-server

adb push frida-server /data/local/tmp/
adb shell "chmod 755 /data/local/tmp/frida-server"

# Démarrer le serveur (en root)
adb shell "su -c 'nohup /data/local/tmp/frida-server &'"

# Vérifier que Frida fonctionne
frida-ps -U  # lister les processus sur le device
```

#### Scripts Frida essentiels

```javascript
// ===== hook_basics.js =====
// Hooker une méthode Java simple

Java.perform(function() {
    // Trouver la classe cible
    var MainActivity = Java.use("com.target.app.MainActivity");

    // Hooker une méthode — .implementation remplace l'implémentation originale
    MainActivity.checkPremium.implementation = function() {
        console.log("[*] checkPremium() appelé — retourner true");
        return true;  // bypass de la vérification premium
    };

    // Hooker avec overload (méthodes surchargées)
    var Utils = Java.use("com.target.app.Utils");
    Utils.decrypt.overload("java.lang.String", "java.lang.String").implementation = function(data, key) {
        console.log("[*] decrypt() appelé avec :");
        console.log("    data = " + data);
        console.log("    key  = " + key);

        // Appeler la vraie fonction et afficher le résultat
        var result = this.decrypt(data, key);
        console.log("    résultat = " + result);
        return result;  // retourner le vrai résultat
    };
});
```

```javascript
// ===== hook_crypto.js =====
// Intercepter toutes les opérations cryptographiques

Java.perform(function() {
    var MessageDigest = Java.use("java.security.MessageDigest");

    // Hook getInstance pour voir les algorithmes utilisés
    MessageDigest.getInstance.overload("java.lang.String").implementation = function(algo) {
        console.log("[CRYPTO] MessageDigest.getInstance(\"" + algo + "\")");
        if (algo.toLowerCase() === "md5" || algo.toLowerCase() === "sha1") {
            console.log("  [!] ALGORITHME FAIBLE DÉTECTÉ !");
        }
        return this.getInstance(algo);
    };

    var Cipher = Java.use("javax.crypto.Cipher");
    Cipher.getInstance.overload("java.lang.String").implementation = function(transformation) {
        console.log("[CRYPTO] Cipher.getInstance(\"" + transformation + "\")");
        if (transformation.includes("ECB")) {
            console.log("  [!] MODE ECB DANGEREUX !");
        }
        return this.getInstance(transformation);
    };

    // Hook doFinal pour voir les données en clair
    Cipher.doFinal.overload("[B").implementation = function(input) {
        var inputHex = bytesToHex(input);
        var result = this.doFinal(input);
        var resultHex = bytesToHex(result);
        console.log("[CRYPTO] doFinal:");
        console.log("  input  = " + inputHex);
        console.log("  output = " + resultHex);
        return result;
    };
});

function bytesToHex(bytes) {
    var hex = '';
    for (var i = 0; i < bytes.length; i++) {
        hex += ('0' + (bytes[i] & 0xFF).toString(16)).slice(-2);
    }
    return hex;
}
```

```javascript
// ===== hook_http.js =====
// Intercepter toutes les requêtes HTTP/HTTPS (même avec SSL Pinning)

Java.perform(function() {
    // Hook OkHttp3 (lib très courante)
    try {
        var OkHttpClient = Java.use("okhttp3.OkHttpClient");
        var Request = Java.use("okhttp3.Request");
        var RequestBuilder = Java.use("okhttp3.Request$Builder");

        // Hook sur l'exécution des requêtes
        var RealCall = Java.use("okhttp3.internal.connection.RealCall");
        RealCall.execute.implementation = function() {
            var request = this.request.value;
            console.log("[HTTP] " + request.method() + " " + request.url().toString());

            // Afficher les headers
            var headers = request.headers();
            for (var i = 0; i < headers.size(); i++) {
                console.log("  " + headers.name(i) + ": " + headers.value(i));
            }

            var response = this.execute();
            console.log("[HTTP] Response: " + response.code());
            return response;
        };
    } catch(e) {
        console.log("OkHttp3 non trouvé: " + e);
    }

    // Hook sur HttpURLConnection (API Android standard)
    try {
        var URL = Java.use("java.net.URL");
        URL.openConnection.overload().implementation = function() {
            console.log("[HTTP] Connexion vers: " + this.toString());
            return this.openConnection();
        };
    } catch(e) {}
});
```

```bash
# Lancer un script Frida
frida -U -n "Target App" -l hook_basics.js

# Attacher à un processus par nom de package
frida -U -f com.target.app -l hook_basics.js --no-pause

# Mode REPL interactif
frida -U -n "Target App"
# Puis dans le REPL :
> Java.perform(function() { Java.use("com.target.app.Utils").secretMethod.implementation = function() { return "hacked"; }; })

# Lister les classes chargées
frida -U -n "Target App" -e "Java.perform(function() { Java.enumerateLoadedClasses({ onMatch: function(cls) { if(cls.includes('target')) console.log(cls); }, onComplete: function() {} }); })"
```

### 5.2 Objection — Frida avec interface plus simple

Objection est une couche d'abstraction au-dessus de Frida qui fournit une CLI conviviale.

```bash
# Installation
pip3 install objection

# Lancer sur une app en cours d'exécution
objection -g com.target.app explore

# ===== Dans le shell Objection =====

# Android keystore — lister les entrées
android keystore list

# Android SharedPreferences — lister toutes les valeurs
android shared_preferences print

# Bypass SSL Pinning (méthode automatique)
android sslpinning disable

# Bypass root detection
android root disable

# Lister les activités
android hooking list activities

# Démarrer une activité exportée
android intent launch_activity com.target.app.HiddenAdminActivity

# Dump de la base SQLite
sqlite connect /data/data/com.target.app/databases/users.db
sqlite execute select * from users;

# Hooks automatiques sur toutes les méthodes d'une classe
android hooking watch class com.target.app.CryptoManager

# Dump mémoire
memory dump all /tmp/memory_dump.bin

# Chercher dans la mémoire
memory search --string "password"
memory search --pattern "Bearer [A-Za-z0-9+/=]+"
```

---

## 6. SSL Pinning — Bypass Complet

### 6.1 Qu'est-ce que le SSL Pinning ?

Le SSL Pinning (ou Certificate Pinning) est une technique qui force l'application à n'accepter qu'un certificat TLS spécifique (ou une clé publique spécifique), au lieu de faire confiance à toute autorité de certification reconnue par le système.

```
Sans SSL Pinning :
App → Vérifie le cert → Toute CA valide acceptée → Proxy Burp possible

Avec SSL Pinning :
App → Compare le cert avec le cert épinglé → Seul le cert du serveur réel accepté → Proxy bloqué
```

```java
// Implémentation typique du certificate pinning avec OkHttp
OkHttpClient client = new OkHttpClient.Builder()
    .certificatePinner(
        new CertificatePinner.Builder()
            // Hash SHA-256 de la clé publique du certificat
            .add("api.example.com",
                 "sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=")
            .add("api.example.com",
                 "sha256/BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB=")  // backup pin
            .build()
    )
    .build();
```

### 6.2 Bypass avec Frida

```javascript
// ===== ssl_bypass_universal.js =====
// Bypass SSL Pinning pour la plupart des implémentations courantes

Java.perform(function() {
    console.log("[*] SSL Pinning Bypass — démarrage...");

    // ===== 1. OkHttp3 CertificatePinner =====
    try {
        var CertificatePinner = Java.use("okhttp3.CertificatePinner");
        CertificatePinner.check.overload('java.lang.String', 'java.util.List')
            .implementation = function(hostname, peerCertificates) {
                console.log("[BYPASS] OkHttp3 CertificatePinner.check() pour: " + hostname);
                return; // ne rien faire = bypass
            };
        console.log("[+] OkHttp3 CertificatePinner bypassed");
    } catch(e) { console.log("[-] OkHttp3 non trouvé"); }

    // ===== 2. TrustManager personnalisé =====
    try {
        var X509TrustManager = Java.use("javax.net.ssl.X509TrustManager");
        var SSLContext = Java.use("javax.net.ssl.SSLContext");

        // Créer un TrustManager qui accepte tout
        var TrustManager = Java.registerClass({
            name: "com.frida.TrustEverything",
            implements: [X509TrustManager],
            methods: {
                checkClientTrusted: function(chain, authType) {},
                checkServerTrusted: function(chain, authType) {},
                getAcceptedIssuers: function() { return []; }
            }
        });

        var SSLContextImpl = Java.use("com.android.org.conscrypt.SSLContextImpl");
        SSLContextImpl.engineGetSocketFactory.implementation = function() {
            var trustManagers = [TrustManager.$new()];
            var sslContext = SSLContext.getInstance("TLS");
            sslContext.init(null, trustManagers, null);
            return sslContext.getSocketFactory();
        };
        console.log("[+] TrustManager personnalisé installé");
    } catch(e) { console.log("[-] Erreur TrustManager: " + e); }

    // ===== 3. HttpsURLConnection =====
    try {
        var HttpsURLConnection = Java.use("javax.net.ssl.HttpsURLConnection");
        HttpsURLConnection.setDefaultHostnameVerifier.implementation = function(hostnameVerifier) {
            console.log("[BYPASS] setDefaultHostnameVerifier remplacé");
            return; // ne pas changer le verifier
        };
        HttpsURLConnection.setSSLSocketFactory.implementation = function(factory) {
            console.log("[BYPASS] setSSLSocketFactory ignoré");
        };
        console.log("[+] HttpsURLConnection bypassed");
    } catch(e) {}

    // ===== 4. Retrofit (utilise OkHttp sous le capot) =====
    // Déjà couvert par OkHttp3 bypass

    // ===== 5. Network Security Config Android 7+ =====
    try {
        var NetworkSecurityTrustManager = Java.use(
            "android.security.net.config.NetworkSecurityTrustManager"
        );
        NetworkSecurityTrustManager.checkPins.implementation = function(chain) {
            console.log("[BYPASS] NetworkSecurityTrustManager.checkPins() bypassed");
            return; // bypass des pins déclarés dans network_security_config.xml
        };
        console.log("[+] NetworkSecurityTrustManager bypassed");
    } catch(e) {}

    // ===== 6. Conscrypt (moteur TLS d'Android) =====
    try {
        var ConscryptOpenSSLX509Certificate = Java.use(
            "com.android.org.conscrypt.OpenSSLX509Certificate"
        );
        // Intercepter la vérification de la chaîne de certificats
        var Platform = Java.use("com.android.org.conscrypt.Platform");
        Platform.checkServerTrusted.implementation = function(tm, chain, authType, engine) {
            console.log("[BYPASS] Conscrypt Platform.checkServerTrusted bypassed");
            return;
        };
        console.log("[+] Conscrypt bypassed");
    } catch(e) {}

    console.log("[*] SSL Pinning Bypass terminé !");
});
```

```bash
# Lancer le bypass SSL
frida -U -f com.target.app -l ssl_bypass_universal.js --no-pause

# Ou via Objection (plus simple)
objection -g com.target.app explore --startup-command "android sslpinning disable"
```

### 6.3 Bypass avec Magisk (Root)

Sur un device rooté avec Magisk, le module **MagiskTrustUserCerts** ajoute automatiquement les certificats utilisateur (comme le certificat Burp) dans le trust store système. Cela bypasse le SSL Pinning de niveau OS.

```bash
# 1. Installer Magisk sur l'émulateur ou le device
# 2. Dans Magisk Manager → Modules → Search "MagiskTrustUserCerts"
# 3. Installer et rebooter

# 4. Ajouter le certificat Burp dans Settings > Security > Install certificate

# Méthode manuelle sans module :
# Exporter le certificat Burp en format DER
# Calculer le hash pour le nom de fichier
CERT_FILE="burp_certificate.der"
HASH=$(openssl x509 -inform DER -subject_hash_old -in "$CERT_FILE" | head -1)
cp "$CERT_FILE" "${HASH}.0"

# Pousser dans le trust store système
adb push "${HASH}.0" /sdcard/
adb shell "su -c 'cp /sdcard/${HASH}.0 /system/etc/security/cacerts/'"
adb shell "su -c 'chmod 644 /system/etc/security/cacerts/${HASH}.0'"
adb reboot
```

### 6.4 Cas difficiles — Pinning natif

Certaines apps implémentent le pinning en code natif (C/C++), plus difficile à bypasser.

```javascript
// Bypass du pinning natif via Frida — hook sur les fonctions SSL C
// Cible : libssl.so ou libboringssl.so

var SSL_CTX_set_custom_verify = Module.findExportByName("libssl.so", "SSL_CTX_set_custom_verify");
if (SSL_CTX_set_custom_verify) {
    Interceptor.attach(SSL_CTX_set_custom_verify, {
        onEnter: function(args) {
            // mode = SSL_VERIFY_NONE (0) pour désactiver la vérification
            args[1] = ptr(0);
            console.log("[BYPASS] SSL_CTX_set_custom_verify → NONE");
        }
    });
}

// BoringSSL (utilisé par beaucoup d'apps modernes)
var boring_verify = Module.findExportByName(null, "SSL_CTX_set_verify");
if (boring_verify) {
    Interceptor.attach(boring_verify, {
        onEnter: function(args) {
            args[1] = ptr(0); // SSL_VERIFY_NONE
        }
    });
}
```

---

## 7. Stockage Insécurisé — Analyse et Exploitation

### 7.1 SharedPreferences

```bash
# Accéder aux SharedPreferences d'une app (root requis)
adb shell "su -c 'ls /data/data/com.target.app/shared_prefs/'"

# Lire un fichier de préférences
adb shell "su -c 'cat /data/data/com.target.app/shared_prefs/user_prefs.xml'"
```

```xml
<!-- Exemple de SharedPreferences vulnérables -->
<?xml version='1.0' encoding='utf-8' standalone='yes' ?>
<map>
    <!-- ❌ Token d'authentification en clair dans les prefs ! -->
    <string name="auth_token">eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoiMTIzIn0.xxx</string>
    <string name="user_email">admin@example.com</string>
    <!-- ❌ Mot de passe en clair -->
    <string name="cached_password">MonMotDePasse123!</string>
    <boolean name="is_premium" value="false" />
    <boolean name="is_admin" value="false" />
</map>
```

```bash
# Modifier les SharedPreferences pour bypass
# Méthode 1 : éditer le fichier XML directement (root)
adb shell "su -c 'sed -i \"s/value=\\\"false\\\" name=\\\"is_admin\\\"/value=\\\"true\\\" name=\\\"is_admin\\\"/g\" /data/data/com.target.app/shared_prefs/user_prefs.xml'"

# Méthode 2 : via Frida
frida -U -n "Target App" -e "
Java.perform(function() {
    var context = Java.use('android.app.ActivityThread').currentApplication().getApplicationContext();
    var prefs = context.getSharedPreferences('user_prefs', 0);
    var editor = prefs.edit();
    editor.putBoolean('is_admin', true);
    editor.putBoolean('is_premium', true);
    editor.commit();
    console.log('[*] SharedPreferences modifiées !');
});
"
```

### 7.2 Bases de données SQLite

```bash
# Trouver toutes les bases de données de l'app
adb shell "su -c 'find /data/data/com.target.app -name \"*.db\" -o -name \"*.sqlite\" -o -name \"*.db3\"'"

# Copier la DB sur la machine de pentest
adb shell "su -c 'cp /data/data/com.target.app/databases/app.db /sdcard/'"
adb pull /sdcard/app.db ./

# Ouvrir avec sqlite3
sqlite3 app.db

# Dans sqlite3 :
.tables                              -- lister les tables
.schema users                        -- voir la structure
SELECT * FROM users;                 -- dump complet
SELECT * FROM sessions;              -- tokens de session
SELECT * FROM credentials;          -- identifiants
.dump                                -- dump SQL complet
.quit
```

```python
#!/usr/bin/env python3
# Script d'analyse automatique des bases SQLite mobiles
import sqlite3
import sys
import os

SENSITIVE_PATTERNS = [
    'password', 'passwd', 'pwd', 'secret', 'token', 'key',
    'pin', 'credit', 'card', 'ssn', 'email', 'phone'
]

def analyze_sqlite(db_path):
    print(f"\n[*] Analyse: {db_path}")
    conn = sqlite3.connect(db_path)
    cursor = conn.cursor()

    # Lister les tables
    cursor.execute("SELECT name FROM sqlite_master WHERE type='table'")
    tables = cursor.fetchall()
    print(f"[+] Tables trouvées: {[t[0] for t in tables]}")

    for table in tables:
        table_name = table[0]
        # Obtenir les colonnes
        cursor.execute(f"PRAGMA table_info({table_name})")
        columns = cursor.fetchall()
        col_names = [col[1] for col in columns]

        # Détecter les colonnes sensibles
        sensitive_cols = [
            col for col in col_names
            if any(p in col.lower() for p in SENSITIVE_PATTERNS)
        ]

        if sensitive_cols:
            print(f"\n[!] Table '{table_name}' — colonnes sensibles: {sensitive_cols}")
            cursor.execute(f"SELECT {', '.join(sensitive_cols)} FROM {table_name} LIMIT 10")
            rows = cursor.fetchall()
            for row in rows:
                print(f"    {row}")

    conn.close()

if __name__ == "__main__":
    for db_file in sys.argv[1:]:
        if os.path.exists(db_file):
            analyze_sqlite(db_file)
```

### 7.3 Fichiers sur le stockage externe

```bash
# Le stockage externe est accessible par TOUTES les apps avec READ_EXTERNAL_STORAGE
# Avant Android 10, n'importe quelle app peut lire /sdcard/

# Chercher des données sensibles sur le stockage externe
adb shell "find /sdcard/ -name '*.db' -o -name '*.sqlite' -o -name '*.log' -o -name '*.xml' -o -name '*.json' 2>/dev/null"

# Exemple : apps qui sauvegardent des données sensibles sur /sdcard/
adb shell "find /sdcard/ -path '*/com.target.app/*' -type f"

# Dump complet du stockage externe
adb pull /sdcard/ ./sdcard_dump/
```

```java
// ❌ VULNÉRABLE — Écriture de données sensibles sur le stockage externe
File file = new File(Environment.getExternalStorageDirectory(),
    "myapp/session_" + userId + ".json");
file.getParentFile().mkdirs();
FileWriter writer = new FileWriter(file);
writer.write("{\"token\":\"" + authToken + "\",\"user_id\":" + userId + "}");
writer.close();
// Ce fichier est lisible par TOUTES les apps avec READ_EXTERNAL_STORAGE !

// ✅ SÉCURISÉ — Utiliser le stockage interne (Context.getFilesDir())
File file = new File(context.getFilesDir(), "session.json");
// Accessible uniquement par l'app elle-même
```

### 7.4 Android Keystore — Stockage sécurisé

```java
// ✅ Utiliser le KeyStore pour stocker des clés de façon sécurisée
public class SecureStorage {
    private static final String KEY_ALIAS = "MyAppMasterKey";

    // Initialiser ou récupérer la clé
    private SecretKey getOrCreateKey() throws Exception {
        KeyStore keyStore = KeyStore.getInstance("AndroidKeyStore");
        keyStore.load(null);

        if (!keyStore.containsAlias(KEY_ALIAS)) {
            // Créer une nouvelle clé dans le Keystore matériel
            KeyGenerator keyGenerator = KeyGenerator.getInstance(
                KeyProperties.KEY_ALGORITHM_AES, "AndroidKeyStore");
            keyGenerator.init(new KeyGenParameterSpec.Builder(
                KEY_ALIAS,
                KeyProperties.PURPOSE_ENCRYPT | KeyProperties.PURPOSE_DECRYPT)
                .setBlockModes(KeyProperties.BLOCK_MODE_GCM)
                .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_NONE)
                .setKeySize(256)
                // La clé ne peut pas être exportée hors du Keystore
                .setUserAuthenticationRequired(false)
                .build());
            return keyGenerator.generateKey();
        }
        return ((KeyStore.SecretKeyEntry) keyStore.getEntry(KEY_ALIAS, null)).getSecretKey();
    }

    public byte[] encrypt(String data) throws Exception {
        SecretKey key = getOrCreateKey();
        Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
        cipher.init(Cipher.ENCRYPT_MODE, key);
        byte[] iv = cipher.getIV();
        byte[] encrypted = cipher.doFinal(data.getBytes());

        // Préfixer avec l'IV pour le déchiffrement
        byte[] result = new byte[iv.length + encrypted.length];
        System.arraycopy(iv, 0, result, 0, iv.length);
        System.arraycopy(encrypted, 0, result, iv.length, encrypted.length);
        return result;
    }
}
```

---

## 8. Interception du Trafic Réseau avec Burp Suite

### 8.1 Configuration de Burp Suite avec un émulateur Android

```bash
# ===== Étape 1 : Configurer Burp Suite =====
# Dans Burp Suite :
# Proxy → Options → Proxy Listeners → Add
# Bind to port: 8080
# Bind to address: All interfaces
# → OK

# ===== Étape 2 : Exporter le certificat Burp =====
# Proxy → Options → CA Certificate → Export Certificate
# Format: DER (.cer) → burp_certificate.der

# ===== Étape 3 : Configurer le proxy sur l'émulateur =====
# Settings → Wi-Fi → Long press sur WiredSSID → Modify network
# Advanced options → Proxy → Manual
# Host: 10.0.2.2 (adresse du PC host depuis l'émulateur AVD)
# Port: 8080

# Pour Genymotion : l'IP du PC host est 10.0.3.2
# Pour device physique sur WiFi : l'IP locale du PC

# ===== Étape 4 : Installer le certificat CA (Android < 10) =====
# Transformer en format PEM avec le bon nom de fichier
CERT_HASH=$(openssl x509 -inform DER -subject_hash_old -in burp_certificate.der | head -1)
openssl x509 -inform DER -in burp_certificate.der -out "${CERT_HASH}.0"

# L'installer dans le trust store système (émulateur rooté)
adb push "${CERT_HASH}.0" /sdcard/
adb shell "su -c 'mount -o rw,remount /system'"
adb shell "su -c 'cp /sdcard/${CERT_HASH}.0 /system/etc/security/cacerts/'"
adb shell "su -c 'chmod 644 /system/etc/security/cacerts/${CERT_HASH}.0'"
adb reboot

# Pour Android 10+ avec user_certs dans network_security_config :
# Méthode A : Patcher le APK (ajouter <certificates src="user"/> dans network_security_config.xml)
# Méthode B : Utiliser le bypass Frida SSL
```

### 8.2 Utiliser le port forwarding ADB

```bash
# Alternative au proxy WiFi — port forwarding via USB
# Avantage : fonctionne même si l'app ignore les paramètres proxy système

# Rediriger le port 8080 de l'appareil vers le PC
adb reverse tcp:8080 tcp:8080

# Dans Burp : écouter sur 127.0.0.1:8080
# Configurer l'app pour utiliser 127.0.0.1:8080 comme proxy
# (ou utiliser l'app ProxyDroid sur un device rooté)
```

### 8.3 Intercepter avec Burp — Exemple pratique

```
Flux de capture :

App Android → proxy 10.0.2.2:8080 → Burp Suite → Internet
                                          ↑
                             Capture + modification possible

Exemple de requête interceptée :
POST /api/v1/login HTTP/1.1
Host: api.example.com
Content-Type: application/json
User-Agent: TargetApp/1.2.3 (Android)

{"username":"user@email.com","password":"hunter2","device_id":"abc123"}

→ Modifier le corps : {"username":"admin@example.com","password":"hunter2","device_id":"abc123"}
→ Tester une injection : {"username":"admin@example.com","password":"' OR 1=1 --","device_id":"abc123"}
```

```python
#!/usr/bin/env python3
# Automatiser les tests sur le trafic intercepté avec l'API Burp
# (Burp Suite Professional uniquement)

import requests
import json

BURP_API = "http://127.0.0.1:1337"  # API Burp (Pro)

def get_recent_requests():
    """Récupérer les dernières requêtes interceptées."""
    response = requests.get(f"{BURP_API}/v0.1/proxy/history")
    return response.json()

def test_sqli_on_requests():
    """Tester une injection SQL sur tous les paramètres POST récents."""
    payloads = ["'", "' OR '1'='1", "'; DROP TABLE users--", "1 UNION SELECT null--"]
    history = get_recent_requests()

    for req in history[-10:]:  # 10 dernières requêtes
        if req.get('method') == 'POST':
            try:
                body = json.loads(req.get('body', '{}'))
                for key, value in body.items():
                    for payload in payloads:
                        body_test = body.copy()
                        body_test[key] = payload
                        print(f"[TEST] {req['url']} — {key}={payload}")
                        # Envoyer la requête modifiée...
            except json.JSONDecodeError:
                pass

test_sqli_on_requests()
```

---

## 9. Reverse Engineering — Workflow Complet

### 9.1 Méthodologie d'audit

```
Phase 1 : Reconnaissance passive
├── Télécharger l'APK (Google Play via APKPure/APKMirror, ou directement depuis l'appareil)
├── Analyser le manifest
├── Analyser les permissions
└── Identifier la surface d'attaque

Phase 2 : Analyse statique
├── jadx → code Java
├── Recherche patterns (credentials, URLs, crypto)
├── MobSF rapport automatique
└── Audit des librairies embarquées

Phase 3 : Analyse dynamique
├── Setup Frida + Burp
├── Intercepter le trafic réseau
├── Hooker les fonctions sensibles
└── Analyser le stockage

Phase 4 : Exploitation
├── Bypass authentication
├── Bypass SSL Pinning
├── Extraire données
└── Escalade de privilèges

Phase 5 : Rapport
├── CVSS scoring de chaque finding
├── POC (Proof of Concept)
├── Recommandations de remédiation
└── Rapport executive + technique
```

### 9.2 Extraire l'APK depuis un device

```bash
# Trouver le chemin de l'APK installée
adb shell pm list packages | grep "target"
# package:com.target.app

adb shell pm path com.target.app
# package:/data/app/~~XXXX==/com.target.app-YYYY==/base.apk

# Copier l'APK sur la machine de pentest
adb pull /data/app/~~XXXX==/com.target.app-YYYY==/base.apk target.apk

# Pour les app splits (Android Bundle)
adb shell pm path com.target.app
# base.apk + split_config.arm64_v8a.apk + split_config.xxhdpi.apk...
# Les télécharger tous et analyser le base.apk
```

### 9.3 Workflow complet illustré

```bash
#!/bin/bash
# === WORKFLOW PENTEST ANDROID COMPLET ===

APP_PACKAGE="com.target.vulnerableapp"
OUTPUT_DIR="pentest_${APP_PACKAGE}"
mkdir -p "$OUTPUT_DIR"

echo "=== [1/6] Extraction APK ==="
APK_PATH=$(adb shell pm path "$APP_PACKAGE" | grep base | cut -d: -f2 | tr -d '\r')
adb pull "$APK_PATH" "$OUTPUT_DIR/target.apk"

echo "=== [2/6] Décompilation apktool ==="
apktool d "$OUTPUT_DIR/target.apk" -o "$OUTPUT_DIR/apktool_out/" --no-res

echo "=== [3/6] Décompilation jadx ==="
jadx -d "$OUTPUT_DIR/jadx_out/" "$OUTPUT_DIR/target.apk" 2>/dev/null

echo "=== [4/6] Analyse automatique du manifest ==="
python3 << PYEOF
import xml.etree.ElementTree as ET

# Manifest décodé par apktool
tree = ET.parse("$OUTPUT_DIR/apktool_out/AndroidManifest.xml")
root = tree.getroot()

android_ns = "http://schemas.android.com/apk/res/android"
app = root.find('application')

findings = []

# Checks basiques
attrs = {
    'debuggable': ('CRITIQUE', 'App en mode debug en production'),
    'allowBackup': ('WARN', 'Backup ADB possible — données extractibles'),
    'usesCleartextTraffic': ('CRITIQUE', 'Trafic HTTP non chiffré autorisé'),
}

for attr, (severity, msg) in attrs.items():
    val = app.get(f'{{{android_ns}}}{attr}')
    if val == 'true':
        findings.append(f"[{severity}] {msg}")

# Composants exportés
for tag in ['activity', 'service', 'receiver', 'provider']:
    for comp in root.iter(tag):
        exported = comp.get(f'{{{android_ns}}}exported')
        name = comp.get(f'{{{android_ns}}}name', '')
        permission = comp.get(f'{{{android_ns}}}permission', '')
        if exported == 'true' and not permission:
            findings.append(f"[WARN] {tag.upper()} exporté sans permission: {name}")

for f in findings:
    print(f)
PYEOF

echo "=== [5/6] Recherche de credentials hardcodés ==="
echo "[*] Strings suspects dans le code :"
grep -r "password\|secret\|api_key\|private_key\|apikey\|token" \
  "$OUTPUT_DIR/jadx_out/sources/" -i --include="*.java" \
  | grep -v "//\|test\|example\|sample" \
  | grep -v "\.setPassword\|getString\|getPassword" \
  | head -20

echo "[*] URLs hardcodées :"
grep -roh 'https\?://[^"'\'' ]*' "$OUTPUT_DIR/jadx_out/sources/" \
  | sort -u | head -20

echo "=== [6/6] Résumé ==="
echo "APK : $OUTPUT_DIR/target.apk"
echo "Sources Java : $OUTPUT_DIR/jadx_out/sources/"
echo "Smali : $OUTPUT_DIR/apktool_out/smali/"
echo ""
echo "Prochaines étapes :"
echo "  1. Analyser les sources Java dans jadx-gui"
echo "  2. Démarrer Frida + Burp pour l'analyse dynamique"
echo "  3. Tester les composants exportés"
```

---

## 10. CTF Mobile — Plateformes et Exemples

### 10.1 OWASP UnCrackable Apps

Les UnCrackable Apps sont des apps Android spécialement conçues pour l'apprentissage du reverse engineering.

```bash
# Télécharger UnCrackable Level 1
wget https://github.com/OWASP/owasp-mastg/raw/master/Crackmes/Android/Level_01/UnCrackable-Level1.apk

# Objectif Level 1 : trouver le mot de passe secret

# Étape 1 : Décompiler avec jadx
jadx -d uncr1/ UnCrackable-Level1.apk

# Étape 2 : Chercher la logique de vérification
grep -r "verify\|check\|secret\|password" uncr1/sources/ -i | head -20
```

```java
// Code jadx de UnCrackable Level 1 (simplifié)
public class MainActivity extends Activity {

    // La vérification est ici
    private boolean verify(String input) {
        String str = "8d127684cbc37c17616d806cf50473cc";  // MD5 de quelque chose
        // Déchiffrement avec une clé hardcodée
        String decrypted = sg.vantagepoint.a.a.a(
            b("8d127684cbc37c17616d806cf50473cc"),
            Base64.decode("5UJiFctbmgbDoLXmpL12mkno8HT4Lv8dlat8FxR2GOc=", 0)
        );
        return input.equals(decrypted);
    }
}
```

```javascript
// Solution avec Frida — intercepter la comparaison
Java.perform(function() {
    // Approche 1 : Hooker la méthode verify
    var MainActivity = Java.use("sg.vantagepoint.uncrackable1.MainActivity");
    MainActivity.verify.implementation = function(input) {
        console.log("[*] verify() appelé avec: " + input);
        var result = this.verify(input);
        console.log("[*] verify() retourne: " + result);
        return result;
    };

    // Approche 2 : Intercepter la méthode de déchiffrement directement
    var a = Java.use("sg.vantagepoint.a.a");
    a.a.implementation = function(key, data) {
        var decrypted = this.a(key, data);
        // Convertir le tableau de bytes en string
        var result = "";
        for (var i = 0; i < decrypted.length; i++) {
            result += String.fromCharCode(decrypted[i]);
        }
        console.log("[*] Secret déchiffré: " + result);
        return decrypted;
    };
});
```

### 10.2 HackTheBox Mobile

```
Catégories de challenges HTB Mobile :
├── APK Reverse : trouver un flag dans le code/ressources
├── Network Traffic : analyser un pcap de trafic mobile
├── Binary Exploitation : vulnérabilité dans une lib native .so
└── Forensics : analyser un backup ou une image de device

Plateformes CTF avec challenges mobiles :
├── HackTheBox (hackthebox.com/tracks/mobile)
├── Root-Me (root-me.org) → catégorie "Android"
├── OWASP iGoat (iOS)
├── OWASP DVIA (Damn Vulnerable iOS Application)
├── OWASP DVSA (Damn Vulnerable Android App)
└── CTFtime.org (filtrer par tag "mobile")
```

```bash
# Checklist CTF Mobile — Android

# 1. Strings dans les ressources
strings UnCrackable.apk | grep -i "flag\|ctf\|HTB{\|FLAG{"

# 2. Dans les ressources compilées
unzip -p target.apk resources.arsc | strings | grep -i "flag"

# 3. Dans les assets
unzip target.apk "assets/*" -d extracted/
find extracted/assets/ -type f | xargs strings | grep "flag"

# 4. Dans les classes.dex
unzip -p target.apk classes.dex | strings | grep "HTB{"

# 5. Après décompilation jadx
grep -r "HTB{\|flag\|FLAG" jadx_out/ -i

# 6. Dans les clés de chiffrement
grep -r "Base64\.\|AES\|key\|iv" jadx_out/sources/ | grep -v "//\|import"

# 7. Certificats embarqués
unzip target.apk "META-INF/*" -d extracted/
openssl x509 -text -in extracted/META-INF/CERT.RSA 2>/dev/null | grep -i "flag\|ctf"
```

### 10.3 Exemple de challenge complet résolu

```
Challenge : "Secure Login" (type CTF)
Description : "Une app de login sécurisée... ou pas ?"
Objectif : Se connecter en tant qu'admin

Étape 1 — Décompiler
jadx -d out/ secure_login.apk

Étape 2 — Analyser le LoginActivity
```

```java
// Code décompilé par jadx
public class LoginActivity extends AppCompatActivity {
    private static final String ADMIN_HASH = "5f4dcc3b5aa765d61d8327deb882cf99";

    public void onLoginClick(View view) {
        String username = this.usernameField.getText().toString();
        String password = this.passwordField.getText().toString();

        // Hash MD5 du password
        String hash = md5(password);

        if (username.equals("admin") && hash.equals(ADMIN_HASH)) {
            showFlag();
        }
    }
}
```

```bash
# Étape 3 — Identifier le hash MD5
echo "5f4dcc3b5aa765d61d8327deb882cf99"
# C'est le MD5 de "password" !
# Vérification : echo -n "password" | md5sum → 5f4dcc3b5aa765d61d8327deb882cf99 ✓

# Ou utiliser crackstation.net / hashcat
echo "5f4dcc3b5aa765d61d8327deb882cf99" > hash.txt
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt
# → password

# Connexion : admin / password → FLAG{md5_is_not_a_password_hash}
```

---

## 11. Contre-mesures — Sécuriser une Application Mobile

> [!tip] Philosophie
> La sécurité mobile est une question de **profondeur de défense**. Aucune protection n'est absolue, mais multiplier les couches rend l'attaque exponentiellement plus coûteuse pour l'attaquant.

### 11.1 Checklist de sécurisation Android

#### Configuration (AndroidManifest.xml)

```xml
<application
    android:allowBackup="false"          <!-- Désactiver le backup ADB -->
    android:debuggable="false"           <!-- JAMAIS true en prod -->
    android:networkSecurityConfig="@xml/network_security_config"
    android:usesCleartextTraffic="false" <!-- Interdire HTTP -->
    android:extractNativeLibs="false">   <!-- Empêcher extraction des .so -->

    <!-- Ne déclarer android:exported que si nécessaire -->
    <!-- Toujours avec android:permission si exporté -->
    <activity
        android:name=".LoginActivity"
        android:exported="true">
        <!-- Démarrable uniquement depuis le launcher, pas depuis d'autres apps -->
    </activity>

    <!-- Protéger les providers avec une permission signature -->
    <provider
        android:name=".SecureProvider"
        android:authorities="com.example.app.provider"
        android:exported="false" />  <!-- Ne jamais exporter sans raison -->
</application>
```

```xml
<!-- network_security_config.xml sécurisé -->
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config cleartextTrafficPermitted="false">
        <trust-anchors>
            <!-- Seulement les CAs système — pas de certificats utilisateur -->
            <certificates src="system" />
        </trust-anchors>
    </base-config>

    <!-- Certificate Pinning pour les domaines critiques -->
    <domain-config>
        <domain includeSubdomains="true">api.example.com</domain>
        <pin-set expiration="2025-12-31">
            <pin digest="SHA-256">AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=</pin>
            <!-- Backup pin obligatoire ! -->
            <pin digest="SHA-256">BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB=</pin>
        </pin-set>
    </domain-config>
</network-security-config>
```

#### Code — Bonnes pratiques

```java
// ===== Stockage sécurisé avec EncryptedSharedPreferences =====
// (Android Jetpack Security - androidx.security:security-crypto)

import androidx.security.crypto.EncryptedSharedPreferences;
import androidx.security.crypto.MasterKey;

MasterKey masterKey = new MasterKey.Builder(context)
    .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
    .build();

SharedPreferences securePrefs = EncryptedSharedPreferences.create(
    context,
    "secure_prefs",
    masterKey,
    EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
    EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
);

// Usage identique aux SharedPreferences normales
securePrefs.edit().putString("auth_token", token).apply();
String token = securePrefs.getString("auth_token", null);
```

```java
// ===== Détection de Root =====
// Ne pas se fier uniquement à des checks facilement contournables
// Combiner plusieurs heuristiques

public class RootDetector {

    public static boolean isDeviceRooted() {
        return checkSuBinary()
            || checkRootApps()
            || checkDangerousProps()
            || checkRWPaths()
            || checkTestKeys();
    }

    private static boolean checkSuBinary() {
        String[] paths = {
            "/system/bin/su", "/system/xbin/su", "/sbin/su",
            "/system/su", "/system/bin/.ext/su", "/system/usr/we-need-root/su-backup",
            "/data/local/xbin/su", "/data/local/bin/su", "/data/local/su"
        };
        for (String path : paths) {
            if (new File(path).exists()) return true;
        }
        return false;
    }

    private static boolean checkRootApps() {
        String[] packages = {
            "com.topjohnwu.magisk",          // Magisk
            "com.koushikdutta.superuser",     // Superuser
            "eu.chainfire.supersu",           // SuperSU
            "com.noshufou.android.su",        // Superuser
            "com.thirdparty.superuser",
            "com.zachspong.temprootremovejb"
        };
        PackageManager pm = context.getPackageManager();
        for (String pkg : packages) {
            try {
                pm.getPackageInfo(pkg, 0);
                return true; // app trouvée
            } catch (PackageManager.NameNotFoundException ignored) {}
        }
        return false;
    }

    private static boolean checkDangerousProps() {
        // Propriétés système présentes sur les devices rootés
        String[] props = {"ro.debuggable", "ro.secure"};
        // ro.secure=0 ou ro.debuggable=1 indique un device de développement/rooté
        return Build.TAGS != null && Build.TAGS.contains("test-keys");
    }

    private static boolean checkRWPaths() {
        // Essayer d'écrire dans /system (normalement en lecture seule)
        try {
            File testFile = new File("/system/test_root_detection");
            testFile.createNewFile();
            testFile.delete();
            return true; // on a pu écrire = rooté
        } catch (Exception ignored) {
            return false;
        }
    }
}
```

```java
// ===== Contre-mesures Anti-Debugging =====
public class AntiDebug {

    // Détecter si un debugger est attaché
    public static boolean isDebuggerAttached() {
        return android.os.Debug.isDebuggerConnected()
            || android.os.Debug.waitingForDebugger();
    }

    // Détecter Frida
    public static boolean isFridaPresent() {
        // Méthode 1 : chercher le process frida-server
        try {
            Runtime runtime = Runtime.getRuntime();
            Process process = runtime.exec("ps -A");
            BufferedReader reader = new BufferedReader(new InputStreamReader(process.getInputStream()));
            String line;
            while ((line = reader.readLine()) != null) {
                if (line.contains("frida") || line.contains("gdb") || line.contains("lldb")) {
                    return true;
                }
            }
        } catch (Exception ignored) {}

        // Méthode 2 : chercher les ports Frida (27042 par défaut)
        try {
            Socket socket = new Socket();
            socket.connect(new InetSocketAddress("127.0.0.1", 27042), 100);
            socket.close();
            return true; // port ouvert = frida-server probablement actif
        } catch (Exception ignored) {}

        return false;
    }

    // Intégration dans le lifecycle de l'app
    public static void checkAndExitIfCompromised(Activity activity) {
        if (BuildConfig.DEBUG) return; // ne vérifier qu'en prod

        if (isDebuggerAttached() || isFridaPresent() || RootDetector.isDeviceRooted()) {
            // Ne pas crasher bêtement (ça aide l'attaquant à localiser la protection)
            // Préférer : sauter à un état aléatoire, modifier le comportement, logger
            activity.finishAffinity();
            System.exit(0);
        }
    }
}
```

> [!warning] Limites des protections runtime
> Toutes ces protections **peuvent être contournées** par un attaquant déterminé avec Frida. Leur rôle est d'augmenter le coût de l'attaque, pas de la rendre impossible. La vraie sécurité réside dans la **validation côté serveur** et le **chiffrement correctement implémenté**.

### 11.2 Checklist MASVS (Mobile Application Security Verification Standard)

```
MASVS-STORAGE (Stockage)
├── ✅ Données sensibles uniquement dans le stockage interne
├── ✅ SharedPreferences chiffrées (EncryptedSharedPreferences)
├── ✅ Bases SQLite chiffrées (SQLCipher)
├── ✅ Clés dans le Android KeyStore, jamais dans le code
├── ✅ Rien de sensible dans les logs
└── ✅ Clipboard cleared après copie de données sensibles

MASVS-CRYPTO (Cryptographie)
├── ✅ AES-256-GCM pour le chiffrement symétrique
├── ✅ RSA-OAEP ou ECDH pour le chiffrement asymétrique
├── ✅ Argon2/bcrypt/PBKDF2 pour les mots de passe (JAMAIS MD5/SHA1 seuls)
├── ✅ IV/Nonce aléatoire unique pour chaque chiffrement
└── ✅ Pas de crypto custom — utiliser uniquement les libs standard

MASVS-NETWORK (Réseau)
├── ✅ TLS 1.2+ pour toutes les connexions
├── ✅ Certificate Pinning sur les endpoints critiques
├── ✅ Validation du hostname
└── ✅ Pas de HTTP (cleartext)

MASVS-AUTH (Authentification)
├── ✅ Authentification côté serveur uniquement
├── ✅ Tokens OAuth2 avec expiration courte
├── ✅ Biométrie via BiometricPrompt (API officielle)
└── ✅ Rate limiting et lockout côté serveur

MASVS-CODE (Code)
├── ✅ ProGuard/R8 activé en release (obfuscation)
├── ✅ Signature APK avec clé de production
├── ✅ Pas de android:debuggable="true" en prod
├── ✅ Vérification de l'intégrité de l'APK (SafetyNet/Play Integrity)
└── ✅ Pas de composants exportés inutilement
```

### 11.3 Obfuscation avec ProGuard/R8

```groovy
// build.gradle — activer R8 (successeur de ProGuard)
android {
    buildTypes {
        release {
            minifyEnabled true          // activer l'obfuscation
            shrinkResources true        // supprimer les ressources inutilisées
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'),
                         'proguard-rules.pro'
        }
    }
}
```

```
# proguard-rules.pro — règles d'obfuscation

# Garder les classes de l'API publique lisibles
-keep class com.example.app.api.** { *; }

# Garder les modèles de données (pour la sérialisation JSON)
-keep class com.example.app.models.** { *; }

# Renommer agressivement tout le reste
-renamesourcefileattribute SourceFile
-keepattributes SourceFile,LineNumberTable  # garder pour le crash reporting

# Obfusquer les noms de méthodes et champs
-obfuscationdictionary proguard-dictionary.txt
```

```bash
# Vérifier l'efficacité de l'obfuscation
jadx -d after_obfuscation/ release.apk

# Avant obfuscation :
# → com.example.app.LoginActivity.checkPassword(String)
# → com.example.app.utils.CryptoManager.encrypt(byte[], SecretKey)

# Après obfuscation R8 agressif :
# → a.b.c.a.a(String)
# → a.b.d.b.a(byte[], Object)
```

---

## 12. Challenges et Exercices Pratiques

### Challenge 1 — Analyse d'un APK vulnérable

**Objectif** : Trouver les 5 vulnérabilités dans l'APK DVAA (Damn Vulnerable Android App).

```bash
# Télécharger DVAA
wget https://github.com/K3170Malone/dvaa/raw/master/DVAA.apk

# Missions :
# 1. Trouver les credentials hardcodés dans le code Java
# 2. Identifier les composants Android exportés exploitables
# 3. Lire les données sensibles dans le stockage (SharedPreferences + SQLite)
# 4. Intercepter le trafic HTTP non chiffré avec Burp
# 5. Bypasser la vérification de licence en hookant avec Frida

# Indices :
# - jadx DVAA.apk → chercher dans com.app.dvaa.activities.*
# - adb shell content query --uri "content://..."
# - grep -r "HTTP\|cleartext\|allowBackup" dans le manifest décodé
```

### Challenge 2 — SSL Pinning Bypass

**Objectif** : Intercepter le trafic HTTPS d'une app qui implémente le certificate pinning.

```bash
# Installer l'app PinningTest (à créer ou utiliser une vraie app avec pinning)
# Symptôme : Burp est configuré mais aucune requête n'arrive pour cette app

# Mission :
# 1. Identifier le type de pinning (OkHttp, TrustManager custom, native)
# 2. Écrire le script Frida approprié pour le bypasser
# 3. Capturer et analyser le trafic dans Burp

# Méthodologie :
# - jadx → chercher "CertificatePinner\|TrustManager\|X509\|checkServerTrusted"
# - Identifier les librairies réseau (OkHttp3, Retrofit, Volley, URLSession)
# - Adapter le script ssl_bypass_universal.js selon le cas
```

### Challenge 3 — CTF OWASP UnCrackable Level 2

```bash
wget https://github.com/OWASP/owasp-mastg/raw/master/Crackmes/Android/Level_02/UnCrackable-Level2.apk

# Ce level est plus difficile :
# - Détection de root (à bypasser)
# - Logique de validation en code natif (libfoo.so)
# - Nécessite de hooker des fonctions C natives avec Frida

# Indices :
# 1. Bypasser la détection de root d'abord
# 2. Trouver la fonction native dans libfoo.so avec strings/radare2
# 3. Hooker la fonction avec Frida Interceptor.attach()
```

```javascript
// Squelette de solution pour UnCrackable Level 2
Java.perform(function() {
    // Étape 1 : Bypass root detection
    var System = Java.use("java.lang.System");
    var Runtime = Java.use("java.lang.Runtime");

    var MainActivity = Java.use("sg.vantagepoint.uncrackable2.MainActivity");
    MainActivity.onResume.implementation = function() {
        this.onResume();
        // Bypass de l'appel à System.exit()
    };

    // Étape 2 : Hook la fonction native
    // Trouver la lib avec : frida -U -n "UnCrackable2" -e "Process.enumerateModules()"
    var libfoo = Module.findBaseAddress("libfoo.so");
    if (libfoo) {
        // Chercher l'export "Java_sg_vantagepoint_uncrackable2_CodeCheck_bar"
        var codeCheckFunc = Module.findExportByName("libfoo.so",
            "Java_sg_vantagepoint_uncrackable2_CodeCheck_bar");

        Interceptor.attach(codeCheckFunc, {
            onEnter: function(args) {
                // args[0] = JNIEnv*, args[1] = jobject, args[2] = jstring (input)
                var input = Java.vm.getEnv().getStringUtfChars(args[2], null).readUtf8String();
                console.log("[*] Code vérifié: " + input);
            },
            onLeave: function(retval) {
                console.log("[*] Résultat: " + retval);
                retval.replace(1); // forcer retour = true
            }
        });
    }
});
```

### Challenge 4 — Mini CTF maison

**Objectif** : Créer ton propre challenge mobile pour tes camarades.

```java
// Template de mini-challenge Android
// Cacher un flag dans plusieurs couches :

public class ChallengeActivity extends AppCompatActivity {

    // Couche 1 : Flag partiellement hardcodé
    private static final String PART1 = "CTF{m0b1l3_";

    // Couche 2 : Obfusqué en base64
    private static final String ENCODED_PART2 = "c2VjdXIxdHlf";  // "secur1ty_"

    // Couche 3 : Dans les ressources
    // res/values/strings.xml → <string name="hidden">4nd_fr1d4}</string>

    // Le flag complet : CTF{m0b1l3_secur1ty_4nd_fr1d4}

    public String revealFlag(String input) {
        if (input.equals("frida")) {
            String part2 = new String(Base64.decode(ENCODED_PART2, Base64.DEFAULT));
            String part3 = getString(R.string.hidden);
            return PART1 + part2 + part3;
        }
        return "Try harder...";
    }
}
```

---

## 13. Tableau de Synthèse — OWASP Mobile Top 10

| # | Vulnérabilité | Outil Principal | Sévérité | Contre-mesure |
|---|--------------|-----------------|----------|---------------|
| M1 | Credentials hardcodés | jadx, strings | Critique | Variables d'env, Keystore |
| M2 | Supply chain | MobSF, dependency scan | Haute | Audit deps, SBOM |
| M3 | Auth/Authz faible | Frida, Burp | Critique | Validation serveur, OAuth2 |
| M4 | Validation inputs | Burp, intents | Haute | Paramétrage SQL, sanitize |
| M5 | Comm non chiffrée | Burp Suite | Critique | TLS 1.3, cert pinning |
| M6 | Privacy controls | MobSF, trafic | Haute | RGPD, privacy by design |
| M7 | Binary protections | jadx, apktool | Moyenne | ProGuard R8, SafetyNet |
| M8 | Misconfiguration | apktool+script | Haute | Manifest audit, CIS benchmark |
| M9 | Stockage insécurisé | adb, sqlite3 | Critique | EncryptedPrefs, Keystore |
| M10 | Crypto faible | jadx, grep | Haute | AES-GCM, Argon2 |

---

## 14. Ressources et Pour Aller Plus Loin

```
Documentation officielle :
├── OWASP Mobile Application Security Testing Guide (MASTG)
│   https://mas.owasp.org/MASTG/
├── OWASP Mobile Application Security Verification Standard (MASVS)
│   https://mas.owasp.org/MASVS/
├── Android Security Documentation
│   https://developer.android.com/topic/security/best-practices
└── Android Developers — Security tips
    https://developer.android.com/training/articles/security-tips

Outils essentiels (tous open source) :
├── jadx : https://github.com/skylot/jadx
├── apktool : https://apktool.org/
├── Frida : https://frida.re/
├── Objection : https://github.com/sensepost/objection
├── MobSF : https://github.com/MobSF/Mobile-Security-Framework-MobSF
└── Drozer : https://github.com/WithSecureLabs/drozer

Labs et CTF :
├── OWASP MAS Crackmes : https://mas.owasp.org/crackmes/
├── HackTheBox Mobile track : https://app.hackthebox.com/tracks/mobile
├── DVAA / DVIA (Vulnerable Apps)
└── Hpandro CTF : https://hpandro1337.github.io/ctf/

Certifications :
├── GREM (GIAC Reverse Engineering Malware) — inclut mobile
├── eLearnSecurity eMAPT (Mobile Application Penetration Tester)
└── OSCP (pas spécifique mobile mais reconnu)
```

> [!tip] Progression recommandée Holberton
> 1. Maîtriser ADB et jadx (semaine 1)
> 2. Faire UnCrackable Levels 1, 2, 3 (semaine 2)
> 3. Installer Frida et bypasser SSL Pinning sur une vraie app (semaine 3)
> 4. Faire 3 challenges HTB Mobile (semaine 4)
> 5. Auditer une app open source (F-Droid) et écrire un rapport (semaine 5-6)
