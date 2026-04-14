# Python Async et Concurrence

## Pourquoi la concurrence ?

Un programme classique execute ses instructions **sequentiellement** : une operation apres l'autre. Mais dans la realite, beaucoup de taches impliquent de l'**attente** : une requete reseau, une lecture de fichier, un calcul lourd sur un autre coeur. La concurrence permet d'utiliser ce temps d'attente de maniere productive.

> [!tip] Analogie
> Imaginez un chef cuisinier. En mode **sequentiel**, il met le poulet au four, attend 45 minutes immobile, puis prepare la salade. En mode **concurrent**, il met le poulet au four, prepare la salade pendant la cuisson, et revient quand le four sonne. Le chef est un seul processeur, mais il gere plusieurs taches en parallele grace a l'attente.

---

## I/O-bound vs CPU-bound

Avant de choisir une strategie de concurrence, il faut comprendre la nature du probleme :

```
┌─────────────────────────────────────────────────────────────────┐
│                    Types de taches                               │
├──────────────────────────┬──────────────────────────────────────┤
│       I/O-bound          │           CPU-bound                  │
├──────────────────────────┼──────────────────────────────────────┤
│ Le programme attend une  │ Le programme calcule                 │
│ ressource externe        │ intensivement                        │
├──────────────────────────┼──────────────────────────────────────┤
│ - Requetes HTTP          │ - Calculs mathematiques              │
│ - Lecture/ecriture disque│ - Traitement d'images                │
│ - Requetes base de       │ - Compression de fichiers            │
│   donnees                │ - Machine learning                   │
│ - Attente utilisateur    │ - Cryptographie                      │
├──────────────────────────┼──────────────────────────────────────┤
│ Solution : threading     │ Solution : multiprocessing           │
│ ou asyncio               │                                      │
└──────────────────────────┴──────────────────────────────────────┘
```

```python
import time

# Tache I/O-bound : le CPU attend la reponse du serveur
def telecharger_page(url):
    import requests
    response = requests.get(url)  # Le CPU ne fait rien pendant l'attente
    return response.text

# Tache CPU-bound : le CPU travaille en permanence
def calculer_primes(n):
    primes = []
    for num in range(2, n):
        is_prime = all(num % i != 0 for i in range(2, int(num**0.5) + 1))
        if is_prime:
            primes.append(num)
    return primes
```

> [!info] Regle d'or
> - **I/O-bound** → `threading` ou `asyncio` (le programme attend, on peut faire autre chose)
> - **CPU-bound** → `multiprocessing` (on a besoin de vrais coeurs en parallele)
> - Si vous utilisez `threading` pour du CPU-bound en Python, vous n'aurez **aucun** gain de performance a cause du GIL.

---

## Le GIL (Global Interpreter Lock)

Le GIL est le concept le plus important a comprendre pour la concurrence en Python. C'est un verrou **global** dans l'interpreteur CPython qui empeche plusieurs threads d'executer du bytecode Python simultanement.

```
┌─────────────────────────────────────────────────────────────┐
│                    CPython avec GIL                          │
│                                                             │
│   Thread 1    Thread 2    Thread 3                          │
│   ┌─────┐    ┌─────┐    ┌─────┐                            │
│   │ RUN │    │WAIT │    │WAIT │   Temps 1 : Thread 1 a     │
│   └─────┘    └─────┘    └─────┘              le GIL        │
│                                                             │
│   ┌─────┐    ┌─────┐    ┌─────┐                            │
│   │WAIT │    │ RUN │    │WAIT │   Temps 2 : Thread 2 a     │
│   └─────┘    └─────┘    └─────┘              le GIL        │
│                                                             │
│   ┌─────┐    ┌─────┐    ┌─────┐                            │
│   │WAIT │    │WAIT │    │ RUN │   Temps 3 : Thread 3 a     │
│   └─────┘    └─────┘    └─────┘              le GIL        │
│                                                             │
│   >>> Un seul thread execute du Python a la fois <<<        │
└─────────────────────────────────────────────────────────────┘
```

