# Introduction au Développement Mobile

## Pourquoi le mobile est différent

Le développement mobile n'est pas du développement web avec un écran plus petit. C'est un écosystème à part entière, avec des contraintes spécifiques, des modèles de distribution propres, et des attentes utilisateurs radicalement différentes.

> [!tip] Contexte chiffres 2024-2025
> - **6,8 milliards** de smartphones actifs dans le monde
> - **55 %** du trafic web mondial vient du mobile
> - **90 %** du temps sur mobile** passe dans des apps natives** (vs navigateur)
> - Apple App Store : 1,8 million d'apps — Google Play : 2,6 millions d'apps
> - Un utilisateur passe en moyenne **4h30/jour** sur son smartphone

---

## L'Écosystème Mobile : iOS vs Android

### Parts de marché mondiales

```
Marché mondial (2024) :
┌──────────────────────────────────────────────────┐
│  Android : 71,8 %  ████████████████████████████  │
│  iOS     : 27,6 %  ██████████                    │
│  Autres  :  0,6 %  ▌                             │
└──────────────────────────────────────────────────┘

Marché américain (penchant iOS) :
  Android : 44 %  ████████████████
  iOS     : 56 %  ████████████████████████

Marché indien (penchant Android) :
  Android : 95 %  █████████████████████████████████████
  iOS     :  5 %  ██
```

> [!info] Implication pour les devs
> En Europe et Amérique du Nord, ignorer iOS = perdre ~40-55 % des utilisateurs. Dans les marchés émergents (Inde, Asie du Sud-Est, Afrique), Android domine largement. Connaître sa cible géographique **avant** de choisir sa stratégie de développement est non-négociable.

### Les deux plateformes en détail

| Dimension | iOS (Apple) | Android (Google) |
|---|---|---|
| **Langage natif** | Swift (+ Objective-C legacy) | Kotlin (+ Java legacy) |
| **IDE officiel** | Xcode (macOS uniquement) | Android Studio (multi-plateforme) |
| **Store** | App Store | Google Play |
| **Validation store** | Review manuelle, 1-3 jours | Review semi-automatique, quelques heures |
| **Fragmentation** | Très faible (Apple contrôle tout) | Très élevée (des centaines de modèles) |
| **Monétisation** | Commission 15-30 % | Commission 15-30 % |
| **Sideloading** | Très restreint (EU : obligatoire depuis 2024) | Autorisé nativement |
| **Coût développeur** | 99 $/an (Apple Developer Program) | 25 $ une seule fois (Play Console) |
| **Architecture CPU** | ARM64 (Apple Silicon) | ARM64, ARM32, x86, x86_64 |
| **Mises à jour OS** | Adoption très rapide (>90 % sur iOS 16+) | Lente (fragmentation versions) |

> [!warning] La fragmentation Android
> Android doit tourner sur des téléphones à 50 € comme sur des flagships à 1 500 €. Un bug qui n'apparaît que sur un Samsung Galaxy A32 avec Android 12 modifié par Samsung (One UI) est un problème réel. iOS est bien plus prévisible car Apple contrôle entièrement le hardware et le logiciel.

---

## Les Stores : Distribution et Règles

### Apple App Store

L'App Store est la **seule** voie de distribution officielle iOS (hors Enterprise et TestFlight).

**Guidelines critiques :**
- Pas de contenu adult sans catégorie dédiée et contrôle parental
- Les apps doivent faire ce qu'elles prétendent faire (no dark patterns)
- **In-App Purchase obligatoire** via le système Apple pour tout achat numérique (commission 30 %, réduite à 15 % pour les petits développeurs)
- Privacy Nutrition Labels obligatoires (déclarer TOUTES les données collectées)
- **App Tracking Transparency (ATT)** : demander explicitement la permission de pister l'utilisateur
- Revue humaine : entre 24h et 3 jours

