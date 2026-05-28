# 04 - Réseaux de Neurones et Deep Learning

> [!info] Objectif du cours
> Comprendre l'architecture et l'entraînement des réseaux de neurones, de la théorie du perceptron jusqu'à l'implémentation pratique avec Keras/TensorFlow et PyTorch. Inclut backpropagation, régularisation, et optimisation avancée.

---

## 1. Le Neurone Artificiel — Perceptron

### 1.1 Modèle du Neurone

Un neurone artificiel calcule une **somme pondérée** de ses entrées, y ajoute un biais, et applique une **fonction d'activation** :

$$z = \sum_{i=1}^{n} w_i x_i + b = \vec{w}^T \vec{x} + b$$
$$a = f(z) \quad \text{(activation)}$$

```python
import numpy as np
import matplotlib.pyplot as plt

def perceptron(x, w, b, activation_fn):
    """Neurone artificiel simple."""
    z = np.dot(w, x) + b
    return activation_fn(z)

# Exemple : porte logique AND
# Entrées : x1, x2 ∈ {0, 1}
# Sortie  : 1 si x1=1 ET x2=1

def step_fn(z):
    return 1 if z >= 0 else 0

w = np.array([1.0, 1.0])  # poids
b = -1.5                   # biais

print("Porte AND avec perceptron :")
for x1 in [0, 1]:
    for x2 in [0, 1]:
        x = np.array([x1, x2])
        out = perceptron(x, w, b, step_fn)
        print(f"  x1={x1}, x2={x2} → {out}  (attendu: {x1 and x2})")
```

### 1.2 Fonctions d'Activation

| Fonction | Formule | Domaine | Utilisation |
|---------|---------|---------|-------------|
| **ReLU** | $\max(0, z)$ | $[0, +\infty)$ | Couches cachées (défaut) |
| **Sigmoid** | $\frac{1}{1+e^{-z}}$ | $(0, 1)$ | Sortie classification binaire |
| **Softmax** | $\frac{e^{z_i}}{\sum_j e^{z_j}}$ | $(0,1)^n, \sum=1$ | Sortie multiclasse |
| **Tanh** | $\frac{e^z - e^{-z}}{e^z + e^{-z}}$ | $(-1, 1)$ | LSTM, RNN |
| **Leaky ReLU** | $\max(\alpha z, z)$ | $\mathbb{R}$ | Évite "dying ReLU" |
| **GELU** | $z \cdot \Phi(z)$ | $\mathbb{R}$ | Transformers (BERT, GPT) |

```python
def relu(z):        return np.maximum(0, z)
def leaky_relu(z, alpha=0.01): return np.where(z >= 0, z, alpha * z)
def sigmoid(z):     return 1 / (1 + np.exp(-z))
def tanh(z):        return np.tanh(z)
def softmax(z):
    e_z = np.exp(z - np.max(z))   # soustraire max pour la stabilité numérique
    return e_z / e_z.sum()

# Dérivées (pour la backpropagation)
def relu_prime(z):     return (z > 0).astype(float)
def sigmoid_prime(z):  s = sigmoid(z); return s * (1 - s)
def tanh_prime(z):     return 1 - tanh(z)**2

# Comparaison visuelle
z = np.linspace(-5, 5, 300)
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

functions = [('ReLU', relu), ('Sigmoid', sigmoid), ('Tanh', tanh), ('Leaky ReLU', leaky_relu)]
for name, fn in functions:
    axes[0].plot(z, fn(z), label=name, linewidth=2)
axes[0].set_title('Fonctions d\'activation')
axes[0].legend()
axes[0].grid(True)
axes[0].set_ylim(-1.5, 3)

derivatives = [('ReLU\'', relu_prime), ('Sigmoid\'', sigmoid_prime), ('Tanh\'', tanh_prime)]
for name, fn in derivatives:
    axes[1].plot(z, fn(z), label=name, linewidth=2)
axes[1].set_title('Dérivées des fonctions d\'activation')
axes[1].legend()
axes[1].grid(True)

plt.tight_layout()
plt.savefig('activation_functions.png', dpi=150)
plt.close()

# Démonstration Softmax
logits = np.array([2.0, 1.0, 0.1, -1.0, 3.0])
proba = softmax(logits)
print(f"Logits : {logits}")
print(f"Softmax: {proba.round(4)}")
print(f"Somme  : {proba.sum():.6f}")   # doit être 1.0
print(f"Classe prédite : {np.argmax(proba)}")
```