```python
# Demonstration : threading n'accelere PAS le CPU-bound
import threading
import time

def calcul_lourd():
    total = 0
    for i in range(10_000_000):
        total += i * i
    return total

# Version sequentielle
start = time.time()
calcul_lourd()
calcul_lourd()
print(f"Sequentiel : {time.time() - start:.2f}s")

# Version threaded (PAS plus rapide a cause du GIL !)
start = time.time()
t1 = threading.Thread(target=calcul_lourd)
t2 = threading.Thread(target=calcul_lourd)
t1.start()
t2.start()
t1.join()
t2.join()
print(f"Threade : {time.time() - start:.2f}s")  # Meme temps ou pire !
```

> [!warning] Le GIL est specifique a CPython
> Le GIL n'existe que dans l'implementation de reference (CPython). D'autres implementations comme Jython (Java) ou PyPy n'ont pas necessairement cette limitation. Python 3.13+ propose un mode experimental `--disable-gil` (PEP 703) mais ce n'est pas encore le defaut.

En C, les threads POSIX (`pthread`) n'ont pas ce probleme : chaque thread execute du code machine en parallele sur des coeurs differents. Le GIL est un compromis de CPython pour simplifier la gestion memoire (reference counting).

---

## Threading

### Les bases : Thread

```python
import threading
import time

def tache_longue(nom, duree):
    print(f"[{nom}] Debut...")
    time.sleep(duree)  # Simule une operation I/O
    print(f"[{nom}] Fin apres {duree}s")

# Creer et demarrer des threads
t1 = threading.Thread(target=tache_longue, args=("Telechargement", 3))
t2 = threading.Thread(target=tache_longue, args=("Requete DB", 2))

t1.start()  # Lance le thread (non-bloquant)
t2.start()

# Attendre la fin des deux threads
t1.join()   # Bloque jusqu'a la fin de t1
t2.join()

print("Tout est termine !")
# Total : ~3s au lieu de 5s en sequentiel
```

### Daemon Threads

Un daemon thread est un thread d'arriere-plan qui se termine automatiquement quand le programme principal se termine :

```python
import threading
import time

def surveillance():
    """Thread daemon qui tourne en arriere-plan."""
    while True:
        print("Surveillance en cours...")
        time.sleep(1)

# daemon=True : le thread meurt quand le programme principal se termine
t = threading.Thread(target=surveillance, daemon=True)
t.start()

time.sleep(3)
print("Programme principal termine")
# Le thread daemon est automatiquement arrete ici
```

> [!info] Daemon vs non-daemon
> - **Non-daemon** (defaut) : le programme attend la fin du thread avant de quitter
> - **Daemon** : le thread est tue quand le programme principal se termine
> - Utile pour les taches de surveillance, logging, heartbeat

### Lock et synchronisation

Quand plusieurs threads accedent a la meme ressource, il y a un risque de **race condition** :

```python
import threading

# PROBLEME : race condition
compteur = 0

def incrementer():
    global compteur
    for _ in range(100_000):
        compteur += 1  # PAS atomique ! (read, add, write)

t1 = threading.Thread(target=incrementer)
t2 = threading.Thread(target=incrementer)
t1.start()
t2.start()
t1.join()
t2.join()
print(compteur)  # Resultat imprevisible ! (souvent < 200000)
```

```python
# SOLUTION : Lock
import threading

compteur = 0
lock = threading.Lock()

def incrementer_safe():
    global compteur
    for _ in range(100_000):
        with lock:  # Acquire le lock, le relache automatiquement
            compteur += 1

t1 = threading.Thread(target=incrementer_safe)
t2 = threading.Thread(target=incrementer_safe)
t1.start()
t2.start()
t1.join()
t2.join()
print(compteur)  # Toujours 200000
```

### RLock et Semaphore

Un `RLock` (Reentrant Lock) peut etre acquis plusieurs fois par le **meme** thread sans bloquer, utile pour les fonctions recursives. Avec un `Lock` normal, ca provoquerait un deadlock.

Un semaphore limite le nombre de threads pouvant acceder a une ressource simultanement :

```python
import threading
import time

# Maximum 3 connexions simultanees
semaphore = threading.Semaphore(3)

def acceder_serveur(id_thread):
    with semaphore:
        print(f"Thread {id_thread} connecte")
        time.sleep(2)  # Simule un traitement
        print(f"Thread {id_thread} deconnecte")

threads = [threading.Thread(target=acceder_serveur, args=(i,)) for i in range(10)]
for t in threads:
    t.start()
for t in threads:
    t.join()
# Les threads passent par groupes de 3
```

### ThreadPoolExecutor

La maniere **moderne** et recommandee d'utiliser les threads :

