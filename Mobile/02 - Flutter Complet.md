# Flutter Complet

## Qu'est-ce que Flutter ?

**Flutter** est le framework de Google pour créer des applications natives compilées pour mobile (iOS, Android), web, desktop et embarqué — à partir d'un seul codebase Dart. Lancé en 2018, il est utilisé par Google Pay, BMW, eBay, et des milliers de startups.

> [!tip] Ce qui rend Flutter unique
> Contrairement à React Native qui utilise des composants natifs, Flutter **dessine tout lui-même** via son moteur graphique (Skia, remplacé par Impeller depuis Flutter 3.10). C'est comme avoir un moteur de jeu vidéo intégré : performance maximale, pixel-perfect identique sur iOS et Android, mais style légèrement différent des guidelines natives.

---

## Dart — Le Langage de Flutter

### Pourquoi Dart ?

Google avait le choix. Ils auraient pu utiliser JavaScript, Kotlin, ou même C++. Ils ont choisi Dart pour des raisons techniques précises :

| Raison | Explication |
|---|---|
| **AOT + JIT** | Dart compile en natif (AOT) pour les releases, et en interprété (JIT) pour le développement (hot reload) |
| **Null safety** | Système de types qui élimine les NullPointerException au compile-time |
| **Syntaxe familière** | Ressemble à Java/Kotlin/Swift — courbe d'apprentissage faible |
| **Dart VM** | VM très performante, GC efficace pour les UIs |
| **Single-threaded event loop** | Modèle async simple, pas de callback hell |

### Syntaxe Dart — Bases Essentielles

```dart
// === TYPES ET VARIABLES ===
// Types fondamentaux
int age = 25;
double prix = 19.99;
String nom = "Alice";
bool estConnecte = true;
List<String> tags = ['flutter', 'dart', 'mobile'];
Map<String, int> scores = {'Alice': 95, 'Bob': 87};

// var = inférence de type
var message = "Hello"; // String inféré
final url = "https://api.example.com"; // const runtime (ne peut pas changer)
const pi = 3.14159; // const compile-time (vraie constante)

// === NULL SAFETY ===
String? prenom = null;   // nullable (avec ?)
String nomSure = "Bob";  // non-nullable (sans ?)

// Opérateur ?? (null-aware)
String affichage = prenom ?? "Anonyme"; // "Anonyme" si prenom est null

// Opérateur ?. (null-conditional)
int? longueur = prenom?.length; // null si prenom est null

// Opérateur ! (force unwrap — dangereux)
String valeur = prenom!; // Lance une exception si null

// === FONCTIONS ===
// Fonction simple
int addition(int a, int b) => a + b; // arrow function

// Paramètres nommés (très utilisés en Flutter)
Widget bouton({
  required String label,   // obligatoire
  VoidCallback? onPressed, // optionnel nullable
  Color color = Colors.blue, // optionnel avec valeur par défaut
}) {
  // ...
}

// Appel
bouton(label: "Valider", color: Colors.green);

// === CLASSES ===
class Utilisateur {
  final String id;
  String nom;
  String? email;
  
  // Constructeur avec paramètres nommés
  const Utilisateur({
    required this.id,
    required this.nom,
    this.email,
  });
  
  // Constructeur factory (pattern courant avec JSON)
  factory Utilisateur.fromJson(Map<String, dynamic> json) {
    return Utilisateur(
      id: json['id'] as String,
      nom: json['nom'] as String,
      email: json['email'] as String?,
    );
  }
  
  // Méthode
  Map<String, dynamic> toJson() => {
    'id': id,
    'nom': nom,
    if (email != null) 'email': email,
  };
  
  // toString pour le debug
  @override
  String toString() => 'Utilisateur(id: $id, nom: $nom)';
}
```

### Async/Await en Dart