---

## 2. Réseau Multicouche (MLP)

### 2.1 Architecture

Un **Multilayer Perceptron** est composé de :
- **Couche d'entrée** : $n$ neurones (une par feature)
- **Couches cachées** : $L$ couches avec des neurones et activations
- **Couche de sortie** : $k$ neurones (une par classe)

```
Entrée   Couche 1   Couche 2   Sortie
x₁  ──┐                       ┌── ŷ₁
x₂  ──┤  ○○○○○   ──  ○○○○   ──┤
x₃  ──┤                       ├── ŷ₂
...  ─┘                       └── ...
```

### 2.2 Forward Pass

```python
class MLP:
    """MLP simple — implémentation NumPy pour la pédagogie."""
    
    def __init__(self, layer_sizes, activations):
        """
        layer_sizes : [n_input, n_hidden1, ..., n_output]
        activations : liste de fonctions d'activation
        """
        self.activations = activations
        self.params = {}
        
        np.random.seed(42)
        for i in range(len(layer_sizes) - 1):
            n_in  = layer_sizes[i]
            n_out = layer_sizes[i + 1]
            # Initialisation He (pour ReLU)
            scale = np.sqrt(2.0 / n_in)
            self.params[f'W{i}'] = np.random.randn(n_out, n_in) * scale
            self.params[f'b{i}'] = np.zeros(n_out)
    
    def forward(self, X):
        """
        Forward pass : calcule les activations de chaque couche.
        X : (n_samples, n_features)
        """
        self.cache = {'A-1': X}
        A = X
        
        for i, activation in enumerate(self.activations):
            W = self.params[f'W{i}']
            b = self.params[f'b{i}']
            Z = A @ W.T + b                # (n_samples, n_out)
            A = activation(Z)
            self.cache[f'Z{i}'] = Z
            self.cache[f'A{i}'] = A
        
        return A   # sortie finale
    
    def predict(self, X):
        proba = self.forward(X)
        return np.argmax(proba, axis=1)


# Test sur MNIST toy
from sklearn.datasets import load_digits
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import OneHotEncoder

digits = load_digits()
X_d = digits.data / 16.0    # normaliser 0-1
y_d = digits.target

X_tr, X_te, y_tr, y_te = train_test_split(X_d, y_d, test_size=0.2, random_state=42)

# MLP : 64 → 128 → 64 → 10
def relu(z): return np.maximum(0, z)
def softmax(z):
    e = np.exp(z - z.max(axis=1, keepdims=True))
    return e / e.sum(axis=1, keepdims=True)

mlp = MLP([64, 128, 64, 10], [relu, relu, softmax])
output = mlp.forward(X_tr[:5])
print(f"Forward pass — shape sortie : {output.shape}")
print(f"Probabilités (premier exemple) : {output[0].round(4)}")
print(f"Classe prédite : {output[0].argmax()} (vraie classe : {y_tr[0]})")
```

---

## 3. Backpropagation

### 3.1 Intuition

La backpropagation applique la **règle de la chaîne** pour calculer $\frac{\partial L}{\partial W^{(l)}}$ pour chaque couche $l$ :

$$\frac{\partial L}{\partial W^{(l)}} = \frac{\partial L}{\partial A^{(l)}} \cdot \frac{\partial A^{(l)}}{\partial Z^{(l)}} \cdot \frac{\partial Z^{(l)}}{\partial W^{(l)}}$$

```
Forward  : x → z₁ → a₁ → z₂ → a₂ → L (loss)
Backward : ∂L/∂W₂ ← ∂L/∂a₂ ← ∂L/∂z₂ ← ∂L/∂a₁ ← ∂L/∂W₁
```

### 3.2 Implémentation Complète

