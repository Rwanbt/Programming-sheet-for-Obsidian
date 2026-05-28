# WebSockets

Les **WebSockets** permettent une communication **bidirectionnelle et persistante** entre le navigateur et le serveur. Contrairement au HTTP classique, ou le client doit demander chaque nouvelle information, avec WebSocket le serveur peut **pousser** des donnees au client a tout moment.

C'est la technologie derriere les chats en temps reel, les tableaux de bord live, les jeux multijoueurs dans le navigateur, les outils collaboratifs comme Figma ou Google Docs.

> [!tip] Analogie
> HTTP classique ressemble a un courrier postal : vous envoyez une lettre (requete), vous attendez la reponse. Pour avoir une nouvelle lettre, vous devez en ecrire une nouvelle. WebSocket, c'est un **appel telephonique** : une fois la connexion etablie, les deux cotes peuvent parler quand ils le souhaitent, sans recomposer le numero a chaque fois.

---

## WebSocket vs HTTP

### Les limites du HTTP pour le temps reel

```
HTTP classique — Polling toutes les 2 secondes :

Client ──[GET /messages]──────► Serveur
Client ◄──[200 OK, vide]──────── Serveur    (rien de nouveau)
... 2 secondes ...
Client ──[GET /messages]──────► Serveur
Client ◄──[200 OK, vide]──────── Serveur    (toujours rien)
... 2 secondes ...
Client ──[GET /messages]──────► Serveur
Client ◄──[200 OK, {msg: "Salut"}]── Serveur   (enfin !)

Problemes :
- Delai de 2 secondes minimum
- Beaucoup de requetes inutiles
- Overhead HTTP (headers) pour chaque requete
- Serveur sous pression pour rien
```

```
WebSocket — Connexion persistante :

Client ──[GET /ws (Upgrade)]──────► Serveur   (1 seule requete initiale)
Client ◄──[101 Switching Protocols]── Serveur  (connexion etablie)

Client ◄──────────────────────────── Serveur   (push immediat quand message)
Client ──────────────────────────►  Serveur    (envoi client immediat)
Client ◄──────────────────────────── Serveur   (reponse immediate)

Connexion reste ouverte jusqu'a fermeture explicite
```

> [!info] Comparaison technique
> | Critere | HTTP Polling | SSE | WebSocket |
> |---|---|---|---|
> | Direction | Client → Serveur | Serveur → Client | Bidirectionnel |
> | Latence | ~1-5s (selon intervalle) | ~50ms | ~1ms |
> | Overhead | Tres eleve (headers repetes) | Faible | Minimal (frames) |
> | Reconnexion auto | N/A | Oui (native) | Non (a implementer) |
> | Protocole | HTTP/1.1, HTTP/2 | HTTP | WS (sur TCP) |
> | Cas d'usage | Legacy, simple | Notifications, logs | Chat, jeux, collaboration |

---

## Le Handshake HTTP → WebSocket

La connexion WebSocket commence toujours par une **requete HTTP ordinaire** avec un en-tete special `Upgrade`.

```
Etape 1 — Requete HTTP du client :

GET /ws/chat HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13

─────────────────────────────────────────────────────

Etape 2 — Reponse du serveur (101 Switching Protocols) :

HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=

─────────────────────────────────────────────────────

Etape 3 — La connexion TCP reste ouverte, protocole WS actif
```

> [!info] La cle Sec-WebSocket-Accept
> Le serveur calcule `Sec-WebSocket-Accept` en concatenant la cle du client avec un GUID fixe (`258EAFA5-...`), puis en hashant en SHA-1 encode en Base64. Ce mecanisme prouve que le serveur est bien un serveur WebSocket (pas juste un proxy HTTP qui aurait laisse passer la requete).

---

## Le Protocole WebSocket

Une fois la connexion etablie, les donnees sont echangees en **frames** (trames) :

```
Frame WebSocket :
┌────────┬────────┬──────────┬────────────┬──────────────┐
│ FIN(1b)│ Opcode │ Mask(1b) │ Payload    │ Masking Key  │
│ RSV(3b)│ (4b)   │ Len (7b) │ Length     │ (optionnel)  │
└────────┴────────┴──────────┴────────────┴──────────────┘
```

**Opcodes principaux** :
- `0x1` : Text frame (UTF-8)
- `0x2` : Binary frame
- `0x8` : Close frame
- `0x9` : Ping frame
- `0xA` : Pong frame (reponse automatique au ping)