```dart
import 'dart:convert';
import 'package:http/http.dart' as http;

// Future = valeur qui sera disponible plus tard (comme une Promise JS)
Future<String> fetchUserName(String id) async {
  try {
    final response = await http.get(
      Uri.parse('https://api.example.com/users/$id'),
    );
    
    if (response.statusCode == 200) {
      final data = jsonDecode(response.body) as Map<String, dynamic>;
      return data['nom'] as String;
    } else {
      throw Exception('Erreur HTTP ${response.statusCode}');
    }
  } catch (e) {
    throw Exception('Impossible de récupérer l\'utilisateur : $e');
  }
}

// Stream = séquence de valeurs asynchrones (comme un Observable RxJS)
Stream<int> compteur() async* {
  for (int i = 0; i < 10; i++) {
    await Future.delayed(const Duration(seconds: 1));
    yield i; // émet une valeur
  }
}

// Utilisation
void main() async {
  final nom = await fetchUserName('123');
  print('Nom : $nom');
  
  // Écouter un stream
  await for (final valeur in compteur()) {
    print('Tick : $valeur');
  }
}
```

---

## Installation et Configuration

### Prérequis

```bash
# 1. Télécharger Flutter SDK
# https://docs.flutter.dev/get-started/install
# Choisir Windows / macOS / Linux

# 2. Extraire et ajouter au PATH
export PATH="$PATH:/home/user/flutter/bin"  # Linux/macOS
# Windows : ajouter via Paramètres système → Variables d'environnement

# 3. Vérifier l'installation
flutter doctor
```

**Sortie typique de `flutter doctor` :**
```
Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel stable, 3.22.0)
[✓] Android toolchain - develop for Android devices (Android SDK 34)
[✓] Xcode - develop for iOS and macOS (Xcode 15.4)
[✓] Android Studio (version 2024.1)
[✓] VS Code (version 1.89.0)
[✓] Connected device (2 available)
[✓] Network resources
```

### Créer un Projet

```bash
# Créer un nouveau projet
flutter create mon_app
flutter create --org com.monentreprise mon_app  # avec package name custom

cd mon_app

# Structure du projet
mon_app/
├── lib/
│   └── main.dart        ← Point d'entrée
├── test/
│   └── widget_test.dart ← Tests
├── android/             ← Config Android native
├── ios/                 ← Config iOS native
├── web/                 ← Config web
├── pubspec.yaml         ← Dépendances (équivalent package.json)
└── pubspec.lock         ← Lock file

# Lancer l'app
flutter run              # sur le device/émulateur connecté
flutter run -d chrome    # en mode web
```

### Hot Reload vs Hot Restart

```
Hot Reload (r) :
  Injecte le nouveau code dans la VM en cours
  ✓ Conserve l'état de l'app (formulaires remplis, navigation)
  ✓ < 1 seconde
  ✗ Ne recharge pas les initState() ni les static variables

Hot Restart (R ou Shift+R) :
  Redémarre la VM entièrement
  ✓ Rechargement complet
  ✗ Perd l'état de l'app
  ✓ ~2-3 secondes
```

---

## Architecture Flutter : Le Widget Tree

### Tout est Widget

En Flutter, **tout** est un Widget : un bouton, un padding, une mise en page, une animation, une couleur de fond. L'UI est un arbre de widgets imbriqués.

```
MaterialApp
└── Scaffold
    ├── AppBar
    │   └── Text("Mon App")
    ├── Body: Column
    │   ├── Container
    │   │   └── Text("Bonjour !")
    │   ├── SizedBox(height: 16)
    │   └── ElevatedButton
    │       └── Text("Cliquer")
    └── BottomNavigationBar
        ├── BottomNavigationBarItem(icon: Icon(Icons.home))
        └── BottomNavigationBarItem(icon: Icon(Icons.settings))
```

### BuildContext