```python
from concurrent.futures import ThreadPoolExecutor, as_completed
import requests
import time

urls = [
    "https://httpbin.org/delay/1",
    "https://httpbin.org/delay/2",
    "https://httpbin.org/delay/1",
    "https://httpbin.org/delay/3",
    "https://httpbin.org/delay/1",
]

def telecharger(url):
    response = requests.get(url)
    return url, response.status_code, len(response.text)

# Pool de 5 threads max
start = time.time()
with ThreadPoolExecutor(max_workers=5) as executor:
    # submit() retourne un Future
    futures = {executor.submit(telecharger, url): url for url in urls}

    # Recuperer les resultats au fur et a mesure
    for future in as_completed(futures):
        url, status, taille = future.result()
        print(f"{url} -> {status} ({taille} octets)")

print(f"Total : {time.time() - start:.2f}s")  # ~3s au lieu de 8s
```

```python
# Version simplifiee avec map()
with ThreadPoolExecutor(max_workers=5) as executor:
    resultats = list(executor.map(telecharger, urls))
    for url, status, taille in resultats:
        print(f"{url} -> {status}")
```

> [!tip] Analogie
> Un `ThreadPoolExecutor` c'est comme un **bureau de poste** avec N guichets. Les clients (taches) font la queue, et des qu'un guichet se libere, le client suivant est servi. Pas besoin de creer un nouveau guichet pour chaque client.

---

## Multiprocessing

Le module `multiprocessing` cree de **vrais processus** systeme, chacun avec son propre interpreteur Python et sa propre memoire. Le GIL n'est plus un probleme car chaque processus a le sien.

### Process de base

```python
from multiprocessing import Process
import os

def calcul_lourd(nom):
    print(f"[{nom}] PID: {os.getpid()}")
    total = sum(i * i for i in range(10_000_000))
    print(f"[{nom}] Resultat: {total}")

if __name__ == "__main__":  # OBLIGATOIRE sur Windows !
    p1 = Process(target=calcul_lourd, args=("Processus 1",))
    p2 = Process(target=calcul_lourd, args=("Processus 2",))

    p1.start()
    p2.start()
    p1.join()
    p2.join()
    print("Termine")
```

> [!warning] `if __name__ == "__main__"` obligatoire
> Sur Windows, `multiprocessing` reimporte le module principal dans chaque processus enfant. Sans cette garde, vous obtiendrez une boucle infinie de creation de processus. C'est une source d'erreur extremement frequente.

### Pool et map

```python
from multiprocessing import Pool
import time

def est_premier(n):
    if n < 2:
        return False
    for i in range(2, int(n**0.5) + 1):
        if n % i == 0:
            return False
    return True

if __name__ == "__main__":
    nombres = list(range(100_000, 200_000))

    # Sequentiel
    start = time.time()
    resultats_seq = [est_premier(n) for n in nombres]
    print(f"Sequentiel : {time.time() - start:.2f}s")

    # Parallele avec Pool
    start = time.time()
    with Pool(processes=4) as pool:
        resultats_par = pool.map(est_premier, nombres)
    print(f"Parallele (4 coeurs) : {time.time() - start:.2f}s")

    print(f"Premiers trouves : {sum(resultats_par)}")
```

### ProcessPoolExecutor

L'interface unifiee via `concurrent.futures` :

```python
from concurrent.futures import ProcessPoolExecutor
import time

def factorielle_lente(n):
    """Calcul volontairement lent pour la demo."""
    resultat = 1
    for i in range(1, n + 1):
        resultat *= i
    return n, resultat

if __name__ == "__main__":
    nombres = [100_000, 150_000, 200_000, 120_000]

    start = time.time()
    with ProcessPoolExecutor(max_workers=4) as executor:
        resultats = list(executor.map(factorielle_lente, nombres))

    for n, fact in resultats:
        print(f"{n}! a {len(str(fact))} chiffres")
    print(f"Temps : {time.time() - start:.2f}s")
```

### Communication entre processus : Queue

Les processus ne partagent pas la memoire. On utilise une `Queue` pour communiquer :