> [!tip] Frames vs Messages HTTP
> Un header HTTP fait ~500 octets. Une frame WebSocket fait 2 a 14 octets. Pour une application de jeu envoyant 60 mises a jour par seconde, cela fait la difference entre 30 KB/s et 720 KB/s d'overhead reseau.

---

## Implementation JavaScript — API Native

### Connexion et evenements de base

```javascript
// Connexion a un serveur WebSocket
const ws = new WebSocket('ws://localhost:8765');
// Pour HTTPS → wss://localhost:8765 (WebSocket Secure)

// Evenement : connexion etablie
ws.addEventListener('open', (event) => {
    console.log('Connexion WebSocket etablie !');
    console.log('Etat :', ws.readyState); // 1 = OPEN
    
    // Envoyer un message
    ws.send(JSON.stringify({
        type: 'salutation',
        contenu: 'Bonjour serveur !'
    }));
});

// Evenement : message recu du serveur
ws.addEventListener('message', (event) => {
    console.log('Message recu :', event.data);
    
    // Generalement, les messages sont du JSON
    try {
        const data = JSON.parse(event.data);
        traiterMessage(data);
    } catch (e) {
        console.error('Message non-JSON :', event.data);
    }
});

// Evenement : erreur
ws.addEventListener('error', (event) => {
    console.error('Erreur WebSocket :', event);
});

// Evenement : connexion fermee
ws.addEventListener('close', (event) => {
    console.log(`Connexion fermee. Code: ${event.code}, Raison: ${event.reason}`);
    // Codes courants :
    // 1000 = fermeture normale
    // 1001 = client part (fermeture navigateur/onglet)
    // 1006 = connexion perdue anormalement (pas de close frame)
    // 1011 = erreur serveur interne
});

// Fermer la connexion proprement
function deconnecter() {
    ws.close(1000, 'Deconnexion utilisateur');
}

// Etats possibles de ws.readyState
// WebSocket.CONNECTING = 0
// WebSocket.OPEN       = 1
// WebSocket.CLOSING    = 2
// WebSocket.CLOSED     = 3
```

### Envoi de donnees

```javascript
// Envoyer du texte / JSON
ws.send('Bonjour');
ws.send(JSON.stringify({ type: 'message', contenu: 'Hello !' }));

// Envoyer des donnees binaires
const buffer = new ArrayBuffer(8);
const view = new DataView(buffer);
view.setFloat32(0, 3.14);
ws.send(buffer);

// Envoyer un Blob (fichier, image)
const file = document.querySelector('input[type=file]').files[0];
ws.send(file);

// Verifier que la connexion est ouverte avant d'envoyer
function envoyerMessage(message) {
    if (ws.readyState === WebSocket.OPEN) {
        ws.send(JSON.stringify(message));
    } else {
        console.warn('WebSocket non connecte, message mis en file...');
        // Implementer une file d'attente si necessaire
        messageQueue.push(message);
    }
}
```

### Reconnexion automatique

```javascript
class WebSocketManager {
    /**
     * Gestionnaire WebSocket avec reconnexion automatique exponentielle.
     */
    constructor(url, options = {}) {
        this.url = url;
        this.maxRetries = options.maxRetries ?? 10;
        this.baseDelay = options.baseDelay ?? 1000; // 1 seconde
        this.maxDelay = options.maxDelay ?? 30000;  // 30 secondes
        this.onMessage = options.onMessage ?? (() => {});
        this.onOpen = options.onOpen ?? (() => {});
        this.onClose = options.onClose ?? (() => {});
        
        this.ws = null;
        this.retryCount = 0;
        this.manuellementFerme = false;
        this.messageQueue = [];
        
        this.connecter();
    }
    
    connecter() {
        this.ws = new WebSocket(this.url);
        
        this.ws.addEventListener('open', (event) => {
            console.log('WebSocket connecte.');
            this.retryCount = 0;
            this.onOpen(event);
            // Vider la file d'attente
            while (this.messageQueue.length > 0) {
                this.ws.send(this.messageQueue.shift());
            }
        });
        
        this.ws.addEventListener('message', (event) => {
            this.onMessage(JSON.parse(event.data));
        });
        
        this.ws.addEventListener('close', (event) => {
            this.onClose(event);
            if (!this.manuellementFerme && this.retryCount < this.maxRetries) {
                const delai = Math.min(
                    this.baseDelay * Math.pow(2, this.retryCount),
                    this.maxDelay
                );
                console.log(`Reconnexion dans ${delai}ms (tentative ${this.retryCount + 1}/${this.maxRetries})...`);
                setTimeout(() => this.connecter(), delai);
                this.retryCount++;
            }
        });
        
        this.ws.addEventListener('error', (event) => {
            console.error('Erreur WebSocket :', event);
        });
    }
    
    envoyer(message) {
        const data = JSON.stringify(message);
        if (this.ws?.readyState === WebSocket.OPEN) {
            this.ws.send(data);
        } else {
            this.messageQueue.push(data);
        }
    }
    
    fermer() {
        this.manuellementFerme = true;
        this.ws?.close(1000, 'Fermeture volontaire');
    }
}

// Utilisation
const manager = new WebSocketManager('ws://localhost:8765', {
    onMessage: (data) => console.log('Message :', data),
    onOpen: () => console.log('Connecte !'),
    onClose: (event) => console.log('Deconnecte :', event.code)
});

manager.envoyer({ type: 'ping', timestamp: Date.now() });
```

