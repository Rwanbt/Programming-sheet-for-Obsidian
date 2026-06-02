# RNN, LSTM et Séries Temporelles

Les réseaux de neurones classiques traitent chaque entrée de façon indépendante, sans tenir compte de l'ordre ou du contexte temporel — une limitation critique pour la parole, le texte ou les données financières. Ce cours couvre les architectures récurrentes (RNN, LSTM, GRU) qui mémorisent les dépendances dans le temps, puis les outils classiques et modernes de prévision de séries temporelles (ARIMA, Prophet, LSTM forecasting). À la fin, tu seras capable d'implémenter un modèle de bout en bout sur des données réelles.

---

## 1. Motivation : pourquoi les réseaux feed-forward échouent sur les séquences

### 1.1 Le problème fondamental

Un réseau de neurones feed-forward (MLP, CNN) reçoit un vecteur d'entrée de taille fixe et produit une sortie. Il n'a aucun mécanisme interne pour « se souvenir » de ce qui s'est passé avant.

Considère la phrase :

> « Le chat que le chien a mordu a fui. »

Pour prédire le pronom correct dans « Il a fui », un modèle doit se souvenir que le sujet principal est « Le chat », introduit 7 tokens plus tôt. Un MLP ne peut pas faire ça : il voit une fenêtre fixe, sans état persistant.

```
Problèmes concrets avec un MLP pour les séquences :

1. Taille d'entrée fixe → impossible de traiter des séquences variables
2. Pas de mémoire → chaque timestep est traité indépendamment
3. Pas de partage de paramètres → apprendre "chat" en position 1 ≠ apprendre "chat" en position 7
```

> [!warning] Erreur classique
> Certains étudiants tentent de "flattten" une séquence de longueur T en vecteur T×F et de l'envoyer dans un MLP. Ça fonctionne pour des séquences **fixées** et **courtes**, mais ça explose en paramètres, ne généralise pas à d'autres longueurs, et ignore l'ordre structurel.

### 1.2 Ce que nous voulons

Un modèle de séquence idéal doit :

| Propriété | Description |
|-----------|-------------|
| **Entrée variable** | Traiter des séquences de longueur quelconque |
| **Mémoire** | Propager de l'information d'un pas de temps au suivant |
| **Partage de paramètres** | Les mêmes poids s'appliquent à chaque position |
| **Dépendances longues** | Se souvenir d'un token apparu 50 pas plus tôt |

---

## 2. RNN Vanilla : architecture et BPTT

### 2.1 Architecture d'un RNN

Un RNN introduit un **état caché** (hidden state) `h_t` qui joue le rôle de mémoire. À chaque pas de temps `t`, il combine l'entrée courante `x_t` et l'état précédent `h_{t-1}` pour produire le nouvel état `h_t`.

```
         x_1    x_2    x_3    ...   x_T
          ↓      ↓      ↓            ↓
h_0 → [RNN] → [RNN] → [RNN] → ... → [RNN] → h_T
          ↓      ↓      ↓            ↓
         y_1    y_2    y_3           y_T
```

**Équations du RNN vanilla :**

```python
# Notation mathématique :
# h_t = tanh(W_h * h_{t-1} + W_x * x_t + b_h)
# y_t = W_y * h_t + b_y

# En code NumPy (pédagogique) :
import numpy as np

def rnn_step(x_t, h_prev, W_h, W_x, b_h):
    """
    Un pas de temps d'un RNN vanilla.
    
    Args:
        x_t   : vecteur d'entrée au temps t, shape (input_size,)
        h_prev: état caché précédent, shape (hidden_size,)
        W_h   : matrice de poids récurrente, shape (hidden_size, hidden_size)
        W_x   : matrice de poids d'entrée, shape (hidden_size, input_size)
        b_h   : biais, shape (hidden_size,)
    
    Returns:
        h_t   : nouvel état caché, shape (hidden_size,)
    """
    h_t = np.tanh(W_h @ h_prev + W_x @ x_t + b_h)
    return h_t

def rnn_forward(X, h0, W_h, W_x, W_y, b_h, b_y):
    """
    Passe avant complète sur une séquence.
    
    Args:
        X  : séquence d'entrée, shape (T, input_size)
        h0 : état initial, shape (hidden_size,)
    
    Returns:
        outputs : liste de sorties y_t
        states  : liste de tous les h_t
    """
    h = h0
    states = [h0]
    outputs = []
    
    for t in range(len(X)):
        h = rnn_step(X[t], h, W_h, W_x, b_h)
        y = W_y @ h + b_y
        states.append(h)
        outputs.append(y)
    
    return outputs, states
```

### 2.2 Le RNN "déroulé" (unrolled)

Pour visualiser la rétropropagation, on "déroulé" le RNN dans le temps : on remplace la boucle récurrente par une succession de couches partageant les mêmes poids.

```
Poids partagés : W_h, W_x, b_h (identiques à chaque step)

  x_1    x_2    x_3    x_4
   |      |      |      |
   ↓      ↓      ↓      ↓
h_0 → [f] → [f] → [f] → [f] → h_4
       |      |      |      |
      y_1    y_2    y_3    y_4

f(h_{t-1}, x_t) = tanh(W_h·h_{t-1} + W_x·x_t + b_h)
```

### 2.3 BPTT — Backpropagation Through Time

La BPTT est la rétropropagation appliquée au RNN déroulé. Le gradient de la perte par rapport aux poids à l'instant `t` dépend de **tous les instants futurs** via la chaîne de dérivées.

```python
# Illustration du problème du gradient (simplifié)
# Si h_t = tanh(W_h * h_{t-1} + ...)
# alors dL/dW_h implique :
#
# dL/dW_h = Σ_{t=1}^{T} dL/dh_T * (dh_T/dh_t) * (dh_t/dW_h)
#
# où dh_T/dh_t = Π_{k=t+1}^{T} (dh_k/dh_{k-1})
#              = Π_{k=t+1}^{T} diag(1 - tanh²(·)) * W_h
#
# Si ||W_h|| < 1 → produit → 0 (vanishing gradient)
# Si ||W_h|| > 1 → produit → ∞ (exploding gradient)
```

> [!warning] Problème du vanishing gradient
> Lorsque la séquence est longue (T > 20-30), le gradient qui remonte vers les premiers timesteps devient exponentiellement petit. Le réseau ne peut donc **plus apprendre des dépendances longue distance**. C'est le problème fondamental qui a motivé l'invention du LSTM.

### 2.4 Solutions partielles pour le RNN vanilla

| Technique | Description | Limite |
|-----------|-------------|--------|
| **Gradient clipping** | Borner la norme du gradient : `g = g * (max_norm / ||g||)` | Ne résout pas le vanishing |
| **Truncated BPTT** | Ne rétropropager que sur K steps | Perd les dépendances > K |
| **Initialisation orthogonale** | `W_h` initialisée comme matrice orthogonale | Aide mais insuffisant |
| **LSTM/GRU** | Architecture dédiée avec portes | Solution principale |

```python
# Gradient clipping en Keras
import tensorflow as tf
from tensorflow import keras

optimizer = keras.optimizers.Adam(learning_rate=0.001, clipnorm=1.0)
# ou clipvalue=0.5 pour clipper composante par composante
```

---

## 3. LSTM — Long Short-Term Memory

### 3.1 Intuition des portes

Le LSTM (Hochreiter & Schmidhuber, 1997) résout le vanishing gradient en introduisant une **cellule d'état** (cell state) `C_t` qui traverse le réseau avec des modifications minimales, comme un convoyeur. Des **portes** (gates) — des sigmoides entre 0 et 1 — contrôlent combien d'information passe ou est effacée.

```
Analogie : le cell state est un tapis roulant.
Les portes sont des vannes qui décident quoi jeter, quoi ajouter, quoi lire.

 C_{t-1} ──────────────────────────────────→ C_t
             ×(forget)    +(input*gate)
              ↑                ↑
           forget           input
            gate             gate
              ↑                ↑
          [sigmoid]        [sigmoid]×[tanh]
              ↑                ↑
         h_{t-1}, x_t    h_{t-1}, x_t
```

### 3.2 Les 4 équations du LSTM

```python
# Notation :
# σ   = sigmoid
# ⊙   = produit élément par élément (Hadamard)
# x_t = entrée au temps t, shape (input_size,)
# h_{t-1} = hidden state précédent, shape (hidden_size,)
# C_{t-1} = cell state précédent, shape (hidden_size,)

import numpy as np

def sigmoid(x):
    return 1 / (1 + np.exp(-x))

def lstm_step(x_t, h_prev, C_prev, params):
    """
    Un pas de temps LSTM complet.
    
    params = {
        'W_f': matrice forget gate (hidden_size, hidden_size + input_size)
        'b_f': biais forget gate
        'W_i': matrice input gate
        'b_i': biais input gate
        'W_c': matrice cell gate (candidate)
        'b_c': biais cell gate
        'W_o': matrice output gate
        'b_o': biais output gate
    }
    """
    # Concaténation de h_{t-1} et x_t
    combined = np.concatenate([h_prev, x_t])  # shape: (hidden + input,)
    
    # 1. FORGET GATE : qu'est-ce qu'on oublie du cell state précédent ?
    #    f_t = σ(W_f · [h_{t-1}, x_t] + b_f)
    f_t = sigmoid(params['W_f'] @ combined + params['b_f'])
    
    # 2. INPUT GATE : qu'est-ce qu'on ajoute au cell state ?
    #    i_t = σ(W_i · [h_{t-1}, x_t] + b_i)      ← combien on ajoute
    #    C̃_t = tanh(W_c · [h_{t-1}, x_t] + b_c)   ← ce qu'on veut ajouter
    i_t = sigmoid(params['W_i'] @ combined + params['b_i'])
    C_candidate = np.tanh(params['W_c'] @ combined + params['b_c'])
    
    # 3. MISE À JOUR DU CELL STATE
    #    C_t = f_t ⊙ C_{t-1} + i_t ⊙ C̃_t
    C_t = f_t * C_prev + i_t * C_candidate
    
    # 4. OUTPUT GATE : qu'est-ce qu'on expose dans le hidden state ?
    #    o_t = σ(W_o · [h_{t-1}, x_t] + b_o)
    #    h_t = o_t ⊙ tanh(C_t)
    o_t = sigmoid(params['W_o'] @ combined + params['b_o'])
    h_t = o_t * np.tanh(C_t)
    
    return h_t, C_t
```

