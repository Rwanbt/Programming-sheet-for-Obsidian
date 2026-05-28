# 05 - CNN, Transformer et Computer Vision

> [!info] Objectif du cours
> Maîtriser les architectures de vision par ordinateur : des convolutions de base aux réseaux ResNet, au Transfer Learning, à la détection d'objets YOLO, et aux Transformers (BERT, ViT). Implémentations complètes avec Keras.

---

## 1. La Convolution — Fondement des CNNs

### 1.1 Opération de Convolution

Une **convolution** fait glisser un **filtre** (kernel) sur l'image et calcule le produit scalaire à chaque position. Elle détecte des patterns locaux (bords, textures, formes).

$$(\text{Image} * \text{Filtre})[i, j] = \sum_{m}\sum_{n} \text{Image}[i+m, j+n] \cdot \text{Filtre}[m, n]$$

```python
import numpy as np
import matplotlib.pyplot as plt

def convolve2d(image, kernel, padding=0, stride=1):
    """Convolution 2D manuelle — pour la compréhension."""
    H, W = image.shape
    kH, kW = kernel.shape
    
    # Padding
    if padding > 0:
        image = np.pad(image, padding, mode='constant', constant_values=0)
        H, W = image.shape
    
    # Taille de la feature map
    out_H = (H - kH) // stride + 1
    out_W = (W - kW) // stride + 1
    output = np.zeros((out_H, out_W))
    
    for i in range(0, out_H):
        for j in range(0, out_W):
            region = image[i*stride:i*stride+kH, j*stride:j*stride+kW]
            output[i, j] = np.sum(region * kernel)
    
    return output

# Filtres classiques
# Détection de bords horizontaux (filtre Sobel)
sobel_h = np.array([[-1, -2, -1],
                     [ 0,  0,  0],
                     [ 1,  2,  1]])

# Détection de bords verticaux
sobel_v = np.array([[-1,  0,  1],
                     [-2,  0,  2],
                     [-1,  0,  1]])

# Sharpen
sharpen = np.array([[ 0, -1,  0],
                    [-1,  5, -1],
                    [ 0, -1,  0]])

# Exemple sur une image simple
image = np.array([[0, 0, 0, 0, 0, 0],
                  [0, 1, 1, 1, 1, 0],
                  [0, 1, 1, 1, 1, 0],
                  [0, 0, 0, 0, 0, 0],
                  [0, 0, 0, 0, 0, 0]], dtype=float)

feature_map_h = convolve2d(image, sobel_h)
feature_map_v = convolve2d(image, sobel_v)

print(f"Image shape      : {image.shape}")
print(f"Feature map shape: {feature_map_h.shape}")
print(f"\nBords horizontaux :\n{feature_map_h}")
print(f"\nBords verticaux :\n{feature_map_v}")
```

### 1.2 Paramètres de la Convolution

| Paramètre | Effet | Formule |
|-----------|-------|---------|
| **Stride** $s$ | Pas du filtre | $\lfloor\frac{H - k + 2p}{s}\rfloor + 1$ |
| **Padding** $p$ | Conserver la taille | `same` → $p = \lfloor k/2 \rfloor$ |
| **Filters** $f$ | Nombre de feature maps | → sortie à $f$ canaux |

```python
import tensorflow as tf
from tensorflow import keras

# Calcul de la taille de sortie
def output_size(H, k, p=0, s=1):
    return (H - k + 2*p) // s + 1

H = 32   # taille entrée
k = 3    # taille kernel

print("Tailles de sortie selon la configuration :")
print(f"  H={H}, k={k}, p=0, s=1 (valid)  → {output_size(H, k, 0, 1)}")
print(f"  H={H}, k={k}, p=1, s=1 (same)   → {output_size(H, k, 1, 1)}")
print(f"  H={H}, k={k}, p=0, s=2 (stride) → {output_size(H, k, 0, 2)}")

# Convolution Keras
conv_layer = keras.layers.Conv2D(
    filters=32,           # 32 filtres → 32 feature maps
    kernel_size=(3, 3),  # filtre 3x3
    strides=(1, 1),      # pas de 1
    padding='same',      # conserver H et W
    activation='relu',
    kernel_initializer='he_normal'
)

# Test sur un batch d'images
batch = np.random.randn(8, 32, 32, 3).astype(np.float32)   # (batch, H, W, C)
output = conv_layer(batch)
print(f"\nEntrée : {batch.shape}")
print(f"Sortie : {output.shape}")    # (8, 32, 32, 32) — même HxW, 32 canaux
print(f"Paramètres Conv : {3*3*3*32 + 32} (w + bias)")
```