```dart
// BuildContext = localisation du widget dans l'arbre
// Il permet d'accéder aux données de ses ancêtres (Theme, MediaQuery, etc.)

Widget build(BuildContext context) {
  // Accéder au thème
  final theme = Theme.of(context);
  final couleurPrimaire = theme.colorScheme.primary;
  
  // Accéder aux dimensions de l'écran
  final taille = MediaQuery.of(context).size;
  final largeur = taille.width;
  
  // Navigation
  Navigator.of(context).push(...);
  
  // Localisation
  // final l10n = AppLocalizations.of(context)!;
  
  return Container(
    color: couleurPrimaire,
    width: largeur * 0.8,
  );
}
```

### StatelessWidget vs StatefulWidget

```dart
// STATELESS : ne change jamais après construction
// Utiliser quand le widget ne dépend que de ses paramètres
class CarteUtilisateur extends StatelessWidget {
  final String nom;
  final String? avatar;
  
  const CarteUtilisateur({
    super.key,
    required this.nom,
    this.avatar,
  });
  
  @override
  Widget build(BuildContext context) {
    return Card(
      child: ListTile(
        leading: CircleAvatar(
          backgroundImage: avatar != null ? NetworkImage(avatar!) : null,
          child: avatar == null ? Text(nom[0]) : null,
        ),
        title: Text(nom),
      ),
    );
  }
}

// STATEFUL : contient un état mutable qui provoque des rebuilds
// Utiliser quand le widget doit changer en réponse à des interactions
class Compteur extends StatefulWidget {
  const Compteur({super.key});

  @override
  State<Compteur> createState() => _CompteurState();
}

class _CompteurState extends State<Compteur> {
  int _count = 0;
  
  // Cycle de vie :
  @override
  void initState() {
    super.initState();
    // Appelé une seule fois à la création
    // Initialiser les controllers, démarrer des listeners
  }
  
  @override
  void dispose() {
    // Appelé à la destruction — libérer les ressources
    // Penser à dispose() les TextEditingController, AnimationController, etc.
    super.dispose();
  }
  
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('$_count', style: const TextStyle(fontSize: 48)),
        ElevatedButton(
          onPressed: () {
            setState(() {
              // setState déclenche un rebuild du widget
              _count++;
            });
          },
          child: const Text('Incrémenter'),
        ),
      ],
    );
  }
}
```

---

## Widgets Essentiels

### Layout

```dart
// ROW : dispose les enfants horizontalement
Row(
  mainAxisAlignment: MainAxisAlignment.spaceBetween,
  crossAxisAlignment: CrossAxisAlignment.center,
  children: [
    Icon(Icons.star),
    Text('4.8'),
    const Spacer(), // occupe tout l'espace disponible
    Text('(127 avis)'),
  ],
)

// COLUMN : dispose les enfants verticalement
Column(
  mainAxisSize: MainAxisSize.min, // prend le minimum de hauteur
  children: [/* ... */],
)

// STACK : superpose les widgets (comme position: absolute en CSS)
Stack(
  children: [
    Image.network(imageUrl),
    Positioned(
      bottom: 8,
      right: 8,
      child: Chip(label: Text('Nouveau')),
    ),
  ],
)

// CONTAINER : boîte polyvalente (padding, margin, décoration, taille)
Container(
  margin: const EdgeInsets.all(16),
  padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 12),
  width: double.infinity,
  decoration: BoxDecoration(
    color: Colors.blue.shade50,
    borderRadius: BorderRadius.circular(12),
    border: Border.all(color: Colors.blue, width: 1),
    boxShadow: [
      BoxShadow(
        color: Colors.black.withOpacity(0.1),
        blurRadius: 8,
        offset: const Offset(0, 2),
      ),
    ],
  ),
  child: Text('Contenu'),
)

// EXPANDED / FLEXIBLE : dans un Row/Column
Row(
  children: [
    Expanded(flex: 2, child: Text('2/3')),  // prend 2/3
    Expanded(flex: 1, child: Text('1/3')),  // prend 1/3
  ],
)
```