### 3.3 Schéma détaillé du LSTM

```
                     C_{t-1} ──────────×──────────+──────── C_t
                                       │          │
                                      f_t        i_t × C̃_t
                                       │          │
    ┌──────────────────────────────────┤          │
    │                                  │          │
    │  h_{t-1} ──────────────────────┐ │          │
    │  x_t     ──────────────────────┤ │          │
    │                                │ │          │
    │                            [σ]─┘ │          │       h_t ──→
    │                          forget  │          │        │
    │                                  │          │       [×]
    │  h_{t-1}, x_t → [σ]─────────────┤   [tanh](C_t)
    │                 input            │          │
    │                                  │       [σ]─── o_t
    │  h_{t-1}, x_t → [tanh] ─────────┘          │
    │                 candidate                    │
    │                                              │
    │  h_{t-1}, x_t → [σ] ─────────────── o_t ───┘
    │                 output
    └───────────────────────────────────────────────
```

### 3.4 Pourquoi le LSTM résout le vanishing gradient

> [!info] Insight clé
> Le gradient remonte à travers le cell state via des **additions** (non des multiplications par des matrices denses). Si la forget gate reste proche de 1, l'information circule librement sur des centaines de timesteps sans s'atténuer. C'est le principe du "gradient highway".

```
Comparaison du chemin du gradient :

RNN vanilla :
dC_t/dC_{t-k} = Π_{j=t-k+1}^{t} W_h * diag(1 - tanh²(·))
             → produit de matrices → explosion ou disparition

LSTM cell state :
dC_t/dC_{t-k} = Π_{j=t-k+1}^{t} f_j
             → produit de forget gates (valeurs entre 0 et 1, souvent ≈ 1)
             → gradient préservé si f ≈ 1
```

### 3.5 Implémentation Keras

```python
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers
import numpy as np

# ─── Modèle LSTM simple ───────────────────────────────────────────────
def build_lstm_classifier(vocab_size, embed_dim, lstm_units, num_classes):
    """
    Classificateur de texte avec LSTM.
    
    Architecture :
      Embedding → LSTM → Dense → Softmax
    """
    model = keras.Sequential([
        # Embedding : transforme les indices de tokens en vecteurs denses
        layers.Embedding(
            input_dim=vocab_size,
            output_dim=embed_dim,
            mask_zero=True  # ignore le padding
        ),
        
        # LSTM : return_sequences=False → on ne garde que le dernier h_T
        layers.LSTM(
            units=lstm_units,
            dropout=0.2,          # dropout sur les entrées
            recurrent_dropout=0.2  # dropout sur les connexions récurrentes
        ),
        
        layers.Dense(64, activation='relu'),
        layers.Dropout(0.3),
        layers.Dense(num_classes, activation='softmax')
    ])
    
    return model

# Exemple d'utilisation
model = build_lstm_classifier(
    vocab_size=10000,
    embed_dim=128,
    lstm_units=256,
    num_classes=2  # classification binaire (ex: sentiment)
)

model.compile(
    optimizer='adam',
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy']
)

model.summary()
```

---

## 4. GRU — Gated Recurrent Unit

### 4.1 Simplification du LSTM

Le GRU (Cho et al., 2014) simplifie le LSTM en fusionnant le cell state et le hidden state, et en réduisant à 2 portes au lieu de 3. Il est plus rapide à entraîner et souvent aussi performant que le LSTM sur des données de taille moyenne.

```python
def gru_step(x_t, h_prev, params):
    """
    Un pas de temps GRU.
    
    Équations :
    z_t = σ(W_z · [h_{t-1}, x_t] + b_z)     # update gate
    r_t = σ(W_r · [h_{t-1}, x_t] + b_r)     # reset gate
    h̃_t = tanh(W_h · [r_t ⊙ h_{t-1}, x_t])  # candidate hidden state
    h_t = (1 - z_t) ⊙ h_{t-1} + z_t ⊙ h̃_t  # final hidden state
    """
    combined = np.concatenate([h_prev, x_t])
    
    # Update gate : combien de l'ancien h on conserve vs du nouveau
    z_t = sigmoid(params['W_z'] @ combined + params['b_z'])
    
    # Reset gate : combien de h_{t-1} on utilise pour calculer le candidat
    r_t = sigmoid(params['W_r'] @ combined + params['b_r'])
    
    # Candidat : utilise seulement r_t * h_{t-1}, pas tout h_{t-1}
    combined_reset = np.concatenate([r_t * h_prev, x_t])
    h_candidate = np.tanh(params['W_h'] @ combined_reset + params['b_h'])
    
    # Interpolation entre ancien et nouveau
    h_t = (1 - z_t) * h_prev + z_t * h_candidate
    
    return h_t
```

### 4.2 Comparaison LSTM vs GRU

| Critère | LSTM | GRU |
|---------|------|-----|
| **Portes** | 3 (forget, input, output) | 2 (reset, update) |
| **États** | 2 (h_t et C_t) | 1 (h_t) |
| **Paramètres** | 4 × hidden² + 4 × hidden × input | 3 × hidden² + 3 × hidden × input |
| **Vitesse d'entraînement** | Plus lent | ~25% plus rapide |
| **Performances** | Légèrement meilleures sur longues séquences | Similaires sur séquences courtes-moyennes |
| **Mémoire GPU** | Plus élevée | Moins élevée |
| **Recommandation** | Longues dépendances critiques | Cas général, datasets de taille moyenne |

> [!tip] Règle pratique
> Commence toujours par un GRU. Si les performances stagnent et que tes séquences sont longues (> 100 tokens), passe au LSTM. En pratique, la différence est souvent < 1-2%.

```python
# GRU en Keras — syntaxe identique au LSTM
gru_model = keras.Sequential([
    layers.Embedding(vocab_size, embed_dim, mask_zero=True),
    layers.GRU(256, dropout=0.2, recurrent_dropout=0.1),
    layers.Dense(1, activation='sigmoid')
])
```

---

## 5. Bi-directional RNN/LSTM

### 5.1 Principe

Un RNN standard lit la séquence de gauche à droite. Un RNN bidirectionnel (BiRNN) lit la séquence dans **les deux sens** et concatène les états cachés. Cela permet au modèle d'avoir le contexte gauche ET droit à chaque position.

```
Séquence : "Le chat mange la souris"

RNN →  : h1→  h2→  h3→  h4→  h5→
         Le   chat  mange  la  souris

RNN ←  : h1←  h2←  h3←  h4←  h5←
         souris  la  mange  chat  Le

BiRNN  : [h1→, h5←] [h2→, h4←] [h3→, h3←] ...
          (contexte complet à chaque position)
```

> [!warning] Limitation importante
> Les BiRNN nécessitent **toute la séquence à l'avance**. Ils sont donc inutilisables pour la génération en temps réel ou la prévision en ligne (tu ne peux pas connaître le futur). Ils conviennent parfaitement pour la classification, le NER, la traduction quand on dispose de la phrase complète.

### 5.2 Implémentation Keras

```python
# Bi-LSTM pour la classification de texte ou le NER
def build_bilstm_model(vocab_size, embed_dim, lstm_units, num_classes):
    """
    BiLSTM avec deux couches récurrentes empilées.
    """
    model = keras.Sequential([
        layers.Embedding(vocab_size, embed_dim, mask_zero=True),
        
        # Première couche BiLSTM — retourne toute la séquence
        layers.Bidirectional(
            layers.LSTM(lstm_units, return_sequences=True, dropout=0.2)
        ),
        
        # Deuxième couche BiLSTM — retourne seulement le dernier état
        layers.Bidirectional(
            layers.LSTM(lstm_units // 2, dropout=0.2)
        ),
        
        layers.Dense(64, activation='relu'),
        layers.Dropout(0.3),
        layers.Dense(num_classes, activation='softmax')
    ])
    return model

# Note : Bidirectional double la taille de sortie
# LSTM(128) → output shape: (batch, 256) avec Bidirectional
```

---

## 6. Seq2Seq : Encodeur-Décodeur et Attention

### 6.1 Architecture Seq2Seq

Le modèle Seq2Seq (Sutskever et al., 2014) est conçu pour les tâches où l'entrée et la sortie sont des séquences de longueurs **différentes** : traduction, résumé, chatbots.

```
Encodeur                    Décodeur
─────────────────────       ─────────────────────────────────
x_1 → [LSTM]               [LSTM] → y_1
x_2 → [LSTM]               [LSTM] → y_2
x_3 → [LSTM] → context →  [LSTM] → y_3
x_4 → [LSTM]               [LSTM] → <EOS>
         ↓
    context vector
    (h_T de l'encodeur)
```

Le **vecteur de contexte** (context vector) est le seul canal d'information entre encodeur et décodeur. C'est aussi sa principale limitation : résumer une séquence entière en un seul vecteur de taille fixe est un goulot d'étranglement.

### 6.2 Mécanisme d'attention (introduction)

L'attention (Bahdanau et al., 2015) résout ce goulot en permettant au décodeur de **regarder tous les états de l'encodeur**, pas seulement le dernier.