### 1.3 Pooling

```python
# Max Pooling — sous-échantillonnage invariant aux petites translations
max_pool = keras.layers.MaxPooling2D(pool_size=(2, 2), strides=(2, 2))
# (8, 32, 32, 32) → (8, 16, 16, 32)

# Average Pooling
avg_pool = keras.layers.AveragePooling2D(pool_size=(2, 2))

# Global Average Pooling — remplacer Flatten + Dense par un seul scalaire par channel
# (8, H, W, C) → (8, C) : moyenne sur H et W
gap = keras.layers.GlobalAveragePooling2D()

# Exemple numérique de Max Pooling 2x2
feature = np.array([[1, 2, 3, 4],
                     [5, 6, 7, 8],
                     [9, 10, 11, 12],
                     [13, 14, 15, 16]])

def max_pool_2x2(feat):
    out = np.zeros((feat.shape[0]//2, feat.shape[1]//2))
    for i in range(0, feat.shape[0], 2):
        for j in range(0, feat.shape[1], 2):
            out[i//2, j//2] = feat[i:i+2, j:j+2].max()
    return out

print(f"Feature map 4x4 :\n{feature}")
print(f"\nMax Pool 2x2 :\n{max_pool_2x2(feature)}")
```

---

## 2. Architecture CNN Classique

### 2.1 Pattern Conv → BN → ReLU → Pool

```python
def conv_bn_relu(filters, kernel_size=3, strides=1):
    """Bloc Conv + BN + ReLU (standard moderne)."""
    return keras.Sequential([
        keras.layers.Conv2D(filters, kernel_size, strides=strides,
                            padding='same', use_bias=False),
        keras.layers.BatchNormalization(),
        keras.layers.Activation('relu')
    ])

# CNN complet pour CIFAR-10
def build_cnn_cifar10():
    return keras.Sequential([
        # Bloc 1
        keras.layers.Input(shape=(32, 32, 3)),
        keras.layers.Conv2D(32, 3, padding='same', activation='relu'),
        keras.layers.Conv2D(32, 3, padding='same', activation='relu'),
        keras.layers.MaxPooling2D(2, 2),
        keras.layers.Dropout(0.25),
        
        # Bloc 2
        keras.layers.Conv2D(64, 3, padding='same', activation='relu'),
        keras.layers.Conv2D(64, 3, padding='same', activation='relu'),
        keras.layers.MaxPooling2D(2, 2),
        keras.layers.Dropout(0.25),
        
        # Bloc 3
        keras.layers.Conv2D(128, 3, padding='same', activation='relu'),
        keras.layers.BatchNormalization(),
        keras.layers.MaxPooling2D(2, 2),
        keras.layers.Dropout(0.4),
        
        # Tête de classification
        keras.layers.Flatten(),           # (batch, 4, 4, 128) → (batch, 2048)
        keras.layers.Dense(512, activation='relu'),
        keras.layers.Dropout(0.5),
        keras.layers.Dense(10, activation='softmax')   # 10 classes CIFAR-10
    ], name='CNN_CIFAR10')

cnn = build_cnn_cifar10()
cnn.summary()
print(f"Paramètres : {cnn.count_params():,}")
```

### 2.2 Entraînement CIFAR-10

```python
# Chargement CIFAR-10
(X_train, y_train), (X_test, y_test) = keras.datasets.cifar10.load_data()
X_train = X_train.astype('float32') / 255.0
X_test  = X_test.astype('float32') / 255.0
y_train = y_train.flatten()
y_test  = y_test.flatten()

class_names = ['airplane', 'automobile', 'bird', 'cat', 'deer',
               'dog', 'frog', 'horse', 'ship', 'truck']

print(f"CIFAR-10 — Train: {X_train.shape}, Test: {X_test.shape}")

cnn.compile(
    optimizer=keras.optimizers.Adam(1e-3),
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy']
)

history = cnn.fit(
    X_train, y_train,
    epochs=50,
    batch_size=128,
    validation_split=0.1,
    callbacks=[
        keras.callbacks.EarlyStopping(patience=10, restore_best_weights=True),
        keras.callbacks.ReduceLROnPlateau(patience=5, factor=0.3)
    ],
    verbose=1
)

test_loss, test_acc = cnn.evaluate(X_test, y_test, verbose=0)
print(f"Test Accuracy : {test_acc:.4f}")
```