### Listes et Grilles

```dart
// ListView.builder : performant pour les longues listes (lazy loading)
ListView.builder(
  itemCount: articles.length,
  itemBuilder: (context, index) {
    final article = articles[index];
    return ListTile(
      leading: CircleAvatar(child: Text('${index + 1}')),
      title: Text(article.titre),
      subtitle: Text(article.auteur),
      trailing: const Icon(Icons.chevron_right),
      onTap: () => Navigator.push(
        context,
        MaterialPageRoute(builder: (_) => DetailArticle(article: article)),
      ),
    );
  },
)

// GridView.builder : grille performante
GridView.builder(
  gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
    crossAxisCount: 2,
    mainAxisSpacing: 12,
    crossAxisSpacing: 12,
    childAspectRatio: 0.75,
  ),
  itemCount: produits.length,
  itemBuilder: (context, index) => CarteProduit(produit: produits[index]),
)
```

### Scaffold, AppBar, BottomNavigationBar

```dart
class AccueilPage extends StatefulWidget {
  const AccueilPage({super.key});
  @override
  State<AccueilPage> createState() => _AccueilPageState();
}

class _AccueilPageState extends State<AccueilPage> {
  int _indexOnglet = 0;
  
  final List<Widget> _pages = const [
    PageAccueil(),
    PageRecherche(),
    PageProfil(),
  ];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Mon App'),
        backgroundColor: Theme.of(context).colorScheme.primaryContainer,
        actions: [
          IconButton(
            icon: const Icon(Icons.notifications),
            onPressed: () {},
          ),
        ],
      ),
      body: _pages[_indexOnglet],
      bottomNavigationBar: NavigationBar(
        selectedIndex: _indexOnglet,
        onDestinationSelected: (index) => setState(() => _indexOnglet = index),
        destinations: const [
          NavigationDestination(icon: Icon(Icons.home), label: 'Accueil'),
          NavigationDestination(icon: Icon(Icons.search), label: 'Recherche'),
          NavigationDestination(icon: Icon(Icons.person), label: 'Profil'),
        ],
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {},
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

### Formulaires

```dart
class FormulaireConnexion extends StatefulWidget {
  const FormulaireConnexion({super.key});
  @override
  State<FormulaireConnexion> createState() => _FormulaireConnexionState();
}

class _FormulaireConnexionState extends State<FormulaireConnexion> {
  final _formKey = GlobalKey<FormState>();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  bool _passwordVisible = false;
  
  @override
  void dispose() {
    _emailController.dispose();   // TOUJOURS dispose les controllers
    _passwordController.dispose();
    super.dispose();
  }
  
  void _soumettre() {
    if (_formKey.currentState!.validate()) {
      // Formulaire valide
      print('Email : ${_emailController.text}');
    }
  }

  @override
  Widget build(BuildContext context) {
    return Form(
      key: _formKey,
      child: Column(
        children: [
          TextFormField(
            controller: _emailController,
            keyboardType: TextInputType.emailAddress,
            decoration: const InputDecoration(
              labelText: 'Email',
              prefixIcon: Icon(Icons.email),
              border: OutlineInputBorder(),
            ),
            validator: (value) {
              if (value == null || value.isEmpty) return 'Email requis';
              if (!value.contains('@')) return 'Email invalide';
              return null; // null = valide
            },
          ),
          const SizedBox(height: 16),
          TextFormField(
            controller: _passwordController,
            obscureText: !_passwordVisible,
            decoration: InputDecoration(
              labelText: 'Mot de passe',
              prefixIcon: const Icon(Icons.lock),
              border: const OutlineInputBorder(),
              suffixIcon: IconButton(
                icon: Icon(_passwordVisible ? Icons.visibility_off : Icons.visibility),
                onPressed: () => setState(() => _passwordVisible = !_passwordVisible),
              ),
            ),
            validator: (value) {
              if (value == null || value.length < 8) {
                return 'Minimum 8 caractères';
              }
              return null;
            },
          ),
          const SizedBox(height: 24),
          ElevatedButton(
            onPressed: _soumettre,
            style: ElevatedButton.styleFrom(
              minimumSize: const Size(double.infinity, 48),
            ),
            child: const Text('Se connecter'),
          ),
        ],
      ),
    );
  }
}
```

---

## Gestion d'État

### setState — Pour l'état local simple

Voir l'exemple `Compteur` ci-dessus. Adapté pour : états qui concernent un seul widget.

### Provider — Gestion d'état globale légère

```yaml
# pubspec.yaml
dependencies:
  provider: ^6.1.2