```python
# Intuition de l'attention — calcul des poids d'attention
def attention(encoder_outputs, decoder_state):
    """
    Calcule les poids d'attention.
    
    Args:
        encoder_outputs : tous les états de l'encodeur, shape (T_enc, hidden)
        decoder_state   : état caché du décodeur, shape (hidden,)
    
    Returns:
        context  : vecteur de contexte pondéré, shape (hidden,)
        weights  : poids d'attention, shape (T_enc,)
    """
    # Score : similarité entre chaque état encodeur et l'état décodeur
    # (version "dot product attention")
    scores = np.dot(encoder_outputs, decoder_state)  # shape (T_enc,)
    
    # Softmax → distribution de probabilité sur les positions encodeur
    weights = np.exp(scores) / np.sum(np.exp(scores))  # shape (T_enc,)
    
    # Contexte : moyenne pondérée des états encodeur
    context = np.sum(weights[:, np.newaxis] * encoder_outputs, axis=0)
    
    return context, weights
```

> [!info] Vers les Transformers
> Le mécanisme d'attention introduit ici est la brique fondamentale des Transformers (Vaswani et al., 2017). Dans un Transformer, **toute** la dépendance entre tokens est gérée par l'attention — les couches récurrentes disparaissent entièrement. Ce cours n'entre pas dans les détails des Transformers (sujet B3), mais il est crucial de comprendre d'où ils viennent.

### 6.3 Implémentation Seq2Seq simple avec Keras

```python
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers
import numpy as np

# ─── Seq2Seq pour l'inversion de séquences (exemple pédagogique) ──────

VOCAB_SIZE = 20
MAX_LEN = 10
LATENT_DIM = 64

# ── Encodeur ─────────────────────────────────────────────────────────
encoder_inputs = keras.Input(shape=(None,), name='encoder_input')
encoder_embed = layers.Embedding(VOCAB_SIZE, 32)(encoder_inputs)
encoder_lstm = layers.LSTM(LATENT_DIM, return_state=True, name='encoder_lstm')
encoder_outputs, state_h, state_c = encoder_lstm(encoder_embed)
encoder_states = [state_h, state_c]  # context vector pour le décodeur

# ── Décodeur ─────────────────────────────────────────────────────────
decoder_inputs = keras.Input(shape=(None,), name='decoder_input')
decoder_embed = layers.Embedding(VOCAB_SIZE, 32)(decoder_inputs)

# Le décodeur est initialisé avec les états de l'encodeur
decoder_lstm = layers.LSTM(
    LATENT_DIM,
    return_sequences=True,  # on veut toutes les sorties
    return_state=True,
    name='decoder_lstm'
)
decoder_outputs, _, _ = decoder_lstm(decoder_embed, initial_state=encoder_states)
decoder_dense = layers.Dense(VOCAB_SIZE, activation='softmax', name='output')
decoder_outputs = decoder_dense(decoder_outputs)

# ── Modèle d'entraînement ─────────────────────────────────────────────
seq2seq_model = keras.Model(
    inputs=[encoder_inputs, decoder_inputs],
    outputs=decoder_outputs
)

seq2seq_model.compile(
    optimizer='adam',
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy']
)

print(seq2seq_model.summary())
```

---

## 7. Implémentation Keras : cas pratiques complets

### 7.1 LSTM pour la classification de sentiment (IMDb)

```python
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers
from tensorflow.keras.datasets import imdb
from tensorflow.keras.preprocessing.sequence import pad_sequences
import matplotlib.pyplot as plt

# ─── Chargement des données ───────────────────────────────────────────
VOCAB_SIZE = 10000
MAX_LEN = 256
BATCH_SIZE = 64

(X_train, y_train), (X_test, y_test) = imdb.load_data(num_words=VOCAB_SIZE)

# Padding : séquences de longueur uniforme
X_train = pad_sequences(X_train, maxlen=MAX_LEN, padding='post', truncating='post')
X_test  = pad_sequences(X_test,  maxlen=MAX_LEN, padding='post', truncating='post')

print(f"Train: {X_train.shape}, Test: {X_test.shape}")
print(f"Labels: {y_train[:5]}")  # 0 = négatif, 1 = positif

# ─── Architecture ─────────────────────────────────────────────────────
model = keras.Sequential([
    # Embedding learnable
    layers.Embedding(
        input_dim=VOCAB_SIZE,
        output_dim=128,
        input_length=MAX_LEN,
        mask_zero=True
    ),
    
    # Première couche LSTM — empilée
    layers.LSTM(
        128,
        return_sequences=True,  # passe la séquence à la couche suivante
        dropout=0.2,
        recurrent_dropout=0.2
    ),
    
    # Deuxième couche LSTM — résumé final
    layers.LSTM(
        64,
        dropout=0.2,
        recurrent_dropout=0.2
    ),
    
    # Tête de classification
    layers.Dense(32, activation='relu'),
    layers.Dropout(0.5),
    layers.Dense(1, activation='sigmoid')
])

model.compile(
    optimizer=keras.optimizers.Adam(learning_rate=3e-4),
    loss='binary_crossentropy',
    metrics=['accuracy', keras.metrics.AUC(name='auc')]
)

model.summary()

# ─── Entraînement ─────────────────────────────────────────────────────
callbacks = [
    keras.callbacks.EarlyStopping(
        monitor='val_auc',
        patience=3,
        mode='max',
        restore_best_weights=True,
        verbose=1
    ),
    keras.callbacks.ReduceLROnPlateau(
        monitor='val_loss',
        factor=0.5,
        patience=2,
        verbose=1
    )
]

history = model.fit(
    X_train, y_train,
    epochs=15,
    batch_size=BATCH_SIZE,
    validation_split=0.2,
    callbacks=callbacks,
    verbose=1
)

# ─── Évaluation ───────────────────────────────────────────────────────
test_loss, test_acc, test_auc = model.evaluate(X_test, y_test, verbose=0)
print(f"\nTest Accuracy : {test_acc:.4f}")
print(f"Test AUC      : {test_auc:.4f}")

# ─── Visualisation ────────────────────────────────────────────────────
fig, axes = plt.subplots(1, 2, figsize=(12, 4))

axes[0].plot(history.history['accuracy'], label='Train')
axes[0].plot(history.history['val_accuracy'], label='Validation')
axes[0].set_title('Accuracy')
axes[0].legend()

axes[1].plot(history.history['loss'], label='Train')
axes[1].plot(history.history['val_loss'], label='Validation')
axes[1].set_title('Loss')
axes[1].legend()

plt.tight_layout()
plt.savefig('lstm_sentiment_training.png', dpi=150)
plt.show()
```

### 7.2 LSTM pour la génération de texte

```python
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers
import numpy as np
import random

# ─── Préparation du corpus ────────────────────────────────────────────
# On va générer du texte caractère par caractère

corpus = """
Le destin est un labyrinthe dont nous sommes les architectes.
Chaque choix ouvre une porte et en ferme mille autres.
La sagesse n'est pas de tout prévoir, mais de savoir s'adapter.
L'homme qui n'a jamais fait d'erreur n'a jamais rien tenté de nouveau.
Le voyage est plus important que la destination.
"""

# Vocabulaire au niveau caractère
chars = sorted(set(corpus))
char_to_idx = {ch: i for i, ch in enumerate(chars)}
idx_to_char = {i: ch for i, ch in enumerate(chars)}
VOCAB_SIZE = len(chars)
print(f"Vocabulaire : {VOCAB_SIZE} caractères uniques")

# ─── Construction des séquences d'entraînement ────────────────────────
SEQ_LEN = 40      # longueur des séquences d'entrée
STEP = 3          # pas entre deux séquences (stride)

sequences = []
targets = []

for i in range(0, len(corpus) - SEQ_LEN, STEP):
    seq = corpus[i:i + SEQ_LEN]
    target = corpus[i + SEQ_LEN]
    sequences.append([char_to_idx[c] for c in seq])
    targets.append(char_to_idx[target])

X = np.array(sequences)              # shape: (N, SEQ_LEN)
y = np.array(targets)                # shape: (N,)

# One-hot encoding
X_onehot = keras.utils.to_categorical(X, num_classes=VOCAB_SIZE)  # (N, SEQ_LEN, VOCAB)
y_onehot = keras.utils.to_categorical(y, num_classes=VOCAB_SIZE)  # (N, VOCAB)

print(f"X shape: {X_onehot.shape}")
print(f"y shape: {y_onehot.shape}")

# ─── Architecture ─────────────────────────────────────────────────────
model_gen = keras.Sequential([
    layers.LSTM(256, return_sequences=True, input_shape=(SEQ_LEN, VOCAB_SIZE)),
    layers.Dropout(0.3),
    layers.LSTM(128),
    layers.Dropout(0.3),
    layers.Dense(VOCAB_SIZE, activation='softmax')
])

model_gen.compile(
    optimizer=keras.optimizers.RMSprop(learning_rate=0.001),
    loss='categorical_crossentropy'
)

# ─── Fonction de sampling avec température ────────────────────────────
def sample_with_temperature(preds, temperature=1.0):
    """
    Échantillonne depuis la distribution de probabilités avec contrôle de créativité.
    
    temperature = 1.0 → distribution d'origine
    temperature < 1.0 → plus conservative (favorise les tokens probables)
    temperature > 1.0 → plus créatif (distribution plus uniforme)
    """
    preds = np.asarray(preds).astype('float64')
    preds = np.log(preds + 1e-10) / temperature  # log-probabilités mises à l'échelle
    exp_preds = np.exp(preds)
    preds = exp_preds / np.sum(exp_preds)         # renormalisation
    return np.random.choice(len(preds), p=preds)

# ─── Callback de génération pendant l'entraînement ────────────────────
class TextGenerationCallback(keras.callbacks.Callback):
    """Génère du texte à la fin de chaque époque pour suivre la progression."""
    
    def __init__(self, seed_text, num_chars=200):
        self.seed_text = seed_text
        self.num_chars = num_chars
    
    def on_epoch_end(self, epoch, logs=None):
        if (epoch + 1) % 5 == 0:  # afficher toutes les 5 époques
            generated = self.seed_text
            current_seq = [char_to_idx.get(c, 0) for c in self.seed_text[-SEQ_LEN:]]
            
            for _ in range(self.num_chars):
                x_pred = np.zeros((1, SEQ_LEN, VOCAB_SIZE))
                for t, idx in enumerate(current_seq):
                    x_pred[0, t, idx] = 1.0
                
                preds = self.model.predict(x_pred, verbose=0)[0]
                next_idx = sample_with_temperature(preds, temperature=0.7)
                next_char = idx_to_char[next_idx]
                
                generated += next_char
                current_seq = current_seq[1:] + [next_idx]
            
            print(f"\n── Époque {epoch+1} ──")
            print(generated)
            print()

# ─── Entraînement ─────────────────────────────────────────────────────
seed = corpus[:SEQ_LEN]  # 40 premiers caractères comme graine

model_gen.fit(
    X_onehot, y_onehot,
    epochs=50,
    batch_size=128,
    callbacks=[
        TextGenerationCallback(seed),
        keras.callbacks.ModelCheckpoint(
            'best_text_gen.keras',
            save_best_only=True,
            monitor='loss'
        )
    ]
)
```