```python
def cross_entropy_loss(y_pred, y_true_onehot):
    """Cross-entropy loss pour softmax."""
    n = y_pred.shape[0]
    # Clip pour éviter log(0)
    y_pred_clipped = np.clip(y_pred, 1e-9, 1 - 1e-9)
    return -np.sum(y_true_onehot * np.log(y_pred_clipped)) / n

def backprop_step(params, cache, y_true_onehot, lr=0.01):
    """Un pas de backpropagation."""
    n = y_true_onehot.shape[0]
    n_layers = len([k for k in params if k.startswith('W')])
    
    grads = {}
    # Gradient de la loss par rapport à la sortie (softmax + cross-entropy)
    dA = (cache[f'A{n_layers-1}'] - y_true_onehot) / n
    
    for i in reversed(range(n_layers)):
        A_prev = cache[f'A{i-1}'] if i > 0 else cache['A-1']
        
        # Gradient par rapport aux poids et biais
        grads[f'dW{i}'] = dA.T @ A_prev
        grads[f'db{i}'] = dA.mean(axis=0)
        
        if i > 0:
            W = params[f'W{i}']
            dZ = dA @ W
            # Dérivée de ReLU
            dA = dZ * (cache[f'Z{i-1}'] > 0)
    
    # Mise à jour des poids
    for i in range(n_layers):
        params[f'W{i}'] -= lr * grads[f'dW{i}']
        params[f'b{i}'] -= lr * grads[f'db{i}']
    
    return params

print("La backpropagation est correctement implémentée dans les frameworks (TF, PyTorch).")
print("En pratique, utiliser Keras/TensorFlow ou PyTorch.")
```

---

## 4. Descente de Gradient et Optimiseurs

### 4.1 Variantes de la Descente de Gradient

| Variante | Batch | Convergence | Mémoire |
|----------|-------|-------------|---------|
| Batch GD | Tout le dataset | Stable, lente | Élevée |
| Stochastic GD (SGD) | 1 exemple | Bruyante, rapide | Faible |
| Mini-batch GD | 32-512 exemples | Compromis idéal | Modérée |

### 4.2 Optimiseurs Avancés

```python
import tensorflow as tf

# SGD classique
sgd = tf.keras.optimizers.SGD(learning_rate=0.01, momentum=0.9)

# Adam — Adaptive Moment Estimation (le plus utilisé)
# m_t = β₁ m_{t-1} + (1-β₁) g_t        (momentum du 1er ordre)
# v_t = β₂ v_{t-1} + (1-β₂) g_t²       (momentum du 2ème ordre)
# θ = θ - α * m̂_t / (√v̂_t + ε)
adam = tf.keras.optimizers.Adam(
    learning_rate=1e-3,
    beta_1=0.9,       # decay du moment 1
    beta_2=0.999,     # decay du moment 2
    epsilon=1e-7
)

# RMSprop — bon pour les RNNs
rmsprop = tf.keras.optimizers.RMSprop(learning_rate=1e-3, rho=0.9)

# AdamW — Adam + Weight Decay (régularisation L2 découplée)
# Préféré pour les Transformers
adamw = tf.keras.optimizers.AdamW(learning_rate=1e-3, weight_decay=1e-4)

print("Optimiseurs disponibles :")
for opt in [sgd, adam, rmsprop, adamw]:
    print(f"  {opt.__class__.__name__} : lr={opt.learning_rate.numpy():.4f}")
```

### 4.3 Learning Rate Scheduling

```python
# Diminuer le LR au cours de l'entraînement
lr_schedule = tf.keras.optimizers.schedules.ExponentialDecay(
    initial_learning_rate=1e-2,
    decay_steps=1000,
    decay_rate=0.9
)

# Cosine Annealing — très populaire pour les Transformers
cosine_schedule = tf.keras.optimizers.schedules.CosineDecay(
    initial_learning_rate=1e-3,
    decay_steps=10000
)

# ReduceLROnPlateau — callback
reduce_lr = tf.keras.callbacks.ReduceLROnPlateau(
    monitor='val_loss',
    factor=0.5,
    patience=5,
    min_lr=1e-7,
    verbose=1
)
```

---

## 5. Régularisation

### 5.1 Dropout

```python
import tensorflow as tf
from tensorflow import keras

# Dropout : désactiver aléatoirement p% des neurones à chaque step d'entraînement
# → Empêche le co-adaptation, force la redondance

model_dropout = keras.Sequential([
    keras.layers.Dense(256, activation='relu'),
    keras.layers.Dropout(0.3),    # 30% des neurones désactivés en training
    keras.layers.Dense(128, activation='relu'),
    keras.layers.Dropout(0.3),
    keras.layers.Dense(10, activation='softmax')
])

# En inference (model.predict), le Dropout est DÉSACTIVÉ automatiquement
```