```

```dart
// 1. Créer un ChangeNotifier (le "store")
class PanierProvider extends ChangeNotifier {
  final List<Produit> _articles = [];
  
  List<Produit> get articles => List.unmodifiable(_articles);
  int get total => _articles.fold(0, (sum, p) => sum + p.prix);
  
  void ajouter(Produit produit) {
    _articles.add(produit);
    notifyListeners(); // déclenche le rebuild des widgets qui écoutent
  }
  
  void supprimer(String id) {
    _articles.removeWhere((p) => p.id == id);
    notifyListeners();
  }
}

// 2. Fournir le provider en haut de l'arbre
void main() {
  runApp(
    ChangeNotifierProvider(
      create: (_) => PanierProvider(),
      child: const MonApp(),
    ),
  );
}

// 3. Consommer dans un widget
class BadgePanier extends StatelessWidget {
  const BadgePanier({super.key});
  
  @override
  Widget build(BuildContext context) {
    // Rebuild automatique quand notifyListeners() est appelé
    final panier = context.watch<PanierProvider>();
    
    return Badge(
      count: panier.articles.length,
      child: const Icon(Icons.shopping_cart),
    );
  }
}

// Pour accéder sans rebuild (ex : dans un callback)
context.read<PanierProvider>().ajouter(produit);
```

### Riverpod — Provider moderne et type-safe

```dart
// Riverpod est plus moderne que Provider
// Pas de BuildContext requis, meilleur pour les tests

import 'package:flutter_riverpod/flutter_riverpod.dart';

// Définir un provider
final compteurProvider = StateNotifierProvider<CompteurNotifier, int>((ref) {
  return CompteurNotifier();
});

class CompteurNotifier extends StateNotifier<int> {
  CompteurNotifier() : super(0);
  
  void incrementer() => state++;
  void decrementer() => state--;
}

// Utiliser dans un widget (ConsumerWidget au lieu de StatelessWidget)
class PageCompteur extends ConsumerWidget {
  const PageCompteur({super.key});
  
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(compteurProvider);
    
    return Column(children: [
      Text('$count'),
      ElevatedButton(
        onPressed: () => ref.read(compteurProvider.notifier).incrementer(),
        child: const Text('+'),
      ),
    ]);
  }
}
```

---

## Navigation avec go_router

```yaml
dependencies:
  go_router: ^14.2.7
```

```dart
// Configuration des routes
final router = GoRouter(
  initialLocation: '/',
  routes: [
    GoRoute(
      path: '/',
      builder: (context, state) => const PageAccueil(),
      routes: [
        GoRoute(
          path: 'article/:id',
          builder: (context, state) {
            final id = state.pathParameters['id']!;
            return PageArticle(id: id);
          },
        ),
      ],
    ),
    GoRoute(
      path: '/profil',
      builder: (context, state) => const PageProfil(),
    ),
    GoRoute(
      path: '/connexion',
      builder: (context, state) => const PageConnexion(),
    ),
  ],
  // Redirection (ex: auth guard)
  redirect: (context, state) {
    final estConnecte = ref.read(authProvider).estConnecte;
    if (!estConnecte && state.matchedLocation != '/connexion') {
      return '/connexion';
    }
    return null; // pas de redirection
  },
);