---

## Implementation Serveur Python — asyncio + websockets

### Installation

```bash
pip install websockets
```

### Serveur basique

```python
import asyncio
import json
import websockets
from datetime import datetime


async def handler(websocket):
    """Gere une connexion WebSocket individuelle."""
    client_addr = websocket.remote_address
    print(f"Nouveau client connecte : {client_addr}")
    
    try:
        async for message in websocket:
            print(f"Recu de {client_addr} : {message}")
            
            # Parser le JSON
            try:
                data = json.loads(message)
            except json.JSONDecodeError:
                await websocket.send(json.dumps({"erreur": "JSON invalide"}))
                continue
            
            # Traiter le message
            reponse = traiter_message(data)
            
            # Repondre
            await websocket.send(json.dumps(reponse))
            
    except websockets.exceptions.ConnectionClosedOK:
        print(f"Client deconnecte proprement : {client_addr}")
    except websockets.exceptions.ConnectionClosedError as e:
        print(f"Client deconnecte avec erreur : {client_addr} — {e}")


def traiter_message(data: dict) -> dict:
    """Logique metier du traitement de message."""
    msg_type = data.get("type", "inconnu")
    
    if msg_type == "ping":
        return {"type": "pong", "timestamp": datetime.utcnow().isoformat()}
    
    elif msg_type == "echo":
        return {"type": "echo", "contenu": data.get("contenu", "")}
    
    else:
        return {"type": "erreur", "message": f"Type inconnu : {msg_type}"}


async def main():
    print("Serveur WebSocket demarre sur ws://localhost:8765")
    async with websockets.serve(handler, "localhost", 8765):
        await asyncio.Future()  # Tourner indefiniment


if __name__ == "__main__":
    asyncio.run(main())
```

### Chat en temps reel — diffusion a tous les clients

```python
import asyncio
import json
import websockets
from datetime import datetime

# Ensemble de toutes les connexions actives
CLIENTS_CONNECTES: set = set()


async def diffuser(message: str, expediteur=None) -> None:
    """Envoie un message a tous les clients (ou a tous sauf l'expediteur)."""
    if not CLIENTS_CONNECTES:
        return
    
    destinataires = CLIENTS_CONNECTES - {expediteur} if expediteur else CLIENTS_CONNECTES
    
    # Envoyer en parallele a tous les clients connectes
    if destinataires:
        await asyncio.gather(
            *[client.send(message) for client in destinataires],
            return_exceptions=True  # Ignorer les erreurs individuelles
        )


async def handler_chat(websocket):
    """Gere un client du chat."""
    # Enregistrer le client
    CLIENTS_CONNECTES.add(websocket)
    pseudo = f"Utilisateur_{websocket.id.hex[:6]}"
    
    try:
        # Notifier tout le monde
        await diffuser(json.dumps({
            "type": "systeme",
            "contenu": f"{pseudo} a rejoint le chat",
            "timestamp": datetime.utcnow().isoformat(),
            "nb_connectes": len(CLIENTS_CONNECTES)
        }))
        
        async for message in websocket:
            data = json.loads(message)
            
            if data.get("type") == "message":
                # Construire le message a diffuser
                msg_broadcast = json.dumps({
                    "type": "message",
                    "pseudo": data.get("pseudo", pseudo),
                    "contenu": data.get("contenu", ""),
                    "timestamp": datetime.utcnow().isoformat()
                })
                
                # Diffuser a tous (expediteur inclus pour confirmation)
                await diffuser(msg_broadcast)
            
            elif data.get("type") == "changement_pseudo":
                ancien_pseudo = pseudo
                pseudo = data.get("nouveau_pseudo", pseudo)[:20]  # Limiter a 20 chars
                await diffuser(json.dumps({
                    "type": "systeme",
                    "contenu": f"{ancien_pseudo} s'appelle maintenant {pseudo}"
                }))
    
    except websockets.exceptions.ConnectionClosed:
        pass
    
    finally:
        # Desenregistrer le client
        CLIENTS_CONNECTES.discard(websocket)
        await diffuser(json.dumps({
            "type": "systeme",
            "contenu": f"{pseudo} a quitte le chat",
            "nb_connectes": len(CLIENTS_CONNECTES)
        }))


async def main():
    async with websockets.serve(handler_chat, "localhost", 8765):
        print(f"Serveur de chat demarre sur ws://localhost:8765")
        await asyncio.Future()


if __name__ == "__main__":
    asyncio.run(main())
```