```python
from multiprocessing import Process, Queue

def producteur(queue, donnees):
    for item in donnees:
        queue.put(item * 2)  # Envoie le resultat dans la queue
    queue.put(None)  # Signal de fin

def consommateur(queue):
    resultats = []
    while True:
        item = queue.get()  # Bloque jusqu'a recevoir un element
        if item is None:
            break
        resultats.append(item)
    print(f"Resultats recus : {resultats}")

if __name__ == "__main__":
    queue = Queue()
    donnees = [1, 2, 3, 4, 5]

    p1 = Process(target=producteur, args=(queue, donnees))
    p2 = Process(target=consommateur, args=(queue,))

    p1.start()
    p2.start()
    p1.join()
    p2.join()
```

### Memoire partagee

Pour partager des valeurs entre processus, on utilise `Value` et `Array` :

```python
from multiprocessing import Process, Value

def incrementer(compteur_partage):
    for _ in range(10_000):
        with compteur_partage.get_lock():
            compteur_partage.value += 1

if __name__ == "__main__":
    compteur = Value('i', 0)  # 'i' = entier, valeur initiale 0
    p1 = Process(target=incrementer, args=(compteur,))
    p2 = Process(target=incrementer, args=(compteur,))
    p1.start(); p2.start()
    p1.join(); p2.join()
    print(f"Compteur : {compteur.value}")  # 20000
```

> [!info] Quand utiliser multiprocessing ?
> - Calculs **CPU-bound** : mathematiques, traitement d'images, ML
> - Quand vous avez besoin de **vrais** coeurs en parallele
> - Attention au **cout** : creer un processus est plus lourd qu'un thread (memoire separee, serialisation des donnees avec pickle)

---

## Asyncio

`asyncio` est la solution Python pour la programmation asynchrone **mono-thread**. Au lieu de creer plusieurs threads, on utilise une **boucle d'evenements** qui gere les taches de maniere cooperative.

### Concept : la boucle d'evenements

```
┌─────────────────────────────────────────────────────────────────┐
│                    Event Loop (asyncio)                          │
│                                                                 │
│   ┌──────────┐                                                  │
│   │          │◄──── La boucle verifie en permanence             │
│   │  EVENT   │      quelles taches sont pretes                  │
│   │   LOOP   │                                                  │
│   │          │                                                  │
│   └────┬─────┘                                                  │
│        │                                                        │
│   ┌────┼──────────┬──────────────┬──────────────┐               │
│   │    │          │              │              │               │
│   ▼    ▼          ▼              ▼              ▼               │
│ ┌────┐ ┌────┐  ┌────┐       ┌────┐        ┌────┐              │
│ │Task│ │Task│  │Task│       │Task│        │Task│              │
│ │ 1  │ │ 2  │  │ 3  │       │ 4  │        │ 5  │              │
│ │RUN │ │WAIT│  │WAIT│       │RUN │        │DONE│              │
│ └────┘ └────┘  └────┘       └────┘        └────┘              │
│                                                                 │
│   Quand Task 1 fait "await", elle rend la main                 │
│   La boucle passe a la prochaine tache prete                   │
│   Tout se passe dans UN SEUL thread !                          │
└─────────────────────────────────────────────────────────────────┘
```

### Coroutines : async def et await

```python
import asyncio

# Une coroutine est definie avec "async def"
async def saluer(nom, delai):
    print(f"Bonjour {nom} !")
    await asyncio.sleep(delai)  # "await" rend la main a la boucle
    print(f"Au revoir {nom} ! (apres {delai}s)")
    return f"Resultat de {nom}"

# asyncio.run() lance la boucle d'evenements
async def main():
    resultat = await saluer("Alice", 2)
    print(resultat)

asyncio.run(main())
```

> [!warning] On ne peut pas appeler une coroutine comme une fonction normale
> `saluer("Alice", 2)` retourne un objet coroutine, il ne l'execute pas. Il faut utiliser `await` dans une autre coroutine, ou `asyncio.run()` pour la premiere.

### asyncio.gather : executer en parallele

```python
import asyncio
import time

async def telecharger_page(url, delai):
    print(f"Debut telechargement {url}")
    await asyncio.sleep(delai)  # Simule un telechargement
    print(f"Fin telechargement {url}")
    return f"Contenu de {url}"

async def main():
    start = time.time()

    # gather execute toutes les coroutines en parallele
    resultats = await asyncio.gather(
        telecharger_page("page1.html", 2),
        telecharger_page("page2.html", 3),
        telecharger_page("page3.html", 1),
    )

    print(f"\nTermine en {time.time() - start:.2f}s")  # ~3s, pas 6s
    print(f"Resultats : {resultats}")

asyncio.run(main())
```