---

## 8. Séries Temporelles : concepts fondamentaux

### 8.1 Définition et décomposition

Une **série temporelle** est une suite d'observations indexées dans le temps : `y_1, y_2, ..., y_T`. On la décompose classiquement en trois composantes :

```
y_t = Tendance(t) + Saisonnalité(t) + Résidu(t)   [décomposition additive]
y_t = Tendance(t) × Saisonnalité(t) × Résidu(t)   [décomposition multiplicative]
```

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from statsmodels.tsa.seasonal import seasonal_decompose

# ─── Données de température (exemple synthétique réaliste) ────────────
np.random.seed(42)
dates = pd.date_range(start='2020-01-01', periods=365*3, freq='D')

# Simulation : tendance linéaire + saisonnalité annuelle + bruit
trend = np.linspace(10, 15, len(dates))
seasonality = 8 * np.sin(2 * np.pi * np.arange(len(dates)) / 365)
noise = np.random.normal(0, 1.5, len(dates))

temperature = trend + seasonality + noise
ts = pd.Series(temperature, index=dates, name='temperature')

# ─── Décomposition STL ────────────────────────────────────────────────
decomposition = seasonal_decompose(ts, model='additive', period=365)

fig, axes = plt.subplots(4, 1, figsize=(12, 10), sharex=True)
ts.plot(ax=axes[0], title='Série originale')
decomposition.trend.plot(ax=axes[1], title='Tendance')
decomposition.seasonal.plot(ax=axes[2], title='Saisonnalité')
decomposition.resid.plot(ax=axes[3], title='Résidus')
plt.tight_layout()
plt.savefig('decomposition.png', dpi=150)
plt.show()
```

### 8.2 Stationnarité

> [!info] Définition
> Une série temporelle est **strictement stationnaire** si sa distribution jointe est invariante par translation temporelle. En pratique, on utilise la **stationnarité faible** : moyenne constante, variance constante, et autocovariance qui ne dépend que du lag (pas du temps).

```
Série non stationnaire (exemple) :

  y_t
   |          /
   |         /
   |        /
   |       /      ← tendance croissante → moyenne non constante
   |______/
   0  T/4  T/2  3T/4  T

Série stationnaire :

  y_t
   |  ~ ~ ~ ~  ~ ~ ~
   |  ~  ~   ~   ~  ~  ← oscille autour d'une moyenne constante
   |___________________
   0  T/4  T/2  3T/4  T
```

**Tests de stationnarité :**

```python
from statsmodels.tsa.stattools import adfuller, kpss

def test_stationarity(series, name="Série"):
    """
    Applique le test ADF (Augmented Dickey-Fuller) et KPSS.
    
    ADF  : H0 = non stationnaire → p < 0.05 → rejette H0 → stationnaire
    KPSS : H0 = stationnaire     → p < 0.05 → rejette H0 → non stationnaire
    """
    print(f"\n{'='*50}")
    print(f"Tests de stationnarité : {name}")
    print('='*50)
    
    # Test ADF
    adf_result = adfuller(series.dropna(), autolag='AIC')
    print(f"\nTest ADF :")
    print(f"  Statistique    : {adf_result[0]:.4f}")
    print(f"  p-value        : {adf_result[1]:.4f}")
    print(f"  Valeurs crit.  : {adf_result[4]}")
    print(f"  Conclusion     : {'Stationnaire ✓' if adf_result[1] < 0.05 else 'Non stationnaire ✗'}")
    
    # Test KPSS
    kpss_result = kpss(series.dropna(), regression='c', nlags='auto')
    print(f"\nTest KPSS :")
    print(f"  Statistique    : {kpss_result[0]:.4f}")
    print(f"  p-value        : {kpss_result[1]:.4f}")
    print(f"  Conclusion     : {'Non stationnaire ✗' if kpss_result[1] < 0.05 else 'Stationnaire ✓'}")

test_stationarity(ts, "Température brute")

# ─── Rendre une série stationnaire ────────────────────────────────────
# Méthode 1 : différenciation (retirer la tendance)
ts_diff = ts.diff().dropna()

# Méthode 2 : log-transformation puis différenciation (stabilise la variance)
ts_log = np.log(ts + abs(ts.min()) + 1)
ts_log_diff = ts_log.diff().dropna()

# Méthode 3 : décomposition et garder les résidus
ts_stationary = decomposition.resid.dropna()

test_stationarity(ts_diff, "Température différenciée (ordre 1)")
```

### 8.3 Autocorrélation (ACF) et autocorrélation partielle (PACF)

```python
from statsmodels.graphics.tsaplots import plot_acf, plot_pacf

# ─── ACF et PACF ─────────────────────────────────────────────────────
fig, axes = plt.subplots(2, 1, figsize=(12, 8))

plot_acf(
    ts_diff,
    lags=40,
    ax=axes[0],
    title='Autocorrélation (ACF) — série différenciée',
    alpha=0.05  # intervalles de confiance à 95%
)

plot_pacf(
    ts_diff,
    lags=40,
    ax=axes[1],
    title='Autocorrélation Partielle (PACF) — série différenciée',
    alpha=0.05
)

plt.tight_layout()
plt.savefig('acf_pacf.png', dpi=150)
plt.show()

# ─── Lecture des graphiques ────────────────────────────────────────────
# ACF  : corrélation entre y_t et y_{t-k} (toutes les correlations intermédiaires incluses)
# PACF : corrélation entre y_t et y_{t-k} APRÈS avoir retiré les effets des lags intermédiaires
#
# Identification ARIMA :
# ┌──────────────┬──────────────────┬──────────────────────┐
# │ Modèle       │ ACF              │ PACF                 │
# ├──────────────┼──────────────────┼──────────────────────┤
# │ AR(p)        │ décroissance expo│ cutoff après lag p   │
# │ MA(q)        │ cutoff après lag q│ décroissance expo   │
# │ ARMA(p,q)    │ décroissance expo│ décroissance expo    │
# └──────────────┴──────────────────┴──────────────────────┘
```

---

## 9. ARIMA : modèle classique de prévision

### 9.1 Composantes AR, I, MA

**AR(p) — AutoRégression d'ordre p :**
```
y_t = c + φ_1·y_{t-1} + φ_2·y_{t-2} + ... + φ_p·y_{t-p} + ε_t
```
La valeur actuelle dépend des p valeurs précédentes.

**MA(q) — Moyenne Mobile d'ordre q :**
```
y_t = μ + ε_t + θ_1·ε_{t-1} + θ_2·ε_{t-2} + ... + θ_q·ε_{t-q}
```
La valeur actuelle dépend des q erreurs précédentes.

**ARIMA(p, d, q) :**
- `p` : ordre AR
- `d` : ordre de différenciation (I = Integrated, pour rendre stationnaire)
- `q` : ordre MA

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from statsmodels.tsa.arima.model import ARIMA
from statsmodels.tsa.stattools import adfuller
import warnings
warnings.filterwarnings('ignore')

# ─── Données : ventes mensuelles (exemple réaliste) ───────────────────
np.random.seed(123)
n = 120  # 10 ans de données mensuelles
t = np.arange(n)

# Simulation : tendance + saisonnalité mensuelle + bruit AR(1)
trend_ventes = 100 + 0.5 * t
seasonal_ventes = 20 * np.sin(2 * np.pi * t / 12)
ar_noise = np.zeros(n)
for i in range(1, n):
    ar_noise[i] = 0.7 * ar_noise[i-1] + np.random.normal(0, 5)

ventes = trend_ventes + seasonal_ventes + ar_noise
dates_m = pd.date_range(start='2014-01-01', periods=n, freq='MS')
ts_ventes = pd.Series(ventes, index=dates_m, name='ventes')

# ─── Identification : test de stationnarité ───────────────────────────
adf = adfuller(ts_ventes)
print(f"ADF p-value brut : {adf[1]:.4f}")

# Différenciation d'ordre 1
ts_ventes_diff = ts_ventes.diff().dropna()
adf_diff = adfuller(ts_ventes_diff)
print(f"ADF p-value après diff(1) : {adf_diff[1]:.4f}")
# Si p < 0.05 → stationnaire avec d=1

# ─── Sélection automatique des ordres via AIC ─────────────────────────
from itertools import product

def select_arima_order(ts, max_p=4, max_d=2, max_q=4):
    """
    Grid search sur les ordres ARIMA par minimisation de l'AIC.
    """
    best_aic = np.inf
    best_order = None
    results = []
    
    for p, d, q in product(range(max_p+1), range(max_d+1), range(max_q+1)):
        try:
            model = ARIMA(ts, order=(p, d, q))
            fitted = model.fit()
            results.append({'order': (p,d,q), 'aic': fitted.aic, 'bic': fitted.bic})
            if fitted.aic < best_aic:
                best_aic = fitted.aic
                best_order = (p, d, q)
        except Exception:
            continue
    
    results_df = pd.DataFrame(results).sort_values('aic')
    print("\nTop 5 modèles par AIC :")
    print(results_df.head())
    return best_order

best_order = select_arima_order(ts_ventes, max_p=3, max_d=1, max_q=3)
print(f"\nMeilleur ordre ARIMA : {best_order}")

# ─── Ajustement du modèle ─────────────────────────────────────────────
# Séparation train/test (derniers 12 mois = test)
train = ts_ventes[:-12]
test = ts_ventes[-12:]

model = ARIMA(train, order=best_order)
fitted_model = model.fit()

print(fitted_model.summary())

# ─── Prévision ────────────────────────────────────────────────────────
forecast_result = fitted_model.forecast(steps=12)
forecast_ci = fitted_model.get_forecast(steps=12).conf_int(alpha=0.05)

# Visualisation
fig, ax = plt.subplots(figsize=(14, 6))

train.plot(ax=ax, label='Entraînement', color='steelblue')
test.plot(ax=ax, label='Réel (test)', color='orange')
forecast_result.plot(ax=ax, label='Prévision ARIMA', color='red', linestyle='--')

ax.fill_between(
    forecast_ci.index,
    forecast_ci.iloc[:, 0],
    forecast_ci.iloc[:, 1],
    alpha=0.3,
    color='red',
    label='Intervalle de confiance 95%'
)

ax.set_title(f'Prévision ARIMA{best_order}')
ax.legend()
ax.grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig('arima_forecast.png', dpi=150)
plt.show()

# ─── Métriques ────────────────────────────────────────────────────────
from sklearn.metrics import mean_absolute_error, mean_squared_error

mae = mean_absolute_error(test, forecast_result)
rmse = np.sqrt(mean_squared_error(test, forecast_result))
mape = np.mean(np.abs((test.values - forecast_result.values) / test.values)) * 100

print(f"\nMétriques sur le test set :")
print(f"  MAE  : {mae:.2f}")
print(f"  RMSE : {rmse:.2f}")
print(f"  MAPE : {mape:.2f}%")
```