---

## Implementation avec FastAPI

FastAPI a un support WebSocket natif, ce qui en fait un excellent choix pour les APIs qui melangent HTTP REST et WebSocket.

```python
from fastapi import FastAPI, WebSocket, WebSocketDisconnect
from fastapi.staticfiles import StaticFiles
import json
from datetime import datetime
from typing import Dict

app = FastAPI()

# Gestionnaire de connexions
class ConnectionManager:
    """Gere toutes les connexions WebSocket actives."""
    
    def __init__(self):
        # {room_id: [websocket, ...]}
        self.rooms: Dict[str, list[WebSocket]] = {}
    
    async def connecter(self, websocket: WebSocket, room_id: str) -> None:
        await websocket.accept()
        if room_id not in self.rooms:
            self.rooms[room_id] = []
        self.rooms[room_id].append(websocket)
        print(f"Client connecte a la room '{room_id}'. Total: {len(self.rooms[room_id])}")
    
    def deconnecter(self, websocket: WebSocket, room_id: str) -> None:
        if room_id in self.rooms:
            self.rooms[room_id].discard(websocket) if hasattr(self.rooms[room_id], 'discard') else None
            try:
                self.rooms[room_id].remove(websocket)
            except ValueError:
                pass
    
    async def diffuser_room(self, message: str, room_id: str, expediteur: WebSocket = None) -> None:
        """Envoie un message a tous les clients d'une room."""
        if room_id not in self.rooms:
            return
        
        connexions_mortes = []
        for connexion in self.rooms[room_id]:
            if connexion == expediteur:
                continue
            try:
                await connexion.send_text(message)
            except Exception:
                connexions_mortes.append(connexion)
        
        # Nettoyer les connexions mortes
        for mort in connexions_mortes:
            self.rooms[room_id].remove(mort)
    
    async def envoyer_personnel(self, message: str, websocket: WebSocket) -> None:
        """Envoie un message a un seul client."""
        await websocket.send_text(message)


manager = ConnectionManager()


@app.websocket("/ws/chat/{room_id}")
async def endpoint_chat(websocket: WebSocket, room_id: str):
    """Endpoint WebSocket pour le chat par room."""
    await manager.connecter(websocket, room_id)
    
    try:
        while True:
            # Attendre un message du client
            data = await websocket.receive_text()
            message = json.loads(data)
            
            # Enrichir le message
            message["timestamp"] = datetime.utcnow().isoformat()
            message["room"] = room_id
            
            # Diffuser a la room
            await manager.diffuser_room(json.dumps(message), room_id, expediteur=websocket)
            
            # Confirmer a l'expediteur
            await manager.envoyer_personnel(
                json.dumps({"type": "confirme", "message_id": message.get("id")}),
                websocket
            )
    
    except WebSocketDisconnect:
        manager.deconnecter(websocket, room_id)
        await manager.diffuser_room(
            json.dumps({"type": "systeme", "contenu": "Un utilisateur a quitte la room"}),
            room_id
        )


@app.websocket("/ws/dashboard")
async def endpoint_dashboard(websocket: WebSocket):
    """Tableau de bord en temps reel — le serveur pousse des mises a jour."""
    await websocket.accept()
    
    try:
        import asyncio
        import random
        
        while True:
            # Simuler des metriques en temps reel
            metriques = {
                "type": "metriques",
                "cpu": round(random.uniform(20, 80), 1),
                "memoire": round(random.uniform(40, 90), 1),
                "connexions_actives": random.randint(50, 500),
                "requetes_par_seconde": random.randint(100, 2000),
                "timestamp": datetime.utcnow().isoformat()
            }
            
            await websocket.send_text(json.dumps(metriques))
            await asyncio.sleep(1)  # Mise a jour toutes les secondes
    
    except WebSocketDisconnect:
        print("Dashboard client deconnecte.")


# API REST complementaire (voir [[08 - APIs REST avec Flask]] et [[09 - APIs REST avec FastAPI]])
@app.get("/rooms")
async def lister_rooms():
    return {
        "rooms": {
            room: len(clients)
            for room, clients in manager.rooms.items()
        }
    }
```

