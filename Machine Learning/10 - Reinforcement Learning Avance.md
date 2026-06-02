# Reinforcement Learning Avancé

Le Reinforcement Learning (RL) est un paradigme d'apprentissage automatique où un agent apprend à prendre des décisions en interagissant avec un environnement, en maximisant un signal de récompense cumulé. Ce cours couvre l'ensemble du spectre du RL moderne, des fondations mathématiques jusqu'aux algorithmes état-de-l'art comme PPO et SAC, utilisés aujourd'hui dans des systèmes comme AlphaGo, ChatGPT (RLHF) et les robots autonomes. À la fin de ce cours, vous serez capables d'implémenter et d'entraîner des agents sur des environnements complexes en utilisant les outils professionnels du domaine.

---

## 1. Rappels Fondamentaux

### 1.1 Le Processus de Décision de Markov (MDP)

Un **MDP** (Markov Decision Process) est le cadre mathématique formel qui modélise tout problème de RL. Il est défini par un tuple $(S, A, P, R, \gamma)$.

> [!info] Définition formelle d'un MDP
> - **S** : espace des états (fini ou continu)
> - **A** : espace des actions (fini ou continu)
> - **P(s' | s, a)** : probabilité de transition — probabilité d'arriver en état s' en prenant l'action a depuis l'état s
> - **R(s, a, s')** : fonction de récompense — signal scalaire reçu après la transition
> - **γ ∈ [0, 1)** : facteur de discount — pondère les récompenses futures

La **propriété de Markov** est fondamentale : l'état futur ne dépend que de l'état présent, pas de l'historique complet.

```
P(s_{t+1} | s_t, a_t, s_{t-1}, a_{t-1}, ...) = P(s_{t+1} | s_t, a_t)
```

**Cycle d'interaction agent-environnement :**

```
         +------------------+
         |                  |
    s_t  |     AGENT        | a_t
  +----->|                  +------->+
  |      |  π(a|s) : policy |        |
  |      +------------------+        |
  |                                  |
  |      +------------------+        |
  |      |                  |        |
  +------+  ENVIRONNEMENT   |<-------+
  r_t    |                  |
  s_{t+1}|  Transition P    |
         |  Récompense R    |
         +------------------+
```

### 1.2 Politique, Retour et Value Function

**La politique (Policy)** $\pi$ est la stratégie de l'agent. Elle peut être :
- **Déterministe** : $\pi(s) = a$ — une action par état
- **Stochastique** : $\pi(a|s) = P(A=a|S=s)$ — distribution de probabilité sur les actions

**Le retour cumulé (Return)** $G_t$ est la somme des récompenses futures discountées :

$$G_t = R_{t+1} + \gamma R_{t+2} + \gamma^2 R_{t+3} + \ldots = \sum_{k=0}^{\infty} \gamma^k R_{t+k+1}$$

Le facteur $\gamma$ permet deux choses :
1. Assurer la convergence mathématique de la somme infinie
2. Modéliser la préférence pour les récompenses immédiates vs futures

**La fonction de valeur d'état (State-Value Function)** $V^\pi(s)$ mesure la qualité d'être dans un état en suivant la politique $\pi$ :

$$V^\pi(s) = \mathbb{E}_\pi[G_t | S_t = s]$$

**La fonction de valeur action-état (Action-Value / Q-Function)** $Q^\pi(s, a)$ mesure la qualité de prendre l'action $a$ depuis l'état $s$, puis suivre $\pi$ :

$$Q^\pi(s, a) = \mathbb{E}_\pi[G_t | S_t = s, A_t = a]$$

> [!tip] Relation entre V et Q
> On peut passer de Q à V en marginalisant sur les actions :
> $$V^\pi(s) = \sum_a \pi(a|s) \cdot Q^\pi(s, a)$$
> Et de V à Q :
> $$Q^\pi(s, a) = \sum_{s'} P(s'|s,a) \left[ R(s,a,s') + \gamma V^\pi(s') \right]$$

### 1.3 Les Équations de Bellman

Les équations de Bellman expriment une relation récursive fondamentale entre la valeur d'un état et la valeur des états successeurs.

**Équation de Bellman pour $V^\pi$** :

$$V^\pi(s) = \sum_a \pi(a|s) \sum_{s'} P(s'|s,a) \left[ R(s,a,s') + \gamma V^\pi(s') \right]$$

**Équation de Bellman pour $Q^\pi$** :

$$Q^\pi(s,a) = \sum_{s'} P(s'|s,a) \left[ R(s,a,s') + \gamma \sum_{a'} \pi(a'|s') Q^\pi(s',a') \right]$$

**Équations d'optimalité de Bellman** (pour la politique optimale $\pi^*$) :

$$V^*(s) = \max_a \sum_{s'} P(s'|s,a) \left[ R(s,a,s') + \gamma V^*(s') \right]$$

$$Q^*(s,a) = \sum_{s'} P(s'|s,a) \left[ R(s,a,s') + \gamma \max_{a'} Q^*(s',a') \right]$$

> [!info] Pourquoi ces équations sont cruciales
> Trouver $Q^*$ suffit à résoudre le MDP : la politique optimale est simplement $\pi^*(s) = \arg\max_a Q^*(s,a)$. Toute la théorie du RL converge vers l'approximation de $Q^*$ ou de $\pi^*$.

---

## 2. Q-Learning Tabulaire

### 2.1 Algorithme Q-Learning

Le Q-Learning (Watkins, 1989) est un algorithme **model-free** et **off-policy** qui apprend directement $Q^*$ sans modèle de l'environnement.

**Mise à jour Q-Learning :**

$$Q(s_t, a_t) \leftarrow Q(s_t, a_t) + \alpha \left[ r_t + \gamma \max_{a'} Q(s_{t+1}, a') - Q(s_t, a_t) \right]$$

- $\alpha$ : taux d'apprentissage (learning rate)
- $r_t + \gamma \max_{a'} Q(s_{t+1}, a')$ : **cible TD** (Temporal Difference target)
- $r_t + \gamma \max_{a'} Q(s_{t+1}, a') - Q(s_t, a_t)$ : **erreur TD** (TD error)

```python
import numpy as np
import gymnasium as gym
from collections import defaultdict

def q_learning(env_name="FrozenLake-v1", n_episodes=10000,
               alpha=0.1, gamma=0.99, epsilon_start=1.0,
               epsilon_end=0.01, epsilon_decay=0.999):
    """
    Q-Learning tabulaire complet.
    
    Args:
        env_name: nom de l'environnement Gymnasium
        n_episodes: nombre d'épisodes d'entraînement
        alpha: taux d'apprentissage
        gamma: facteur de discount
        epsilon_start: epsilon initial (exploration maximale)
        epsilon_end: epsilon minimal (exploitation)
        epsilon_decay: décroissance multiplicative de epsilon
    """
    env = gym.make(env_name)
    n_states = env.observation_space.n
    n_actions = env.action_space.n
    
    # Table Q initialisée à zéro
    # Q[s][a] = valeur estimée de (s, a)
    Q = np.zeros((n_states, n_actions))
    
    epsilon = epsilon_start
    rewards_per_episode = []
    
    for episode in range(n_episodes):
        state, _ = env.reset()
        total_reward = 0
        done = False
        
        while not done:
            # === Politique epsilon-greedy ===
            if np.random.random() < epsilon:
                action = env.action_space.sample()  # Exploration aléatoire
            else:
                action = np.argmax(Q[state])         # Exploitation greedy
            
            # Exécuter l'action dans l'environnement
            next_state, reward, terminated, truncated, _ = env.step(action)
            done = terminated or truncated
            
            # === Mise à jour de Bellman ===
            td_target = reward + gamma * np.max(Q[next_state]) * (not done)
            td_error = td_target - Q[state, action]
            Q[state, action] += alpha * td_error
            
            state = next_state
            total_reward += reward
        
        # Décroissance d'epsilon
        epsilon = max(epsilon_end, epsilon * epsilon_decay)
        rewards_per_episode.append(total_reward)
        
        # Log toutes les 1000 épisodes
        if (episode + 1) % 1000 == 0:
            avg_reward = np.mean(rewards_per_episode[-100:])
            print(f"Episode {episode+1:5d} | Avg Reward: {avg_reward:.3f} | ε: {epsilon:.4f}")
    
    env.close()
    return Q, rewards_per_episode


def evaluate_policy(Q, env_name="FrozenLake-v1", n_eval=100):
    """Évalue la politique greedy apprise."""
    env = gym.make(env_name, render_mode=None)
    success_count = 0
    
    for _ in range(n_eval):
        state, _ = env.reset()
        done = False
        
        while not done:
            action = np.argmax(Q[state])  # Politique purement greedy
            state, reward, terminated, truncated, _ = env.step(action)
            done = terminated or truncated
        
        if reward > 0:
            success_count += 1
    
    env.close()
    print(f"Taux de succès : {success_count / n_eval * 100:.1f}%")
    return success_count / n_eval


# Entraînement et évaluation
Q_table, rewards = q_learning()
win_rate = evaluate_policy(Q_table)
```

### 2.2 La Politique Epsilon-Greedy

Le dilemme **exploration vs exploitation** est central en RL : l'agent doit exploiter ses connaissances actuelles (greedy) tout en explorant pour découvrir de meilleures stratégies.

```
Probabilité de choisir une action :

P(action aléatoire) = ε
P(action greedy)    = 1 - ε

Décroissance typique :
ε_t = max(ε_min, ε_start × decay^t)
```

> [!warning] Pièges courants avec epsilon-greedy
> - **ε trop bas dès le début** : l'agent exploite des connaissances quasi nulles — converge vers un minimum local
> - **ε qui ne décroît pas** : l'agent continue d'explorer inutilement même quand la politique est bonne
> - **ε trop lent à décroître** : apprentissage très lent en fin d'entraînement

### 2.3 Convergence du Q-Learning

**Théorème de convergence** : Q-Learning converge vers $Q^*$ avec probabilité 1 si :
1. Chaque paire (état, action) est visitée infiniment souvent
2. $\sum_t \alpha_t = \infty$ et $\sum_t \alpha_t^2 < \infty$ (conditions de Robbins-Monro)
3. Les récompenses sont bornées

> [!info] Pourquoi Q-Learning est "off-policy"
> La mise à jour utilise $\max_{a'} Q(s', a')$ — la meilleure action possible — indépendamment de l'action réellement choisie par la politique comportementale (epsilon-greedy). L'agent peut apprendre $Q^*$ même en explorant aléatoirement.

**Comparaison Q-Learning vs SARSA :**

| Critère | Q-Learning | SARSA |
|---------|-----------|-------|
| Type | Off-policy | On-policy |
| Mise à jour | $\max_{a'} Q(s', a')$ | $Q(s', a')$ avec $a' \sim \pi$ |
| Convergence | Vers $Q^*$ (optimale) | Vers $Q^\pi$ (politique actuelle) |
| Comportement falaise | Risqué (prend des raccourcis) | Prudent (évite les risques) |
| Usage | Quand on peut simuler librement | Quand l'agent agit en vrai |

---

## 3. DQN — Deep Q-Network

### 3.1 Motivation : Limites du Q-Learning Tabulaire

Le Q-Learning tabulaire est inapplicable dès que l'espace d'états est grand ou continu :
- **Atari** : état = image 84×84×3 → $256^{84 \times 84 \times 3}$ états possibles
- **Robotique** : état = positions articulaires continues

**Solution** : approximer $Q(s, a; \theta)$ avec un réseau de neurones paramétré par $\theta$.

$$Q(s, a; \theta) \approx Q^*(s, a)$$

### 3.2 Instabilité du Q-Learning avec Réseaux de Neurones

Naïvement remplacer la table par un réseau crée des instabilités majeures :

```
Problème 1 — Corrélations temporelles :
  Les transitions consécutives (s_t, a_t, r_t, s_{t+1}) sont très corrélées
  → le réseau "oublie" en optimisant toujours sur les mêmes patterns récents
  → équivalent d'un humain qui révise toujours le même chapitre

Problème 2 — Distribution changeante des cibles :
  La cible TD est r + γ max_a' Q(s', a'; θ)
  Mais θ est mis à jour à chaque step → la cible bouge en même temps que les prédictions
  → "chasser sa propre queue" → divergence
```

### 3.3 Architecture DQN et Innovations

**DQN (Mnih et al., DeepMind 2013/2015)** résout ces deux problèmes avec deux innovations :

**Innovation 1 : Replay Buffer (Experience Replay)**

```python
from collections import deque
import random

class ReplayBuffer:
    """
    Buffer circulaire stockant les transitions passées.
    Brise les corrélations temporelles en échantillonnant aléatoirement.
    """
    def __init__(self, capacity=100_000):
        self.buffer = deque(maxlen=capacity)
    
    def push(self, state, action, reward, next_state, done):
        """Stocke une transition."""
        self.buffer.append((state, action, reward, next_state, done))
    
    def sample(self, batch_size):
        """Échantillonnage aléatoire uniforme — brise les corrélations."""
        transitions = random.sample(self.buffer, batch_size)
        states, actions, rewards, next_states, dones = zip(*transitions)
        
        return (
            np.array(states, dtype=np.float32),
            np.array(actions, dtype=np.int64),
            np.array(rewards, dtype=np.float32),
            np.array(next_states, dtype=np.float32),
            np.array(dones, dtype=np.float32)
        )
    
    def __len__(self):
        return len(self.buffer)
```

**Innovation 2 : Target Network**

```python
import torch
import torch.nn as nn
import copy

class DQNNetwork(nn.Module):
    """
    Réseau Q pour des observations vectorielles.
    Entrée : état → Sortie : Q(s, a) pour toutes les actions
    """
    def __init__(self, obs_dim, n_actions, hidden_dim=128):
        super().__init__()
        self.network = nn.Sequential(
            nn.Linear(obs_dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, n_actions)
        )
    
    def forward(self, x):
        return self.network(x)


class DQNAgent:
    """
    Agent DQN complet avec replay buffer et target network.
    """
    def __init__(self, obs_dim, n_actions, lr=1e-3, gamma=0.99,
                 buffer_capacity=50_000, batch_size=64, target_update_freq=100):
        self.n_actions = n_actions
        self.gamma = gamma
        self.batch_size = batch_size
        self.target_update_freq = target_update_freq
        self.steps = 0
        
        # Réseau principal (mis à jour à chaque step)
        self.q_network = DQNNetwork(obs_dim, n_actions)
        
        # Réseau cible (mis à jour périodiquement — stabilise l'entraînement)
        # C'est une COPIE FIGÉE du réseau principal
        self.target_network = copy.deepcopy(self.q_network)
        self.target_network.eval()  # Mode inference uniquement
        
        self.optimizer = torch.optim.Adam(self.q_network.parameters(), lr=lr)
        self.replay_buffer = ReplayBuffer(buffer_capacity)
        
        # Politique epsilon-greedy
        self.epsilon = 1.0
        self.epsilon_min = 0.01
        self.epsilon_decay = 0.995
    
    def select_action(self, state):
        """Sélection d'action epsilon-greedy."""
        if np.random.random() < self.epsilon:
            return np.random.randint(self.n_actions)
        
        with torch.no_grad():
            state_tensor = torch.FloatTensor(state).unsqueeze(0)
            q_values = self.q_network(state_tensor)
            return q_values.argmax().item()
    
    def train_step(self):
        """Une étape de gradient descent."""
        if len(self.replay_buffer) < self.batch_size:
            return None  # Pas assez d'expériences
        
        # Échantillonner un mini-batch aléatoire du replay buffer
        states, actions, rewards, next_states, dones = \
            self.replay_buffer.sample(self.batch_size)
        
        states = torch.FloatTensor(states)
        actions = torch.LongTensor(actions)
        rewards = torch.FloatTensor(rewards)
        next_states = torch.FloatTensor(next_states)
        dones = torch.FloatTensor(dones)
        
        # Q-values actuelles : Q(s, a) pour les actions choisies
        current_q = self.q_network(states).gather(1, actions.unsqueeze(1)).squeeze(1)
        
        # Cibles TD avec le TARGET NETWORK (figé)
        with torch.no_grad():
            # max_a' Q_target(s', a') — réseau cible plus stable
            next_q = self.target_network(next_states).max(1)[0]
            # Si done=True, pas de récompense future (épisode terminé)
            target_q = rewards + self.gamma * next_q * (1 - dones)
        
        # Loss Huber (plus robuste que MSE aux outliers)
        loss = nn.SmoothL1Loss()(current_q, target_q)
        
        self.optimizer.zero_grad()
        loss.backward()
        # Gradient clipping — stabilise l'entraînement
        torch.nn.utils.clip_grad_norm_(self.q_network.parameters(), 10)
        self.optimizer.step()
        
        # Mise à jour périodique du target network
        self.steps += 1
        if self.steps % self.target_update_freq == 0:
            self.target_network.load_state_dict(self.q_network.state_dict())
        
        # Décroissance epsilon
        self.epsilon = max(self.epsilon_min, self.epsilon * self.epsilon_decay)
        
        return loss.item()


def train_dqn(env_name="CartPole-v1", n_episodes=500):
    """Entraînement complet d'un agent DQN."""
    env = gym.make(env_name)
    obs_dim = env.observation_space.shape[0]
    n_actions = env.action_space.n
    
    agent = DQNAgent(obs_dim, n_actions)
    rewards_history = []
    
    for episode in range(n_episodes):
        state, _ = env.reset()
        total_reward = 0
        done = False
        
        while not done:
            action = agent.select_action(state)
            next_state, reward, terminated, truncated, _ = env.step(action)
            done = terminated or truncated
            
            # Stocker la transition dans le replay buffer
            agent.replay_buffer.push(state, action, reward, next_state, done)
            
            # Entraîner le réseau
            loss = agent.train_step()
            
            state = next_state
            total_reward += reward
        
        rewards_history.append(total_reward)
        
        if (episode + 1) % 50 == 0:
            avg = np.mean(rewards_history[-50:])
            print(f"Ep {episode+1:4d} | Avg Reward: {avg:6.1f} | ε: {agent.epsilon:.3f}")
    
    env.close()
    return agent, rewards_history
```

### 3.4 DQN pour Atari

Pour les jeux Atari, l'architecture utilise des convolutions sur les frames de jeu :

```python
class AtariDQN(nn.Module):
    """
    Architecture DQN originale pour Atari (Mnih 2015).
    Entrée : 4 frames empilées en niveaux de gris 84x84
    Sortie : Q-value pour chaque action possible
    """
    def __init__(self, n_actions):
        super().__init__()
        
        # Extraction de features visuelles
        self.conv = nn.Sequential(
            nn.Conv2d(4, 32, kernel_size=8, stride=4),  # 84x84 → 20x20
            nn.ReLU(),
            nn.Conv2d(32, 64, kernel_size=4, stride=2),  # 20x20 → 9x9
            nn.ReLU(),
            nn.Conv2d(64, 64, kernel_size=3, stride=1),  # 9x9 → 7x7
            nn.ReLU()
        )
        
        # Calcul de la dimension après convolutions
        conv_out_dim = 64 * 7 * 7  # = 3136
        
        # Couches fully-connected
        self.fc = nn.Sequential(
            nn.Linear(conv_out_dim, 512),
            nn.ReLU(),
            nn.Linear(512, n_actions)
        )
    
    def forward(self, x):
        # x shape: (batch, 4, 84, 84), normalisé entre 0 et 1
        x = x / 255.0
        x = self.conv(x)
        x = x.flatten(start_dim=1)
        return self.fc(x)
```

> [!info] Résultats Atari
> DQN a dépassé un joueur humain expert sur 29 des 49 jeux Atari testés. C'est la première démonstration qu'un seul algorithme peut apprendre des tâches très différentes depuis des pixels bruts — une rupture majeure dans l'histoire du RL.

---

## 4. Améliorations de DQN

### 4.1 Double DQN (DDQN)

**Problème avec DQN standard** : le `max` dans la cible TD cause une **surestimation systématique** des Q-values.

```
Cible DQN standard :
  target = r + γ · max_a' Q_target(s', a'; θ⁻)
  
Problème : on utilise le MÊME réseau pour CHOISIR l'action et ÉVALUER sa valeur
→ les erreurs d'estimation sont maximisées plutôt que moyennées
→ Q-values gonflées, politique sous-optimale
```

**Solution Double DQN (van Hasselt et al., 2015)** :
- **Réseau Q principal** : choisit quelle action est la meilleure
- **Réseau cible** : évalue la valeur de cette action

```python
# DQN standard — surestimation
next_q_standard = self.target_network(next_states).max(1)[0]

# Double DQN — décomposition en deux étapes
with torch.no_grad():
    # Étape 1 : Choisir la meilleure action avec le réseau PRINCIPAL
    best_actions = self.q_network(next_states).argmax(1)
    
    # Étape 2 : Évaluer cette action avec le réseau CIBLE
    next_q_double = self.target_network(next_states).gather(
        1, best_actions.unsqueeze(1)
    ).squeeze(1)

target_q = rewards + self.gamma * next_q_double * (1 - dones)
```

> [!tip] Gain pratique de Double DQN
> Sur les jeux Atari, DDQN améliore les performances sur 80% des jeux et réduit la surestimation des Q-values de 50% en moyenne. Le changement de code est minime — c'est l'une des améliorations les plus rentables.

### 4.2 Dueling DQN

**Intuition** : pour beaucoup d'états, la valeur de l'état $V(s)$ est plus informative que les Q-values individuelles. Dueling DQN décompose explicitement :

$$Q(s, a) = V(s) + A(s, a)$$

où $A(s, a) = Q(s, a) - V(s)$ est l'**Advantage** — combien l'action $a$ est meilleure que la moyenne.

```python
class DuelingDQN(nn.Module):
    """
    Architecture Dueling DQN (Wang et al., 2016).
    Sépare l'estimation de V(s) et A(s,a) en deux flux distincts.
    """
    def __init__(self, obs_dim, n_actions, hidden_dim=128):
        super().__init__()
        
        # Couche partagée (feature extraction)
        self.shared = nn.Sequential(
            nn.Linear(obs_dim, hidden_dim),
            nn.ReLU()
        )
        
        # Flux Value : V(s) — valeur scalaire de l'état
        self.value_stream = nn.Sequential(
            nn.Linear(hidden_dim, hidden_dim // 2),
            nn.ReLU(),
            nn.Linear(hidden_dim // 2, 1)  # Un seul scalaire
        )
        
        # Flux Advantage : A(s, a) — avantage relatif de chaque action
        self.advantage_stream = nn.Sequential(
            nn.Linear(hidden_dim, hidden_dim // 2),
            nn.ReLU(),
            nn.Linear(hidden_dim // 2, n_actions)  # Un scalaire par action
        )
    
    def forward(self, x):
        features = self.shared(x)
        
        value = self.value_stream(features)          # shape: (batch, 1)
        advantage = self.advantage_stream(features)  # shape: (batch, n_actions)
        
        # Combinaison avec normalisation de l'advantage
        # On soustrait la moyenne pour que E[A(s,a)] = 0
        # (si on soustrait le max, l'identifiabilité est forcée mais instable)
        q_values = value + advantage - advantage.mean(dim=1, keepdim=True)
        
        return q_values
```

**Visualisation de l'architecture :**

```
       Input (état)
           |
    +------+------+
    |  Shared FC   |
    +------+------+
           |
    +------+------+
    |              |
  Value         Advantage
  Stream         Stream
    |              |
  V(s)           A(s,a₁)
                 A(s,a₂)
                 A(s,a₃)
    |              |
    +------+------+
           |
  Q(s,a) = V(s) + A(s,a) - mean(A(s,·))
```

### 4.3 Prioritized Experience Replay (PER)

**Problème du replay buffer uniforme** : certaines transitions sont plus instructives que d'autres. Échantillonner uniformément gaspille du temps sur des transitions "ennuyeuses" (déjà bien apprises).

**Idée PER** : échantillonner plus souvent les transitions avec une grande erreur TD (celles où l'agent se trompe le plus).

```python
import heapq

class PrioritizedReplayBuffer:
    """
    Replay buffer avec priorités proportionnelles à l'erreur TD.
    Utilise un sum-tree pour un échantillonnage efficace O(log n).
    """
    def __init__(self, capacity, alpha=0.6, beta_start=0.4, beta_end=1.0):
        self.capacity = capacity
        self.alpha = alpha     # Degré de prioritisation (0 = uniforme, 1 = greedy)
        self.beta = beta_start # Correction IS (importance sampling)
        self.beta_end = beta_end
        
        self.buffer = []
        self.priorities = np.zeros(capacity, dtype=np.float32)
        self.pos = 0
        self.size = 0
        self.max_priority = 1.0
    
    def push(self, state, action, reward, next_state, done):
        """Stocke avec la priorité maximale actuelle (optimisme initial)."""
        transition = (state, action, reward, next_state, done)
        
        if self.size < self.capacity:
            self.buffer.append(transition)
        else:
            self.buffer[self.pos] = transition
        
        # Nouvelle transition → priorité maximale (garantit qu'elle sera vue au moins une fois)
        self.priorities[self.pos] = self.max_priority
        self.pos = (self.pos + 1) % self.capacity
        self.size = min(self.size + 1, self.capacity)
    
    def sample(self, batch_size, beta=None):
        """
        Échantillonnage proportionnel aux priorités.
        Retourne aussi les poids IS pour corriger le biais.
        """
        if beta is None:
            beta = self.beta
        
        priorities = self.priorities[:self.size]
        
        # Probabilités proportionnelles à P(i) = p_i^alpha / sum(p_j^alpha)
        probs = priorities[:self.size] ** self.alpha
        probs /= probs.sum()
        
        indices = np.random.choice(self.size, batch_size, p=probs, replace=False)
        transitions = [self.buffer[idx] for idx in indices]
        
        # Importance Sampling weights pour corriger le biais d'échantillonnage
        # w_i = (1/N · 1/P(i))^beta, normalisé par max(w_j)
        weights = (self.size * probs[indices]) ** (-beta)
        weights /= weights.max()  # Normalisation
        
        states, actions, rewards, next_states, dones = zip(*transitions)
        
        return (
            np.array(states, dtype=np.float32),
            np.array(actions, dtype=np.int64),
            np.array(rewards, dtype=np.float32),
            np.array(next_states, dtype=np.float32),
            np.array(dones, dtype=np.float32),
            indices,
            np.array(weights, dtype=np.float32)
        )
    
    def update_priorities(self, indices, td_errors):
        """Met à jour les priorités après le calcul des erreurs TD."""
        priorities = np.abs(td_errors) + 1e-6  # Epsilon pour éviter les priorités nulles
        self.priorities[indices] = priorities
        self.max_priority = max(self.max_priority, priorities.max())
```

**Tableau récapitulatif des variantes DQN :**

| Variante | Problème résolu | Gain typique |
|----------|----------------|--------------|
| DQN | Instabilité avec réseau de neurones | Baseline |
| Double DQN | Surestimation des Q-values | +10-15% |
| Dueling DQN | Inefficacité en états peu différenciés | +15-20% |
| PER | Sous-utilisation des transitions rares | +15-25% |
| Rainbow | Combinaison de toutes les améliorations | +50-100% |

---

## 5. Policy Gradient — REINFORCE

### 5.1 Motivation : Optimiser Directement la Politique

DQN est une méthode **value-based** : on apprend $Q^*$, puis on dérive $\pi^*$. Les méthodes **policy-based** optimisent directement les paramètres $\theta$ de la politique $\pi_\theta$.

**Avantages des méthodes policy gradient :**
- Naturellement adaptées aux espaces d'actions **continus** (robotique, contrôle)
- Politiques **stochastiques** natives — utile quand l'optimalité est stochastique
- Convergence vers un optimum local garanti (sous conditions)

**Objectif à maximiser :**

$$J(\theta) = \mathbb{E}_{\tau \sim \pi_\theta}[G(\tau)] = \mathbb{E}_{\tau \sim \pi_\theta}\left[\sum_{t=0}^T r_t\right]$$

### 5.2 Le Théorème Policy Gradient

$$\nabla_\theta J(\theta) = \mathbb{E}_{\pi_\theta}\left[\sum_{t=0}^T \nabla_\theta \log \pi_\theta(a_t|s_t) \cdot G_t\right]$$

**Intuition** : augmenter la probabilité des actions qui ont mené à un retour élevé, diminuer celles qui ont mené à un retour faible.

```
Si G_t > 0 : ∇ log π(a|s) pointe vers des paramètres qui augmentent π(a|s)
              → on fait un pas dans cette direction → π(a|s) augmente ✓

Si G_t < 0 : ∇ log π(a|s) pointe vers plus de probabilité
              → on fait un pas dans la direction OPPOSÉE → π(a|s) diminue ✓
```

### 5.3 Algorithme REINFORCE

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.distributions import Categorical
import gymnasium as gym

class PolicyNetwork(nn.Module):
    """
    Réseau de politique stochastique.
    Sortie : distribution de probabilité sur les actions (softmax).
    """
    def __init__(self, obs_dim, n_actions, hidden_dim=128):
        super().__init__()
        self.network = nn.Sequential(
            nn.Linear(obs_dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, n_actions)
        )
    
    def forward(self, x):
        logits = self.network(x)
        return Categorical(logits=logits)  # Distribution discrète
    
    def act(self, state):
        """Échantillonne une action et retourne son log-probabilité."""
        dist = self.forward(state)
        action = dist.sample()
        log_prob = dist.log_prob(action)
        return action.item(), log_prob


def reinforce(env_name="CartPole-v1", n_episodes=1000, lr=1e-3, gamma=0.99):
    """
    Algorithme REINFORCE (Monte Carlo Policy Gradient).
    """
    env = gym.make(env_name)
    obs_dim = env.observation_space.shape[0]
    n_actions = env.action_space.n
    
    policy = PolicyNetwork(obs_dim, n_actions)
    optimizer = optim.Adam(policy.parameters(), lr=lr)
    
    rewards_history = []
    
    for episode in range(n_episodes):
        state, _ = env.reset()
        
        # === Collecte d'un épisode complet ===
        log_probs = []   # Log-probabilités des actions choisies
        rewards = []     # Récompenses reçues
        
        done = False
        while not done:
            state_tensor = torch.FloatTensor(state).unsqueeze(0)
            action, log_prob = policy.act(state_tensor)
            
            next_state, reward, terminated, truncated, _ = env.step(action)
            done = terminated or truncated
            
            log_probs.append(log_prob)
            rewards.append(reward)
            state = next_state
        
        # === Calcul des retours G_t (discountés, de la fin vers le début) ===
        returns = []
        G = 0
        for r in reversed(rewards):
            G = r + gamma * G
            returns.insert(0, G)
        
        returns = torch.tensor(returns, dtype=torch.float32)
        
        # Normalisation des retours (réduit la variance)
        returns = (returns - returns.mean()) / (returns.std() + 1e-8)
        
        # === Calcul de la loss REINFORCE ===
        # Loss = -E[log π(a|s) · G_t]
        # (négatif car on maximise J mais PyTorch minimise)
        policy_loss = []
        for log_prob, G_t in zip(log_probs, returns):
            policy_loss.append(-log_prob * G_t)
        
        loss = torch.stack(policy_loss).sum()
        
        # === Mise à jour des paramètres ===
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()
        
        episode_reward = sum(rewards)
        rewards_history.append(episode_reward)
        
        if (episode + 1) % 100 == 0:
            avg = np.mean(rewards_history[-100:])
            print(f"Episode {episode+1:5d} | Avg Reward: {avg:6.1f}")
    
    env.close()
    return policy, rewards_history
```

### 5.4 Réduction de Variance avec Baseline

**Problème de REINFORCE** : haute variance du gradient. Le retour $G_t$ peut être très grand ou très petit selon la chance, créant des mises à jour instables.

**Solution : soustraire une baseline $b(s_t)$** qui ne biaise pas le gradient :

$$\nabla_\theta J(\theta) = \mathbb{E}_{\pi_\theta}\left[\nabla_\theta \log \pi_\theta(a_t|s_t) \cdot (G_t - b(s_t))\right]$$

La baseline la plus efficace est $V(s_t)$ — la valeur de l'état. On obtient alors l'**Advantage** :

$$A_t = G_t - V(s_t)$$

```python
class PolicyWithBaseline(nn.Module):
    """
    Réseau combiné politique + baseline (value network).
    Partage les couches de features pour l'efficacité.
    """
    def __init__(self, obs_dim, n_actions, hidden_dim=128):
        super().__init__()
        
        # Couches partagées
        self.shared = nn.Sequential(
            nn.Linear(obs_dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, hidden_dim),
            nn.ReLU()
        )
        
        # Tête politique
        self.policy_head = nn.Linear(hidden_dim, n_actions)
        
        # Tête valeur (baseline)
        self.value_head = nn.Linear(hidden_dim, 1)
    
    def forward(self, x):
        features = self.shared(x)
        logits = self.policy_head(features)
        value = self.value_head(features)
        return Categorical(logits=logits), value.squeeze(-1)
```

> [!warning] Biais vs Variance en Policy Gradient
> - **Retour Monte Carlo** $G_t$ : non biaisé, haute variance (attend la fin de l'épisode)
> - **TD(0)** $r + \gamma V(s')$ : biaisé (dépend de $V$ approché), basse variance
> - **n-step returns** : compromis ajustable entre les deux extrêmes

---

## 6. Actor-Critic — A2C

### 6.1 Architecture Actor-Critic

L'Actor-Critic combine une **politique** (l'acteur) et une **fonction de valeur** (le critique) :

```
            État s_t
               |
        +------+------+
        |              |
     ACTEUR         CRITIQUE
   π(a|s; θ)       V(s; φ)
        |              |
   Action a_t      Valeur V(s_t)
        |
   Env → r_t, s_{t+1}
        |              |
        +----- A_t ----+
          A_t = r_t + γV(s_{t+1}) - V(s_t)
```

Le **critique** réduit la variance en apprenant $V(s)$. L'**acteur** optimise la politique guidé par le critique.

### 6.2 Advantage Actor-Critic (A2C)

**L'Advantage** mesure si une action est meilleure ou moins bonne que la moyenne :

$$A_t = r_t + \gamma V(s_{t+1}) - V(s_t)$$

```python
class A2CAgent:
    """
    Advantage Actor-Critic (A2C).
    Version synchrone — plus simple qu'A3C, aussi efficace sur GPU.
    """
    def __init__(self, obs_dim, n_actions, lr_actor=3e-4, lr_critic=1e-3,
                 gamma=0.99, entropy_coef=0.01, value_coef=0.5):
        self.gamma = gamma
        self.entropy_coef = entropy_coef  # Regularisation par entropie
        self.value_coef = value_coef
        
        # Réseau partagé avec deux têtes
        self.network = PolicyWithBaseline(obs_dim, n_actions)
        self.optimizer = optim.Adam(self.network.parameters(), lr=lr_actor)
    
    def compute_returns(self, rewards, values, dones, next_value):
        """
        Calcule les retours n-step avec bootstrap depuis next_value.
        """
        returns = []
        R = next_value
        
        for step in reversed(range(len(rewards))):
            R = rewards[step] + self.gamma * R * (1 - dones[step])
            returns.insert(0, R)
        
        return torch.tensor(returns, dtype=torch.float32)
    
    def update(self, states, actions, rewards, dones, next_state):
        """
        Mise à jour Actor-Critic sur un batch de transitions.
        """
        states_t = torch.FloatTensor(np.array(states))
        actions_t = torch.LongTensor(actions)
        
        # Valeur bootstrap depuis le dernier état
        with torch.no_grad():
            _, next_value = self.network(torch.FloatTensor(next_state).unsqueeze(0))
        
        # Retours discountés
        returns = self.compute_returns(rewards, None, dones, next_value.item())
        
        # Prédictions actuelles
        dist, values = self.network(states_t)
        
        # Advantage
        advantages = returns - values.detach()
        
        # === Loss Acteur ===
        # Maximiser E[log π(a|s) · A_t] → minimiser -E[...]
        log_probs = dist.log_prob(actions_t)
        actor_loss = -(log_probs * advantages).mean()
        
        # === Loss Critique ===
        # Minimiser (V(s) - G_t)² — régression classique
        critic_loss = nn.MSELoss()(values, returns)
        
        # === Bonus Entropie ===
        # Encourage l'exploration en pénalisant les politiques trop certaines
        entropy_loss = -dist.entropy().mean()
        
        # Loss totale
        total_loss = (actor_loss 
                     + self.value_coef * critic_loss 
                     + self.entropy_coef * entropy_loss)
        
        self.optimizer.zero_grad()
        total_loss.backward()
        torch.nn.utils.clip_grad_norm_(self.network.parameters(), 0.5)
        self.optimizer.step()
        
        return {
            'actor_loss': actor_loss.item(),
            'critic_loss': critic_loss.item(),
            'entropy': -entropy_loss.item()
        }
```

> [!info] A2C vs A3C
> **A3C** (Asynchronous Advantage Actor-Critic, Mnih 2016) utilise plusieurs workers asynchrones en parallèle — plus efficace sur CPU multi-cœurs. **A2C** est la version synchronisée : les workers collectent en parallèle et font une mise à jour commune — plus stable sur GPU. En pratique avec Stable-Baselines3, A2C est préféré.

---

## 7. PPO — Proximal Policy Optimization

### 7.1 Problème des Policy Gradients Classiques

Les mises à jour de gradient peuvent être trop grandes, détruisant la politique actuelle :

```
Mise à jour "classique" :
  θ_new = θ_old + α · ∇_θ J(θ)

Problème : si le gradient est grand, θ_new peut être très loin de θ_old
→ La nouvelle politique peut être radicalement différente
→ Performance peut s'effondrer (catastrophic forgetting)
→ Difficile à récupérer sans revenir en arrière
```

**TRPO** (Trust Region Policy Optimization) résout ce problème avec une contrainte sur la divergence KL entre les politiques, mais est complexe à implémenter (optimisation de second ordre).

**PPO** (Schulman et al., 2017) obtient des garanties similaires avec une méthode beaucoup plus simple.

### 7.2 L'Objectif Clip de PPO

**Ratio d'importance sampling :**

$$r_t(\theta) = \frac{\pi_\theta(a_t|s_t)}{\pi_{\theta_{old}}(a_t|s_t)}$$

**Objectif PPO-Clip :**

$$L^{CLIP}(\theta) = \mathbb{E}_t\left[\min\left(r_t(\theta) A_t, \text{clip}(r_t(\theta), 1-\epsilon, 1+\epsilon) A_t\right)\right]$$

```
Intuition visuelle de l'objectif Clip :

r_t = π_new / π_old

Si A_t > 0 (bonne action) :
  - On veut augmenter r_t (rendre l'action plus probable)
  - Mais clip à 1+ε : on ne pousse pas trop fort
  
Si A_t < 0 (mauvaise action) :
  - On veut diminuer r_t (rendre l'action moins probable)
  - Mais clip à 1-ε : on ne pousse pas trop fort non plus

Résultat : la politique ne peut pas changer de plus de ε≈0.2 par mise à jour
→ Garantie implicite de stabilité sans contrainte KL explicite
```

```python
class PPOAgent:
    """
    Proximal Policy Optimization complet.
    Implémentation avec plusieurs epochs sur les données collectées.
    """
    def __init__(self, obs_dim, n_actions, lr=3e-4, gamma=0.99,
                 gae_lambda=0.95, clip_epsilon=0.2, n_epochs=10,
                 batch_size=64, value_coef=0.5, entropy_coef=0.01):
        
        self.gamma = gamma
        self.gae_lambda = gae_lambda       # GAE lambda pour l'advantage
        self.clip_epsilon = clip_epsilon   # ε du clip
        self.n_epochs = n_epochs           # Réutilisation des données
        self.batch_size = batch_size
        self.value_coef = value_coef
        self.entropy_coef = entropy_coef
        
        self.network = PolicyWithBaseline(obs_dim, n_actions)
        self.optimizer = optim.Adam(self.network.parameters(), lr=lr)
    
    def compute_gae(self, rewards, values, dones, next_value):
        """
        Generalized Advantage Estimation (GAE, Schulman 2016).
        Compromis biais/variance contrôlé par lambda.
        
        GAE(λ=0) = r + γV(s') - V(s)  → TD(0), basse variance, biaisé
        GAE(λ=1) = Σ γᵏrₖ - V(s)     → Monte Carlo, non biaisé, haute variance
        """
        advantages = []
        gae = 0
        
        for step in reversed(range(len(rewards))):
            if step == len(rewards) - 1:
                next_val = next_value
            else:
                next_val = values[step + 1]
            
            # Erreur TD à un pas
            delta = rewards[step] + self.gamma * next_val * (1 - dones[step]) - values[step]
            
            # Accumulation GAE (pondération exponentielle)
            gae = delta + self.gamma * self.gae_lambda * (1 - dones[step]) * gae
            advantages.insert(0, gae)
        
        advantages = torch.tensor(advantages, dtype=torch.float32)
        returns = advantages + torch.tensor(values, dtype=torch.float32)
        
        return advantages, returns
    
    def update(self, states, actions, old_log_probs, advantages, returns):
        """
        Mise à jour PPO avec K epochs sur les données collectées.
        C'est là que PPO diffère fondamentalement de A2C.
        """
        states_t = torch.FloatTensor(np.array(states))
        actions_t = torch.LongTensor(actions)
        old_log_probs_t = torch.FloatTensor(old_log_probs)
        advantages_t = torch.FloatTensor(advantages)
        returns_t = torch.FloatTensor(returns)
        
        # Normalisation des advantages (stabilise l'entraînement)
        advantages_t = (advantages_t - advantages_t.mean()) / (advantages_t.std() + 1e-8)
        
        metrics = {'policy_loss': [], 'value_loss': [], 'entropy': [], 'clip_fraction': []}
        
        # === K epochs de mise à jour sur les MÊMES données ===
        # C'est l'innovation clé de PPO : réutiliser les données efficacement
        for epoch in range(self.n_epochs):
            
            # Mini-batch shuffling
            indices = np.random.permutation(len(states))
            
            for start in range(0, len(states), self.batch_size):
                batch_idx = indices[start:start + self.batch_size]
                
                # Forward pass
                dist, values = self.network(states_t[batch_idx])
                new_log_probs = dist.log_prob(actions_t[batch_idx])
                
                # Ratio d'importance sampling
                ratio = torch.exp(new_log_probs - old_log_probs_t[batch_idx])
                
                # Objectif Clip PPO
                advantages_batch = advantages_t[batch_idx]
                
                obj_unclipped = ratio * advantages_batch
                obj_clipped = torch.clamp(ratio, 1 - self.clip_epsilon,
                                         1 + self.clip_epsilon) * advantages_batch
                
                # Prendre le MINIMUM (pessimiste)
                policy_loss = -torch.min(obj_unclipped, obj_clipped).mean()
                
                # Loss valeur (critique)
                value_loss = nn.MSELoss()(values.squeeze(), returns_t[batch_idx])
                
                # Bonus entropie
                entropy_loss = -dist.entropy().mean()
                
                # Loss totale
                loss = (policy_loss 
                       + self.value_coef * value_loss 
                       + self.entropy_coef * entropy_loss)
                
                self.optimizer.zero_grad()
                loss.backward()
                torch.nn.utils.clip_grad_norm_(self.network.parameters(), 0.5)
                self.optimizer.step()
                
                # Métriques
                clip_fraction = (torch.abs(ratio - 1) > self.clip_epsilon).float().mean()
                metrics['policy_loss'].append(policy_loss.item())
                metrics['value_loss'].append(value_loss.item())
                metrics['entropy'].append(-entropy_loss.item())
                metrics['clip_fraction'].append(clip_fraction.item())
        
        return {k: np.mean(v) for k, v in metrics.items()}
```

> [!tip] Hyperparamètres PPO recommandés
> Les valeurs par défaut de Stable-Baselines3 sont robustes pour débuter :
> - `clip_epsilon = 0.2` : contrainte de changement de politique
> - `n_epochs = 10` : réutilisation des données collectées
> - `gae_lambda = 0.95` : équilibre biais/variance de GAE
> - `batch_size = 64` : mini-batches pour la stabilité SGD
> - `learning_rate = 3e-4` : taux d'apprentissage standard Adam

---

## 8. SAC — Soft Actor-Critic

### 8.1 Maximum Entropy Reinforcement Learning

**SAC** (Haarnoja et al., 2018) introduit le paradigme **Maximum Entropy RL** : au lieu de maximiser uniquement la récompense cumulative, on maximise la récompense **plus l'entropie de la politique** :

$$J(\pi) = \mathbb{E}_{\tau \sim \pi}\left[\sum_t r_t + \alpha \mathcal{H}(\pi(\cdot|s_t))\right]$$

où $\mathcal{H}(\pi(\cdot|s_t)) = -\sum_a \pi(a|s_t) \log \pi(a|s_t)$ est l'entropie.

**Pourquoi maximiser l'entropie ?**
- Encourage l'exploration naturellement — la politique reste stochastique
- Robustesse : apprend à résoudre la tâche de **plusieurs façons** différentes
- Évite les minima locaux — la politique ne se spécialise pas trop vite

### 8.2 Architecture SAC pour Actions Continues

SAC est particulièrement adapté aux **espaces d'actions continus** (robotique, contrôle) :

```python
import torch.nn.functional as F
from torch.distributions import Normal

class SACPolicyNetwork(nn.Module):
    """
    Politique gaussienne pour actions continues.
    Sortie : distribution gaussienne N(μ(s), σ(s)) avec reparametrization trick.
    """
    LOG_SIG_MAX = 2
    LOG_SIG_MIN = -20
    
    def __init__(self, obs_dim, action_dim, hidden_dim=256):
        super().__init__()
        
        self.network = nn.Sequential(
            nn.Linear(obs_dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, hidden_dim),
            nn.ReLU()
        )
        
        self.mean_layer = nn.Linear(hidden_dim, action_dim)
        self.log_std_layer = nn.Linear(hidden_dim, action_dim)
    
    def forward(self, state):
        features = self.network(state)
        mean = self.mean_layer(features)
        log_std = self.log_std_layer(features)
        log_std = torch.clamp(log_std, self.LOG_SIG_MIN, self.LOG_SIG_MAX)
        return mean, log_std
    
    def sample(self, state):
        """
        Échantillonne une action avec le reparametrization trick.
        a = tanh(μ + σ·ε), ε ~ N(0, I)
        
        tanh squeeze les actions dans [-1, 1] — nécessaire pour les envs continus
        """
        mean, log_std = self.forward(state)
        std = log_std.exp()
        
        # Reparametrization : permet de rétropropager à travers l'échantillonnage
        normal = Normal(mean, std)
        x = normal.rsample()  # x = mean + std * eps, eps ~ N(0,1)
        
        # Squash dans [-1, 1] via tanh
        action = torch.tanh(x)
        
        # Calcul du log-prob (avec correction de Jacobien pour tanh)
        # log π(a|s) = log N(x|μ,σ) - Σ log(1 - tanh²(x))
        log_prob = normal.log_prob(x)
        log_prob -= torch.log(1 - action.pow(2) + 1e-6)
        log_prob = log_prob.sum(dim=-1, keepdim=True)
        
        return action, log_prob


class SACQNetwork(nn.Module):
    """
    Réseau Q pour actions continues.
    Entrée : (état, action) → Sortie : Q(s, a) scalaire
    """
    def __init__(self, obs_dim, action_dim, hidden_dim=256):
        super().__init__()
        self.q = nn.Sequential(
            nn.Linear(obs_dim + action_dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, 1)
        )
    
    def forward(self, state, action):
        x = torch.cat([state, action], dim=-1)
        return self.q(x)


class SACAgent:
    """
    Soft Actor-Critic complet pour espaces d'actions continus.
    Utilise deux Q-networks pour réduire la surestimation (Clipped Double Q).
    """
    def __init__(self, obs_dim, action_dim, lr=3e-4, gamma=0.99,
                 tau=0.005, alpha=0.2, buffer_size=1_000_000, batch_size=256):
        self.gamma = gamma
        self.tau = tau          # Soft update coefficient du target network
        self.alpha = alpha      # Température entropie (peut être auto-tuné)
        self.batch_size = batch_size
        
        # Politique
        self.policy = SACPolicyNetwork(obs_dim, action_dim)
        self.policy_optimizer = optim.Adam(self.policy.parameters(), lr=lr)
        
        # Double Q-Network (réduit la surestimation comme dans TD3)
        self.q1 = SACQNetwork(obs_dim, action_dim)
        self.q2 = SACQNetwork(obs_dim, action_dim)
        self.q1_optimizer = optim.Adam(self.q1.parameters(), lr=lr)
        self.q2_optimizer = optim.Adam(self.q2.parameters(), lr=lr)
        
        # Target networks (soft update)
        self.q1_target = copy.deepcopy(self.q1)
        self.q2_target = copy.deepcopy(self.q2)
        
        self.replay_buffer = ReplayBuffer(buffer_size)
    
    def select_action(self, state, evaluate=False):
        """Sélection d'action — stochastique en train, déterministe en eval."""
        state_t = torch.FloatTensor(state).unsqueeze(0)
        
        with torch.no_grad():
            if evaluate:
                mean, _ = self.policy(state_t)
                action = torch.tanh(mean)
            else:
                action, _ = self.policy.sample(state_t)
        
        return action.squeeze(0).numpy()
    
    def update(self):
        """Mise à jour complète SAC."""
        if len(self.replay_buffer) < self.batch_size:
            return
        
        states, actions, rewards, next_states, dones = \
            self.replay_buffer.sample(self.batch_size)
        
        states_t = torch.FloatTensor(states)
        actions_t = torch.FloatTensor(actions)
        rewards_t = torch.FloatTensor(rewards).unsqueeze(1)
        next_states_t = torch.FloatTensor(next_states)
        dones_t = torch.FloatTensor(dones).unsqueeze(1)
        
        # === Mise à jour des Q-networks ===
        with torch.no_grad():
            # Actions du prochain état selon la politique actuelle
            next_actions, next_log_probs = self.policy.sample(next_states_t)
            
            # Target Q-value (Clipped Double Q pour éviter la surestimation)
            q1_next = self.q1_target(next_states_t, next_actions)
            q2_next = self.q2_target(next_states_t, next_actions)
            q_next = torch.min(q1_next, q2_next)
            
            # Bellman avec terme d'entropie (soft Bellman equation)
            target_q = rewards_t + self.gamma * (1 - dones_t) * (q_next - self.alpha * next_log_probs)
        
        # Loss Q1 et Q2
        q1_loss = nn.MSELoss()(self.q1(states_t, actions_t), target_q)
        q2_loss = nn.MSELoss()(self.q2(states_t, actions_t), target_q)
        
        self.q1_optimizer.zero_grad(); q1_loss.backward(); self.q1_optimizer.step()
        self.q2_optimizer.zero_grad(); q2_loss.backward(); self.q2_optimizer.step()
        
        # === Mise à jour de la politique ===
        new_actions, log_probs = self.policy.sample(states_t)
        
        q1_new = self.q1(states_t, new_actions)
        q2_new = self.q2(states_t, new_actions)
        q_new = torch.min(q1_new, q2_new)
        
        # Maximiser Q(s, a) - α·log π(a|s)
        policy_loss = (self.alpha * log_probs - q_new).mean()
        
        self.policy_optimizer.zero_grad()
        policy_loss.backward()
        self.policy_optimizer.step()
        
        # === Soft update des target networks ===
        # θ_target = τ·θ + (1-τ)·θ_target (plus doux que la copie périodique)
        for param, target_param in zip(self.q1.parameters(), self.q1_target.parameters()):
            target_param.data.copy_(self.tau * param.data + (1 - self.tau) * target_param.data)
        
        for param, target_param in zip(self.q2.parameters(), self.q2_target.parameters()):
            target_param.data.copy_(self.tau * param.data + (1 - self.tau) * target_param.data)
```

**Comparaison des algorithmes deep RL :**

| Algorithme | Type | Espace d'actions | Stabilité | Sample efficiency |
|-----------|------|------------------|-----------|------------------|
| DQN | Value-based, off-policy | Discret | ★★★ | ★★★ |
| DDQN | Value-based, off-policy | Discret | ★★★★ | ★★★ |
| REINFORCE | Policy gradient, on-policy | Discret/Continu | ★★ | ★ |
| A2C | Actor-Critic, on-policy | Discret/Continu | ★★★ | ★★ |
| PPO | Actor-Critic, on-policy | Discret/Continu | ★★★★★ | ★★★ |
| SAC | Actor-Critic, off-policy | Continu | ★★★★ | ★★★★★ |

---

## 9. OpenAI Gymnasium

### 9.1 Installation et Environnements de Base

```bash
# Installation de base
pip install gymnasium

# Avec les environnements classiques (CartPole, LunarLander, etc.)
pip install "gymnasium[classic-control]"

# Avec les jeux Atari
pip install "gymnasium[atari]" "gymnasium[accept-rom-license]"

# Avec Box2D (LunarLander, BipedalWalker)
pip install "gymnasium[box2d]"

# Avec MuJoCo (robotique)
pip install "gymnasium[mujoco]"

# Installation complète
pip install "gymnasium[all]"
```

### 9.2 API Gymnasium

```python
import gymnasium as gym

# Création d'un environnement
env = gym.make("CartPole-v1", render_mode="human")

# Inspection de l'environnement
print(f"Espace d'observations : {env.observation_space}")
print(f"Espace d'actions      : {env.action_space}")
print(f"Type d'obs            : {type(env.observation_space)}")

# Pour CartPole :
# Espace d'observations : Box([-4.8, -inf, -0.419, -inf], [4.8, inf, 0.419, inf], (4,), float32)
# Espace d'actions      : Discrete(2) — 0 = gauche, 1 = droite

# === Boucle d'interaction standard ===
obs, info = env.reset(seed=42)  # Réinitialise l'env, retourne obs initiale

for step in range(1000):
    # Sélectionner une action (ici aléatoire)
    action = env.action_space.sample()
    
    # Exécuter l'action
    obs, reward, terminated, truncated, info = env.step(action)
    
    # terminated : l'épisode s'est terminé naturellement (succès ou échec)
    # truncated  : l'épisode a été coupé (limite de temps)
    
    if terminated or truncated:
        obs, info = env.reset()

env.close()
```

### 9.3 Environnements Classiques

**CartPole-v1** — Équilibre d'un pendule inversé

```
État (4 variables) :
  [position_chariot, vitesse_chariot, angle_perche, vitesse_angulaire]

Actions : 0 (gauche) ou 1 (droite)

Récompense : +1 à chaque step où la perche reste debout
Terminaison : perche > 12° OU chariot hors [-2.4, 2.4]
Succès : survivre 500 steps → récompense totale = 500
```

**LunarLander-v3** — Alunissage

```python
env = gym.make("LunarLander-v2")  # Discret : 4 actions (rien, gauche, bas, droite)
# env = gym.make("LunarLanderContinuous-v2")  # Continu : 2 actions continues

# État (8 variables) :
# [x, y, vx, vy, angle, vitesse_angulaire, contact_gauche, contact_droit]

# Récompenses :
# +100 à +140 : atterrissage réussi dans la zone
# -100        : crash
# -0.3/step   : utilisation des propulseurs latéraux
```

**MountainCar-v0** — Voiture sur une montagne (sparse rewards)

```
Problème : la récompense est -1 à CHAQUE step (sauf succès)
→ Très difficile par simple exploration aléatoire
→ Excellent exemple pour tester curriculum learning et HER (Hindsight Experience Replay)
```

```python
# Exploration des espaces d'actions
import gymnasium as gym
import numpy as np

envs_to_explore = [
    "CartPole-v1",
    "LunarLander-v2",
    "MountainCar-v0",
    "Pendulum-v1",          # Continu
    "BipedalWalker-v3",     # Continu, difficile
]

for env_name in envs_to_explore:
    env = gym.make(env_name)
    obs_space = env.observation_space
    act_space = env.action_space
    
    print(f"\n{'='*50}")
    print(f"Environnement : {env_name}")
    print(f"Observation   : {obs_space}")
    
    if hasattr(obs_space, 'shape'):
        print(f"  Shape       : {obs_space.shape}")
    
    print(f"Action        : {act_space}")
    
    if hasattr(act_space, 'n'):
        print(f"  N actions   : {act_space.n}")
    elif hasattr(act_space, 'shape'):
        print(f"  Shape       : {act_space.shape}")
        print(f"  Low/High    : {act_space.low} / {act_space.high}")
    
    env.close()
```

### 9.4 Enregistrement de Vidéos

```python
from gymnasium.wrappers import RecordVideo

# Enregistre des vidéos automatiquement
env = gym.make("CartPole-v1", render_mode="rgb_array")
env = RecordVideo(env, video_folder="./videos",
                  episode_trigger=lambda ep: ep % 100 == 0)  # Toutes les 100 épisodes

obs, _ = env.reset()
for _ in range(10_000):
    action = env.action_space.sample()
    obs, reward, terminated, truncated, _ = env.step(action)
    if terminated or truncated:
        obs, _ = env.reset()

env.close()
```

---

## 10. Wrappers : Vectorisation et Normalisation

### 10.1 Envs Vectorisés

La vectorisation parallélise N environnements pour collecter des expériences plus rapidement :

```python
import gymnasium as gym
from gymnasium.vector import SyncVectorEnv, AsyncVectorEnv

def make_env(env_name, seed=0):
    """Factory function pour créer un environnement seedé."""
    def _init():
        env = gym.make(env_name)
        env.reset(seed=seed)
        return env
    return _init

# Création de 4 environnements en parallèle (synchrone)
n_envs = 4
vec_env = SyncVectorEnv([make_env("CartPole-v1", seed=i) for i in range(n_envs)])

# L'API est identique mais les shapes ajoutent une dimension de batch
obs, info = vec_env.reset()
print(f"Shape des observations : {obs.shape}")  # (4, 4) — 4 envs × 4 obs

# Les actions doivent être un tableau de taille n_envs
actions = vec_env.action_space.sample()  # (4,) — une action par env
obs, rewards, terminateds, truncateds, infos = vec_env.step(actions)

print(f"Shape rewards : {rewards.shape}")  # (4,)

vec_env.close()

# Asynchrone — meilleure performance si les envs sont lents (simulation physique)
async_env = AsyncVectorEnv([make_env("LunarLander-v2", seed=i) for i in range(8)])
```

### 10.2 Wrappers Standards

```python
from gymnasium.wrappers import (
    NormalizeObservation,
    NormalizeReward,
    FrameStack,
    GrayscaleObservation,
    ResizeObservation,
    ClipAction,
    RescaleAction,
    TimeLimit
)

# Pipeline de wrappers pour Atari
def make_atari_env(env_name, n_stack=4):
    env = gym.make(env_name, render_mode=None)
    env = GrayscaleObservation(env)          # RGB → Niveaux de gris
    env = ResizeObservation(env, shape=(84, 84))  # Redimensionner
    env = FrameStack(env, n_stack)           # Empiler 4 frames consécutives
    return env

# Pour les envs continus : normaliser les observations
def make_continuous_env(env_name):
    env = gym.make(env_name)
    env = NormalizeObservation(env)      # Moyenne 0, variance 1
    env = NormalizeReward(env)           # Normalise les récompenses
    env = ClipAction(env)                # Clamp les actions dans l'espace valide
    return env

# Limite la durée des épisodes
env = TimeLimit(gym.make("MountainCar-v0"), max_episode_steps=200)

# Wrappers personnalisés
class RewardScalerWrapper(gym.RewardWrapper):
    """Multiplie toutes les récompenses par un facteur."""
    def __init__(self, env, scale=0.01):
        super().__init__(env)
        self.scale = scale
    
    def reward(self, reward):
        return reward * self.scale


class ObservationClipWrapper(gym.ObservationWrapper):
    """Clip les observations dans [-clip_value, clip_value]."""
    def __init__(self, env, clip_value=5.0):
        super().__init__(env)
        self.clip_value = clip_value
    
    def observation(self, obs):
        return np.clip(obs, -self.clip_value, self.clip_value)
```

---

## 11. Stable-Baselines3

### 11.1 Installation et Utilisation de Base

```bash
pip install stable-baselines3
pip install stable-baselines3[extra]  # Avec tensorboard, optuna, etc.
```

```python
import gymnasium as gym
from stable_baselines3 import DQN, PPO, SAC, A2C, TD3
from stable_baselines3.common.env_util import make_vec_env
from stable_baselines3.common.evaluation import evaluate_policy

# === DQN pour CartPole (discret) ===
env = gym.make("CartPole-v1")

model = DQN(
    policy="MlpPolicy",     # Réseau fully-connected (Multi-Layer Perceptron)
    env=env,
    verbose=1,              # Affiche les logs d'entraînement
    learning_rate=1e-3,
    buffer_size=50_000,
    learning_starts=1000,   # Commence à apprendre après 1000 steps
    batch_size=32,
    gamma=0.99,
    train_freq=4,           # Entraîne le réseau toutes les 4 transitions
    gradient_steps=1,
    target_update_interval=1000,
    exploration_fraction=0.1,
    exploration_final_eps=0.05,
    tensorboard_log="./logs/dqn_cartpole/"
)

# Entraînement
model.learn(total_timesteps=50_000, log_interval=100)

# Sauvegarde
model.save("dqn_cartpole")

# Chargement
loaded_model = DQN.load("dqn_cartpole", env=env)

# Évaluation
mean_reward, std_reward = evaluate_policy(
    loaded_model, env, n_eval_episodes=10, deterministic=True
)
print(f"Récompense moyenne : {mean_reward:.2f} ± {std_reward:.2f}")

env.close()
```

```python
# === PPO pour LunarLander ===
vec_env = make_vec_env("LunarLander-v2", n_envs=4)  # 4 envs en parallèle

model = PPO(
    policy="MlpPolicy",
    env=vec_env,
    verbose=1,
    learning_rate=3e-4,
    n_steps=1024,          # Transitions collectées avant chaque mise à jour
    batch_size=64,
    n_epochs=10,
    gamma=0.99,
    gae_lambda=0.95,
    clip_range=0.2,
    ent_coef=0.01,
    vf_coef=0.5,
    max_grad_norm=0.5,
    tensorboard_log="./logs/ppo_lunarlander/"
)

model.learn(total_timesteps=500_000)
model.save("ppo_lunarlander")
vec_env.close()


# === SAC pour Pendulum (continu) ===
env = gym.make("Pendulum-v1")

model = SAC(
    policy="MlpPolicy",
    env=env,
    verbose=1,
    learning_rate=3e-4,
    buffer_size=1_000_000,
    learning_starts=100,
    batch_size=256,
    tau=0.005,
    gamma=0.99,
    train_freq=1,
    gradient_steps=1,
    ent_coef="auto",       # Auto-tuning du coefficient d'entropie
    target_entropy="auto",
    tensorboard_log="./logs/sac_pendulum/"
)

model.learn(total_timesteps=100_000)
env.close()
```

### 11.2 Politiques Disponibles

```python
# MlpPolicy — réseau fully-connected (observations vectorielles)
model = PPO("MlpPolicy", env)

# CnnPolicy — réseau convolutionnel (observations images)
model = DQN("CnnPolicy", gym.make("BreakoutNoFrameskip-v4"))

# MultiInputPolicy — observations dict (plusieurs sources)
# Utile quand l'état est une combinaison d'image + vecteur
model = PPO("MultiInputPolicy", env_with_dict_obs)

# Architecture personnalisée
from stable_baselines3.common.torch_layers import BaseFeaturesExtractor

class CustomCNN(BaseFeaturesExtractor):
    """Extracteur de features CNN personnalisé."""
    def __init__(self, observation_space, features_dim=256):
        super().__init__(observation_space, features_dim)
        n_input_channels = observation_space.shape[0]
        
        self.cnn = nn.Sequential(
            nn.Conv2d(n_input_channels, 32, kernel_size=8, stride=4),
            nn.ReLU(),
            nn.Conv2d(32, 64, kernel_size=4, stride=2),
            nn.ReLU(),
            nn.Flatten()
        )
        
        # Calculer la dimension de sortie
        with torch.no_grad():
            n_flatten = self.cnn(torch.as_tensor(observation_space.sample()[None]).float()).shape[1]
        
        self.linear = nn.Sequential(
            nn.Linear(n_flatten, features_dim),
            nn.ReLU()
        )
    
    def forward(self, observations):
        return self.linear(self.cnn(observations))

policy_kwargs = dict(
    features_extractor_class=CustomCNN,
    features_extractor_kwargs=dict(features_dim=256),
    net_arch=[256, 128]  # Couches après le features extractor
)

model = PPO("CnnPolicy", env, policy_kwargs=policy_kwargs)
```

### 11.3 Callbacks

Les callbacks permettent d'injecter du code personnalisé pendant l'entraînement :

```python
from stable_baselines3.common.callbacks import (
    BaseCallback,
    EvalCallback,
    StopTrainingOnRewardThreshold,
    CheckpointCallback,
    CallbackList
)

# === Callback de sauvegarde automatique ===
checkpoint_callback = CheckpointCallback(
    save_freq=10_000,           # Sauvegarder tous les 10k steps
    save_path="./checkpoints/",
    name_prefix="ppo_lunar"
)

# === Callback d'évaluation périodique ===
eval_env = gym.make("LunarLander-v2")
eval_callback = EvalCallback(
    eval_env,
    best_model_save_path="./best_model/",
    log_path="./eval_logs/",
    eval_freq=5_000,           # Évaluer toutes les 5k steps
    n_eval_episodes=10,
    deterministic=True,
    render=False
)

# === Stop quand la performance atteint un seuil ===
stop_callback = StopTrainingOnRewardThreshold(
    reward_threshold=200,       # Arrêter si récompense moyenne > 200
    verbose=1
)

# Combiner plusieurs callbacks
callback_list = CallbackList([checkpoint_callback, eval_callback])

# === Callback personnalisé ===
class WandBCallback(BaseCallback):
    """Envoie les métriques vers Weights & Biases."""
    
    def __init__(self, verbose=0):
        super().__init__(verbose)
        self.training_metrics = []
    
    def _on_step(self) -> bool:
        """Appelé à chaque step. Retourner False pour arrêter l'entraînement."""
        
        # Accès aux infos de l'épisode courant
        if self.locals.get("dones", [False])[0]:
            # Episode terminé — log les métriques
            info = self.locals.get("infos", [{}])[0]
            if "episode" in info:
                ep_reward = info["episode"]["r"]
                ep_length = info["episode"]["l"]
                print(f"Step {self.num_timesteps} | Ep Reward: {ep_reward:.1f}")
        
        return True  # True = continuer l'entraînement
    
    def _on_rollout_end(self):
        """Appelé à la fin de chaque rollout (pour PPO/A2C)."""
        pass
    
    def _on_training_end(self):
        """Appelé à la fin de l'entraînement."""
        print("Entraînement terminé !")


# Entraînement avec callbacks
model = PPO("MlpPolicy", vec_env, verbose=1, tensorboard_log="./logs/")
model.learn(
    total_timesteps=1_000_000,
    callback=CallbackList([
        checkpoint_callback,
        eval_callback,
        WandBCallback()
    ])
)
```

---

## 12. TensorBoard pour le Monitoring

### 12.1 Lancer TensorBoard

```bash
# Lancer TensorBoard (dans un terminal séparé)
tensorboard --logdir ./logs/

# Ouvrir dans le navigateur : http://localhost:6006
```

### 12.2 Métriques Clés en RL

```python
# SB3 log automatiquement dans TensorBoard :
# - rollout/ep_rew_mean  : récompense moyenne par épisode
# - rollout/ep_len_mean  : durée moyenne des épisodes
# - train/value_loss     : loss du critique
# - train/policy_loss    : loss de l'acteur (PPO)
# - train/entropy_loss   : entropie de la politique
# - train/clip_fraction  : fraction des ratios PPO clippés
# - train/approx_kl      : KL divergence approximée

# Ajouter des métriques personnalisées dans un callback
from torch.utils.tensorboard import SummaryWriter

class CustomTensorBoardCallback(BaseCallback):
    """Ajoute des métriques personnalisées à TensorBoard."""
    
    def __init__(self, verbose=0):
        super().__init__(verbose)
        self.episode_rewards = []
        self.episode_lengths = []
    
    def _on_step(self):
        # Récupérer les infos d'épisode
        for info in self.locals.get("infos", []):
            if "episode" in info:
                self.episode_rewards.append(info["episode"]["r"])
                self.episode_lengths.append(info["episode"]["l"])
                
                # Log vers TensorBoard via le logger SB3
                self.logger.record("custom/episode_reward", info["episode"]["r"])
                self.logger.record("custom/episode_length", info["episode"]["l"])
                
                # Moving average sur les 100 derniers épisodes
                if len(self.episode_rewards) >= 100:
                    avg_100 = np.mean(self.episode_rewards[-100:])
                    self.logger.record("custom/avg_reward_100", avg_100)
        
        return True
```

### 12.3 Visualisation de la Politique Apprise

```python
import matplotlib.pyplot as plt

def visualize_q_values(model, env_name="CartPole-v1", n_points=50):
    """
    Visualise les Q-values d'un modèle DQN sur une grille d'états.
    """
    # Créer une grille d'états (2 premières dimensions pour la visualisation)
    x_range = np.linspace(-2.4, 2.4, n_points)   # Position
    theta_range = np.linspace(-0.2, 0.2, n_points)  # Angle
    
    q_left = np.zeros((n_points, n_points))
    q_right = np.zeros((n_points, n_points))
    
    for i, x in enumerate(x_range):
        for j, theta in enumerate(theta_range):
            state = np.array([[x, 0, theta, 0]], dtype=np.float32)
            state_tensor = torch.FloatTensor(state)
            
            with torch.no_grad():
                q_vals = model.policy.q_net(state_tensor).numpy()[0]
            
            q_left[j, i] = q_vals[0]
            q_right[j, i] = q_vals[1]
    
    fig, axes = plt.subplots(1, 3, figsize=(15, 5))
    
    axes[0].imshow(q_left, origin='lower', extent=[-2.4, 2.4, -0.2, 0.2],
                   cmap='RdYlGn', aspect='auto')
    axes[0].set_title('Q(s, action=gauche)')
    axes[0].set_xlabel('Position'); axes[0].set_ylabel('Angle')
    plt.colorbar(axes[0].images[0], ax=axes[0])
    
    axes[1].imshow(q_right, origin='lower', extent=[-2.4, 2.4, -0.2, 0.2],
                   cmap='RdYlGn', aspect='auto')
    axes[1].set_title('Q(s, action=droite)')
    plt.colorbar(axes[1].images[0], ax=axes[1])
    
    # Politique greedy (argmax)
    policy_map = (q_right > q_left).astype(int)
    axes[2].imshow(policy_map, origin='lower', extent=[-2.4, 2.4, -0.2, 0.2],
                   cmap='bwr', aspect='auto', vmin=0, vmax=1)
    axes[2].set_title('Politique greedy (0=gauche, 1=droite)')
    
    plt.tight_layout()
    plt.savefig("q_values_visualization.png", dpi=150)
    plt.show()
```

---

## 13. Évaluation des Agents

### 13.1 Courbes d'Apprentissage

```python
import matplotlib.pyplot as plt
import pandas as pd
from pathlib import Path

def plot_learning_curves(log_dirs, labels, smooth_window=50):
    """
    Trace les courbes d'apprentissage depuis les logs TensorBoard.
    Utilise tensorboard.backend.event_processing pour parser les events.
    """
    from tensorboard.backend.event_processing.event_accumulator import EventAccumulator
    
    fig, axes = plt.subplots(2, 2, figsize=(15, 10))
    
    metrics = [
        ("rollout/ep_rew_mean", "Récompense moyenne", axes[0, 0]),
        ("rollout/ep_len_mean", "Durée des épisodes", axes[0, 1]),
        ("train/value_loss", "Loss Valeur", axes[1, 0]),
        ("train/entropy_loss", "Entropie", axes[1, 1]),
    ]
    
    for log_dir, label in zip(log_dirs, labels):
        ea = EventAccumulator(log_dir)
        ea.Reload()
        
        for tag, title, ax in metrics:
            if tag in ea.Tags()['scalars']:
                events = ea.Scalars(tag)
                steps = [e.step for e in events]
                values = [e.value for e in events]
                
                # Lissage par moyenne mobile
                if smooth_window > 1:
                    values_smooth = pd.Series(values).rolling(smooth_window).mean()
                else:
                    values_smooth = values
                
                ax.plot(steps, values_smooth, label=label)
                ax.fill_between(steps, 
                               pd.Series(values).rolling(smooth_window).min(),
                               pd.Series(values).rolling(smooth_window).max(),
                               alpha=0.1)
    
    for tag, title, ax in metrics:
        ax.set_title(title)
        ax.set_xlabel("Steps d'entraînement")
        ax.legend()
        ax.grid(True, alpha=0.3)
    
    plt.tight_layout()
    plt.savefig("learning_curves.png", dpi=150)
    plt.show()


def benchmark_agents(env_name, algorithms, n_train_steps=200_000, n_eval=50):
    """
    Compare plusieurs algorithmes sur le même environnement.
    Retourne un DataFrame avec les résultats.
    """
    results = {}
    
    for algo_name, AlgoClass, kwargs in algorithms:
        print(f"\n=== Entraînement {algo_name} ===")
        
        # Environnements multiples pour PPO/A2C
        if algo_name in ["PPO", "A2C"]:
            env = make_vec_env(env_name, n_envs=4)
        else:
            env = gym.make(env_name)
        
        model = AlgoClass("MlpPolicy", env, verbose=0, **kwargs)
        
        # Enregistrement des performances pendant l'entraînement
        eval_env = gym.make(env_name)
        rewards_during_training = []
        
        class PeriodicEvalCallback(BaseCallback):
            def __init__(self, eval_env, eval_freq=5000):
                super().__init__()
                self.eval_env = eval_env
                self.eval_freq = eval_freq
            
            def _on_step(self):
                if self.num_timesteps % self.eval_freq == 0:
                    mean_r, _ = evaluate_policy(self.model, self.eval_env,
                                               n_eval_episodes=10, deterministic=True)
                    rewards_during_training.append((self.num_timesteps, mean_r))
                return True
        
        model.learn(total_timesteps=n_train_steps,
                   callback=PeriodicEvalCallback(eval_env))
        
        # Évaluation finale
        final_mean, final_std = evaluate_policy(model, eval_env,
                                               n_eval_episodes=n_eval)
        
        results[algo_name] = {
            'training_curve': rewards_during_training,
            'final_mean': final_mean,
            'final_std': final_std
        }
        
        env.close()
        eval_env.close()
        print(f"  Final: {final_mean:.1f} ± {final_std:.1f}")
    
    return results


# Exemple de benchmark
algorithms_to_compare = [
    ("DQN", DQN, {"learning_rate": 1e-3}),
    ("PPO", PPO, {"learning_rate": 3e-4, "n_steps": 1024}),
    ("A2C", A2C, {"learning_rate": 7e-4}),
]

results = benchmark_agents("CartPole-v1", algorithms_to_compare)
```

### 13.2 Métriques d'Évaluation

```python
def compute_agent_metrics(model, env_name, n_episodes=100):
    """
    Calcule un ensemble complet de métriques d'évaluation.
    """
    env = gym.make(env_name)
    
    all_rewards = []
    all_lengths = []
    all_actions = []
    success_count = 0
    
    for episode in range(n_episodes):
        obs, _ = env.reset()
        episode_reward = 0
        episode_length = 0
        done = False
        
        while not done:
            action, _ = model.predict(obs, deterministic=True)
            obs, reward, terminated, truncated, info = env.step(action)
            done = terminated or truncated
            
            episode_reward += reward
            episode_length += 1
            all_actions.append(action)
        
        all_rewards.append(episode_reward)
        all_lengths.append(episode_length)
        
        # Critère de succès spécifique à l'env
        if env_name == "LunarLander-v2" and episode_reward > 200:
            success_count += 1
    
    env.close()
    
    rewards_array = np.array(all_rewards)
    
    metrics = {
        "mean_reward": np.mean(rewards_array),
        "std_reward": np.std(rewards_array),
        "median_reward": np.median(rewards_array),
        "min_reward": np.min(rewards_array),
        "max_reward": np.max(rewards_array),
        "p25_reward": np.percentile(rewards_array, 25),
        "p75_reward": np.percentile(rewards_array, 75),
        "mean_length": np.mean(all_lengths),
        "success_rate": success_count / n_episodes,
        "action_entropy": -np.sum(  # Entropie des actions choisies
            [p * np.log(p + 1e-8) for p in 
             np.unique(all_actions, return_counts=True)[1] / len(all_actions)]
        )
    }
    
    print("=== Métriques d'évaluation ===")
    for k, v in metrics.items():
        print(f"  {k:20s}: {v:.4f}")
    
    return metrics
```

---

## 14. Multi-Agent Reinforcement Learning (MARL)

### 14.1 Introduction au MARL

Dans le **MARL**, plusieurs agents interagissent simultanément dans le même environnement. La dynamique devient fondamentalement différente car les transitions dépendent des actions de **tous** les agents.

```
Environnement multi-agents :

  Agent₁ ──a₁──→ +──────────────+ ──r₁, s₁──→ Agent₁
  Agent₂ ──a₂──→ | ENVIRONNEMENT| ──r₂, s₂──→ Agent₂
  Agent₃ ──a₃──→ +──────────────+ ──r₃, s₃──→ Agent₃
  
MDP multi-agents : (S, A₁×A₂×...×Aₙ, P, R₁,...,Rₙ, γ)
```

**Taxonomie des problèmes MARL :**

| Paradigme | Récompenses | Exemples |
|-----------|-------------|---------|
| Coopératif | Partagées ou alignées | Navigation de robots, réseau électrique |
| Compétitif | Opposées (zero-sum) | Jeux de plateau, Poker |
| Mixte | Hybride | Soccer, négociation, traffic |

### 14.2 Défis Spécifiques au MARL

> [!warning] Non-stationnarité — Le défi fondamental du MARL
> Quand l'agent A apprend, la politique de A change. Du point de vue de l'agent B, l'environnement dans lequel il apprend **change continuellement**. La propriété de Markov est violée : la "cible" d'apprentissage bouge pour chaque agent. Les algorithmes de RL standard divergent en MARL naïf.

**Autres défis :**
- **Credit assignment** en coopération : comment attribuer la récompense partagée à l'action de chaque agent ?
- **Scalabilité** : l'espace des actions joint grandit exponentiellement avec le nombre d'agents
- **Communication** : les agents peuvent-ils se transmettre de l'information ?

### 14.3 Approches Classiques

**Independent Q-Learning (IQL)** — baselines naïf

```python
# Chaque agent ignore l'existence des autres
# Traite les autres agents comme faisant partie de l'environnement
# Fonctionne parfois en pratique malgré la non-stationnarité

agents = {
    agent_id: DQNAgent(obs_dim, n_actions)
    for agent_id in env.agents
}

# Entraînement indépendant
for episode in range(n_episodes):
    obs = env.reset()
    done = {agent: False for agent in env.agents}
    
    while not all(done.values()):
        actions = {
            agent: agents[agent].select_action(obs[agent])
            for agent in env.agents if not done[agent]
        }
        
        next_obs, rewards, dones, infos = env.step(actions)
        
        # Mise à jour indépendante de chaque agent
        for agent in env.agents:
            if not done[agent]:
                agents[agent].replay_buffer.push(
                    obs[agent], actions[agent], rewards[agent],
                    next_obs[agent], dones[agent]
                )
                agents[agent].train_step()
        
        obs = next_obs
        done = dones
```

**CTDE : Centralized Training, Decentralized Execution**

```
Paradigme moderne en MARL coopératif :

  Phase d'entraînement (centralisée) :
    Critic global qui voit TOUTES les observations et actions
    → peut estimer précisément la valeur des états joints
  
  Phase d'exécution (décentralisée) :
    Chaque agent n'a accès qu'à son observation locale
    → applicable en conditions réelles

  Avantage : résout le credit assignment pendant l'entraînement
             sans sacrifier l'autonomie à l'exécution
```

```python
# MADDPG — Multi-Agent DDPG (architecture CTDE)
# Chaque agent a sa propre politique, mais les critiques sont centralisés

class MADDPGCritic(nn.Module):
    """
    Critique centralisé qui observe TOUT l'état joint.
    Utilisé uniquement pendant l'entraînement.
    """
    def __init__(self, total_obs_dim, total_action_dim, hidden_dim=256):
        super().__init__()
        # Entrée : concaténation de TOUTES les observations et actions
        self.network = nn.Sequential(
            nn.Linear(total_obs_dim + total_action_dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, hidden_dim),
            nn.ReLU(),
            nn.Linear(hidden_dim, 1)
        )
    
    def forward(self, all_obs, all_actions):
        """
        all_obs     : (batch, n_agents × obs_dim)
        all_actions : (batch, n_agents × action_dim)
        """
        x = torch.cat([all_obs, all_actions], dim=-1)
        return self.network(x)
```

### 14.4 PettingZoo — Bibliothèque MARL

```bash
pip install pettingzoo
pip install "pettingzoo[atari]"   # Jeux Atari multi-agents
pip install "pettingzoo[classic]"  # Jeux de cartes, échecs...
```

```python
from pettingzoo.classic import chess_v6
from pettingzoo.butterfly import cooperative_pong_v5

# Environnement AEC (Agent Environment Cycle)
# Les agents agissent tour à tour
env = chess_v6.env()
env.reset()

for agent in env.agent_iter():
    observation, reward, termination, truncation, info = env.last()
    
    if termination or truncation:
        action = None
    else:
        # Ici : utiliser la politique de l'agent correspondant
        action = env.action_space(agent).sample()
    
    env.step(action)

env.close()

# Environnement Parallel (tous les agents agissent simultanément)
env = cooperative_pong_v5.parallel_env()
observations, infos = env.reset()

while env.agents:
    actions = {agent: env.action_space(agent).sample()
               for agent in env.agents}
    observations, rewards, terminations, truncations, infos = env.step(actions)

env.close()
```

---

## 15. Curriculum Learning et Transfer Learning en RL

### 15.1 Curriculum Learning

**Problème** : certains environnements sont trop difficiles pour être appris directement (récompenses trop rares, état initial trop aléatoire). Un humain apprendrait à marcher avant de courir — le Curriculum Learning applique ce principe.

**Automatic Domain Randomization (ADR)** — approach OpenAI :

```python
class CurriculumWrapper(gym.Wrapper):
    """
    Wrapper implémentant un curriculum progressif.
    Commence avec une tâche facile, augmente la difficulté
    selon les performances de l'agent.
    """
    def __init__(self, env, initial_difficulty=0.1, 
                 success_threshold=0.8, n_eval_episodes=50):
        super().__init__(env)
        self.difficulty = initial_difficulty  # 0.0 = facile, 1.0 = difficile
        self.success_threshold = success_threshold
        self.n_eval_episodes = n_eval_episodes
        self.episode_rewards = []
    
    def reset(self, **kwargs):
        """Configure la difficulté en fonction du curriculum."""
        # Exemple : BipedalWalker — difficulty contrôle le terrain
        options = {"hardcore": self.difficulty > 0.7}
        return self.env.reset(options=options, **kwargs)
    
    def step(self, action):
        obs, reward, terminated, truncated, info = self.env.step(action)
        
        if terminated or truncated:
            self.episode_rewards.append(info.get("episode_reward", 0))
        
        return obs, reward, terminated, truncated, info
    
    def update_curriculum(self):
        """
        Augmente la difficulté si l'agent réussit suffisamment souvent.
        Appelé périodiquement dans le callback d'entraînement.
        """
        if len(self.episode_rewards) >= self.n_eval_episodes:
            # Estimer le taux de succès récent
            recent = self.episode_rewards[-self.n_eval_episodes:]
            success_rate = np.mean([r > 0 for r in recent])
            
            if success_rate > self.success_threshold:
                # L'agent maîtrise le niveau actuel → augmenter
                self.difficulty = min(1.0, self.difficulty + 0.1)
                self.episode_rewards = []
                print(f"Curriculum : difficulté → {self.difficulty:.1f} "
                      f"(taux succès : {success_rate:.1%})")
            elif success_rate < 0.3:
                # L'agent régresse → réduire légèrement
                self.difficulty = max(0.1, self.difficulty - 0.05)


class CurriculumCallback(BaseCallback):
    """Met à jour le curriculum à intervalles réguliers."""
    
    def __init__(self, curriculum_env, update_freq=5000, verbose=0):
        super().__init__(verbose)
        self.curriculum_env = curriculum_env
        self.update_freq = update_freq
    
    def _on_step(self):
        if self.num_timesteps % self.update_freq == 0:
            self.curriculum_env.update_curriculum()
        return True
```

**Hindsight Experience Replay (HER)** — pour les récompenses clairsemées

```python
# HER est implémenté dans Stable-Baselines3
from stable_baselines3 import HerReplayBuffer

# FetchReach : atteindre une position cible (récompense = 0 sauf si atteint)
env = gym.make("FetchReach-v2")  # Retourne un dict d'observation

model = SAC(
    policy="MultiInputPolicy",
    env=env,
    replay_buffer_class=HerReplayBuffer,
    replay_buffer_kwargs=dict(
        n_sampled_goal=4,           # 4 buts de remplacement par transition
        goal_selection_strategy="future",  # Stratégie "future" (finale, episode, random)
    ),
    verbose=1
)

# HER reformule les transitions échouées comme des succès
# avec le goal finalement atteint → génère du signal là où il n'y en avait pas
model.learn(total_timesteps=100_000)
```

### 15.2 Transfer Learning en RL

**Sim-to-Real Transfer** — entraîner en simulation, déployer sur vrai robot :

```python
class DomainRandomizationWrapper(gym.Wrapper):
    """
    Randomise les paramètres physiques à chaque épisode.
    Rend l'agent robuste aux variations → meilleur transfert sim→réel.
    """
    def __init__(self, env, randomize_gravity=True, randomize_friction=True):
        super().__init__(env)
        self.randomize_gravity = randomize_gravity
        self.randomize_friction = randomize_friction
    
    def reset(self, **kwargs):
        # Randomiser la gravité ±20%
        if self.randomize_gravity:
            gravity = np.random.uniform(8.0, 12.0)
            self.env.unwrapped.gravity = gravity
        
        # Randomiser le frottement ±30%
        if self.randomize_friction:
            friction = np.random.uniform(0.7, 1.3)
            # Appliquer au modèle physique...
        
        return self.env.reset(**kwargs)


def fine_tune_pretrained(source_env, target_env, pretrained_model_path,
                         fine_tune_steps=50_000):
    """
    Transfer learning : fine-tune un modèle pré-entraîné sur un nouvel env.
    
    Cas d'usage : entraîner sur CartPole (facile) → fine-tune sur MountainCar
    """
    # Charger le modèle pré-entraîné
    model = PPO.load(pretrained_model_path)
    
    # Créer un nouvel env cible
    target = gym.make(target_env)
    
    # Vérifier la compatibilité des espaces
    assert model.observation_space.shape == target.observation_space.shape, \
        "Les espaces d'observation doivent être compatibles !"
    
    # Remplacer l'environnement (les poids sont conservés)
    model.set_env(make_vec_env(target_env, n_envs=4))
    
    # Fine-tuning avec un taux d'apprentissage réduit
    model.learning_rate = 1e-4  # Plus petit que l'entraînement initial
    
    model.learn(total_timesteps=fine_tune_steps, reset_num_timesteps=False)
    
    target.close()
    return model


def progressive_neural_networks_concept():
    """
    Progressive Neural Networks (PNN, Rusu 2016) :
    
    Source task       Target task
    Column 1          Column 2
    
    Layer 1 (frozen) ──→ Layer 1 (new)
    Layer 2 (frozen) ──→ Layer 2 (new)  ← lateral connections
    Layer 3 (frozen) ──→ Layer 3 (new)
    
    Avantage : zéro catastrophic forgetting
    Inconvénient : architecture grandit avec le nombre de tâches
    """
    pass
```

> [!info] Curriculum Learning vs Transfer Learning
> - **Curriculum Learning** : même agent, tâches de plus en plus difficiles (un seul problème final)
> - **Transfer Learning** : agent pré-entraîné sur tâche A, adapté à tâche B différente
> - **Meta-Learning (MAML)** : apprendre à apprendre — l'agent s'initialise bien sur de nouvelles tâches en quelques gradient steps

---

## Exercices Pratiques

### Exercice 1 — Débutant : Q-Learning sur FrozenLake

```python
"""
Challenge : Implémenter un agent Q-Learning sur FrozenLake-v1 (carte 4x4).
Objectif : atteindre un taux de succès > 70% en 10 000 épisodes.

Consignes :
1. Implémenter la table Q et la mise à jour de Bellman
2. Implémenter epsilon-greedy avec décroissance
3. Tracer la courbe du taux de succès tous les 500 épisodes
4. Expérimenter avec different γ (0.9, 0.95, 0.99) — quel impact ?
"""
import gymnasium as gym
import numpy as np
import matplotlib.pyplot as plt

def votre_solution():
    env = gym.make("FrozenLake-v1", is_slippery=True)
    
    # TODO : implémenter Q-Learning ici
    raise NotImplementedError("À vous de jouer !")

votre_solution()
```

### Exercice 2 — Intermédiaire : DQN Custom sur CartPole

```python
"""
Challenge : Implémenter DQN depuis zéro sur CartPole-v1.
Objectif : obtenir une récompense moyenne > 450 sur 100 épisodes consécutifs.

Composants à implémenter :
1. ReplayBuffer avec capacité 10 000
2. DQNNetwork avec 2 couches cachées de 64 neurons
3. Target Network mis à jour toutes les 200 steps
4. Loss Huber (SmoothL1Loss)
5. Gradient clipping à 10

Bonus : ajouter Double DQN et mesurer l'amélioration.
"""
```

### Exercice 3 — Avancé : PPO sur LunarLander

```python
"""
Challenge : Entraîner PPO (depuis Stable-Baselines3) sur LunarLander-v2.
Objectif : récompense moyenne > 200 en moins de 1M steps.

Tâches :
1. Créer un vecteur de 4 environnements
2. Configurer PPO avec les bons hyperparamètres
3. Ajouter EvalCallback + CheckpointCallback
4. Logger vers TensorBoard
5. Tracer la courbe d'apprentissage finale
6. Comparer avec SAC (même budget computationnel)

Question bonus : pourquoi PPO apprend-il plus vite que SAC sur LunarLander ?
(Hint : regarder la dimension de l'espace d'actions)
"""
```

### Exercice 4 — Expert : Multi-Agent Coopératif

```python
"""
Challenge : entraîner 2 agents coopératifs sur un gridworld personnalisé.
Les deux agents doivent atteindre leur cible sans collision.

1. Implémenter un GridWorldEnv PettingZoo avec :
   - 2 agents, grille 8x8
   - Récompense partagée si les DEUX atteignent leur cible
   - Pénalité -1 en cas de collision entre agents
   - Terminaison si un agent quitte la grille

2. Entraîner avec IQL (Q-Learning indépendant)
3. Analyser le comportement émergent :
   - Les agents apprennent-ils à s'éviter ?
   - Convergent-ils vers une division du territoire ?
   - Que se passe-t-il si on augmente à 4 agents ?
"""
```

---

## Récapitulatif — Choisir le Bon Algorithme

```
                    Espace d'actions
                         |
          +--------------+--------------+
          |                             |
       DISCRET                      CONTINU
          |                             |
    +-----+-----+               +------+------+
    |           |               |             |
  Simple      Complexe       Off-policy   On-policy
    |           |               |             |
  DQN        Rainbow          SAC/TD3      PPO/A2C
 DDQN        DQN+PER
             Dueling
  
  
  Critères supplémentaires :
  
  Sample efficiency prioritaire  → SAC (off-policy, réutilise les données)
  Stabilité et robustesse        → PPO (contrainte clip, très stable)
  Environnement discret simple   → DQN ou Q-Learning tabulaire
  Multi-agent coopératif         → MADDPG, QMIX
  Récompenses clairsemées        → PPO + HER ou Curriculum
  Déploiement temps-réel         → PPO (politique déterministe à l'inférence)
```

> [!tip] Recommandations pratiques
> 1. **Toujours commencer par PPO** (Stable-Baselines3) avec les hyperparamètres par défaut — c'est le meilleur point de départ dans 80% des cas
> 2. **Passer à SAC** si l'espace d'actions est continu et la sample efficiency est critique
> 3. **DQN** uniquement pour les espaces discrets avec des architectures spécialisées (CNN Atari)
> 4. **Ne jamais oublier** la normalisation des observations — c'est souvent plus important que le choix d'algorithme
> 5. **TensorBoard dès le premier run** — impossible de débugger le RL sans visualisation des courbes

> [!warning] Pièges classiques des débutants en RL
> - Hyperparamètres trop agressifs dès le départ → divergence
> - Oublier de normaliser les observations → gradients explosifs
> - Comparer des algorithmes avec des budgets de steps différents — inéquitable
> - Conclure d'un seul run avec une seule seed — le RL est stochastique, utiliser 5+ seeds
> - Ne pas monitorer l'entropie de la politique — si elle s'effondre trop vite, l'agent est piégé dans un minimum local