// Navigation
context.go('/article/42');           // remplace l'historique
context.push('/profil');             // empile sur l'historique
context.pop();                        // retour
context.go('/article/${article.id}'); // avec paramètre
```

---

## Appels API avec Dio

```yaml
dependencies:
  dio: ^5.6.0
```

```dart
// Service API centralisé
class ApiService {
  late final Dio _dio;
  
  ApiService() {
    _dio = Dio(BaseOptions(
      baseUrl: 'https://api.monapp.com/v1',
      connectTimeout: const Duration(seconds: 10),
      receiveTimeout: const Duration(seconds: 30),
    ));
    
    // Intercepteur pour les tokens d'auth
    _dio.interceptors.add(InterceptorsWrapper(
      onRequest: (options, handler) async {
        final token = await _getToken();
        options.headers['Authorization'] = 'Bearer $token';
        handler.next(options);
      },
      onError: (error, handler) {
        if (error.response?.statusCode == 401) {
          // Logout automatique si token expiré
          logout();
        }
        handler.next(error);
      },
    ));
  }
  
  Future<List<Article>> getArticles({int page = 1}) async {
    try {
      final response = await _dio.get('/articles', 
        queryParameters: {'page': page, 'limit': 20},
      );
      final data = response.data as List<dynamic>;
      return data.map((json) => Article.fromJson(json)).toList();
    } on DioException catch (e) {
      throw _handleError(e);
    }
  }
  
  Exception _handleError(DioException e) {
    switch (e.type) {
      case DioExceptionType.connectionTimeout:
        return Exception('Connexion timeout');
      case DioExceptionType.receiveTimeout:
        return Exception('Serveur trop lent');
      case DioExceptionType.badResponse:
        return Exception('Erreur ${e.response?.statusCode}');
      default:
        return Exception('Erreur réseau');
    }
  }
}

// Afficher les données avec FutureBuilder
class ListeArticles extends StatelessWidget {
  const ListeArticles({super.key});
  
  @override
  Widget build(BuildContext context) {
    return FutureBuilder<List<Article>>(
      future: ApiService().getArticles(),
      builder: (context, snapshot) {
        if (snapshot.connectionState == ConnectionState.waiting) {
          return const Center(child: CircularProgressIndicator());
        }
        
        if (snapshot.hasError) {
          return Center(
            child: Column(children: [
              const Icon(Icons.error, size: 48, color: Colors.red),
              Text('Erreur : ${snapshot.error}'),
              ElevatedButton(
                onPressed: () => setState(() {}), // retry
                child: const Text('Réessayer'),
              ),
            ]),
          );
        }
        
        final articles = snapshot.data!;
        return ListView.builder(
          itemCount: articles.length,
          itemBuilder: (context, i) => CarteArticle(article: articles[i]),
        );
      },
    );
  }
}
```

---

## Persistance des Données

### shared_preferences — Données simples

```dart
import 'package:shared_preferences/shared_preferences.dart';

// Sauvegarder
final prefs = await SharedPreferences.getInstance();
await prefs.setString('token', 'eyJhbGci...');
await prefs.setBool('onboarding_done', true);
await prefs.setInt('theme_index', 1);

// Lire
final token = prefs.getString('token');
final onboardingFait = prefs.getBool('onboarding_done') ?? false;

// Supprimer
await prefs.remove('token');
await prefs.clear(); // tout supprimer
```

### Hive — Base de données NoSQL rapide

```dart
// Parfait pour les modèles simples sans relations
import 'package:hive_flutter/hive_flutter.dart';

// Initialisation
await Hive.initFlutter();
final box = await Hive.openBox<Article>('articles');

// CRUD
box.put('article_1', article);
final article = box.get('article_1');
await box.delete('article_1');