### 5.2 Batch Normalization

```python
# BatchNorm : normalise les activations au sein d'un mini-batch
# z_norm = (z - μ_batch) / √(σ²_batch + ε)
# sortie = γ * z_norm + β   (γ, β apprenables)

model_bn = keras.Sequential([
    keras.layers.Dense(256),
    keras.layers.BatchNormalization(),   # AVANT activation !
    keras.layers.Activation('relu'),
    keras.layers.Dense(128),
    keras.layers.BatchNormalization(),
    keras.layers.Activation('relu'),
    keras.layers.Dense(10, activation='softmax')
])

# Avantages BatchNorm :
# - Permet un LR plus élevé → convergence plus rapide
# - Réduit la dépendance à l'initialisation
# - Léger effet de régularisation
```

### 5.3 L1/L2 Regularization

```python
from tensorflow.keras import regularizers

model_reg = keras.Sequential([
    keras.layers.Dense(256, activation='relu',
                       kernel_regularizer=regularizers.l2(1e-4)),
    keras.layers.Dense(128, activation='relu',
                       kernel_regularizer=regularizers.l1_l2(l1=1e-5, l2=1e-4)),
    keras.layers.Dense(10, activation='softmax')
])
```

---

## 6. TensorFlow/Keras vs PyTorch

| Aspect | TensorFlow/Keras | PyTorch |
|--------|-----------------|---------|
| **Paradigme** | Graphe statique (TF) / dynamique (eager) | Graphe dynamique (eager par défaut) |
| **API** | Sequential, Functional, Subclassing | `nn.Module` + dérivation |
| **Déploiement** | TFLite, TF Serving, TF.js | TorchScript, TorchServe |
| **Popularité** | Production, industrie | Recherche, papiers |
| **Debug** | Plus difficile | Python natif (pdb) |
| **Courbe d'apprentissage** | Douce (Keras) | Légèrement plus raide |

---

## 7. Implémentation Keras — MNIST Complet

### 7.1 Sequential API

```python
import tensorflow as tf
from tensorflow import keras
import numpy as np
import matplotlib.pyplot as plt

# Chargement MNIST
(X_train, y_train), (X_test, y_test) = keras.datasets.mnist.load_data()

# Préprocessing
X_train = X_train.reshape(-1, 784).astype('float32') / 255.0
X_test  = X_test.reshape(-1, 784).astype('float32') / 255.0

print(f"Train : {X_train.shape} → {y_train.shape}")
print(f"Test  : {X_test.shape} → {y_test.shape}")
print(f"Classes : {np.unique(y_train)}")

# Architecture MLP
model = keras.Sequential([
    keras.layers.Input(shape=(784,)),
    keras.layers.Dense(512, activation='relu'),
    keras.layers.BatchNormalization(),
    keras.layers.Dropout(0.3),
    keras.layers.Dense(256, activation='relu'),
    keras.layers.BatchNormalization(),
    keras.layers.Dropout(0.3),
    keras.layers.Dense(128, activation='relu'),
    keras.layers.Dropout(0.2),
    keras.layers.Dense(10, activation='softmax')
], name='MLP_MNIST')

model.summary()
# Total params: ~550,000
```

### 7.2 Compilation et Callbacks

```python
# Compilation
model.compile(
    optimizer=keras.optimizers.Adam(learning_rate=1e-3),
    loss='sparse_categorical_crossentropy',   # pour labels entiers (pas one-hot)
    metrics=['accuracy']
)

# Callbacks — comportements automatiques pendant l'entraînement
callbacks = [
    # Arrêter si la validation ne s'améliore pas
    keras.callbacks.EarlyStopping(
        monitor='val_loss',
        patience=10,
        restore_best_weights=True,   # restaurer les meilleurs poids
        verbose=1
    ),
    # Sauvegarder le meilleur modèle
    keras.callbacks.ModelCheckpoint(
        filepath='best_mnist_model.keras',
        monitor='val_accuracy',
        save_best_only=True,
        verbose=1
    ),
    # Réduire le LR si la validation stagne
    keras.callbacks.ReduceLROnPlateau(
        monitor='val_loss',
        factor=0.5,
        patience=5,
        min_lr=1e-7,
        verbose=1
    ),
    # TensorBoard — visualisation de l'entraînement
    keras.callbacks.TensorBoard(
        log_dir='logs/mnist',
        histogram_freq=1
    )
]

# Entraînement
history = model.fit(
    X_train, y_train,
    epochs=50,
    batch_size=256,
    validation_split=0.1,   # 10% du train pour la validation
    callbacks=callbacks,
    verbose=1
)
```