```bash
# Lancer le serveur FastAPI
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

---

## Flask-SocketIO

Flask-SocketIO est une surcouche de Socket.io qui simplifie les WebSockets avec gestion automatique des rooms, namespaces, et reconnexion.

```bash
pip install flask-socketio eventlet
```

```python
from flask import Flask, render_template
from flask_socketio import SocketIO, emit, join_room, leave_room, send
import json

app = Flask(__name__)
app.config['SECRET_KEY'] = 'secret_holberton'
socketio = SocketIO(app, cors_allowed_origins="*")

# Stockage en memoire (en prod : utiliser Redis — voir [[03 - Redis et Caches]])
utilisateurs_connectes = {}


@socketio.on('connect')
def on_connect():
    """Appele quand un client se connecte."""
    from flask import request
    print(f"Client connecte : {request.sid}")
    emit('connexion_confirmee', {'sid': request.sid})


@socketio.on('disconnect')
def on_disconnect():
    from flask import request
    sid = request.sid
    if sid in utilisateurs_connectes:
        pseudo = utilisateurs_connectes.pop(sid)
        emit('systeme', {'contenu': f'{pseudo} a quitte le chat'}, broadcast=True)
    print(f"Client deconnecte : {sid}")


@socketio.on('rejoindre_room')
def on_rejoindre_room(data):
    """Un client rejoint une room de chat."""
    from flask import request
    room = data.get('room', 'general')
    pseudo = data.get('pseudo', f'Anon_{request.sid[:4]}')
    
    utilisateurs_connectes[request.sid] = pseudo
    join_room(room)
    
    # Notifier la room
    emit('systeme', {
        'contenu': f'{pseudo} a rejoint #{room}',
        'room': room
    }, room=room)


@socketio.on('message_chat')
def on_message_chat(data):
    """Diffuse un message dans une room."""
    from flask import request
    room = data.get('room', 'general')
    pseudo = utilisateurs_connectes.get(request.sid, 'Inconnu')
    
    emit('nouveau_message', {
        'pseudo': pseudo,
        'contenu': data.get('contenu', ''),
        'room': room
    }, room=room)


@socketio.on('quitter_room')
def on_quitter_room(data):
    from flask import request
    room = data.get('room', 'general')
    leave_room(room)
    emit('systeme', {'contenu': 'Vous avez quitte la room'})


if __name__ == '__main__':
    socketio.run(app, host='0.0.0.0', port=5000, debug=True)
```

---

## Client JavaScript complet — Application Chat

```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <title>Chat WebSocket</title>
    <style>
        body { font-family: sans-serif; max-width: 800px; margin: 0 auto; padding: 20px; }
        #messages { height: 400px; overflow-y: auto; border: 1px solid #ccc; padding: 10px; margin-bottom: 10px; }
        .message { margin: 5px 0; }
        .message.systeme { color: gray; font-style: italic; }
        .message.moi { color: blue; }
        input, button { padding: 8px; margin: 5px 0; }
        #input-message { width: 70%; }
        #statut { color: gray; font-size: 0.9em; }
    </style>