---

## 3. ResNet — Skip Connections

### 3.1 Problème du Gradient Vanishing et Solution

Plus un réseau est profond, plus les gradients s'atténuent en remontant vers les premières couches (gradient vanishing). ResNet résout cela avec des **connexions résiduelles** :

$$\text{Sortie} = F(x) + x \quad \text{(au lieu de juste } F(x))$$

Ainsi, $\frac{\partial L}{\partial x} = \frac{\partial L}{\partial \text{sortie}} \cdot (F'(x) + 1)$ — le gradient "+1" assure un flux minimal.

```python
def residual_block(x, filters, stride=1):
    """Bloc résiduel standard."""
    shortcut = x
    
    # Branche principale
    out = keras.layers.Conv2D(filters, 3, strides=stride, padding='same',
                               use_bias=False)(x)
    out = keras.layers.BatchNormalization()(out)
    out = keras.layers.Activation('relu')(out)
    
    out = keras.layers.Conv2D(filters, 3, padding='same', use_bias=False)(out)
    out = keras.layers.BatchNormalization()(out)
    
    # Projection si les dimensions changent (stride > 1 ou channels différents)
    if stride != 1 or x.shape[-1] != filters:
        shortcut = keras.layers.Conv2D(filters, 1, strides=stride,
                                        use_bias=False)(x)
        shortcut = keras.layers.BatchNormalization()(shortcut)
    
    # Skip connection : addition
    out = keras.layers.Add()([out, shortcut])
    out = keras.layers.Activation('relu')(out)
    return out

def build_resnet_mini(input_shape=(32, 32, 3), n_classes=10):
    """Mini-ResNet pour CIFAR-10."""
    inputs = keras.Input(shape=input_shape)
    
    x = keras.layers.Conv2D(32, 3, padding='same', use_bias=False)(inputs)
    x = keras.layers.BatchNormalization()(x)
    x = keras.layers.Activation('relu')(x)
    
    x = residual_block(x, 32)
    x = residual_block(x, 64, stride=2)   # réduction spatiale
    x = residual_block(x, 64)
    x = residual_block(x, 128, stride=2)
    x = residual_block(x, 128)
    
    x = keras.layers.GlobalAveragePooling2D()(x)
    outputs = keras.layers.Dense(n_classes, activation='softmax')(x)
    
    return keras.Model(inputs, outputs, name='MiniResNet')

resnet = build_resnet_mini()
resnet.summary()
```

---

## 4. Transfer Learning

### 4.1 Pourquoi le Transfer Learning ?

Entraîner un ResNet50 sur ImageNet (1.2M images, 1000 classes) prend des semaines sur plusieurs GPUs. Le Transfer Learning réutilise ces poids pré-entraînés sur une nouvelle tâche.

```
Modèle pré-entraîné (ImageNet)
     ↓
Couches convolutives → Features génériques (bords, textures, formes)
     ↓
Nouvelle tête de classification
```

### 4.2 Feature Extraction vs Fine-Tuning

```python
# Chargement VGG16 / ResNet50 / EfficientNet sans la tête de classification
base_model = keras.applications.ResNet50(
    weights='imagenet',    # poids ImageNet pré-entraînés
    include_top=False,     # exclure la couche Dense(1000, softmax)
    input_shape=(224, 224, 3)
)

print(f"ResNet50 paramètres : {base_model.count_params():,}")

# OPTION 1 : Feature Extraction — congeler le backbone
base_model.trainable = False

model_fe = keras.Sequential([
    base_model,
    keras.layers.GlobalAveragePooling2D(),
    keras.layers.Dense(256, activation='relu'),
    keras.layers.Dropout(0.5),
    keras.layers.Dense(5, activation='softmax')   # 5 nouvelles classes
])

model_fe.compile(
    optimizer=keras.optimizers.Adam(1e-3),
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy']
)

# Compter les paramètres entraînables
trainable = sum([w.numpy().size for w in model_fe.trainable_weights])
total = model_fe.count_params()
print(f"Paramètres entraînables : {trainable:,} / {total:,} ({100*trainable/total:.1f}%)")

# OPTION 2 : Fine-Tuning — débloquer les dernières couches
base_model.trainable = True

# Geler jusqu'à la couche conv5 (fine-tuner seulement les 20 dernières couches)
for layer in base_model.layers[:-20]:
    layer.trainable = False

# Utiliser un LR très faible pour le fine-tuning
model_fe.compile(
    optimizer=keras.optimizers.Adam(1e-5),   # LR 100x plus faible
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy']
)
```

### 4.3 Exemple Pratique — Classification de Fleurs

```python
# Chargement EfficientNetB0 — meilleur compromis taille/performance
base_efficientnet = keras.applications.EfficientNetB0(
    weights='imagenet',
    include_top=False,
    input_shape=(224, 224, 3)
)
base_efficientnet.trainable = False

# EfficientNet a sa propre normalisation intégrée
inputs = keras.Input(shape=(224, 224, 3))

# Preprocessing EfficientNet (pas besoin de normaliser manuellement)
x = keras.applications.efficientnet.preprocess_input(inputs)
x = base_efficientnet(x, training=False)
x = keras.layers.GlobalAveragePooling2D()(x)
x = keras.layers.Dense(256, activation='relu')(x)
x = keras.layers.Dropout(0.4)(x)
outputs = keras.layers.Dense(102, activation='softmax')(x)   # Oxford 102 Flowers

model_efficientnet = keras.Model(inputs, outputs)
model_efficientnet.compile(
    optimizer='adam',
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy']
)
print("EfficientNetB0 prêt pour transfer learning")
print(f"Paramètres backbone : {base_efficientnet.count_params():,}")
```

---

## 5. Data Augmentation

### 5.1 ImageDataGenerator (Keras)

```python
from tensorflow.keras.preprocessing.image import ImageDataGenerator

# Augmentation du training set
train_gen = ImageDataGenerator(
    rotation_range=20,          # rotation aléatoire ±20°
    width_shift_range=0.2,      # translation horizontale
    height_shift_range=0.2,     # translation verticale
    horizontal_flip=True,       # miroir horizontal
    vertical_flip=False,        # pas de flip vertical pour les photos naturelles
    zoom_range=0.2,             # zoom aléatoire
    shear_range=0.1,            # cisaillement
    brightness_range=[0.8, 1.2],  # variation de luminosité
    fill_mode='nearest'         # remplissage des pixels nouveaux
)

# Pas d'augmentation sur le test set !
test_gen = ImageDataGenerator()

# Chargement depuis un dossier
# train_generator = train_gen.flow_from_directory(
#     'data/train/',
#     target_size=(224, 224),
#     batch_size=32,
#     class_mode='categorical'
# )

# Augmentation inline (Keras layers — préféré en TF2)
data_augmentation = keras.Sequential([
    keras.layers.RandomFlip('horizontal'),
    keras.layers.RandomRotation(0.1),
    keras.layers.RandomZoom(0.1),
    keras.layers.RandomContrast(0.1),
    keras.layers.RandomBrightness(0.1),
], name='data_augmentation')
```

### 5.2 Albumentations — Plus Puissant

```python
# pip install albumentations
try:
    import albumentations as A
    
    # Pipeline d'augmentation avancée
    transform = A.Compose([
        A.RandomRotate90(p=0.5),
        A.Flip(p=0.5),
        A.Transpose(p=0.2),
        
        # Augmentations photométriques
        A.OneOf([
            A.MotionBlur(blur_limit=5),
            A.MedianBlur(blur_limit=5),
            A.GaussianBlur(blur_limit=5),
        ], p=0.2),
        
        A.OneOf([
            A.CLAHE(clip_limit=2),
            A.RandomBrightnessContrast(),
            A.HueSaturationValue(),
        ], p=0.3),
        
        # Distorsions géométriques
        A.ShiftScaleRotate(shift_limit=0.0625, scale_limit=0.1,
                           rotate_limit=45, p=0.3),
        A.ElasticTransform(p=0.1),
        
        # Normalisation finale
        A.Normalize(mean=[0.485, 0.456, 0.406],
                    std=[0.229, 0.224, 0.225]),
    ])
    print("Albumentations disponible")
except ImportError:
    print("Albumentations non installé : pip install albumentations")
```

---

## 6. Détection d'Objets — YOLO

### 6.1 Concepts Clés

```
Bounding Box = (x_center, y_center, width, height)
               (normalisé par la taille de l'image : 0 à 1)

Détection YOLO :
  1. Diviser l'image en grille S×S
  2. Chaque cellule prédit B boîtes englobantes
  3. Chaque boîte : (x, y, w, h, confidence) + C probabilités de classe
```

#### Intersection over Union (IoU)

$$\text{IoU} = \frac{\text{Aire}(A \cap B)}{\text{Aire}(A \cup B)} \in [0, 1]$$

```python
def compute_iou(box1, box2):
    """
    Calcule l'IoU entre deux boîtes englobantes.
    Format : [x_min, y_min, x_max, y_max]
    """
    # Intersection
    x_left   = max(box1[0], box2[0])
    y_top    = max(box1[1], box2[1])
    x_right  = min(box1[2], box2[2])
    y_bottom = min(box1[3], box2[3])
    
    if x_right < x_left or y_bottom < y_top:
        return 0.0   # pas d'intersection
    
    intersection = (x_right - x_left) * (y_bottom - y_top)
    
    # Union
    area1 = (box1[2] - box1[0]) * (box1[3] - box1[1])
    area2 = (box2[2] - box2[0]) * (box2[3] - box2[1])
    union = area1 + area2 - intersection
    
    return intersection / union

# Test
box_pred  = [50, 50, 200, 200]    # boîte prédite
box_truth = [70, 70, 220, 210]    # boîte réelle (ground truth)
iou = compute_iou(box_pred, box_truth)
print(f"IoU = {iou:.4f}")   # 0 = aucun overlap, 1 = parfait
```

#### Non-Maximum Suppression (NMS)

```python
def non_maximum_suppression(boxes, scores, iou_threshold=0.5):
    """
    Supprimer les boîtes redondantes.
    Garder seulement la boîte avec le meilleur score dans chaque région.
    """
    indices = np.argsort(scores)[::-1]   # tri par score décroissant
    kept = []
    
    while len(indices) > 0:
        best = indices[0]
        kept.append(best)
        
        # Calculer IoU avec toutes les boîtes restantes
        rest = indices[1:]
        ious = [compute_iou(boxes[best], boxes[i]) for i in rest]
        
        # Garder seulement celles avec IoU < threshold
        indices = rest[np.array(ious) < iou_threshold]
    
    return kept

boxes = np.array([[100, 100, 200, 200],
                  [110, 105, 210, 205],   # redondant avec le premier
                  [300, 300, 400, 400],
                  [305, 295, 405, 395]])  # redondant avec le troisième
scores = np.array([0.9, 0.75, 0.85, 0.7])

kept_idx = non_maximum_suppression(boxes, scores, iou_threshold=0.5)
print(f"Boîtes retenues : {kept_idx}")
print(f"Scores : {scores[kept_idx]}")
```

### 6.2 Utiliser YOLO avec Ultralytics

```python
# pip install ultralytics
try:
    from ultralytics import YOLO
    
    # Charger YOLOv8 pré-entraîné
    model = YOLO('yolov8n.pt')   # 'n' = nano (le plus léger)
    
    # Inférence sur une image
    # results = model('path/to/image.jpg')
    # results[0].show()   # afficher les détections
    # results[0].boxes   # coordonnées des boîtes
    # results[0].boxes.cls   # classes détectées
    # results[0].boxes.conf  # confidences
    
    # Fine-tuning sur dataset personnalisé
    # model.train(data='custom_dataset.yaml', epochs=50, imgsz=640)
    
    print("YOLO disponible via ultralytics")
except ImportError:
    print("ultralytics non installé : pip install ultralytics")
```

---

## 7. Le Mécanisme d'Attention

### 7.1 Self-Attention

L'attention calcule pour chaque élément d'une séquence une **combinaison pondérée** de tous les autres éléments. Contrairement aux CNNs (réceptif local), l'attention a un **champ réceptif global**.

$$\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d_k}}\right)V$$