### 9.2 Diagnostic des résidus

```python
# ─── Analyse des résidus — vérification des hypothèses ───────────────
import scipy.stats as stats

residuals = fitted_model.resid

fig, axes = plt.subplots(2, 2, figsize=(12, 8))

# 1. Distribution des résidus
axes[0, 0].hist(residuals, bins=30, density=True, alpha=0.7, color='steelblue')
x_norm = np.linspace(residuals.min(), residuals.max(), 100)
axes[0, 0].plot(x_norm, stats.norm.pdf(x_norm, residuals.mean(), residuals.std()),
                'r-', label='Distribution normale théorique')
axes[0, 0].set_title('Distribution des résidus')
axes[0, 0].legend()

# 2. Q-Q plot (normalité)
stats.probplot(residuals, dist="norm", plot=axes[0, 1])
axes[0, 1].set_title('Q-Q Plot (normalité des résidus)')

# 3. Résidus dans le temps
residuals.plot(ax=axes[1, 0])
axes[1, 0].axhline(0, color='red', linestyle='--')
axes[1, 0].set_title('Résidus vs temps')

# 4. ACF des résidus (doit être bruit blanc)
plot_acf(residuals, lags=30, ax=axes[1, 1], alpha=0.05)
axes[1, 1].set_title('ACF des résidus (doit être bruit blanc)')

plt.tight_layout()
plt.savefig('residual_diagnostics.png', dpi=150)
plt.show()

# Test de Ljung-Box (bruit blanc ?)
from statsmodels.stats.diagnostic import acorr_ljungbox
lb_test = acorr_ljungbox(residuals, lags=[10, 20], return_df=True)
print("\nTest de Ljung-Box (H0 = résidus = bruit blanc) :")
print(lb_test)
print("→ Si p > 0.05 pour tous les lags : résidus = bruit blanc ✓")
```

> [!warning] SARIMA pour les données saisonnières
> Si ta série a une saisonnalité marquée (ventes par mois, température par saison), utilise **SARIMA(p,d,q)(P,D,Q,s)** où `s` est la période saisonnière. Pour des données mensuelles avec saisonnalité annuelle : `s=12`. Statsmodels propose `SARIMAX` qui couvre ce cas.

---

## 10. Prophet (Facebook/Meta)

### 10.1 Principe de Prophet

Prophet (Taylor & Letham, 2018) est un modèle additif qui décompose la série en :
- **Tendance** (trend) : linéaire ou logistique, avec détection automatique de changements de pente (changepoints)
- **Saisonnalité** (seasonality) : Fourier series pour capter des cycles annuels, hebdomadaires, journaliers
- **Effets de vacances/événements** (holidays) : composante manuelle
- **Terme d'erreur** : bruit iid

```
y(t) = g(t) + s(t) + h(t) + ε_t
```

> [!tip] Quand utiliser Prophet
> Prophet est idéal pour : données de business (ventes, trafic web), prévision à horizon moyen (1 semaine – 1 an), présence de jours fériés et effets calendaires. Il est robuste aux valeurs manquantes et aux outliers. Il n'est **pas** idéal pour : données haute fréquence (sub-horaire), séries financières avec forte dépendance, prévision à très long terme.

### 10.2 Installation et utilisation

```python
# Installation : pip install prophet
# Sur certains systèmes : conda install -c conda-forge prophet

from prophet import Prophet
from prophet.plot import plot_plotly, plot_components_plotly
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from sklearn.metrics import mean_absolute_error, mean_squared_error

# ─── Données : ventes quotidiennes d'un e-commerce ────────────────────
np.random.seed(42)
dates_daily = pd.date_range(start='2021-01-01', periods=365*2, freq='D')

# Simulation réaliste
t = np.arange(len(dates_daily))
trend_daily = 500 + 1.2 * t
seasonal_annual = 150 * np.sin(2 * np.pi * t / 365 - np.pi/2)  # pic en été
seasonal_weekly = 80 * np.sin(2 * np.pi * (t % 7) / 7)         # pic le week-end
black_friday_idx = np.where(
    (dates_daily.month == 11) & (dates_daily.day.isin(range(24, 30)))
)[0]
black_friday_effect = np.zeros(len(dates_daily))
black_friday_effect[black_friday_idx] = 500

noise_daily = np.random.normal(0, 50, len(dates_daily))
ventes_daily = trend_daily + seasonal_annual + seasonal_weekly + black_friday_effect + noise_daily

# Format Prophet : colonnes 'ds' (date) et 'y' (valeur)
df_prophet = pd.DataFrame({
    'ds': dates_daily,
    'y': ventes_daily
})

print(df_prophet.head())
print(f"\nShape : {df_prophet.shape}")

# ─── Split train/test ─────────────────────────────────────────────────
cutoff = '2022-06-30'
df_train = df_prophet[df_prophet['ds'] <= cutoff]
df_test  = df_prophet[df_prophet['ds'] > cutoff]
print(f"Train : {len(df_train)} jours | Test : {len(df_test)} jours")

# ─── Définition des événements spéciaux (Black Friday) ────────────────
holidays = pd.DataFrame({
    'holiday': 'black_friday',
    'ds': pd.to_datetime([
        '2021-11-26', '2021-11-27',
        '2022-11-25', '2022-11-26'
    ]),
    'lower_window': -2,  # effet 2 jours avant
    'upper_window': 1    # effet 1 jour après
})

# ─── Instanciation et configuration du modèle ─────────────────────────
model_prophet = Prophet(
    # Tendance
    growth='linear',              # 'linear' ou 'logistic'
    changepoint_prior_scale=0.05, # flexibilité de la tendance (0.001-0.5)
    n_changepoints=25,            # nombre de points de changement potentiels
    
    # Saisonnalités
    yearly_seasonality=True,      # activer la saisonnalité annuelle
    weekly_seasonality=True,      # activer la saisonnalité hebdomadaire
    daily_seasonality=False,      # désactiver (données journalières, pas horaires)
    seasonality_prior_scale=10,   # flexibilité de la saisonnalité
    seasonality_mode='additive',  # 'additive' ou 'multiplicative'
    
    # Événements
    holidays=holidays,
    
    # Incertitude
    interval_width=0.95,          # intervalles de confiance à 95%
    mcmc_samples=0                # 0 = optimisation MAP (rapide) ; >0 = MCMC (lent mais incertitude mieux calibrée)
)

# Ajouter une saisonnalité mensuelle personnalisée
model_prophet.add_seasonality(
    name='monthly',
    period=30.5,
    fourier_order=5  # plus élevé = plus de flexibilité mais risque de surapprentissage
)

# ─── Entraînement ─────────────────────────────────────────────────────
model_prophet.fit(df_train)

# ─── Prévision ────────────────────────────────────────────────────────
# Créer un dataframe de dates futures
future = model_prophet.make_future_dataframe(
    periods=len(df_test),
    freq='D',
    include_history=True  # inclure les dates d'entraînement
)

forecast = model_prophet.predict(future)

# Colonnes importantes de forecast :
# - ds        : date
# - yhat      : prévision centrale
# - yhat_lower: borne basse (intervalle de confiance)
# - yhat_upper: borne haute
# - trend, trend_lower, trend_upper
# - weekly, yearly, monthly (composantes séparées)
print("\nColonnes du forecast :")
print([c for c in forecast.columns])

# ─── Visualisation ────────────────────────────────────────────────────
fig1 = model_prophet.plot(forecast, figsize=(14, 6))
fig1.suptitle('Prévision Prophet — Ventes quotidiennes', fontsize=14)
plt.tight_layout()
plt.savefig('prophet_forecast.png', dpi=150)

fig2 = model_prophet.plot_components(forecast, figsize=(14, 12))
plt.tight_layout()
plt.savefig('prophet_components.png', dpi=150)
plt.show()

# ─── Évaluation sur le test set ───────────────────────────────────────
forecast_test = forecast[forecast['ds'].isin(df_test['ds'])]

mae = mean_absolute_error(df_test['y'].values, forecast_test['yhat'].values)
rmse = np.sqrt(mean_squared_error(df_test['y'].values, forecast_test['yhat'].values))
mape = np.mean(np.abs((df_test['y'].values - forecast_test['yhat'].values) / df_test['y'].values)) * 100

print(f"\nMétriques Prophet (test set) :")
print(f"  MAE  : {mae:.2f}")
print(f"  RMSE : {rmse:.2f}")
print(f"  MAPE : {mape:.2f}%")

# ─── Cross-validation Prophet ─────────────────────────────────────────
from prophet.diagnostics import cross_validation, performance_metrics

# Validation croisée temporelle :
# initial = taille du premier ensemble d'entraînement
# period  = espacement entre chaque point de coupure
# horizon = horizon de prévision évalué
df_cv = cross_validation(
    model_prophet,
    initial='365 days',
    period='90 days',
    horizon='60 days',
    parallel="processes"  # parallélisation
)

df_perf = performance_metrics(df_cv)
print("\nPerformances cross-validation :")
print(df_perf[['horizon', 'mae', 'rmse', 'mape']].tail(10))
```