### asyncio.create_task : lancer des taches

```python
import asyncio

async def tache_longue(nom, duree):
    await asyncio.sleep(duree)
    return f"{nom} terminee"

async def main():
    # create_task lance la coroutine en arriere-plan
    task1 = asyncio.create_task(tache_longue("Tache A", 2))
    task2 = asyncio.create_task(tache_longue("Tache B", 3))

    # On peut faire autre chose pendant ce temps
    print("Les taches tournent en arriere-plan...")
    await asyncio.sleep(1)
    print("1 seconde passee, on continue...")

    # Attendre les resultats
    resultat1 = await task1
    resultat2 = await task2
    print(resultat1, resultat2)

asyncio.run(main())
```

> [!info] gather vs create_task
> - `asyncio.gather()` : lance et attend plusieurs coroutines d'un coup
> - `asyncio.create_task()` : lance une coroutine en arriere-plan, on recupère le resultat plus tard
> - `create_task` offre plus de controle (annulation, verification du statut)

### aiohttp : requetes HTTP asynchrones

`requests` est **synchrone** et bloque le thread. Pour de l'async, on utilise `aiohttp` :

```python
# pip install aiohttp
import asyncio
import aiohttp
import time

async def telecharger(session, url):
    async with session.get(url) as response:
        contenu = await response.text()
        return url, response.status, len(contenu)

async def main():
    urls = [
        "https://httpbin.org/delay/1",
        "https://httpbin.org/delay/2",
        "https://httpbin.org/delay/1",
        "https://httpbin.org/delay/3",
        "https://httpbin.org/delay/1",
    ]

    start = time.time()
    async with aiohttp.ClientSession() as session:
        taches = [telecharger(session, url) for url in urls]
        resultats = await asyncio.gather(*taches)

    for url, status, taille in resultats:
        print(f"{url} -> {status} ({taille} octets)")
    print(f"\nTotal : {time.time() - start:.2f}s")  # ~3s au lieu de 8s

asyncio.run(main())
```

### Async context managers

```python
import asyncio

class ConnexionDB:
    """Simule une connexion base de donnees asynchrone."""

    async def __aenter__(self):
        print("Connexion a la DB...")
        await asyncio.sleep(0.5)  # Simule la connexion
        print("Connecte !")
        return self

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        print("Fermeture de la connexion...")
        await asyncio.sleep(0.2)
        print("Deconnecte !")

    async def executer(self, requete):
        print(f"Execution : {requete}")
        await asyncio.sleep(0.3)
        return [{"id": 1, "nom": "Alice"}, {"id": 2, "nom": "Bob"}]

async def main():
    async with ConnexionDB() as db:
        resultats = await db.executer("SELECT * FROM users")
        print(f"Resultats : {resultats}")

asyncio.run(main())
```

### Async generators

```python
import asyncio

async def stream_donnees(n):
    """Genere des donnees de maniere asynchrone."""
    for i in range(n):
        await asyncio.sleep(0.5)  # Simule la reception de donnees
        yield f"Donnee {i}"

async def main():
    async for donnee in stream_donnees(5):
        print(f"Recu : {donnee}")

asyncio.run(main())
```

---

## Tableau comparatif