- $Q$ (Query) : "je cherche"
- $K$ (Key) : "voici ce que je suis"
- $V$ (Value) : "voici ce que je fournis"
- $\sqrt{d_k}$ : facteur de scaling pour éviter des produits scalaires trop grands

```python
def scaled_dot_product_attention(Q, K, V, mask=None):
    """
    Q : (batch, seq_q, d_k)
    K : (batch, seq_k, d_k)
    V : (batch, seq_k, d_v)
    """
    d_k = Q.shape[-1]
    
    # Scores : (batch, seq_q, seq_k)
    scores = Q @ K.transpose(0, 2, 1) / np.sqrt(d_k)
    
    if mask is not None:
        scores = np.where(mask, -1e9, scores)   # masquer les positions padding
    
    # Attention weights
    def softmax_axis(x, axis=-1):
        e = np.exp(x - x.max(axis=axis, keepdims=True))
        return e / e.sum(axis=axis, keepdims=True)
    
    attn_weights = softmax_axis(scores)    # (batch, seq_q, seq_k)
    
    # Context
    context = attn_weights @ V             # (batch, seq_q, d_v)
    return context, attn_weights

# Exemple : attention sur une séquence de 5 tokens
batch_size, seq_len, d_k, d_v = 2, 5, 64, 64
Q = np.random.randn(batch_size, seq_len, d_k)
K = np.random.randn(batch_size, seq_len, d_k)
V = np.random.randn(batch_size, seq_len, d_v)

context, weights = scaled_dot_product_attention(Q, K, V)
print(f"Q, K, V shapes : {Q.shape}")
print(f"Context shape  : {context.shape}")
print(f"Weights shape  : {weights.shape}")
print(f"Weights sum    : {weights[0].sum(axis=-1)}")   # doit être ≈ [1, 1, 1, 1, 1]
```