### 10.3 Tuning des hyperparamètres Prophet

```python
import itertools
from prophet.diagnostics import cross_validation, performance_metrics

# ─── Grid search des hyperparamètres clés ────────────────────────────
param_grid = {
    'changepoint_prior_scale': [0.001, 0.05, 0.5],
    'seasonality_prior_scale': [0.01, 1.0, 10.0],
    'seasonality_mode': ['additive', 'multiplicative']
}

all_params = [
    dict(zip(param_grid.keys(), v))
    for v in itertools.product(*param_grid.values())
]

rmse_scores = []

for params in all_params:
    m = Prophet(holidays=holidays, **params)
    m.fit(df_train)
    
    df_cv_tmp = cross_validation(
        m,
        initial='300 days',
        period='60 days',
        horizon='30 days',
        parallel="processes"
    )
    df_p = performance_metrics(df_cv_tmp, rolling_window=1)
    rmse_scores.append(df_p['rmse'].values[0])

# Trouver les meilleurs paramètres
best_idx = np.argmin(rmse_scores)
print(f"Meilleurs paramètres : {all_params[best_idx]}")
print(f"RMSE : {rmse_scores[best_idx]:.2f}")
```

---

## 11. LSTM pour le forecasting de séries temporelles

### 11.1 Préparation des données : la fenêtre glissante

Le LSTM pour le forecasting utilise un paradigme **supervisé** : on transforme la série temporelle en paires (entrée, cible) via une fenêtre glissante (sliding window).

```
Série originale : [10, 12, 11, 14, 15, 13, 16, 18, 17, 20]

Avec window_size = 3, horizon = 1 :

  Entrée          → Cible
  [10, 12, 11]   → 14
  [12, 11, 14]   → 15
  [11, 14, 15]   → 13
  [14, 15, 13]   → 16
  [15, 13, 16]   → 18
  [13, 16, 18]   → 17
  [16, 18, 17]   → 20
```

```python
import numpy as np
import pandas as pd
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers
from sklearn.preprocessing import MinMaxScaler
import matplotlib.pyplot as plt

# ─── Données : températures de Melbourne (synthétique réaliste) ───────
np.random.seed(0)
n_points = 1460  # 4 ans de données quotidiennes

t = np.arange(n_points)
temp = (
    15                                                    # baseline
    + 10 * np.sin(2 * np.pi * t / 365 - np.pi/2)        # saisonnalité annuelle
    + 2 * np.sin(2 * np.pi * t / 7)                      # saisonnalité hebdomadaire
    + 0.002 * t                                           # légère tendance
    + np.random.normal(0, 2, n_points)                   # bruit
)

dates_temp = pd.date_range('2020-01-01', periods=n_points, freq='D')
df_temp = pd.DataFrame({'date': dates_temp, 'temp': temp})

# ─── Normalisation ────────────────────────────────────────────────────
# IMPORTANT : normaliser AVANT de créer les fenêtres
# et garder le scaler pour l'inverse-transformation
scaler = MinMaxScaler(feature_range=(0, 1))

values = df_temp['temp'].values.reshape(-1, 1)
values_scaled = scaler.fit_transform(values)  # → [0, 1]

# ─── Création des fenêtres glissantes ────────────────────────────────
def create_sliding_window(data, window_size, horizon=1):
    """
    Transforme une série temporelle en dataset supervisé.
    
    Args:
        data       : array 1D ou 2D de shape (T,) ou (T, features)
        window_size: nombre de pas de temps en entrée
        horizon    : nombre de pas de temps à prédire
    
    Returns:
        X : shape (N, window_size, features)
        y : shape (N, horizon)
    """
    X, y = [], []
    
    for i in range(len(data) - window_size - horizon + 1):
        X.append(data[i : i + window_size])
        y.append(data[i + window_size : i + window_size + horizon])
    
    return np.array(X), np.array(y)

WINDOW_SIZE = 30   # 30 jours en entrée
HORIZON = 7        # prédire les 7 prochains jours

X, y = create_sliding_window(values_scaled, WINDOW_SIZE, HORIZON)
print(f"X shape : {X.shape}")  # (N, 30, 1)
print(f"y shape : {y.shape}")  # (N, 7)

# ─── Split temporel (PAS de mélange aléatoire pour les séries temporelles !)
split_ratio = 0.8
split_idx = int(len(X) * split_ratio)

X_train, X_test = X[:split_idx], X[split_idx:]
y_train, y_test = y[:split_idx], y[split_idx:]

print(f"\nSplit :")
print(f"  Train : {X_train.shape[0]} échantillons")
print(f"  Test  : {X_test.shape[0]} échantillons")
```

> [!warning] Ne jamais mélanger les données temporelles
> Contrairement à la classification d'images, on ne fait **jamais** de `shuffle=True` sur des séries temporelles. Mélanger briserait l'ordre temporel et entraînerait une **fuite de données du futur vers le passé** (data leakage), ce qui donnerait des métriques artificiellement bonnes mais un modèle inutilisable en production.

### 11.2 Architecture LSTM pour le forecasting

```python
# ─── Architecture multi-variante (avec features engineered) ──────────

def build_lstm_forecaster(
    window_size,
    horizon,
    n_features=1,
    lstm_units=(128, 64),
    dense_units=32,
    dropout_rate=0.2
):
    """
    LSTM pour la prévision de séries temporelles.
    
    Supporte des entrées multivariées (plusieurs features par timestep).
    
    Args:
        window_size : longueur de la fenêtre d'entrée
        horizon     : nombre de pas à prédire
        n_features  : nombre de features à chaque timestep
        lstm_units  : tuple des tailles des couches LSTM
        dense_units : taille de la couche dense cachée
        dropout_rate: taux de dropout
    
    Returns:
        keras.Model
    """
    inputs = keras.Input(shape=(window_size, n_features))
    
    x = inputs
    
    # Couches LSTM empilées
    for i, units in enumerate(lstm_units):
        return_seq = (i < len(lstm_units) - 1)  # True sauf pour la dernière couche
        x = layers.LSTM(
            units,
            return_sequences=return_seq,
            dropout=dropout_rate,
            recurrent_dropout=dropout_rate / 2
        )(x)
        x = layers.BatchNormalization()(x)  # stabilise l'entraînement
    
    # Couches denses pour la projection finale
    x = layers.Dense(dense_units, activation='relu')(x)
    x = layers.Dropout(dropout_rate)(x)
    outputs = layers.Dense(horizon)(x)  # pas d'activation pour la régression
    
    model = keras.Model(inputs=inputs, outputs=outputs)
    return model


model_lstm_ts = build_lstm_forecaster(
    window_size=WINDOW_SIZE,
    horizon=HORIZON,
    n_features=1,
    lstm_units=(128, 64),
    dense_units=32
)

model_lstm_ts.compile(
    optimizer=keras.optimizers.Adam(learning_rate=1e-3),
    loss='huber',  # Huber loss = robuste aux outliers, mélange MAE et MSE
    metrics=[
        keras.metrics.MeanAbsoluteError(name='mae'),
        keras.metrics.RootMeanSquaredError(name='rmse')
    ]
)

model_lstm_ts.summary()

# ─── Entraînement ─────────────────────────────────────────────────────
callbacks_ts = [
    keras.callbacks.EarlyStopping(
        monitor='val_loss',
        patience=10,
        restore_best_weights=True
    ),
    keras.callbacks.ReduceLROnPlateau(
        monitor='val_loss',
        factor=0.5,
        patience=5,
        min_lr=1e-6,
        verbose=1
    ),
    keras.callbacks.ModelCheckpoint(
        'best_lstm_forecaster.keras',
        save_best_only=True,
        monitor='val_loss'
    )
]

history_ts = model_lstm_ts.fit(
    X_train, y_train,
    epochs=100,
    batch_size=32,
    validation_split=0.15,
    callbacks=callbacks_ts,
    verbose=1,
    shuffle=False  # IMPORTANT : ne pas mélanger les données temporelles
)

# ─── Prévision et inverse-transformation ──────────────────────────────
y_pred_scaled = model_lstm_ts.predict(X_test)

# Remettre à l'échelle originale
# y_pred_scaled shape: (N, HORIZON) → besoin de (N*HORIZON, 1) pour le scaler
y_pred = scaler.inverse_transform(
    y_pred_scaled.reshape(-1, 1)
).reshape(-1, HORIZON)

y_true = scaler.inverse_transform(
    y_test.reshape(-1, 1)
).reshape(-1, HORIZON)

print(f"\ny_pred shape : {y_pred.shape}")
print(f"y_true shape : {y_true.shape}")

# ─── Visualisation des prévisions ─────────────────────────────────────
# On visualise les prévisions du premier pas uniquement (horizon=1)
fig, axes = plt.subplots(2, 1, figsize=(14, 10))

# Prévision J+1
ax = axes[0]
ax.plot(y_true[:, 0], label='Réel', alpha=0.7, color='steelblue')
ax.plot(y_pred[:, 0], label='Prévision J+1', alpha=0.7, color='red', linestyle='--')
ax.set_title('Prévision LSTM — 1 jour à l\'avance')
ax.legend()
ax.grid(True, alpha=0.3)

# Prévision J+7
ax = axes[1]
ax.plot(y_true[:, -1], label='Réel', alpha=0.7, color='steelblue')
ax.plot(y_pred[:, -1], label='Prévision J+7', alpha=0.7, color='orange', linestyle='--')
ax.set_title('Prévision LSTM — 7 jours à l\'avance')
ax.legend()
ax.grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('lstm_ts_forecast.png', dpi=150)
plt.show()
```