### 7.3 Évaluation et Visualisation

```python
# Évaluation sur le test set
test_loss, test_acc = model.evaluate(X_test, y_test, verbose=0)
print(f"\nTest Loss     : {test_loss:.4f}")
print(f"Test Accuracy : {test_acc:.4f}")

# Courbes d'apprentissage
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(12, 4))

ax1.plot(history.history['accuracy'], label='Train')
ax1.plot(history.history['val_accuracy'], label='Validation')
ax1.set_title('Accuracy')
ax1.legend()
ax1.grid(True)

ax2.plot(history.history['loss'], label='Train')
ax2.plot(history.history['val_loss'], label='Validation')
ax2.set_title('Loss')
ax2.legend()
ax2.grid(True)

plt.tight_layout()
plt.savefig('mnist_training_curves.png', dpi=150)
plt.close()

# Prédictions et matrice de confusion
y_pred = model.predict(X_test)
y_pred_classes = np.argmax(y_pred, axis=1)

from sklearn.metrics import confusion_matrix, classification_report
import seaborn as sns

cm = confusion_matrix(y_test, y_pred_classes)
fig, ax = plt.subplots(figsize=(10, 8))
sns.heatmap(cm, annot=True, fmt='d', cmap='Blues', ax=ax)
ax.set_xlabel('Prédit')
ax.set_ylabel('Réel')
ax.set_title('Matrice de Confusion MNIST')
plt.savefig('confusion_matrix_mnist.png', dpi=150)
plt.close()

print(classification_report(y_test, y_pred_classes))
```

### 7.4 Functional API — Pour les Architectures Complexes

```python
# La Functional API permet des architectures non-linéaires
# (multiples entrées, sorties, connexions skip, etc.)

inputs = keras.Input(shape=(784,), name='image_input')

# Branche principale
x = keras.layers.Dense(512, activation='relu', name='dense_1')(inputs)
x = keras.layers.BatchNormalization()(x)
x = keras.layers.Dropout(0.3)(x)

# Skip connection (comme ResNet)
shortcut = keras.layers.Dense(256, activation=None)(inputs)   # projection

x = keras.layers.Dense(256, activation=None)(x)
x = keras.layers.Add()([x, shortcut])     # addition résiduelle
x = keras.layers.Activation('relu')(x)
x = keras.layers.BatchNormalization()(x)
x = keras.layers.Dropout(0.2)(x)

outputs = keras.layers.Dense(10, activation='softmax', name='predictions')(x)

model_functional = keras.Model(inputs=inputs, outputs=outputs, name='MLP_ResNet')
model_functional.summary()

# Visualiser l'architecture (nécessite pydot et graphviz)
# keras.utils.plot_model(model_functional, 'model_architecture.png', show_shapes=True)
```

---

## 8. GPU Setup et Entraînement Accéléré

```python
# Vérifier la disponibilité du GPU
print("GPUs disponibles :", tf.config.list_physical_devices('GPU'))
print("Version TF :", tf.__version__)

# Entraîner sur GPU — automatique si disponible
# Sur Colab : Runtime → Change runtime type → GPU

# Mémoire GPU limitée : eviter l'OOM
gpus = tf.config.experimental.list_physical_devices('GPU')
if gpus:
    for gpu in gpus:
        tf.config.experimental.set_memory_growth(gpu, True)
    print("Croissance mémoire GPU activée")

# Mixed Precision — accélération sur GPUs modernes (Ampere+)
from tensorflow.keras import mixed_precision
mixed_precision.set_global_policy('mixed_float16')
# Les calculs en float16, les poids stockés en float32
# Gain : 2x vitesse sur GPU, 2x moins de mémoire

# PyTorch équivalent :
# device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
# model.to(device)
# X = X.to(device)
```

---