### 7.2 Multi-Head Attention

```python
import tensorflow as tf
from tensorflow import keras

class MultiHeadAttention(keras.layers.Layer):
    def __init__(self, d_model, num_heads, **kwargs):
        super().__init__(**kwargs)
        self.num_heads = num_heads
        self.d_model = d_model
        assert d_model % num_heads == 0
        
        self.d_k = d_model // num_heads
        
        self.W_q = keras.layers.Dense(d_model)
        self.W_k = keras.layers.Dense(d_model)
        self.W_v = keras.layers.Dense(d_model)
        self.W_o = keras.layers.Dense(d_model)
    
    def split_heads(self, x, batch_size):
        """(batch, seq, d_model) → (batch, num_heads, seq, d_k)"""
        x = tf.reshape(x, (batch_size, -1, self.num_heads, self.d_k))
        return tf.transpose(x, perm=[0, 2, 1, 3])
    
    def call(self, query, key, value, mask=None, training=None):
        batch_size = tf.shape(query)[0]
        
        Q = self.split_heads(self.W_q(query), batch_size)
        K = self.split_heads(self.W_k(key), batch_size)
        V = self.split_heads(self.W_v(value), batch_size)
        
        # Attention scalée
        scores = tf.matmul(Q, K, transpose_b=True) / tf.math.sqrt(
            tf.cast(self.d_k, tf.float32))
        if mask is not None:
            scores += mask * -1e9
        
        weights = tf.nn.softmax(scores, axis=-1)
        context = tf.matmul(weights, V)
        
        # Recombiner les têtes
        context = tf.transpose(context, perm=[0, 2, 1, 3])
        context = tf.reshape(context, (batch_size, -1, self.d_model))
        
        return self.W_o(context), weights

# Test
mha = MultiHeadAttention(d_model=128, num_heads=8)
x = tf.random.normal((4, 20, 128))   # (batch=4, seq=20, d_model=128)
output, weights = mha(x, x, x)
print(f"Multi-Head Attention sortie : {output.shape}")
print(f"Poids d'attention : {weights.shape}")
```