```
┌──────────────────┬──────────────────┬──────────────────┬──────────────────┐
│                  │    threading     │ multiprocessing  │     asyncio      │
├──────────────────┼──────────────────┼──────────────────┼──────────────────┤
│ Cas d'usage      │ I/O-bound        │ CPU-bound        │ I/O-bound        │
│                  │ (reseau, disque) │ (calculs lourds) │ (beaucoup de     │
│                  │                  │                  │  connexions)     │
├──────────────────┼──────────────────┼──────────────────┼──────────────────┤
│ Impact du GIL    │ Limite (pas de   │ Aucun (processus │ Aucun (mono-     │
│                  │ vrai parallelisme│ separes)         │ thread)          │
│                  │ CPU)             │                  │                  │
├──────────────────┼──────────────────┼──────────────────┼──────────────────┤
│ Memoire          │ Partagee (meme   │ Separee (chaque  │ Partagee (meme   │
│                  │ espace memoire)  │ processus a sa   │ espace memoire)  │
│                  │                  │ propre memoire)  │                  │
├──────────────────┼──────────────────┼──────────────────┼──────────────────┤
│ Complexite       │ Moyenne (locks,  │ Elevee (IPC,     │ Moyenne (async/  │
│                  │ race conditions) │ serialisation)   │ await, ecosystem)│
├──────────────────┼──────────────────┼──────────────────┼──────────────────┤
│ Scalabilite      │ Dizaines de      │ Nombre de coeurs │ Milliers de      │
│                  │ threads          │ CPU              │ connexions       │
├──────────────────┼──────────────────┼──────────────────┼──────────────────┤
│ Creation         │ Leger            │ Lourd            │ Tres leger       │
│                  │ (~1 Mo/thread)   │ (~30+ Mo/proc)   │ (~1 Ko/tache)    │
├──────────────────┼──────────────────┼──────────────────┼──────────────────┤
│ Debugging        │ Difficile        │ Moyen            │ Moyen            │
│                  │ (non deterministe│ (processus       │ (stack traces    │
│                  │  race conditions)│  isoles)         │  lisibles)       │
└──────────────────┴──────────────────┴──────────────────┴──────────────────┘
```

---

## concurrent.futures : interface unifiee

Le module `concurrent.futures` offre la meme interface pour threads et processus :

```python
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor
import time

def tache(x):
    """Peut etre executee en thread ou en processus."""
    time.sleep(0.1)
    return x ** 2

donnees = list(range(20))

# Meme code, juste changer l'Executor
for ExecutorClass in [ThreadPoolExecutor, ProcessPoolExecutor]:
    nom = ExecutorClass.__name__
    start = time.time()

    with ExecutorClass(max_workers=4) as executor:
        resultats = list(executor.map(tache, donnees))

    print(f"{nom} : {time.time() - start:.2f}s -> {resultats[:5]}...")
```

> [!tip] Analogie
> `concurrent.futures` c'est comme une **interface universelle** de prise electrique. Que vous branchiez un appareil francais (thread) ou americain (processus), l'interface est la meme (`submit`, `map`, `as_completed`). Seul l'adaptateur change.

---

## Exemples pratiques

### Web scraper asynchrone

```python
import asyncio
import aiohttp
from dataclasses import dataclass

@dataclass
class PageInfo:
    url: str
    status: int
    taille: int
    titre: str

async def analyser_page(session, url):
    """Telecharge et analyse une page web."""
    try:
        async with session.get(url, timeout=aiohttp.ClientTimeout(total=10)) as resp:
            html = await resp.text()
            # Extraction basique du titre
            debut = html.find("<title>")
            fin = html.find("</title>")
            titre = html[debut + 7:fin] if debut != -1 and fin != -1 else "Sans titre"
            return PageInfo(url, resp.status, len(html), titre)
    except Exception as e:
        return PageInfo(url, 0, 0, f"Erreur : {e}")

async def scraper(urls, max_concurrent=10):
    """Scrape plusieurs URLs avec limitation de concurrence."""
    semaphore = asyncio.Semaphore(max_concurrent)

    async def scrape_avec_limite(session, url):
        async with semaphore:
            return await analyser_page(session, url)

    async with aiohttp.ClientSession() as session:
        taches = [scrape_avec_limite(session, url) for url in urls]
        return await asyncio.gather(*taches)

async def main():
    urls = [
        "https://python.org",
        "https://docs.python.org",
        "https://pypi.org",
        "https://flask.palletsprojects.com",
        "https://fastapi.tiangolo.com",
    ]

    resultats = await scraper(urls)
    for page in resultats:
        print(f"[{page.status}] {page.url}")
        print(f"       Titre: {page.titre} ({page.taille} octets)")

asyncio.run(main())
```

### Traitement parallele de fichiers

```python
from concurrent.futures import ProcessPoolExecutor
from pathlib import Path
import hashlib

def analyser_fichier(chemin):
    path = Path(chemin)
    contenu = path.read_bytes()
    return {
        "fichier": path.name,
        "taille": len(contenu),
        "hash": hashlib.md5(contenu).hexdigest(),
    }

if __name__ == "__main__":
    fichiers = list(Path("./mon_projet").rglob("*.py"))
    with ProcessPoolExecutor() as executor:
        resultats = sorted(executor.map(analyser_fichier, fichiers),
                          key=lambda r: r["taille"], reverse=True)
    for r in resultats[:10]:
        print(f"{r['fichier']:30s} {r['taille']:>10,} octets")
```