</head>
<body>
    <h1>Chat en temps reel</h1>
    <div id="statut">Deconnecte</div>
    
    <div>
        <input type="text" id="input-pseudo" placeholder="Votre pseudo" value="Alice">
        <input type="text" id="input-room" placeholder="Room" value="general">
        <button onclick="connecter()">Rejoindre</button>
        <button onclick="deconnecter()">Quitter</button>
    </div>
    
    <div id="messages"></div>
    
    <div>
        <input type="text" id="input-message" placeholder="Votre message...">
        <button onclick="envoyerMessage()">Envoyer</button>
    </div>

    <script>
        let ws = null;
        let pseudo = '';
        let room = '';
        
        function connecter() {
            pseudo = document.getElementById('input-pseudo').value.trim();
            room = document.getElementById('input-room').value.trim();
            
            if (!pseudo || !room) {
                alert('Entrez un pseudo et une room.');
                return;
            }
            
            // Se connecter au serveur FastAPI
            ws = new WebSocket(`ws://localhost:8000/ws/chat/${room}`);
            
            ws.addEventListener('open', () => {
                majStatut('Connecte a #' + room, 'green');
                afficherMessage({ type: 'systeme', contenu: `Vous avez rejoint #${room}` });
                
                // Envoyer une salutation
                ws.send(JSON.stringify({
                    type: 'rejoindre',
                    pseudo: pseudo,
                    room: room
                }));
            });
            
            ws.addEventListener('message', (event) => {
                const data = JSON.parse(event.data);
                afficherMessage(data);
            });
            
            ws.addEventListener('close', (event) => {
                majStatut(`Deconnecte (code: ${event.code})`, 'red');
                ws = null;
            });
            
            ws.addEventListener('error', () => {
                majStatut('Erreur de connexion', 'red');
            });
        }
        
        function deconnecter() {
            if (ws) {
                ws.close(1000, 'Deconnexion volontaire');
            }
        }
        
        function envoyerMessage() {
            const input = document.getElementById('input-message');
            const contenu = input.value.trim();
            
            if (!contenu || !ws || ws.readyState !== WebSocket.OPEN) return;
            
            ws.send(JSON.stringify({
                type: 'message',
                pseudo: pseudo,
                contenu: contenu,
                room: room,
                id: Date.now()
            }));
            
            // Afficher localement
            afficherMessage({ type: 'message', pseudo: pseudo + ' (moi)', contenu: contenu });
            input.value = '';
        }
        
        function afficherMessage(data) {
            const div = document.createElement('div');
            div.className = 'message ' + (data.type === 'systeme' ? 'systeme' : '');
            
            if (data.type === 'systeme') {
                div.textContent = `[Systeme] ${data.contenu}`;
            } else {
                const heure = new Date().toLocaleTimeString('fr-FR', { hour: '2-digit', minute: '2-digit' });
                div.textContent = `[${heure}] ${data.pseudo}: ${data.contenu}`;
                if (data.pseudo?.includes('(moi)')) div.className += ' moi';
            }
            
            const messages = document.getElementById('messages');
            messages.appendChild(div);
            messages.scrollTop = messages.scrollHeight;
        }
        
        function majStatut(texte, couleur) {
            const el = document.getElementById('statut');
            el.textContent = texte;
            el.style.color = couleur;
        }
        
        // Envoyer avec Enter
        document.getElementById('input-message').addEventListener('keydown', (e) => {
            if (e.key === 'Enter') envoyerMessage();
        });
    </script>
</body>
</html>
```

---

## Rooms et Namespaces avec Socket.io

Socket.io ajoute des concepts de haut niveau au-dessus des WebSockets bruts.

```javascript
// CLIENT JavaScript — Socket.io
import { io } from 'socket.io-client';

// Namespace par defaut '/'
const socket = io('http://localhost:5000');

// Namespace specifique (isolation logique)
const chatSocket = io('http://localhost:5000/chat');
const notifSocket = io('http://localhost:5000/notifications');

// Rejoindre une room (cote client — la room est geree cote serveur)
socket.emit('rejoindre_room', { room: 'equipe-backend', pseudo: 'Alice' });

// Ecouter des evenements specifiques
socket.on('nouveau_message', (data) => {
    console.log(`[${data.room}] ${data.pseudo}: ${data.contenu}`);
});

socket.on('utilisateur_rejoint', (data) => {
    console.log(`${data.pseudo} a rejoint la room`);
});

// Envoyer a une room
socket.emit('message_chat', {
    room: 'equipe-backend',
    contenu: 'Revue de code a 14h ?'
});
```

---

## Exemple : Tableau de Bord Live

```javascript
// CLIENT — Tableau de bord de metriques
const wsDashboard = new WebSocket('ws://localhost:8000/ws/dashboard');
const charts = {};