---

## 8. Architecture Transformer

### 8.1 Encodeur Transformer

```python
class TransformerEncoderBlock(keras.layers.Layer):
    def __init__(self, d_model, num_heads, d_ff, dropout_rate=0.1, **kwargs):
        super().__init__(**kwargs)
        self.mha = MultiHeadAttention(d_model, num_heads)
        
        # Feed-Forward Network
        self.ffn = keras.Sequential([
            keras.layers.Dense(d_ff, activation='relu'),
            keras.layers.Dense(d_model)
        ])
        
        # Layer Normalization
        self.layernorm1 = keras.layers.LayerNormalization(epsilon=1e-6)
        self.layernorm2 = keras.layers.LayerNormalization(epsilon=1e-6)
        
        self.dropout1 = keras.layers.Dropout(dropout_rate)
        self.dropout2 = keras.layers.Dropout(dropout_rate)
    
    def call(self, x, training=None, mask=None):
        # Self-attention + residual
        attn_out, _ = self.mha(x, x, x, mask=mask, training=training)
        attn_out = self.dropout1(attn_out, training=training)
        out1 = self.layernorm1(x + attn_out)         # Pre-norm ou Post-norm
        
        # FFN + residual
        ffn_out = self.ffn(out1)
        ffn_out = self.dropout2(ffn_out, training=training)
        out2 = self.layernorm2(out1 + ffn_out)
        
        return out2

def get_positional_encoding(max_len, d_model):
    """Encodage positionnel sinusoïdal (Vaswani et al. 2017)."""
    positions = np.arange(max_len)[:, np.newaxis]    # (max_len, 1)
    dims = np.arange(d_model)[np.newaxis, :]          # (1, d_model)
    
    angles = positions / np.power(10000, (2 * (dims // 2)) / d_model)
    
    # sin pour les indices pairs, cos pour les impairs
    angles[:, 0::2] = np.sin(angles[:, 0::2])
    angles[:, 1::2] = np.cos(angles[:, 1::2])
    
    return tf.cast(angles[np.newaxis, :, :], dtype=tf.float32)  # (1, max_len, d_model)

# Visualisation positional encoding
pe = get_positional_encoding(100, 128).numpy()[0]
print(f"Positional Encoding shape : {pe.shape}")
```