// Écouter les changements
ValueListenableBuilder<Box<Article>>(
  valueListenable: box.listenable(),
  builder: (context, box, _) {
    final articles = box.values.toList();
    return ListView.builder(/* ... */);
  },
)
```

---

## Animations

```dart
// AnimatedContainer — animation implicite (simple)
AnimatedContainer(
  duration: const Duration(milliseconds: 300),
  curve: Curves.easeInOut,
  width: _isExpanded ? 300 : 100,
  height: _isExpanded ? 200 : 100,
  color: _isExpanded ? Colors.blue : Colors.red,
  child: const Text('Cliquer pour animer'),
)

// Hero — animation de transition entre pages
// Page source :
Hero(
  tag: 'photo_${article.id}', // tag unique
  child: Image.network(article.imageUrl),
)

// Page destination :
Hero(
  tag: 'photo_${article.id}', // même tag
  child: Image.network(article.imageUrl, width: double.infinity),
)
```

---

## Tests Flutter

```dart
// widget_test.dart
import 'package:flutter_test/flutter_test.dart';

void main() {
  testWidgets('Le compteur s\'incrémente', (tester) async {
    // Monter le widget
    await tester.pumpWidget(const MaterialApp(home: Compteur()));
    
    // Vérifier l'état initial
    expect(find.text('0'), findsOneWidget);
    
    // Interagir
    await tester.tap(find.byType(ElevatedButton));
    await tester.pump(); // déclencher le rebuild
    
    // Vérifier le nouvel état
    expect(find.text('1'), findsOneWidget);
  });
}
```

---

## Déploiement

### Android (APK / AAB)

```bash
# Build release APK (direct install)
flutter build apk --release

# Build release AAB (Play Store)
flutter build appbundle --release

# Signer (via key.properties)
# android/key.properties :
storePassword=monPassword
keyPassword=monKeyPassword
keyAlias=monAlias
storeFile=/chemin/vers/keystore.jks
```

### iOS

```bash
# Prérequis : Mac + Xcode + compte Apple Developer
flutter build ios --release
# Ouvrir ios/Runner.xcworkspace dans Xcode
# Product → Archive → Distribute App → App Store Connect
```

### Flavors (Dev / Staging / Prod)

```dart
// lib/config/flavor.dart
enum Flavor { dev, staging, prod }

class AppConfig {
  final Flavor flavor;
  final String apiBaseUrl;
  final String appName;
  
  const AppConfig({
    required this.flavor,
    required this.apiBaseUrl,
    required this.appName,
  });
  
  static const dev = AppConfig(
    flavor: Flavor.dev,
    apiBaseUrl: 'https://dev-api.monapp.com',
    appName: '[DEV] Mon App',
  );
  
  static const prod = AppConfig(
    flavor: Flavor.prod,
    apiBaseUrl: 'https://api.monapp.com',
    appName: 'Mon App',
  );
}

// Lancement avec flavor
// flutter run --dart-define=FLAVOR=dev
// flutter build apk --dart-define=FLAVOR=prod
```

---

## Projet Complet : App Todo List

```dart
// lib/main.dart
import 'package:flutter/material.dart';

void main() => runApp(const MonApp());

class MonApp extends StatelessWidget {
  const MonApp({super.key});
  
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Todo App',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const PageTodo(),
    );
  }
}

class Todo {
  final String id;
  final String titre;
  bool estFait;
  
  Todo({required this.id, required this.titre, this.estFait = false});
}

class PageTodo extends StatefulWidget {
  const PageTodo({super.key});
  @override
  State<PageTodo> createState() => _PageTodoState();
}

class _PageTodoState extends State<PageTodo> {
  final List<Todo> _todos = [];
  final _controller = TextEditingController();
  
  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
  
  void _ajouter() {
    if (_controller.text.trim().isEmpty) return;
    setState(() {
      _todos.add(Todo(
        id: DateTime.now().toIso8601String(),
        titre: _controller.text.trim(),
      ));
      _controller.clear();
    });
  }
  