> [!warning] Raisons fréquentes de rejet App Store
> - Bugs ou crashes lors de la review (Apple teste vraiment l'app)
> - Liens brisés, placeholder content
> - Fonctionnalités cachées derrière un login impossible à tester
> - Duplication d'apps Apple natives sans valeur ajoutée claire
> - Violation des règles IAP (ex : bouton "Acheter" qui pointe vers un site web)

### Google Play Store

Plus permissif à l'entrée, mais surveillance continue.

**Politiques critiques :**
- **Target API level** : Google exige de cibler une API récente (dans les 12 mois de la version Android actuelle)
- **64-bit requirement** : toutes les apps doivent avoir des binaires 64 bits
- Déclarations de permissions : toute permission doit être justifiée
- **Data Safety section** : similaire aux Privacy Labels d'Apple
- Play Integrity API pour vérifier l'intégrité des appareils

```
Processus de publication :
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   Développer │───>│  Signer APK  │───>│  Upload Play │───>│  Review      │
│   et tester  │    │  / AAB       │    │  Console     │    │  (quelques h)│
└──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
```

> [!info] APK vs AAB
> **APK** (Android Package) : l'ancien format, contient tout pour toutes les architectures.
> **AAB** (Android App Bundle) : le nouveau format obligatoire sur Play Store. Google optimise le téléchargement en n'envoyant que les ressources nécessaires à l'appareil de l'utilisateur (20-30 % plus léger).

---

## Les Approches de Développement

Il n'existe pas une seule façon de faire du mobile. Chaque approche a ses compromis.

### 1. Développement Natif iOS (Swift)

```swift
// Swift — moderne, type-safe, compilé
import SwiftUI

struct ContentView: View {
    @State private var count = 0
    
    var body: some View {
        VStack(spacing: 20) {
            Text("Compteur : \(count)")
                .font(.largeTitle)
                .fontWeight(.bold)
            
            Button("Incrémenter") {
                count += 1
            }
            .buttonStyle(.borderedProminent)
        }
    }
}
```

**Avantages :**
- Performance maximale (accès direct aux APIs Apple)
- Accès immédiat aux nouvelles fonctionnalités iOS dès le jour J
- Meilleure intégration UX (animations, gestures, widgets iOS)
- Outils Apple (Xcode, Instruments) de grande qualité

**Inconvénients :**
- iOS uniquement
- Xcode requis = Mac obligatoire
- Courbe d'apprentissage SwiftUI/UIKit

### 2. Développement Natif Android (Kotlin)

```kotlin
// Kotlin — concis, null-safe, interopérable Java
import androidx.compose.material3.*
import androidx.compose.runtime.*

@Composable
fun CounterScreen() {
    var count by remember { mutableStateOf(0) }
    
    Column(
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center
    ) {
        Text(
            text = "Compteur : $count",
            style = MaterialTheme.typography.headlineLarge
        )
        Button(onClick = { count++ }) {
            Text("Incrémenter")
        }
    }
}
```

**Avantages :**
- Performance maximale sur Android
- Accès complet aux APIs Android
- Jetpack Compose (UI moderne déclarative)
- Développement possible sur Windows/Linux/Mac

**Inconvénients :**
- Android uniquement
- Fragmentation à gérer
- API souvent plus complexes qu'iOS

### 3. React Native (JavaScript/TypeScript)

```typescript
// React Native — bridge JavaScript → natif
import React, { useState } from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';

export default function CounterScreen() {
  const [count, setCount] = useState(0);
  
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Compteur : {count}</Text>
      <TouchableOpacity 
        style={styles.button}
        onPress={() => setCount(count + 1)}
      >
        <Text style={styles.buttonText}>Incrémenter</Text>
      </TouchableOpacity>
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, justifyContent: 'center', alignItems: 'center' },
  title: { fontSize: 28, fontWeight: 'bold', marginBottom: 20 },
  button: { backgroundColor: '#007AFF', padding: 16, borderRadius: 8 },
  buttonText: { color: 'white', fontSize: 16 },
});
```

**Architecture React Native :**
```
JavaScript Engine (Hermes)
         ↓
     Bridge JS ↔ Natif  (ancienne architecture)
         ↓
   Composants natifs (UIKit / Android Views)

Nouvelle architecture (JSI - 2024) :
JavaScript → JSI → C++ Host Objects → Code natif
(pas de sérialisation JSON, synchrone, bien plus rapide)
```

### 4. Flutter (Dart)

Flutter est détaillé dans [[02 - Flutter Complet]]. En résumé : Flutter ne pas des composants natifs — il dessine **tout lui-même** avec son moteur Skia/Impeller. Cela donne une cohérence visuelle parfaite cross-platform mais une intégration visuelle légèrement différente des guidelines iOS/Android.

### 5. WebView Apps (Ionic, Capacitor, Cordova)

```
Application = WebView (navigateur embarqué) + plugins natifs

Architecture :
┌─────────────────────────────────────────┐
│          Application Native (shell)     │
│  ┌───────────────────────────────────┐  │
│  │         WebView (Chrome/WKWebView)│  │
│  │  ┌─────────────────────────────┐  │  │
│  │  │   Votre code Web            │  │  │
│  │  │   HTML + CSS + JavaScript   │  │  │
│  │  └─────────────────────────────┘  │  │
│  └───────────────────────────────────┘  │
│         ↕ Plugins (caméra, GPS...)      │
└─────────────────────────────────────────┘
```

Adapté pour : apps simples avec équipe web existante, MVPs rapides, apps peu intensives.

---

## Comparaison des Approches

| Critère | Natif iOS | Natif Android | React Native | Flutter | WebView |
|---|---|---|---|---|---|
| **Performance** | ★★★★★ | ★★★★★ | ★★★★☆ | ★★★★☆ | ★★☆☆☆ |
| **Partage de code** | 0 % | 0 % | 70-90 % | 90-95 % | 90-100 % |
| **Accès hardware** | ★★★★★ | ★★★★★ | ★★★★☆ | ★★★☆☆ | ★★☆☆☆ |
| **Coût développement** | Élevé × 2 | Élevé × 2 | Moyen | Moyen | Faible |
| **Look & Feel natif** | ★★★★★ | ★★★★★ | ★★★★☆ | ★★★☆☆ | ★★☆☆☆ |
| **Courbe apprentissage** | Steep | Steep | Modérée (si React) | Modérée | Faible |
| **Maintenance** | Élevée × 2 | Élevée × 2 | Modérée | Modérée | Faible |
| **Mises à jour iOS/Android** | Immédiat | Immédiat | Délai | Délai | Délai |
| **Taille équipe idéale** | Grande | Grande | Petite/Moyenne | Petite/Moyenne | Petite |

> [!tip] Comment choisir ?
> - **Startup MVP** → Flutter ou React Native (rapidité, partage de code)
> - **App critique performance** (jeux, audio, AR) → Natif
> - **Grande équipe avec spécialistes** → Natif sur chaque plateforme
> - **Équipe web existante** → React Native
> - **Design très personnalisé cross-platform** → Flutter
> - **App interne entreprise simple** → WebView/Ionic

---

## Concepts Universels du Mobile

### Lifecycle d'une Application Mobile

Toutes les apps mobiles partagent des états de cycle de vie similaires :

```
iOS (SwiftUI) :                    Android (Jetpack) :

  Active (Foreground)                Active (Resumed)
       ↑ ↓                               ↑ ↓
  Inactive                           Paused
       ↑ ↓                               ↑ ↓
  Background                         Stopped
       ↑ ↓                               ↑ ↓
  Suspended                          Destroyed
       ↑ ↓
  Not Running
```

**Points critiques :**
- `Background` / `Stopped` : sauvegarder l'état, arrêter les opérations lourdes, libérer les ressources
- `Foreground` / `Resumed` : reprendre les connexions, rafraîchir les données
- L'OS peut **tuer** votre app à tout moment pour libérer de la mémoire

```kotlin
// Android - gestion du lifecycle
class MainActivity : AppCompatActivity() {
    
    override fun onResume() {
        super.onResume()
        // App visible et interactive → démarrer caméra, GPS, etc.
        startLocationUpdates()
    }
    
    override fun onPause() {
        super.onPause()
        // App partiellement visible → sauvegarder données non sauvegardées
        saveUserProgress()
    }
    
    override fun onStop() {
        super.onStop()
        // App complètement invisible → arrêter les opérations lourdes
        stopLocationUpdates()
        releaseNetworkConnections()
    }
}
```

### Navigation Mobile

Deux paradigmes dominants :

```
Stack Navigation :
Screen A → Screen B → Screen C
              ←Back         ←Back

Tab Navigation :
┌──────────────────────────────┐
│         Contenu              │
│                              │
├──────┬──────┬──────┬─────────┤
│ Home │Search│ Cart │ Profile │  ← Tab Bar (iOS) / Bottom Nav (Android)
└──────┴──────┴──────┴─────────┘

Drawer Navigation :
┌──────────────────────────────┐
│ ≡  Mon App              🔔  │
│                              │
│  Contenu principal            │
│                              │
└──────────────────────────────┘
Swipe gauche → menu latéral
```

### Gestion d'État et Offline-First

Le mobile doit **fonctionner sans connexion**. Le pattern offline-first :

```
Architecture Offline-First :

  Réseau  ─────────────────────────────────────────>  API Server
              ↑ sync quand disponible ↓
  App     ←──────────────── Cache Local (SQLite/Room/CoreData)
              ↑ lit toujours du cache ↓
  UI      ←─────────── Source de vérité locale
```

**Stratégies de cache :**
- **Cache-First** : lire le cache, mettre à jour en arrière-plan (emails, actualités)
- **Network-First** : tenter le réseau, fallback cache (données critiques)
- **Cache-Only** : jamais de réseau (données statiques, préférences)
- **Network-Only** : pas de cache (paiements, données temps réel)

### Push Notifications

```
Architecture Push Notifications :

  Votre Serveur Backend
         ↓  (envoi de la notification via SDK)
  APNs (Apple Push Notification service)   ←── iOS uniquement
  FCM  (Firebase Cloud Messaging)          ←── Android + iOS possible
         ↓
  Appareil de l'utilisateur
         ↓
  Notification affichée dans l'OS
```

```javascript
// Exemple : envoi d'une notification via FCM (Node.js)
const admin = require('firebase-admin');

async function envoyerNotification(token, titre, corps) {
  const message = {
    token: token,
    notification: {
      title: titre,
      body: corps,
    },
    android: {
      priority: 'high',
    },
    apns: {
      payload: {
        aps: {
          sound: 'default',
          badge: 1,
        },
      },
    },
  };
  
  const response = await admin.messaging().send(message);
  console.log('Notification envoyée :', response);
}
```

### Permissions

Les permissions doivent être demandées **au moment où l'utilisateur en a besoin** (contextual permission request), jamais au démarrage de l'app.

```
Mauvaise pratique :
App ouvre → demande IMMÉDIATEMENT accès aux contacts, localisation, caméra
↓
L'utilisateur refuse tout par peur → app non fonctionnelle

Bonne pratique :
Utilisateur clique "Prendre une photo" → demander la caméra
Utilisateur active le tracking → demander la localisation
```

| Catégorie de permission | iOS | Android |
|---|---|---|
| **Localisation** | `NSLocationWhenInUseUsageDescription` | `ACCESS_FINE_LOCATION` |
| **Caméra** | `NSCameraUsageDescription` | `CAMERA` |
| **Contacts** | `NSContactsUsageDescription` | `READ_CONTACTS` |
| **Notifications** | Demande runtime | Depuis Android 13 : `POST_NOTIFICATIONS` |
| **Microphone** | `NSMicrophoneUsageDescription` | `RECORD_AUDIO` |

### Deep Linking

Le deep linking permet d'ouvrir une page spécifique d'une app depuis un lien :

```
https://monapp.com/produit/123
         ↓  (Universal Link iOS / App Link Android)
App "MonApp" → page produit #123

Schéma custom :
monapp://produit/123
```

**Universal Links (iOS) / App Links (Android)** sont préférables aux schémas custom car :
- Fonctionnent aussi sur le web si l'app n'est pas installée
- Plus sécurisés (vérification de propriété du domaine)

### Responsive Mobile vs Desktop

```css
/* Le mobile d'abord (Mobile-First) — pour le web mobile */
.container {
  /* Base : mobile */
  padding: 16px;
  font-size: 16px;
}

@media (min-width: 768px) {
  /* Tablette */
  .container {
    padding: 24px;
    max-width: 768px;
    margin: 0 auto;
  }
}

@media (min-width: 1024px) {
  /* Desktop */
  .container {
    padding: 32px;
    max-width: 1200px;
  }
}
```

En natif, les **breakpoints** sont gérés différemment :
- iOS : `UITraitCollection.horizontalSizeClass` (Compact / Regular)
- Android : qualificateurs de ressources (`layout-sw600dp/`)
- Flutter : `MediaQuery.of(context).size.width`

---

## Publication sur les Stores

### App Store (iOS) — Checklist complète

```
1. Compte Apple Developer (99 $/an)
   └── Accès à Xcode, TestFlight, App Store Connect

2. Certificats et Provisioning Profiles
   ├── Development Certificate  → tests en local
   ├── Distribution Certificate → soumission App Store
   └── Provisioning Profile → lie app + certificat + appareils

3. App Store Connect
   ├── Bundle ID unique (com.monentreprise.monapp)
   ├── Screenshots pour chaque taille d'écran requise
   │   ├── iPhone 6.7" (obligatoire)
   │   ├── iPad Pro 12.9" (si l'app supporte iPad)
   │   └── etc.
   ├── Description (localisation recommandée)
   ├── Keywords (100 caractères max — ASO)
   ├── Privacy Policy URL (OBLIGATOIRE)
   ├── App Privacy → Data Nutrition Labels
   └── Age Rating

4. Soumission
   ├── Archive dans Xcode
   ├── Upload via Xcode ou Transporter
   ├── Sélection de la build dans App Store Connect
   └── Soumission pour review
```

### Google Play (Android) — Checklist

```
1. Compte Google Play Console (25 $ une fois)

2. Signature de l'app
   ├── Keystore (fichier .jks) — GARDER PRÉCIEUSEMENT
   │   keytool -genkey -v -keystore release.jks \
   │           -alias monapp -keyalg RSA -keysize 2048
   ├── Play App Signing (Google gère la clé finale)
   └── Upload Key (vous signez votre AAB avec ça)

3. Build Release
   ./gradlew bundleRelease  # génère un .aab
   # Signer avec l'upload key

4. Play Console
   ├── Listing (titre, description, icône, screenshots)
   ├── Content rating (questionnaire)
   ├── Data Safety (équivalent Privacy Labels)
   ├── Pricing (gratuit/payant/freemium)
   └── Pays cibles

5. Canaux de distribution
   ├── Internal testing (20 testeurs max)
   ├── Closed testing (alpha — liste d'emails)
   ├── Open testing (beta publique)
   └── Production (rollout progressif : 10% → 50% → 100%)
```

> [!tip] Rollout progressif
> Ne jamais déployer à 100 % immédiatement en production. Commencer à 10 %, surveiller les crashs et les avis, puis augmenter progressivement. Si un bug critique est détecté, stopper le rollout avant qu'il n'atteigne tout le monde.

### ASO — App Store Optimization

L'ASO est le SEO des stores. Facteurs de classement :

| Facteur | Poids | Action |
|---|---|---|
| **Titre** | Très élevé | Inclure le keyword principal |
| **Keywords / Subtitle** | Élevé | Recherche de mots-clés compétitifs |
| **Avis et notes** | Très élevé | Demander les avis au bon moment |
| **Téléchargements** | Élevé | Campagnes d'acquisition |
| **Rétention** | Élevé | UX soignée, onboarding clair |
| **Screenshots** | Conversion | A/B tester les visuels |
| **Mises à jour régulières** | Modéré | Signale une app active |

---

## Exercices Pratiques

### Exercice 1 — Analyse comparative (30 min)
Choisissez une app populaire (Instagram, Spotify, Google Maps) et répondez :
1. Quelle approche de développement pensez-vous qu'ils utilisent et pourquoi ?
2. Listez 5 fonctionnalités qui nécessitent des permissions spécifiques
3. Décrivez comment fonctionne leur navigation (stack, tabs, drawer ?)
4. L'app fonctionne-t-elle offline ? Quelles données sont disponibles sans connexion ?

### Exercice 2 — Arbre de décision (20 min)
Vous êtes CTO d'une startup. On vous demande de choisir une approche pour ces 3 projets :
- **Projet A** : App bancaire avec authentification biométrique, sécurité maximale, budget 500k€
- **Projet B** : Réseau social de photos pour une communauté de photographes, équipe de 2 devs
- **Projet C** : App interne de gestion de stock pour 200 employés dans une usine, budget serré

Pour chaque projet : choisir l'approche + justifier en 3 bullet points.

### Exercice 3 — Compliance Store (45 min)
Prenez une idée d'app de votre choix. Remplissez :
1. La liste des permissions nécessaires et leur justification contextuelle
2. La description App Store (500 caractères max) + 5 keywords
3. Les raisons possibles de rejet par Apple (au moins 3 risques identifiés)
4. Votre stratégie de rollout : à quel pourcentage commencez-vous et quelles métriques surveillez-vous ?

### Exercice 4 — Architecture Offline (1h)
Dessinez le schéma d'architecture d'une app de liste de courses offline-first :
- Les données doivent être disponibles sans connexion
- La synchronisation se fait dès que le réseau revient
- Plusieurs utilisateurs peuvent partager la même liste
Précisez : quelle stratégie de cache, où est la source de vérité, comment gérer les conflits de synchronisation ?

> [!info] Aller plus loin
> - **Lecture** : "Programming iOS" de Matt Neuburg — référence iOS complète
> - **Pratique** : Créer un compte développeur gratuit (sandbox) pour explorer App Store Connect
> - **Outils** : Android Studio est gratuit, Xcode nécessite macOS (ou une VM Hackintosh)
> - **Communautés** : r/iOSProgramming, r/androiddev, Stack Overflow mobile tags