---

## 9. BERT — Fine-Tuning pour la Classification de Texte

```python
# pip install transformers
try:
    from transformers import (BertTokenizer, TFBertForSequenceClassification,
                               BertConfig)
    import tensorflow as tf
    
    # Charger le tokenizer BERT pré-entraîné
    tokenizer = BertTokenizer.from_pretrained('bert-base-uncased')
    
    # Tokenisation
    texts = ["This movie is absolutely amazing!",
             "Terrible film, complete waste of time.",
             "It was okay, nothing special."]
    
    encoding = tokenizer(
        texts,
        max_length=128,
        padding='max_length',
        truncation=True,
        return_tensors='tf'
    )
    
    print(f"Input IDs shape : {encoding['input_ids'].shape}")
    print(f"Attention mask  : {encoding['attention_mask'].shape}")
    
    # Charger BERT pour classification binaire (positif/négatif)
    model_bert = TFBertForSequenceClassification.from_pretrained(
        'bert-base-uncased',
        num_labels=2
    )
    
    # Fine-tuning
    optimizer = tf.keras.optimizers.Adam(learning_rate=2e-5)
    model_bert.compile(
        optimizer=optimizer,
        loss=model_bert.hf_compute_loss,
        metrics=['accuracy']
    )
    
    print("BERT prêt pour fine-tuning")
    print(f"Paramètres : {model_bert.count_params():,}")

except ImportError:
    print("transformers non installé : pip install transformers")
```

---

## 10. Vision Transformer (ViT)

### 10.1 Principe : Découper l'Image en Patches

```python
def patchify(images, patch_size=16):
    """
    Découper les images en patches.
    images : (batch, H, W, C)
    Retourne : (batch, n_patches, patch_size*patch_size*C)
    """
    batch, H, W, C = images.shape
    n_patches_h = H // patch_size
    n_patches_w = W // patch_size
    n_patches = n_patches_h * n_patches_w
    
    # Reshape en patches
    x = images.reshape(batch, n_patches_h, patch_size, n_patches_w, patch_size, C)
    x = x.transpose(0, 1, 3, 2, 4, 5)   # (batch, nh, nw, ph, pw, C)
    x = x.reshape(batch, n_patches, patch_size * patch_size * C)
    return x

# Démonstration
images = np.random.randn(4, 224, 224, 3).astype(np.float32)
patches = patchify(images, patch_size=16)
print(f"Images shape  : {images.shape}")
print(f"Patches shape : {patches.shape}")   # (4, 196, 768)
# 224/16 = 14 → 14×14 = 196 patches, chacun de 16×16×3 = 768 dimensions

# Mini-ViT avec Keras
class ViT_Mini(keras.Model):
    def __init__(self, image_size=224, patch_size=16, d_model=128,
                 num_heads=4, n_layers=4, n_classes=10, **kwargs):
        super().__init__(**kwargs)
        self.patch_size = patch_size
        n_patches = (image_size // patch_size) ** 2
        patch_dim = patch_size * patch_size * 3
        
        # Projection linéaire des patches
        self.patch_embedding = keras.layers.Dense(d_model)
        
        # Token [CLS] apprenable
        self.cls_token = self.add_weight('cls_token', shape=(1, 1, d_model))
        
        # Positional embedding
        self.pos_embedding = self.add_weight(
            'pos_embedding', shape=(1, n_patches + 1, d_model)
        )
        
        # Transformer encoders
        self.transformer_blocks = [
            TransformerEncoderBlock(d_model, num_heads, d_model * 4)
            for _ in range(n_layers)
        ]
        
        self.norm = keras.layers.LayerNormalization()
        self.classifier = keras.layers.Dense(n_classes, activation='softmax')
    
    def call(self, x, training=None):
        batch_size = tf.shape(x)[0]
        
        # Patchification et embedding
        # (batch, H, W, C) → (batch, n_patches, patch_dim)
        patches = tf.image.extract_patches(
            x,
            sizes=[1, self.patch_size, self.patch_size, 1],
            strides=[1, self.patch_size, self.patch_size, 1],
            rates=[1, 1, 1, 1],
            padding='VALID'
        )
        patches = tf.reshape(patches, (batch_size, -1, patches.shape[-1]))
        x = self.patch_embedding(patches)
        
        # Ajouter le token [CLS]
        cls_tokens = tf.repeat(self.cls_token, batch_size, axis=0)
        x = tf.concat([cls_tokens, x], axis=1)
        
        # Positional embedding
        x = x + self.pos_embedding
        
        # Transformer
        for block in self.transformer_blocks:
            x = block(x, training=training)
        
        x = self.norm(x)
        
        # Extraire le token [CLS] pour la classification
        cls_output = x[:, 0, :]
        return self.classifier(cls_output)

print("ViT Mini prêt.")
```