### 11.3 Feature engineering pour les LSTM de séries temporelles

```python
# ─── Enrichissement multivarié ────────────────────────────────────────
def add_time_features(df, date_col='date'):
    """
    Ajoute des features temporelles cycliques encodées.
    L'encodage cyclique (sin/cos) préserve la continuité :
    mois 12 et mois 1 sont proches, pas distants.
    """
    df = df.copy()
    df['day_of_year'] = df[date_col].dt.dayofyear
    df['day_of_week'] = df[date_col].dt.dayofweek
    df['month'] = df[date_col].dt.month
    df['week_of_year'] = df[date_col].dt.isocalendar().week.astype(int)
    
    # Encodage cyclique
    df['doy_sin'] = np.sin(2 * np.pi * df['day_of_year'] / 365)
    df['doy_cos'] = np.cos(2 * np.pi * df['day_of_year'] / 365)
    df['dow_sin'] = np.sin(2 * np.pi * df['day_of_week'] / 7)
    df['dow_cos'] = np.cos(2 * np.pi * df['day_of_week'] / 7)
    df['month_sin'] = np.sin(2 * np.pi * df['month'] / 12)
    df['month_cos'] = np.cos(2 * np.pi * df['month'] / 12)
    
    # Lags (valeurs passées comme features)
    df['temp_lag1'] = df['temp'].shift(1)
    df['temp_lag7'] = df['temp'].shift(7)
    df['temp_lag14'] = df['temp'].shift(14)
    df['temp_lag365'] = df['temp'].shift(365)  # même jour l'an dernier
    
    # Rolling statistics
    df['temp_rolling_mean_7']  = df['temp'].rolling(7).mean()
    df['temp_rolling_std_7']   = df['temp'].rolling(7).std()
    df['temp_rolling_mean_30'] = df['temp'].rolling(30).mean()
    
    return df.dropna()

df_enriched = add_time_features(df_temp)
print(f"Features : {list(df_enriched.columns)}")
print(f"Shape après feature engineering : {df_enriched.shape}")

# ─── Pipeline complet multivarié ──────────────────────────────────────
feature_cols = [
    'temp', 'doy_sin', 'doy_cos', 'dow_sin', 'dow_cos',
    'month_sin', 'month_cos', 'temp_lag1', 'temp_lag7',
    'temp_rolling_mean_7', 'temp_rolling_std_7'
]

# Normaliser toutes les features
scaler_multi = MinMaxScaler()
data_multi = scaler_multi.fit_transform(df_enriched[feature_cols])

# Créer les fenêtres multivariées
X_multi, y_multi = create_sliding_window(data_multi, WINDOW_SIZE, HORIZON)
# X_multi shape: (N, 30, 11)
# y_multi shape: (N, 7, 11) → on ne prédit que la colonne 0 (temp)
y_multi = y_multi[:, :, 0]  # ne garder que la température cible

print(f"\nDataset multivarié :")
print(f"  X : {X_multi.shape}")
print(f"  y : {y_multi.shape}")

# Construire le modèle multivarié
n_features_multi = X_multi.shape[2]
model_multi = build_lstm_forecaster(
    window_size=WINDOW_SIZE,
    horizon=HORIZON,
    n_features=n_features_multi
)
```

---

## 12. Métriques d'évaluation des séries temporelles

### 12.1 Les 5 métriques essentielles

```python
import numpy as np

def compute_all_metrics(y_true, y_pred, series_name=""):
    """
    Calcule toutes les métriques d'évaluation pour une prévision.
    
    Args:
        y_true : valeurs réelles, array 1D
        y_pred : valeurs prédites, array 1D
    
    Returns:
        dict avec toutes les métriques
    """
    y_true = np.array(y_true).flatten()
    y_pred = np.array(y_pred).flatten()
    
    # MAE — Mean Absolute Error
    # Interprétation : erreur moyenne en unités absolues
    # Avantage : robuste aux outliers, facilement interprétable
    mae = np.mean(np.abs(y_true - y_pred))
    
    # MSE — Mean Squared Error
    # Pénalise les grandes erreurs quadratiquement
    mse = np.mean((y_true - y_pred) ** 2)
    
    # RMSE — Root Mean Squared Error
    # Même unité que y_true, pénalise les grandes erreurs
    rmse = np.sqrt(mse)
    
    # MAPE — Mean Absolute Percentage Error
    # Erreur relative en % → comparable entre séries d'amplitudes différentes
    # ATTENTION : explose si y_true ≈ 0 (division par zéro)
    epsilon = 1e-10  # éviter la division par zéro
    mape = np.mean(np.abs((y_true - y_pred) / (y_true + epsilon))) * 100
    
    # SMAPE — Symmetric MAPE
    # Version symétrique du MAPE, entre 0 et 200%
    # Moins sensible aux valeurs proches de zéro
    # 0% = prévision parfaite, 200% = erreur maximale
    smape = np.mean(
        2 * np.abs(y_true - y_pred) / (np.abs(y_true) + np.abs(y_pred) + epsilon)
    ) * 100
    
    # R² — Coefficient de détermination
    # Proportion de variance expliquée par le modèle
    # 1.0 = parfait, 0.0 = modèle nul (prédire la moyenne), < 0 = pire que la moyenne
    ss_res = np.sum((y_true - y_pred) ** 2)
    ss_tot = np.sum((y_true - np.mean(y_true)) ** 2)
    r2 = 1 - (ss_res / (ss_tot + epsilon))
    
    metrics = {
        'MAE': mae,
        'MSE': mse,
        'RMSE': rmse,
        'MAPE (%)': mape,
        'SMAPE (%)': smape,
        'R²': r2
    }
    
    if series_name:
        print(f"\n{'─'*40}")
        print(f"Métriques : {series_name}")
        print('─'*40)
        for name, value in metrics.items():
            print(f"  {name:<15}: {value:.4f}")
    
    return metrics

# ─── Exemple d'utilisation ────────────────────────────────────────────
np.random.seed(42)
y_true_demo = np.sin(np.linspace(0, 4*np.pi, 100)) * 10 + 50
y_pred_good = y_true_demo + np.random.normal(0, 1, 100)
y_pred_bad  = y_true_demo + np.random.normal(0, 5, 100)

metrics_good = compute_all_metrics(y_true_demo, y_pred_good, "Bon modèle")
metrics_bad  = compute_all_metrics(y_true_demo, y_pred_bad, "Mauvais modèle")
```

### 12.2 Tableau comparatif des métriques

| Métrique | Formule | Unité | Sensible aux outliers | Interprétable en % | Problème principal |
|----------|---------|-------|----------------------|-------------------|-------------------|
| **MAE** | `mean(|y - ŷ|)` | Idem que y | Faible | Non | Ne pénalise pas les grandes erreurs |
| **MSE** | `mean((y - ŷ)²)` | y² | Élevée | Non | Unité difficile à interpréter |
| **RMSE** | `√MSE` | Idem que y | Élevée | Non | Pénalise beaucoup les outliers |
| **MAPE** | `mean(|e|/|y|) × 100` | % | Faible | Oui | Explose si y ≈ 0 |
| **SMAPE** | `mean(2|e|/(|y|+|ŷ|)) × 100` | % | Faible | Oui | Asymétrique en pratique |
| **R²** | `1 - SS_res/SS_tot` | Sans unité | Élevée | Relatif | Peut être trompeur |

> [!tip] Quelle métrique choisir ?
> - **Comparer des modèles sur la même série** : MAE ou RMSE (même unité)
> - **Comparer sur des séries d'amplitudes différentes** : MAPE ou SMAPE
> - **Présenter des résultats à un non-technicien** : MAPE (en %)
> - **Détecter les grandes erreurs rares** : RMSE (sensible aux outliers)
> - **Données avec valeurs nulles** : SMAPE (plus stable que MAPE)

### 12.3 Baseline naïve — point de comparaison obligatoire

```python
def naive_forecast(ts, horizon=1, method='last'):
    """
    Prévisions naïves de référence.
    Tout modèle doit au minimum battre ces baselines.
    
    Methods:
        'last'     : dernier point observé
        'seasonal' : même valeur que la même période de l'année précédente
        'drift'    : extrapolation linéaire de la tendance
    """
    ts = np.array(ts)
    n = len(ts)
    
    if method == 'last':
        # Prévision constante = dernière valeur
        return np.full(horizon, ts[-1])
    
    elif method == 'seasonal':
        # Prévision saisonnière = même période période précédente
        # Pour des données journalières avec saisonnalité annuelle (s=365)
        s = 365
        forecasts = []
        for h in range(1, horizon + 1):
            idx = n - s + h - 1
            if idx >= 0:
                forecasts.append(ts[idx])
            else:
                forecasts.append(ts[-1])  # fallback
        return np.array(forecasts)
    
    elif method == 'drift':
        # Extrapolation de la tendance moyenne
        slope = (ts[-1] - ts[0]) / (n - 1)
        return np.array([ts[-1] + slope * h for h in range(1, horizon + 1)])
    
    else:
        raise ValueError(f"Méthode inconnue : {method}")

# ─── Comparaison modèles vs baselines ────────────────────────────────
print("\n" + "="*60)
print("TABLEAU DE BORD DES PERFORMANCES")
print("="*60)

# Simulation comparative
y_true_final = y_true[:200, 0]  # 200 premiers pas, horizon 1

# Baselines
naive_last     = naive_forecast(np.concatenate([scaler.inverse_transform(X_test[:1, -1]).flatten(), y_true_final[:-1]]), horizon=1, method='last')
naive_seasonal = naive_forecast(np.concatenate([scaler.inverse_transform(X_test[:200, -1]).flatten()]), horizon=1, method='last')

models_scores = {
    'ARIMA (example)': (5.2, 6.8, 3.1),   # MAE, RMSE, MAPE
    'Prophet':         (4.1, 5.5, 2.4),
    'LSTM 1-step':     (3.2, 4.3, 1.8),
    'Naive (last)':    (7.8, 9.2, 4.5),
    'Naive (seasonal)':(4.9, 6.1, 2.8),
}

print(f"\n{'Modèle':<20} {'MAE':>8} {'RMSE':>8} {'MAPE%':>8}")
print('-' * 46)
for model_name, (mae, rmse, mape) in models_scores.items():
    print(f"{model_name:<20} {mae:>8.2f} {rmse:>8.2f} {mape:>8.2f}")
```