wsDashboard.addEventListener('message', (event) => {
    const metriques = JSON.parse(event.data);
    
    // Mettre a jour les indicateurs
    document.getElementById('cpu').textContent = metriques.cpu + '%';
    document.getElementById('memoire').textContent = metriques.memoire + '%';
    document.getElementById('connexions').textContent = metriques.connexions_actives;
    document.getElementById('rps').textContent = metriques.requetes_par_seconde;
    
    // Mettre a jour les couleurs selon les seuils
    const cpuEl = document.getElementById('cpu');
    cpuEl.style.color = metriques.cpu > 80 ? 'red' : metriques.cpu > 60 ? 'orange' : 'green';
    
    // Ajouter un point au graphique (Chart.js)
    if (charts.cpu) {
        const time = new Date(metriques.timestamp).toLocaleTimeString();
        charts.cpu.data.labels.push(time);
        charts.cpu.data.datasets[0].data.push(metriques.cpu);
        // Garder les 60 dernieres secondes
        if (charts.cpu.data.labels.length > 60) {
            charts.cpu.data.labels.shift();
            charts.cpu.data.datasets[0].data.shift();
        }
        charts.cpu.update('none'); // Mise a jour sans animation
    }
});
```

---

## Scaling WebSockets avec Redis

Quand on a plusieurs instances du serveur (load balancing), un client connecte a l'instance A ne peut pas recevoir les messages emis depuis l'instance B. Redis resout ce probleme via son mecanisme Pub/Sub.

```python
# Avec Flask-SocketIO + Redis comme message queue
from flask_socketio import SocketIO

# En production avec multiple workers
socketio = SocketIO(
    app,
    message_queue='redis://localhost:6379/',  # Redis comme broker
    channel='mon_app_chat'
)

# Toutes les instances partagent maintenant les messages via Redis
# Instance A emits → Redis → Instance B recoit → pousse au client
```

```python
# Avec FastAPI + Redis Pub/Sub (architecture manuelle)
import asyncio
import json
import redis.asyncio as aioredis
from fastapi import FastAPI, WebSocket

app = FastAPI()
redis_client = aioredis.from_url("redis://localhost:6379")

# Ensemble global de connexions (par noeud)
connexions: set[WebSocket] = set()


async def redis_listener():
    """Ecoute Redis et diffuse aux clients WS connectes sur ce noeud."""
    pubsub = redis_client.pubsub()
    await pubsub.subscribe("chat_global")
    
    async for message in pubsub.listen():
        if message["type"] == "message":
            # Diffuser aux clients WebSocket de CE noeud
            morts = set()
            for ws in connexions:
                try:
                    await ws.send_text(message["data"])
                except Exception:
                    morts.add(ws)
            connexions -= morts


@app.on_event("startup")
async def startup():
    asyncio.create_task(redis_listener())


@app.websocket("/ws/chat")
async def ws_chat(websocket: WebSocket):
    await websocket.accept()
    connexions.add(websocket)
    
    try:
        while True:
            data = await websocket.receive_text()
            # Publier dans Redis → tous les noeuds recevront le message
            await redis_client.publish("chat_global", data)
    except Exception:
        connexions.discard(websocket)
```

Voir [[03 - Redis et Caches]] pour les details sur Redis Pub/Sub et la configuration du cluster.

---

## Comparaison SSE vs WebSocket vs Long Polling

> [!info] Guide de choix
> | Scenario | Recommandation |
> |---|---|
> | Notifications serveur uniquement | SSE (Server-Sent Events) |
> | Chat, jeu multijoueur | WebSocket |
> | Dashboard de metriques | SSE ou WebSocket |
> | Collaboration en temps reel (type Google Docs) | WebSocket |
> | Support IE (vieux navigateurs) | Long Polling (fallback) |
> | Derriere un proxy HTTP strict | SSE (passe mieux les proxies) |
> | Bande passante minimale | WebSocket (frames legeres) |

```javascript
// SSE — Simple mais unidirectionnel
const eventSource = new EventSource('/events');
eventSource.addEventListener('message', (e) => {
    console.log('SSE recu :', e.data);
});