---

## 11. Comparaison CNN vs Transformer pour la Vision

| Aspect | CNN (ResNet, EfficientNet) | ViT |
|--------|---------------------------|-----|
| **Données** | Fonctionne bien avec peu de données | Nécessite beaucoup de données |
| **Inductive bias** | Localité + équivariance translation | Aucun → apprend tout |
| **Champ réceptif** | Local (dépend de la profondeur) | Global dès la 1ère couche |
| **Complexité** | $O(HWC^2)$ | $O(n^2 d)$ (quadratique en séquence) |
| **Performance SOTA** | Très bonne avec TL | Meilleure sur grand dataset |
| **Interprétabilité** | Activation maps | Attention maps |

---

## 12. Exercices Pratiques

> [!tip] Exercice 1 — Filtres de Convolution
> Implémentez la convolution 2D en NumPy (comme dans le cours). Appliquez les filtres Sobel horizontal, vertical, et diagonal sur une image chargée avec `PIL`. Affichez les feature maps côte à côte. Quel filtre détecte le mieux les bords ?

> [!tip] Exercice 2 — CNN CIFAR-10
> Entraînez le CNN du cours sur CIFAR-10. Puis ajoutez de la data augmentation (`RandomFlip`, `RandomRotation`, `RandomZoom`). Comparez l'accuracy test avec et sans augmentation après 30 epochs.

> [!tip] Exercice 3 — Transfer Learning Flowers
> Utilisez le dataset `tf_flowers` (disponible via `tensorflow_datasets`). Chargez EfficientNetB0 pré-entraîné sur ImageNet. Phase 1 : feature extraction (backbone gelé) — 10 epochs. Phase 2 : fine-tuning des 30 dernières couches avec lr=1e-5 — 20 epochs. Comparez les performances des deux phases.

> [!tip] Exercice 4 — IoU et NMS
> Implémentez les fonctions `compute_iou` et `non_maximum_suppression`. Générez 20 boîtes aléatoires avec des scores aléatoires. Appliquez NMS avec threshold=0.4 et 0.7. Visualisez les boîtes avant et après NMS avec Matplotlib.

> [!tip] Exercice 5 — Attention Visualization
> Fine-tunez BERT sur le dataset IMDb (`tensorflow_datasets` ou Kaggle). Extrayez les poids d'attention de la dernière couche et visualisez-les sous forme de heatmap pour une phrase positive et une phrase négative. Quels mots reçoivent le plus d'attention pour chaque sentiment ?

---

## Liens

- [[04 - Reseaux de Neurones et Deep Learning]] — bases des réseaux de neurones
- [[06 - ML Non Supervise et Reinforcement Learning]] — VAE et GAN
- [[03 - ML Supervise avec Scikit-Learn]] — évaluation des modèles

> [!info] Ressources
> - "Attention Is All You Need" (Vaswani et al., 2017) — article original Transformer
> - "An Image is Worth 16x16 Words" (Dosovitskiy et al., 2020) — article ViT
> - Stanford CS231n : Computer Vision — http://cs231n.stanford.edu/
> - Hugging Face Transformers : https://huggingface.co/docs/transformers/
> - YOLO Ultralytics : https://docs.ultralytics.com/