## 9. Exemple PyTorch Complet

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader, TensorDataset

# Données
X_tr_t = torch.FloatTensor(X_train)
y_tr_t = torch.LongTensor(y_train)
X_te_t = torch.FloatTensor(X_test)
y_te_t = torch.LongTensor(y_test)

dataset = TensorDataset(X_tr_t, y_tr_t)
loader  = DataLoader(dataset, batch_size=256, shuffle=True)

# Modèle PyTorch — définition par héritage de nn.Module
class MLP_PyTorch(nn.Module):
    def __init__(self, input_dim=784, hidden_dims=[512, 256, 128], n_classes=10):
        super().__init__()
        
        layers = []
        prev_dim = input_dim
        for h in hidden_dims:
            layers.extend([
                nn.Linear(prev_dim, h),
                nn.BatchNorm1d(h),
                nn.ReLU(),
                nn.Dropout(0.3)
            ])
            prev_dim = h
        layers.append(nn.Linear(prev_dim, n_classes))
        
        self.network = nn.Sequential(*layers)
    
    def forward(self, x):
        return self.network(x)

device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
model_pt = MLP_PyTorch().to(device)
print(f"Modèle PyTorch sur : {device}")
print(f"Paramètres totaux : {sum(p.numel() for p in model_pt.parameters()):,}")

# Entraînement PyTorch
criterion = nn.CrossEntropyLoss()
optimizer = optim.Adam(model_pt.parameters(), lr=1e-3)
scheduler = optim.lr_scheduler.ReduceLROnPlateau(optimizer, patience=3)

def train_epoch(model, loader, optimizer, criterion, device):
    model.train()   # mode entraînement (Dropout actif)
    total_loss, total_correct = 0.0, 0
    for X_batch, y_batch in loader:
        X_batch, y_batch = X_batch.to(device), y_batch.to(device)
        optimizer.zero_grad()          # remettre les gradients à 0
        logits = model(X_batch)        # forward pass
        loss = criterion(logits, y_batch)
        loss.backward()                # backpropagation
        optimizer.step()               # mise à jour des poids
        
        total_loss    += loss.item()
        total_correct += (logits.argmax(1) == y_batch).sum().item()
    
    return total_loss / len(loader), total_correct / len(loader.dataset)

def eval_epoch(model, X, y, device):
    model.eval()   # mode évaluation (Dropout désactivé)
    with torch.no_grad():
        logits = model(X.to(device))
        preds  = logits.argmax(1).cpu()
        acc    = (preds == y).float().mean().item()
    return acc

# Boucle d'entraînement
n_epochs = 20
print(f"\n{'Epoch':>5} | {'Train Loss':>12} | {'Train Acc':>10} | {'Test Acc':>10}")
print("-" * 48)
for epoch in range(n_epochs):
    tr_loss, tr_acc = train_epoch(model_pt, loader, optimizer, criterion, device)
    te_acc = eval_epoch(model_pt, X_te_t, y_te_t, device)
    scheduler.step(tr_loss)
    
    if (epoch + 1) % 5 == 0:
        print(f"{epoch+1:>5} | {tr_loss:>12.4f} | {tr_acc:>10.4f} | {te_acc:>10.4f}")

print(f"\nAccuracy finale sur test : {te_acc:.4f}")
```

---

## 10. Éviter le Surapprentissage — Stratégies Combinées

```python
# Stratégie complète anti-overfitting

model_robust = keras.Sequential([
    keras.layers.Input(shape=(784,)),
    
    # L2 regularization sur les poids
    keras.layers.Dense(512, activation='relu',
                       kernel_regularizer=keras.regularizers.l2(1e-4),
                       kernel_initializer='he_normal'),
    keras.layers.BatchNormalization(),
    keras.layers.Dropout(0.4),
    
    keras.layers.Dense(256, activation='relu',
                       kernel_regularizer=keras.regularizers.l2(1e-4),
                       kernel_initializer='he_normal'),
    keras.layers.BatchNormalization(),
    keras.layers.Dropout(0.3),
    
    keras.layers.Dense(10, activation='softmax')
])

model_robust.compile(
    optimizer=keras.optimizers.AdamW(learning_rate=1e-3, weight_decay=1e-4),
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy']
)