  void _basculer(String id) {
    setState(() {
      final todo = _todos.firstWhere((t) => t.id == id);
      todo.estFait = !todo.estFait;
    });
  }
  
  void _supprimer(String id) {
    setState(() => _todos.removeWhere((t) => t.id == id));
  }

  @override
  Widget build(BuildContext context) {
    final restants = _todos.where((t) => !t.estFait).length;
    
    return Scaffold(
      appBar: AppBar(
        title: Text('Todo ($restants restants)'),
        backgroundColor: Theme.of(context).colorScheme.primaryContainer,
      ),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(16),
            child: Row(
              children: [
                Expanded(
                  child: TextField(
                    controller: _controller,
                    decoration: const InputDecoration(
                      hintText: 'Nouvelle tâche...',
                      border: OutlineInputBorder(),
                    ),
                    onSubmitted: (_) => _ajouter(),
                  ),
                ),
                const SizedBox(width: 8),
                ElevatedButton(
                  onPressed: _ajouter,
                  child: const Icon(Icons.add),
                ),
              ],
            ),
          ),
          Expanded(
            child: _todos.isEmpty
              ? const Center(child: Text('Aucune tâche. Ajoutez-en une !'))
              : ListView.builder(
                  itemCount: _todos.length,
                  itemBuilder: (context, index) {
                    final todo = _todos[index];
                    return Dismissible(
                      key: Key(todo.id),
                      onDismissed: (_) => _supprimer(todo.id),
                      background: Container(color: Colors.red),
                      child: CheckboxListTile(
                        value: todo.estFait,
                        onChanged: (_) => _basculer(todo.id),
                        title: Text(
                          todo.titre,
                          style: TextStyle(
                            decoration: todo.estFait
                              ? TextDecoration.lineThrough
                              : null,
                          ),
                        ),
                      ),
                    );
                  },
                ),
          ),
        ],
      ),
    );
  }
}
```

---

## Exercices Pratiques

### Exercice 1 — Premier widget (30 min)
Créez un widget `CarteMeteo` qui affiche :
- La ville (paramètre)
- La température (paramètre double)
- Une icône selon la température (< 10°C : ❄️, 10-25°C : ☀️, > 25°C : 🔥)
- Un bouton "Actualiser" qui affiche un SnackBar "Actualisation en cours..."

### Exercice 2 — Liste avec API (1h)
Utilisez l'API publique `https://jsonplaceholder.typicode.com/todos` pour :
1. Récupérer 20 todos avec le package `http`
2. Les afficher dans un `ListView.builder`
3. Gérer les états loading / error / success avec `FutureBuilder`
4. Permettre de marquer un todo comme complété (localement)

### Exercice 3 — Navigation multi-pages (45 min)
Créez une app avec 3 pages via `go_router` :
- `/` → liste de films (titre + année, données hardcodées)
- `/film/:id` → détail du film avec `Hero` animation sur l'image
- `/favoris` → liste des films favoris (ajouter/retirer depuis le détail)
Gérer les favoris avec `Provider`.

### Exercice 4 — Projet complet (3h)
Étendre l'app Todo du cours :
1. Ajouter des catégories (Travail / Personnel / Courses)
2. Persister les todos avec `shared_preferences`
3. Ajouter un filtre par catégorie
4. Ajouter une date d'échéance et un tri par date
5. Écrire 3 widget tests pour les fonctionnalités principales

> [!info] Ressources Flutter
> - **Documentation officielle** : docs.flutter.dev (excellente, avec cookbook)
> - **pub.dev** : repository de packages Dart/Flutter
> - **Flutter Gems** : catalogue visuel de packages
> - **Riverpod.dev** : documentation Riverpod avec exemples
> - **Very Good Ventures** : blog de bonnes pratiques Flutter avancées