---

## Comparaison avec le C et JavaScript

En C, la concurrence utilise `fork()` pour les processus et `pthread` pour les threads. La difference majeure : pas de GIL, les threads sont vraiment paralleles sur plusieurs coeurs. Mais il faut gerer manuellement les mutex, les conditions, et la memoire partagee.

```python
# Equivalences simplifiees :
# C fork()              -> Python multiprocessing.Process()
# C pthread_create()    -> Python threading.Thread()
# C pthread_mutex_lock  -> Python threading.Lock()
# C sem_wait/sem_post   -> Python threading.Semaphore()
```

L'asyncio de Python est tres similaire a l'event loop de JavaScript/Node.js :

```python
# Python asyncio            |  JavaScript (Node.js)
# ========================= | =========================
# async def foo():          |  async function foo() {
#     await bar()            |      await bar()
# asyncio.gather(a, b, c)   |  Promise.all([a, b, c])
# asyncio.create_task(foo())|  foo()  // (Promises sont eager)
# asyncio.run(main())       |  (automatique dans Node.js)
```

> [!info] Differences cles avec JavaScript
> - En JS, l'event loop est **toujours** actif. En Python, il faut l'activer avec `asyncio.run()`
> - Les Promises JS sont **eagerly evaluated**. Les coroutines Python sont **lazy** (ne s'executent que quand on les `await` ou les passe a `create_task`)
> - Python a aussi le threading et le multiprocessing. Node.js est fondamentalement mono-thread

---

## Carte Mentale

```
                    Async et Concurrence
                            │
            ┌───────────────┼───────────────┐
            │               │               │
        Threading      Multiprocessing    Asyncio
            │               │               │
       ┌────┤          ┌────┤          ┌────┤
       │    │          │    │          │    │
     Thread │       Process │      async def│
       │    │          │    │          │    │
     Lock   │        Pool   │       await   │
       │    │          │    │          │    │
    RLock   │       Queue   │      gather   │
       │    │          │    │          │    │
  Semaphore │    Value/ │    create_task │
       │    │    Array  │          │    │
   daemon   │          │       aiohttp  │
       │    │     ProcessPool   │    │
  ThreadPool│     Executor     async with
   Executor │                      │
             │                  async for
         GIL │
             │
        I/O-bound           CPU-bound
             │                   │
        ┌────┤              ┌────┤
        │    │              │    │
    requetes │          calculs  │
    reseau   │          lourds   │
        │    │              │    │
    fichiers │          images   │
        │    │              │    │
      BDD    │            ML     │
             │                   │
       concurrent.futures        │
       (interface unifiee)       │
```

---

## Exercices

### Exercice 1 : ThreadPoolExecutor

Ecrivez un programme qui telecharge le contenu de 10 URLs differentes en utilisant `ThreadPoolExecutor`. Mesurez le temps d'execution et comparez avec une version sequentielle. Affichez les resultats (URL, status code, taille) tries par taille decroissante.

### Exercice 2 : Multiprocessing CPU-bound

Ecrivez un programme qui calcule la somme des nombres premiers entre 1 et 1 000 000 en decoupant le travail sur 4 processus. Chaque processus traite un quart de la plage. Utilisez une `Queue` pour recuperer les resultats partiels. Comparez avec la version sequentielle.

### Exercice 3 : Web scraper async

Creez un scraper asynchrone avec `aiohttp` qui :
1. Telecharge une liste de 20 URLs
2. Limite la concurrence a 5 requetes simultanees (Semaphore)
3. Extrait le titre de chaque page
4. Gere les erreurs (timeout, connexion refusee)
5. Affiche un rapport avec le temps total

### Exercice 4 : Producteur-Consommateur async

Implementez un pattern producteur-consommateur avec `asyncio.Queue` :
- Un producteur genere des nombres aleatoires toutes les 0.1s
- Trois consommateurs traitent ces nombres (calcul du carre, attente simulee)
- Le programme s'arrete apres que le producteur a genere 50 nombres
- Affichez les statistiques a la fin (temps total, nombres traites par consommateur)

---

## Liens

- [[06 - Modules Packages et Venv]] - Modules, packages et environnements virtuels
- [[08 - APIs REST avec Flask]] - Creation d'APIs REST avec Flask