# Callbacks anti-overfitting
callbacks_robust = [
    keras.callbacks.EarlyStopping(patience=15, restore_best_weights=True),
    keras.callbacks.ReduceLROnPlateau(patience=7, factor=0.3, min_lr=1e-7),
]

# Note : data augmentation (voir cours CNN) est une autre technique clé
print("Stratégies anti-overfitting :")
print("  1. Dropout (0.2-0.5)")
print("  2. L1/L2 regularization")
print("  3. BatchNormalization")
print("  4. EarlyStopping")
print("  5. Data Augmentation")
print("  6. Plus de données")
print("  7. Modèle moins profond / moins large")
```

---

## 11. Métriques d'Entraînement à Surveiller

> [!warning] Signes d'alerte à surveiller
> - **Loss train baisse mais val_loss monte** → Overfitting → augmenter régularisation
> - **Les deux loss restent élevées** → Underfitting → augmenter la capacité du modèle
> - **Loss oscillante et instable** → Learning rate trop élevé → diminuer lr
> - **Loss ne bouge plus** → Learning rate trop faible ou gradient vanishing

```python
def analyser_historique(history):
    """Analyse automatique de la courbe d'apprentissage."""
    tr_loss = np.array(history.history['loss'])
    val_loss = np.array(history.history['val_loss'])
    
    best_epoch = np.argmin(val_loss)
    final_gap = tr_loss[-1] - val_loss[-1]
    
    print(f"Meilleure epoch : {best_epoch+1} (val_loss={val_loss[best_epoch]:.4f})")
    print(f"Amélioration    : {(val_loss[0] - val_loss[best_epoch])*100/val_loss[0]:.1f}%")
    
    if final_gap < -0.05:
        print("⚠ Possible UNDERFITTING — loss val < loss train")
    elif final_gap > 0.1:
        print("⚠ OVERFITTING détecté — augmenter la régularisation")
    else:
        print("✓ Bon compromis bias-variance")

# analyser_historique(history)
```

---

## 12. Exercices Pratiques

> [!tip] Exercice 1 — Fonctions d'Activation
> Implémentez en NumPy pur (sans bibliothèque) : ReLU, Leaky ReLU, Sigmoid, Tanh et leurs dérivées. Vérifiez vos dérivées par différences finies. Tracez les 4 fonctions et leurs dérivées sur $[-5, 5]$.

> [!tip] Exercice 2 — MLP sur Fashion-MNIST
> Fashion-MNIST est plus difficile que MNIST (10 classes de vêtements). Chargez-le avec `keras.datasets.fashion_mnist.load_data()`. Entraînez un MLP avec BatchNorm et Dropout. Objectif : accuracy > 88%. Affichez la matrice de confusion et identifiez les classes les plus confondues.

> [!tip] Exercice 3 — Comparaison d'Optimiseurs
> Sur le même MLP (MNIST ou Fashion-MNIST), comparez SGD (lr=0.01, momentum=0.9), Adam (lr=1e-3) et RMSprop (lr=1e-3). Tracez les courbes de loss et comparez la vitesse de convergence et l'accuracy finale.

> [!tip] Exercice 4 — Régularisation
> Sur un petit dataset (500 exemples de MNIST), entraînez 4 modèles : (1) sans régularisation, (2) avec Dropout seul, (3) avec L2 seul, (4) avec les deux. Tracez les courbes train vs val et comparez le gap de généralisation.

> [!tip] Exercice 5 — PyTorch
> Réimplémentez l'Exercice 2 en PyTorch. Utilisez `nn.Module`, `DataLoader`, et `torch.optim.Adam`. Ajoutez un `torch.utils.tensorboard.SummaryWriter` pour visualiser la loss.

---

## Liens

- [[01 - Mathematiques pour le ML]] — calcul différentiel et backpropagation
- [[03 - ML Supervise avec Scikit-Learn]] — ML classique
- [[05 - CNN Transformer et Computer Vision]] — architectures avancées
- [[06 - ML Non Supervise et Reinforcement Learning]] — autoencoders et GANs

> [!info] Ressources
> - Documentation Keras : https://keras.io/
> - PyTorch Tutorials : https://pytorch.org/tutorials/
> - "Deep Learning" — Goodfellow, Bengio, Courville (livre de référence)
> - Stanford CS231n : http://cs231n.stanford.edu/