---

## 13. Exercices pratiques

### Exercice 1 — RNN et LSTM (fondamentaux)

> [!info] Exercice 1 : Comprendre le vanishing gradient
> 1. Implémente un RNN vanilla en NumPy pur (sans Keras) sur une séquence binaire simple
> 2. Entraîne-le avec BPTT sur `T=5`, puis `T=20`, puis `T=50`
> 3. Log le gradient de `W_h` à chaque époque
> 4. Trace la norme du gradient en fonction de T — observe la disparition

```python
# Squelette de départ pour l'exercice 1
import numpy as np

# Données : copier le premier élément d'une séquence à la fin
# Entrée  : [1, 0, 1, 1, 0, 0, 1, ...]
# Cible   : [1, 0, 1, 1, 0, 0, 1, ...] (même valeur T steps plus tard)

def generate_copy_task(seq_len, num_samples=100):
    """
    Génère des séquences pour la tâche "copier le premier bit".
    C'est une tâche qui nécessite explicitement une mémoire longue.
    """
    X = np.random.randint(0, 2, (num_samples, seq_len, 1)).astype(float)
    y = X[:, 0, :]  # cible = premier élément
    return X, y

# TODO : implémenter le RNN forward + BPTT
# TODO : mesurer la norme ||dL/dW_h|| en fonction de seq_len
# TODO : comparer avec LSTM en Keras (layers.LSTM(32))
```

### Exercice 2 — Classificateur de sentiment personnalisé

> [!info] Exercice 2 : Dataset IMDB francophone
> 1. Utilise le dataset [Allocine](https://huggingface.co/datasets/allocine) (critiques de films en français) depuis HuggingFace
> 2. Tokenise avec `keras.preprocessing.text.Tokenizer`
> 3. Compare : LSTM vs GRU vs BiLSTM sur le même jeu de données
> 4. Affiche la matrice de confusion et le rapport de classification

```python
# Aide : chargement du dataset Allocine
# pip install datasets
from datasets import load_dataset

dataset = load_dataset("allocine")
train_texts = [item['review'] for item in dataset['train']]
train_labels = [item['label'] for item in dataset['train']]
# 0 = négatif, 1 = positif

# À toi de jouer :
# 1. Tokenisation
# 2. Padding
# 3. 3 modèles à comparer
# 4. Tableau de résultats (accuracy, F1)
```

### Exercice 3 — Forecasting sur données réelles

> [!info] Exercice 3 : Prévision de la consommation électrique
> 1. Télécharge les données de consommation électrique de RTE France ou utilise le dataset UCI "Household Power Consumption"
> 2. Analyse la stationnarité et la saisonnalité
> 3. Entraîne ARIMA, Prophet et LSTM
> 4. Compare sur les 30 derniers jours avec MAE, RMSE et MAPE
> 5. Bonus : génère des intervalles de confiance pour le LSTM (via Monte Carlo Dropout)

```python
# Aide : Monte Carlo Dropout pour les intervalles de confiance LSTM
def predict_with_uncertainty(model, X, n_iterations=100):
    """
    Estime l'incertitude via Monte Carlo Dropout.
    Requires le modèle d'avoir des couches Dropout actives en inférence.
    """
    # Pour activer le Dropout en inférence, utiliser training=True
    predictions = np.array([
        model(X, training=True).numpy()
        for _ in range(n_iterations)
    ])
    
    mean_pred = predictions.mean(axis=0)
    std_pred  = predictions.std(axis=0)
    
    lower = mean_pred - 1.96 * std_pred  # intervalle 95%
    upper = mean_pred + 1.96 * std_pred
    
    return mean_pred, lower, upper

# mean_pred, lower, upper = predict_with_uncertainty(model_lstm_ts, X_test[:50])
```

### Exercice 4 — Génération de musique (challenge)

> [!info] Exercice 4 (Challenge) : LSTM pour générer des mélodies
> 1. Représente les notes MIDI comme des tokens (60 notes possibles + silences)
> 2. Entraîne un LSTM caractère-par-caractère sur un corpus de mélodies encodées
> 3. Génère une mélodie de 64 notes à partir d'une amorce de 8 notes
> 4. Varie la température (0.3, 0.7, 1.2) et commente l'effet

---

## 14. Bonnes pratiques et pièges fréquents

### 14.1 Pièges à éviter

> [!warning] Data leakage dans les séries temporelles
> C'est le piège le plus commun. Tu fais du data leakage si :
> - Tu normalises **avant** de splitter (le scaler "voit" les données de test)
> - Tu utilises `train_test_split(shuffle=True)` sur des données temporelles
> - Tu calcules des features rolling sur la série entière puis tu splittes
>
> **Solution** : toujours travailler dans l'ordre chronologique. Fitter le scaler uniquement sur le train set.

```python
# ❌ MAUVAIS : data leakage
from sklearn.preprocessing import MinMaxScaler
scaler = MinMaxScaler()
data_normalized = scaler.fit_transform(all_data)   # voit les données de test !
X_train, X_test = split(data_normalized)

# ✅ CORRECT : fit sur train uniquement
scaler = MinMaxScaler()
train_data = scaler.fit_transform(train_data)   # fit + transform sur train
test_data  = scaler.transform(test_data)         # transform seulement sur test
```

> [!warning] Oublier l'inverse-transform
> Après la normalisation, les métriques se calculent toujours sur les **données à l'échelle originale**. Un MAPE de 2% sur des données normalisées [0,1] ne signifie pas la même chose que 2% sur les ventes réelles.

```python
# Exemple de pipeline correct pour l'évaluation
y_pred_scaled = model.predict(X_test)

# Inverse transform AVANT de calculer les métriques
y_pred_original = scaler.inverse_transform(y_pred_scaled.reshape(-1, 1)).flatten()
y_true_original = scaler.inverse_transform(y_test.reshape(-1, 1)).flatten()

mae = mean_absolute_error(y_true_original, y_pred_original)
print(f"MAE en unités originales : {mae:.2f}")
```

### 14.2 Choisir le bon modèle

| Situation | Modèle recommandé | Raison |
|-----------|------------------|--------|
| Petite série (< 200 points), saisonnalité connue | ARIMA / SARIMA | Robuste, peu de données nécessaires |
| Données avec jours fériés, effets calendaires | Prophet | Gestion native des événements |
| Grande série (> 5000 points), patterns complexes | LSTM | Capacité à apprendre des patterns non linéaires |
| Plusieurs series similaires à prédire en même temps | LSTM multivarié | Transfert de connaissances entre séries |
| Besoin d'intervalles de confiance bien calibrés | Prophet ou ARIMA | Distribution théorique des erreurs |
| Temps réel, latence critique | GRU (moins lourd que LSTM) | Moins de paramètres, inférence plus rapide |

### 14.3 Checklist avant de soumettre un modèle de prévision

```
□ Test de stationnarité réalisé (ADF + KPSS)
□ Données normalisées après split (pas avant)
□ Pas de shuffle sur les données temporelles
□ Baseline naïve calculée (le modèle doit la battre)
□ Métriques calculées sur les données à l'échelle originale
□ Overfitting vérifié (train loss ≈ val loss)
□ Résidus analysés (bruit blanc ?)
□ Prévision visualisée avec intervalles de confiance
□ Performance stable sur plusieurs horizons (J+1, J+7, J+30)
```

---

## 15. Récapitulatif et liens avec les autres cours

### 15.1 Carte mentale du cours

```
                    SÉQUENCES ET TEMPS
                          │
           ┌──────────────┼──────────────┐
           │              │              │
      Réseaux          Modèles        Outils
      récurrents        classiques      modernes
           │              │              │
    ┌──────┤         ┌────┤         ┌────┤
    │      │         │    │         │    │
   RNN    LSTM      AR    MA      Prophet LSTM-TS
    │      │         │    │
   GRU  BiLSTM    ARIMA   │
    │              │    SARIMA
  Seq2Seq      différenciation
    │           stationnarité
  Attention
    │
[→ Transformers B3]
```

### 15.2 Liens avec les autres cours Holberton

| Cours | Connexion |
|-------|-----------|
| **B2 — Réseaux de neurones** | Les RNN sont des MLP avec connexions récurrentes |
| **B2 — Régularisation** | Dropout, L2, early stopping s'appliquent identiquement aux LSTM |
| **B2 — Optimisation** | Adam, RMSProp, gradient clipping — mêmes règles |
| **B3 — Transformers / Attention** | L'attention introduite au §6 est le cœur des Transformers |
| **B3 — NLP avancé** | BERT, GPT utilisent des mécanismes issus du Seq2Seq |
| **B1 — Statistiques** | Stationnarité, ACF, PACF viennent de la statistique classique |

> [!tip] Pour aller plus loin
> - **Temporal Fusion Transformer** (TFT) : architecture récente qui combine attention + LSTMs pour le forecasting multi-horizons avec intervalles de confiance
> - **N-BEATS / N-HiTS** : réseaux purement feed-forward qui surpassent souvent les LSTM sur les séries longues sans nécessiter de feature engineering
> - **WaveNet** : CNN dilaté pour les séquences audio — une alternative aux RNN quand la séquence est très longue
> - **Darts** : librairie Python qui uniforme ARIMA, Prophet, LSTM, Transformers dans une API cohérente