// WebSocket — Bidirectionnel
const ws = new WebSocket('ws://localhost:8765');
ws.send('message du client');
ws.onmessage = (e) => console.log('WS recu :', e.data);
```

---

## Gestion des Erreurs et Cas Limites

```python
import asyncio
import websockets
import json
from websockets.exceptions import ConnectionClosedError, ConnectionClosedOK


async def handler_robuste(websocket):
    """Handler avec gestion complete des cas d'erreur."""
    try:
        async for message in websocket:
            # Valider la taille du message
            if len(message) > 64 * 1024:  # 64 KB max
                await websocket.send(json.dumps({
                    "erreur": "Message trop volumineux (max 64 KB)"
                }))
                continue
            
            # Valider le JSON
            try:
                data = json.loads(message)
            except json.JSONDecodeError:
                await websocket.send(json.dumps({"erreur": "JSON invalide"}))
                continue
            
            # Traiter avec timeout
            try:
                resultat = await asyncio.wait_for(
                    traiter_message_async(data),
                    timeout=5.0  # 5 secondes max de traitement
                )
                await websocket.send(json.dumps(resultat))
            except asyncio.TimeoutError:
                await websocket.send(json.dumps({"erreur": "Timeout de traitement"}))
    
    except ConnectionClosedOK:
        pass  # Deconnexion normale
    except ConnectionClosedError as e:
        print(f"Connexion perdue : {e.code} — {e.reason}")
    except Exception as e:
        print(f"Erreur inattendue : {e}")
        try:
            await websocket.send(json.dumps({"erreur": "Erreur serveur interne"}))
            await websocket.close(1011, "Erreur interne")
        except Exception:
            pass  # La connexion est peut-etre deja morte


async def traiter_message_async(data: dict) -> dict:
    """Traitement async d'un message."""
    await asyncio.sleep(0.1)  # Simuler un traitement IO
    return {"type": "reponse", "contenu": f"Traite : {data}"}
```

---

## Exercices Pratiques

### Exercice 1 : Echo Server
1. Implementez un serveur WebSocket Python (asyncio + websockets) qui renvoie chaque message recu, mais en majuscules
2. Testez avec le client JavaScript natif (navigateur)
3. Ajoutez un compteur de messages envoyes/recus visible dans la reponse

### Exercice 2 : Chat Multi-Room
En utilisant FastAPI + WebSocket :
1. Implementez une API avec des rooms de chat (/ws/chat/{room_id})
2. Les clients doivent s'identifier avec un pseudo a la connexion
3. Les messages sont diffuses a tous les membres de la room
4. Implementez la liste des membres en ligne (`/rooms/{room_id}/membres`)
5. Testez avec deux onglets de navigateur sur la meme room

### Exercice 3 : Tableau de Bord Live
1. Creez un serveur qui envoie des "metriques" (CPU simule, connexions, etc.) toutes les secondes
2. Creez une page HTML qui affiche ces metriques en temps reel (pas de rechargement)
3. Ajoutez un historique des 30 dernieres valeurs de CPU (tableau HTML ou graphique)

### Exercice 4 : Reconnexion Automatique
1. Implementez le `WebSocketManager` JavaScript avec reconnexion exponentielle
2. Testez : demarrez le serveur, connectez le client, arretez le serveur, redemarrez-le
3. Verifiez que le client se reconnecte automatiquement et reprend les messages

### Exercice 5 : Rate Limiting WebSocket
Integrez le rate limiter de [[03 - Redis et Caches]] dans un serveur WebSocket :
1. Maximum 10 messages par seconde par client
2. Si depasse : envoyer une erreur et fermer la connexion
3. Utilisez un Redis local pour stocker les compteurs

Voir aussi [[09 - APIs REST avec FastAPI]] pour integrer les WebSockets dans une API REST complete, et [[03 - JavaScript Asynchrone]] pour bien comprendre les Promises et async/await utilises cote client.

> [!warning] A retenir
> - WebSocket = connexion TCP persistante et bidirectionnelle apres un handshake HTTP unique
> - Le protocole WS reduit drastiquement l'overhead vs HTTP polling (2-14 octets vs 500+ octets)
> - Toujours implementer la reconnexion automatique cote client (delai exponentiel)
> - Valider la taille et le format des messages cote serveur
> - Pour scaler sur plusieurs instances : Redis Pub/Sub comme broker de messages (voir [[03 - Redis et Caches]])
> - SSE suffit pour les notifications serveur → client ; WebSocket requis pour le bidirectionnel
