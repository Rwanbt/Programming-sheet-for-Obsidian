# 06 - ML Non Supervisé et Reinforcement Learning

> [!info] Objectif du cours
> Maîtriser les techniques d'apprentissage sans labels : clustering, réduction de dimensionnalité, autoencoders, GANs — et les bases du Reinforcement Learning avec Q-Learning et Deep Q-Networks. Ce cours complète [[04 - Reseaux de Neurones et Deep Learning]] et s'appuie sur [[02 - NumPy Pandas et Visualisation]].

---

## 1. Clustering

### 1.1 K-Means

K-Means partitionne les données en $k$ clusters en minimisant la variance intra-cluster :

$$J = \sum_{i=1}^{k} \sum_{x \in C_i} \|x - \mu_i\|^2$$

**Algorithme :**
1. Initialiser $k$ centroïdes aléatoirement
2. Assigner chaque point au centroïde le plus proche
3. Recalculer les centroïdes (moyenne des points du cluster)
4. Répéter jusqu'à convergence

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from sklearn.cluster import KMeans, DBSCAN, AgglomerativeClustering
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import silhouette_score, silhouette_samples
from sklearn.datasets import make_blobs, make_moons, load_iris
import warnings
warnings.filterwarnings('ignore')

# Implémentation K-Means from scratch
class KMeans_Scratch:
    def __init__(self, n_clusters=3, max_iter=100, random_state=42):
        self.k = n_clusters
        self.max_iter = max_iter
        self.random_state = random_state
    
    def fit(self, X):
        np.random.seed(self.random_state)
        n, d = X.shape
        
        # Initialisation : choisir k points aléatoires comme centroïdes
        idx = np.random.choice(n, self.k, replace=False)
        self.centroids = X[idx].copy()
        
        for iteration in range(self.max_iter):
            old_centroids = self.centroids.copy()
            
            # Assigner chaque point au centroïde le plus proche
            distances = np.linalg.norm(X[:, np.newaxis, :] - self.centroids, axis=2)
            self.labels = np.argmin(distances, axis=1)
            
            # Recalculer les centroïdes
            for k in range(self.k):
                mask = self.labels == k
                if mask.sum() > 0:
                    self.centroids[k] = X[mask].mean(axis=0)
            
            # Vérifier la convergence
            if np.allclose(old_centroids, self.centroids, atol=1e-6):
                print(f"  Convergé en {iteration+1} itérations")
                break
        
        # Calculer l'inertie (somme des distances au carré)
        self.inertia = sum(
            np.linalg.norm(X[self.labels == k] - self.centroids[k])**2
            for k in range(self.k)
        )
        return self
    
    def predict(self, X):
        distances = np.linalg.norm(X[:, np.newaxis, :] - self.centroids, axis=2)
        return np.argmin(distances, axis=1)

# Dataset de test
X_blobs, y_true = make_blobs(n_samples=400, n_features=2,
                              centers=4, cluster_std=0.8, random_state=42)

kmeans_scratch = KMeans_Scratch(n_clusters=4)
kmeans_scratch.fit(X_blobs)

print(f"K-Means scratch — Inertie : {kmeans_scratch.inertia:.2f}")
print(f"Distribution des clusters : {np.bincount(kmeans_scratch.labels)}")
```

### 1.2 Méthode du Coude (Elbow Method) et Silhouette Score

```python
# Trouver le bon nombre de clusters
inertias = []
silhouettes = []
k_range = range(2, 11)

for k in k_range:
    km = KMeans(n_clusters=k, random_state=42, n_init=10)
    labels = km.fit_predict(X_blobs)
    inertias.append(km.inertia_)
    silhouettes.append(silhouette_score(X_blobs, labels))

fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(14, 5))

ax1.plot(k_range, inertias, 'o-', color='blue', linewidth=2)
ax1.set_title('Méthode du Coude — Inertie')
ax1.set_xlabel('Nombre de clusters k')
ax1.set_ylabel('Inertie')
ax1.axvline(x=4, color='red', linestyle='--', label='k optimal')
ax1.legend()
ax1.grid(True)

ax2.plot(k_range, silhouettes, 's-', color='orange', linewidth=2)
ax2.set_title('Silhouette Score')
ax2.set_xlabel('Nombre de clusters k')
ax2.set_ylabel('Silhouette Score')
ax2.axvline(x=4, color='red', linestyle='--', label='k optimal')
ax2.legend()
ax2.grid(True)

plt.tight_layout()
plt.savefig('elbow_silhouette.png', dpi=150)
plt.close()

best_k = list(k_range)[np.argmax(silhouettes)]
print(f"Meilleur k par silhouette : {best_k} (score={max(silhouettes):.4f})")
```

> [!info] Silhouette Score
> $$s(i) = \frac{b(i) - a(i)}{\max(a(i), b(i))} \in [-1, 1]$$
> - $a(i)$ : distance moyenne aux autres points du **même** cluster
> - $b(i)$ : distance moyenne aux points du **cluster le plus proche**
> - **Proche de 1** = bon clustering, **0** = frontière, **négatif** = mauvais

### 1.3 DBSCAN — Clustering Basé sur la Densité

DBSCAN (Density-Based Spatial Clustering) ne nécessite pas de spécifier $k$ et détecte les clusters de forme arbitraire.

| Paramètre | Rôle |
|-----------|------|
| `eps` | Rayon du voisinage |
| `min_samples` | Nombre minimum de points dans le voisinage pour être "core point" |

```python
# K-Means vs DBSCAN sur des données en forme de lune
X_moons, y_moons = make_moons(n_samples=400, noise=0.1, random_state=42)

# K-Means échoue sur des formes non-convexes
km_moons = KMeans(n_clusters=2, random_state=42)
labels_km = km_moons.fit_predict(X_moons)

# DBSCAN réussit
dbscan = DBSCAN(eps=0.2, min_samples=5)
labels_db = dbscan.fit_predict(X_moons)

print(f"K-Means silhouette    : {silhouette_score(X_moons, labels_km):.4f}")
print(f"DBSCAN silhouette     : {silhouette_score(X_moons, labels_db[labels_db >= 0]):.4f}")
print(f"DBSCAN clusters trouvés : {len(np.unique(labels_db[labels_db >= 0]))}")
print(f"DBSCAN outliers         : {(labels_db == -1).sum()} points (-1 = bruit)")

fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(12, 5))
for ax, labels, title in [(ax1, labels_km, 'K-Means (k=2)'),
                            (ax2, labels_db, 'DBSCAN (eps=0.2)')]:
    ax.scatter(X_moons[:, 0], X_moons[:, 1], c=labels, cmap='Set1', s=30, alpha=0.8)
    ax.set_title(title)
    ax.grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig('kmeans_vs_dbscan.png', dpi=150)
plt.close()
```

### 1.4 Clustering Hiérarchique

```python
from sklearn.cluster import AgglomerativeClustering
from scipy.cluster.hierarchy import dendrogram, linkage

X_iris, y_iris = load_iris(return_X_y=True)

# Dendrogramme — visualiser la hiérarchie
linkage_matrix = linkage(X_iris, method='ward')

fig, ax = plt.subplots(figsize=(12, 5))
dendrogram(linkage_matrix, ax=ax, truncate_mode='level', p=5,
           color_threshold=8)
ax.set_title('Dendrogramme — Dataset Iris')
ax.set_xlabel('Échantillons')
ax.set_ylabel('Distance Ward')
plt.tight_layout()
plt.savefig('dendrogram_iris.png', dpi=150)
plt.close()

# Comparaison des méthodes de clustering
print("\nComparaison clustering sur Iris :")
print(f"{'Méthode':<25} {'Silhouette':>12}")
print("-" * 38)
for name, model in [
    ("K-Means (k=3)",           KMeans(n_clusters=3, random_state=42, n_init=10)),
    ("DBSCAN (eps=0.5)",        DBSCAN(eps=0.5, min_samples=5)),
    ("Agglomerative Ward",      AgglomerativeClustering(n_clusters=3, linkage='ward')),
    ("Agglomerative Complete",  AgglomerativeClustering(n_clusters=3, linkage='complete'))
]:
    labels = model.fit_predict(StandardScaler().fit_transform(X_iris))
    valid = labels[labels >= 0]
    if len(np.unique(valid)) > 1:
        score = silhouette_score(X_iris[labels >= 0], valid)
        print(f"{name:<25} {score:>12.4f}")
```

---

## 2. Réduction de Dimensionnalité

### 2.1 PCA — Analyse en Composantes Principales

PCA trouve les directions de variance maximale dans les données (vecteurs propres de la matrice de covariance).

$$X_{\text{réduit}} = X_{\text{centré}} \cdot V_k^T$$

où $V_k$ contient les $k$ premiers vecteurs propres.

```python
from sklearn.decomposition import PCA
from sklearn.preprocessing import StandardScaler

# Dataset : visages MNIST (784 dimensions → 2D)
from sklearn.datasets import load_digits
digits = load_digits()
X_dig = digits.data    # (1797, 64)
y_dig = digits.target

# PCA mathématique
X_centered = X_dig - X_dig.mean(axis=0)
Sigma = np.cov(X_centered.T)    # matrice de covariance (64, 64)
eigenvalues, eigenvectors = np.linalg.eig(Sigma)
idx = np.argsort(eigenvalues.real)[::-1]
eigenvalues = eigenvalues.real[idx]

# Variance expliquée cumulée
var_exp = eigenvalues / eigenvalues.sum()
cumsum_var = np.cumsum(var_exp)
n_components_95 = np.argmax(cumsum_var >= 0.95) + 1
print(f"Composantes pour 95% de variance : {n_components_95} (sur 64)")

# PCA Scikit-Learn
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X_dig)

pca_full = PCA()
pca_full.fit(X_scaled)

pca_2d = PCA(n_components=2, random_state=42)
X_pca_2d = pca_2d.fit_transform(X_scaled)

print(f"\nPCA 2D — Variance expliquée : {pca_2d.explained_variance_ratio_.sum()*100:.1f}%")

# Visualisation 2D
fig, ax = plt.subplots(figsize=(10, 8))
scatter = ax.scatter(X_pca_2d[:, 0], X_pca_2d[:, 1],
                      c=y_dig, cmap='tab10', alpha=0.7, s=20)
plt.colorbar(scatter, ax=ax, label='Classe')
ax.set_title('PCA 2D — Digits (MNIST)')
ax.set_xlabel(f'PC1 ({pca_2d.explained_variance_ratio_[0]*100:.1f}%)')
ax.set_ylabel(f'PC2 ({pca_2d.explained_variance_ratio_[1]*100:.1f}%)')
plt.tight_layout()
plt.savefig('pca_digits.png', dpi=150)
plt.close()

# Reconstruction depuis PCA (compression avec perte)
pca_32 = PCA(n_components=32)
X_compressed = pca_32.fit_transform(X_scaled)    # (1797, 32)
X_reconstructed = pca_32.inverse_transform(X_compressed)
X_reconstructed = scaler.inverse_transform(X_reconstructed)

mse = np.mean((X_dig - X_reconstructed)**2)
print(f"\nReconstruction avec 32 composantes :")
print(f"  MSE : {mse:.4f}")
print(f"  Ratio de compression : {64/32:.1f}x")
```

### 2.2 t-SNE — Visualisation Non-Linéaire

t-SNE préserve les **structures locales** et révèle les clusters en 2D. Contrairement à PCA, les axes n'ont pas de signification.

> [!warning] t-SNE — Limitations
> - **Non déterministe** (utiliser `random_state`)
> - **Lent** sur grandes données (O(n²)) → utiliser sur un sous-ensemble
> - **Pas utilisable pour la projection de nouveaux points** (pas de `transform()`)
> - Les distances **entre** clusters n'ont pas de signification

```python
from sklearn.manifold import TSNE

# Sous-ensemble pour la vitesse
n_samples = 1000
idx = np.random.choice(len(X_dig), n_samples, replace=False)
X_sub, y_sub = X_dig[idx], y_dig[idx]

X_sub_scaled = StandardScaler().fit_transform(X_sub)

# t-SNE 2D
tsne = TSNE(
    n_components=2,
    perplexity=30,       # balance local/global (5-50)
    n_iter=1000,         # nombre d'itérations
    learning_rate='auto',
    random_state=42,
    verbose=0
)
X_tsne = tsne.fit_transform(X_sub_scaled)
print(f"t-SNE — shape résultat : {X_tsne.shape}")

fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(14, 6))

# PCA
pca_sub = PCA(n_components=2).fit_transform(X_sub_scaled)
ax1.scatter(pca_sub[:, 0], pca_sub[:, 1], c=y_sub, cmap='tab10', s=20, alpha=0.7)
ax1.set_title('PCA 2D')

# t-SNE
ax2.scatter(X_tsne[:, 0], X_tsne[:, 1], c=y_sub, cmap='tab10', s=20, alpha=0.7)
ax2.set_title('t-SNE')

plt.tight_layout()
plt.savefig('pca_vs_tsne.png', dpi=150)
plt.close()
print("t-SNE révèle des clusters plus séparés que PCA pour les données complexes")
```

### 2.3 UMAP — Plus Rapide et Préserve la Structure Globale

```python
# pip install umap-learn
try:
    import umap
    
    reducer = umap.UMAP(
        n_components=2,
        n_neighbors=15,     # balance local/global (5-100)
        min_dist=0.1,       # compacité des clusters (0-1)
        random_state=42
    )
    X_umap = reducer.fit_transform(X_sub_scaled)
    
    print(f"UMAP — shape résultat : {X_umap.shape}")
    print("UMAP avantages vs t-SNE :")
    print("  ✓ Plus rapide (O(n log n))")
    print("  ✓ Préserve mieux la structure globale")
    print("  ✓ Supporte transform() pour de nouveaux points")
    print("  ✓ Applicable à la réduction de haute dimension (>2)")
    
except ImportError:
    print("UMAP non installé : pip install umap-learn")
```

---

## 3. Autoencoders

### 3.1 Architecture Encoder-Decoder

Un autoencoder apprend à **compresser** l'entrée en un espace latent ($z$) puis à le **reconstruire**.

```
Entrée x → [Encoder] → z (espace latent) → [Decoder] → x̂ (reconstruction)
Loss = ||x - x̂||² (reconstruction loss)
```

```python
import tensorflow as tf
from tensorflow import keras

# Autoencoder sur MNIST
(X_train, _), (X_test, _) = keras.datasets.mnist.load_data()
X_train = X_train.reshape(-1, 784).astype('float32') / 255.0
X_test  = X_test.reshape(-1, 784).astype('float32') / 255.0

# Encoder
encoder_input = keras.Input(shape=(784,), name='image')
x = keras.layers.Dense(256, activation='relu')(encoder_input)
x = keras.layers.Dense(128, activation='relu')(x)
latent = keras.layers.Dense(32, activation='relu', name='latent')(x)   # 32D

# Decoder
x = keras.layers.Dense(128, activation='relu')(latent)
x = keras.layers.Dense(256, activation='relu')(x)
decoder_output = keras.layers.Dense(784, activation='sigmoid', name='reconstruction')(x)

# Modèles séparés pour Encoder et Decoder
autoencoder = keras.Model(encoder_input, decoder_output, name='Autoencoder')
encoder_model = keras.Model(encoder_input, latent, name='Encoder')

autoencoder.compile(
    optimizer='adam',
    loss='mse'   # binary_crossentropy aussi possible pour images binaires
)
autoencoder.summary()

# Entraînement : entrée = sortie (reconstruction)
history = autoencoder.fit(
    X_train, X_train,     # x = y (autoencoder non supervisé)
    epochs=30,
    batch_size=256,
    validation_data=(X_test, X_test),
    callbacks=[keras.callbacks.EarlyStopping(patience=5)],
    verbose=0
)

# Évaluation
reconstructions = autoencoder.predict(X_test[:10], verbose=0)
mse = np.mean((X_test[:10] - reconstructions)**2)
print(f"\nAutoencoder MNIST :")
print(f"  MSE de reconstruction : {mse:.6f}")
print(f"  Compression : 784 → 32 dimensions ({784/32:.1f}x)")

# Visualisation
fig, axes = plt.subplots(2, 10, figsize=(20, 4))
for i in range(10):
    axes[0, i].imshow(X_test[i].reshape(28, 28), cmap='gray')
    axes[0, i].axis('off')
    axes[1, i].imshow(reconstructions[i].reshape(28, 28), cmap='gray')
    axes[1, i].axis('off')
axes[0, 0].set_ylabel('Original', fontsize=12)
axes[1, 0].set_ylabel('Reconstruit', fontsize=12)
plt.savefig('autoencoder_reconstructions.png', dpi=150, bbox_inches='tight')
plt.close()
```

### 3.2 Dénoising Autoencoder

```python
# Ajouter du bruit à l'entrée, apprendre à dénoiser
noise_factor = 0.3
X_train_noisy = X_train + noise_factor * np.random.randn(*X_train.shape)
X_train_noisy = np.clip(X_train_noisy, 0, 1)
X_test_noisy = X_test + noise_factor * np.random.randn(*X_test.shape)
X_test_noisy  = np.clip(X_test_noisy, 0, 1)

denoising_ae = keras.Model(encoder_input, decoder_output, name='DenoisingAE')
denoising_ae.compile(optimizer='adam', loss='mse')

# Entrée bruitée → sortie propre
denoising_ae.fit(
    X_train_noisy, X_train,    # bruitée → originale
    epochs=20, batch_size=256,
    validation_data=(X_test_noisy, X_test),
    verbose=0
)
print("Denoising Autoencoder entraîné")
```

### 3.3 Variational Autoencoder (VAE)

Le VAE ajoute une contrainte probabiliste sur l'espace latent : $z \sim \mathcal{N}(\mu, \sigma^2)$

**Loss VAE :**
$$\mathcal{L} = \underbrace{\|x - \hat{x}\|^2}_{\text{reconstruction}} + \underbrace{D_{KL}(\mathcal{N}(\mu, \sigma^2) \| \mathcal{N}(0, I))}_{\text{régularisation KL}}$$

```python
class Sampling(keras.layers.Layer):
    """Reparameterization trick : z = μ + ε*σ, ε ~ N(0,1)"""
    def call(self, inputs):
        mu, log_var = inputs
        batch = tf.shape(mu)[0]
        dim   = tf.shape(mu)[1]
        epsilon = tf.random.normal(shape=(batch, dim))
        return mu + tf.exp(0.5 * log_var) * epsilon

# VAE Encoder
vae_input = keras.Input(shape=(784,))
x = keras.layers.Dense(256, activation='relu')(vae_input)
x = keras.layers.Dense(128, activation='relu')(x)
mu      = keras.layers.Dense(16, name='mu')(x)
log_var = keras.layers.Dense(16, name='log_var')(x)
z = Sampling()([mu, log_var])

vae_encoder = keras.Model(vae_input, [mu, log_var, z], name='VAE_Encoder')

# VAE Decoder
latent_input = keras.Input(shape=(16,))
x = keras.layers.Dense(128, activation='relu')(latent_input)
x = keras.layers.Dense(256, activation='relu')(x)
vae_output = keras.layers.Dense(784, activation='sigmoid')(x)

vae_decoder = keras.Model(latent_input, vae_output, name='VAE_Decoder')

# VAE complet avec loss personnalisée
class VAE(keras.Model):
    def __init__(self, encoder, decoder, **kwargs):
        super().__init__(**kwargs)
        self.encoder = encoder
        self.decoder = decoder
        self.reconstruction_loss_tracker = keras.metrics.Mean(name='reconstruction_loss')
        self.kl_loss_tracker = keras.metrics.Mean(name='kl_loss')
    
    @property
    def metrics(self):
        return [self.reconstruction_loss_tracker, self.kl_loss_tracker]
    
    def train_step(self, data):
        with tf.GradientTape() as tape:
            mu, log_var, z = self.encoder(data)
            reconstruction = self.decoder(z)
            
            # Reconstruction loss (MSE ou BCE)
            recon_loss = tf.reduce_mean(
                tf.reduce_sum(keras.losses.binary_crossentropy(data, reconstruction))
            )
            
            # KL Divergence loss
            kl_loss = -0.5 * tf.reduce_mean(
                1 + log_var - tf.square(mu) - tf.exp(log_var)
            )
            
            total_loss = recon_loss + kl_loss
        
        grads = tape.gradient(total_loss, self.trainable_weights)
        self.optimizer.apply_gradients(zip(grads, self.trainable_weights))
        
        self.reconstruction_loss_tracker.update_state(recon_loss)
        self.kl_loss_tracker.update_state(kl_loss)
        return {
            'reconstruction_loss': self.reconstruction_loss_tracker.result(),
            'kl_loss': self.kl_loss_tracker.result()
        }

vae = VAE(vae_encoder, vae_decoder)
vae.compile(optimizer=keras.optimizers.Adam(1e-3))
vae.fit(X_train, epochs=30, batch_size=256, verbose=0)

# Génération : échantillonner depuis l'espace latent
n_generated = 16
z_sample = np.random.normal(0, 1, (n_generated, 16)).astype(np.float32)
generated_images = vae_decoder.predict(z_sample, verbose=0)

print("VAE entraîné — génération depuis l'espace latent possible")
print(f"Images générées shape : {generated_images.shape}")
```

---

## 4. Generative Adversarial Networks (GANs)

### 4.1 Architecture GAN

Un GAN est un jeu entre deux réseaux :
- **Générateur** $G$ : génère de fausses données à partir de bruit $z$
- **Discriminateur** $D$ : distingue les vraies données des fausses

$$\min_G \max_D \mathbb{E}_{x \sim p_{data}}[\log D(x)] + \mathbb{E}_{z \sim p_z}[\log(1 - D(G(z)))]$$

```
Bruit z ~ N(0,I) → [Générateur G] → Image générée G(z)
                                         ↓
Vraie image x   →→→→→→→→→→→→→→→→ [Discriminateur D] → Réel ou Faux ?
```

### 4.2 DCGAN — Deep Convolutional GAN

```python
def build_generator(latent_dim=100):
    """Générateur DCGAN pour images 28x28 (MNIST)."""
    model = keras.Sequential([
        keras.Input(shape=(latent_dim,)),
        
        # Projeter et reshape
        keras.layers.Dense(7 * 7 * 256, use_bias=False),
        keras.layers.BatchNormalization(),
        keras.layers.LeakyReLU(0.2),
        keras.layers.Reshape((7, 7, 256)),
        
        # Upsampling 7x7 → 14x14
        keras.layers.Conv2DTranspose(128, 5, strides=2, padding='same',
                                      use_bias=False),
        keras.layers.BatchNormalization(),
        keras.layers.LeakyReLU(0.2),
        
        # Upsampling 14x14 → 28x28
        keras.layers.Conv2DTranspose(64, 5, strides=2, padding='same',
                                      use_bias=False),
        keras.layers.BatchNormalization(),
        keras.layers.LeakyReLU(0.2),
        
        # Sortie : image 28x28x1
        keras.layers.Conv2DTranspose(1, 5, strides=1, padding='same',
                                      activation='tanh')
    ], name='Generator')
    return model

def build_discriminator():
    """Discriminateur DCGAN pour images 28x28."""
    model = keras.Sequential([
        keras.Input(shape=(28, 28, 1)),
        
        keras.layers.Conv2D(64, 5, strides=2, padding='same'),
        keras.layers.LeakyReLU(0.2),
        keras.layers.Dropout(0.3),
        
        keras.layers.Conv2D(128, 5, strides=2, padding='same'),
        keras.layers.LeakyReLU(0.2),
        keras.layers.Dropout(0.3),
        
        keras.layers.Flatten(),
        keras.layers.Dense(1)   # logit (pas de sigmoid — plus stable)
    ], name='Discriminator')
    return model

generator = build_generator(latent_dim=100)
discriminator = build_discriminator()

print("Générateur :")
generator.summary()
print("\nDiscriminateur :")
discriminator.summary()
```

### 4.3 Boucle d'Entraînement GAN

```python
class DCGAN(keras.Model):
    def __init__(self, generator, discriminator, latent_dim=100):
        super().__init__()
        self.generator = generator
        self.discriminator = discriminator
        self.latent_dim = latent_dim
    
    def compile(self, g_optimizer, d_optimizer, loss_fn):
        super().compile()
        self.g_optimizer = g_optimizer
        self.d_optimizer = d_optimizer
        self.loss_fn = loss_fn
        
        self.d_loss_tracker = keras.metrics.Mean(name='d_loss')
        self.g_loss_tracker = keras.metrics.Mean(name='g_loss')
    
    @property
    def metrics(self):
        return [self.d_loss_tracker, self.g_loss_tracker]
    
    def train_step(self, real_images):
        batch_size = tf.shape(real_images)[0]
        
        # === Entraîner le Discriminateur ===
        z = tf.random.normal((batch_size, self.latent_dim))
        fake_images = self.generator(z, training=False)
        
        with tf.GradientTape() as tape:
            real_logits = self.discriminator(real_images, training=True)
            fake_logits = self.discriminator(fake_images, training=True)
            
            # Loss discriminateur : maximiser log D(x) + log(1 - D(G(z)))
            real_loss = self.loss_fn(tf.ones_like(real_logits), real_logits)
            fake_loss = self.loss_fn(tf.zeros_like(fake_logits), fake_logits)
            d_loss = (real_loss + fake_loss) / 2
        
        grads = tape.gradient(d_loss, self.discriminator.trainable_weights)
        self.d_optimizer.apply_gradients(
            zip(grads, self.discriminator.trainable_weights))
        
        # === Entraîner le Générateur ===
        z = tf.random.normal((batch_size, self.latent_dim))
        with tf.GradientTape() as tape:
            fake_images = self.generator(z, training=True)
            fake_logits = self.discriminator(fake_images, training=False)
            
            # Loss générateur : tromper le discriminateur → minimiser log(1 - D(G(z)))
            g_loss = self.loss_fn(tf.ones_like(fake_logits), fake_logits)
        
        grads = tape.gradient(g_loss, self.generator.trainable_weights)
        self.g_optimizer.apply_gradients(
            zip(grads, self.generator.trainable_weights))
        
        self.d_loss_tracker.update_state(d_loss)
        self.g_loss_tracker.update_state(g_loss)
        return {
            'd_loss': self.d_loss_tracker.result(),
            'g_loss': self.g_loss_tracker.result()
        }

# Préparer les données MNIST
(X_mnist, _), _ = keras.datasets.mnist.load_data()
X_mnist = (X_mnist.reshape(-1, 28, 28, 1).astype('float32') - 127.5) / 127.5  # [-1, 1]
dataset = tf.data.Dataset.from_tensor_slices(X_mnist).shuffle(60000).batch(128)

dcgan = DCGAN(generator, discriminator, latent_dim=100)
dcgan.compile(
    g_optimizer=keras.optimizers.Adam(2e-4, beta_1=0.5),
    d_optimizer=keras.optimizers.Adam(2e-4, beta_1=0.5),
    loss_fn=keras.losses.BinaryCrossentropy(from_logits=True)
)

print("DCGAN compilé — prêt pour l'entraînement")
print("Note : entraîner avec dcgan.fit(dataset, epochs=50)")
```

### 4.4 Mode Collapse et Solutions

> [!warning] Mode Collapse
> Le générateur converge vers quelques exemples répétés (ex. toujours le même chiffre). Symptôme : g_loss très faible, d_loss très faible, mais peu de diversité.

```python
print("Techniques contre le mode collapse :")
print("  1. Spectral Normalization — stabilise les gradients du discriminateur")
print("  2. Wasserstein GAN (WGAN) — meilleure fonction de loss")
print("  3. Mini-batch discrimination — D regarde la diversité du batch")
print("  4. Label smoothing — étiquettes 0.9 au lieu de 1.0")
print("  5. Progressive growing (ProGAN) — commencer en basse résolution")
print("  6. Gradient penalty (WGAN-GP)")

# Applications des GANs
print("\nApplications des GANs :")
print("  • Génération d'images photoréalistes (StyleGAN3)")
print("  • Data augmentation pour datasets rares")
print("  • Super-résolution d'images (SRGAN)")
print("  • Transfert de style artistique (CycleGAN)")
print("  • Deepfakes (détection → usage forensique)")
print("  • Drug discovery (génération de molécules)")
```

---

## 5. Reinforcement Learning

### 5.1 Concepts Fondamentaux

```
Agent ←──── Observation (état s_t) ─────┐
  │                                      │
  ├──── Action a_t ──────────────────→ Environnement
  │                                      │
  └──── Récompense r_t ─────────────────┘
```

| Concept | Définition |
|---------|-----------|
| **Agent** | L'entité qui apprend et décide |
| **Environnement** | Le système avec lequel l'agent interagit |
| **État** $s$ | Représentation actuelle de l'environnement |
| **Action** $a$ | Choix de l'agent dans l'état $s$ |
| **Récompense** $r$ | Signal scalaire de feedback |
| **Politique** $\pi$ | Fonction $s \to a$ (stratégie de l'agent) |
| **Episode** | Séquence d'états-actions jusqu'à l'état terminal |

### 5.2 Processus de Décision Markovien (MDP)

$$P(s_{t+1}, r_t | s_t, a_t) \quad \text{(propriété de Markov : l'état futur ne dépend que du présent)}$$

**Retour cumulatif actualisé :**
$$G_t = r_t + \gamma r_{t+1} + \gamma^2 r_{t+2} + \cdots = \sum_{k=0}^{\infty} \gamma^k r_{t+k}$$

où $\gamma \in [0, 1]$ est le **facteur d'actualisation** (importance du futur).

**Fonction de valeur Q :**
$$Q^*(s, a) = \mathbb{E}\left[G_t | s_t = s, a_t = a\right]$$

La politique optimale : $\pi^*(s) = \arg\max_a Q^*(s, a)$

### 5.3 Q-Learning

L'équation de **Bellman** donne la mise à jour Q-Learning :

$$Q(s, a) \leftarrow Q(s, a) + \alpha \left[r + \gamma \max_{a'} Q(s', a') - Q(s, a)\right]$$

```python
import numpy as np

class QLearningAgent:
    """Q-Learning tabular pour des environnements discrets."""
    
    def __init__(self, n_states, n_actions, lr=0.1, gamma=0.99, epsilon=1.0):
        self.n_states = n_states
        self.n_actions = n_actions
        self.lr = lr           # learning rate
        self.gamma = gamma     # facteur d'actualisation
        self.epsilon = epsilon # exploration (greedy epsilon)
        
        # Table Q initialisée à 0
        self.Q = np.zeros((n_states, n_actions))
    
    def select_action(self, state):
        """Politique epsilon-greedy."""
        if np.random.random() < self.epsilon:
            return np.random.randint(self.n_actions)   # exploration
        else:
            return np.argmax(self.Q[state])             # exploitation
    
    def update(self, state, action, reward, next_state, done):
        """Mise à jour Q selon l'équation de Bellman."""
        target = reward
        if not done:
            target += self.gamma * np.max(self.Q[next_state])
        
        self.Q[state, action] += self.lr * (target - self.Q[state, action])
    
    def decay_epsilon(self, epsilon_min=0.01, decay=0.995):
        """Réduire l'exploration au cours du temps."""
        self.epsilon = max(epsilon_min, self.epsilon * decay)


# Environnement simple : FrozenLake-like (4x4 grid)
# 0=gauche, 1=bas, 2=droite, 3=haut
# États 0-15, état 15 = objectif, certains états = trous

class FrozenLake:
    """Environnement grille 4x4 simplifié."""
    
    def __init__(self):
        self.grid_size = 4
        self.n_states  = 16
        self.n_actions = 4   # gauche, bas, droite, haut
        # Trous : états 5, 7, 11, 12
        self.holes  = {5, 7, 11, 12}
        self.goal   = 15
        self.reset()
    
    def reset(self):
        self.state = 0
        return self.state
    
    def step(self, action):
        row = self.state // self.grid_size
        col = self.state % self.grid_size
        
        # Appliquer l'action
        if action == 0: col = max(0, col - 1)        # gauche
        elif action == 1: row = min(3, row + 1)      # bas
        elif action == 2: col = min(3, col + 1)      # droite
        elif action == 3: row = max(0, row - 1)      # haut
        
        self.state = row * self.grid_size + col
        
        if self.state in self.holes:
            return self.state, -1.0, True    # tombé dans un trou
        elif self.state == self.goal:
            return self.state, 1.0, True     # objectif atteint
        else:
            return self.state, -0.01, False  # petit malus pour encourager la vitesse

# Entraînement
env = FrozenLake()
agent = QLearningAgent(n_states=16, n_actions=4, lr=0.1, gamma=0.99, epsilon=1.0)

n_episodes = 5000
rewards_history = []

for episode in range(n_episodes):
    state = env.reset()
    total_reward = 0
    
    for step in range(100):   # max 100 steps par épisode
        action = agent.select_action(state)
        next_state, reward, done = env.step(action)
        agent.update(state, action, reward, next_state, done)
        
        state = next_state
        total_reward += reward
        
        if done:
            break
    
    agent.decay_epsilon()
    rewards_history.append(total_reward)
    
    if (episode + 1) % 1000 == 0:
        recent = np.mean(rewards_history[-100:])
        print(f"Épisode {episode+1:5d} | Récompense moy. (100 ep.) : {recent:.4f} | ε : {agent.epsilon:.3f}")

# Table Q apprise
print(f"\nTable Q apprise (4x4) :")
print(f"Actions : 0=gauche, 1=bas, 2=droite, 3=haut")
print(agent.Q.reshape(4, 4, 4).round(3))

# Politique optimale
policy = np.argmax(agent.Q, axis=1)
policy_names = ['←', '↓', '→', '↑']
print(f"\nPolitique apprise :")
print(np.array([policy_names[a] for a in policy]).reshape(4, 4))
```

### 5.4 Deep Q-Network (DQN)

Quand l'espace d'états est continu ou trop grand pour une table Q, on utilise un réseau de neurones pour approximer $Q(s, a)$.

```python
import tensorflow as tf
from tensorflow import keras
from collections import deque
import random

class DQNAgent:
    """Deep Q-Network avec Experience Replay et Target Network."""
    
    def __init__(self, state_dim, n_actions, lr=1e-3, gamma=0.99,
                 epsilon=1.0, memory_size=10000, batch_size=64):
        self.state_dim  = state_dim
        self.n_actions  = n_actions
        self.gamma      = gamma
        self.epsilon    = epsilon
        self.epsilon_min = 0.01
        self.epsilon_decay = 0.995
        self.batch_size = batch_size
        
        # Replay buffer
        self.memory = deque(maxlen=memory_size)
        
        # Réseau principal et réseau cible (target network)
        self.model        = self._build_network()
        self.target_model = self._build_network()
        self.update_target_network()
        
        self.optimizer = keras.optimizers.Adam(lr)
    
    def _build_network(self):
        """Réseau Q : s → Q(s, a) pour toutes les actions."""
        return keras.Sequential([
            keras.Input(shape=(self.state_dim,)),
            keras.layers.Dense(128, activation='relu'),
            keras.layers.Dense(128, activation='relu'),
            keras.layers.Dense(self.n_actions, activation='linear')  # linéaire pour Q
        ])
    
    def update_target_network(self):
        """Copier les poids du modèle principal vers le modèle cible."""
        self.target_model.set_weights(self.model.get_weights())
    
    def remember(self, state, action, reward, next_state, done):
        """Stocker une transition dans le replay buffer."""
        self.memory.append((state, action, reward, next_state, done))
    
    def select_action(self, state):
        if np.random.random() < self.epsilon:
            return np.random.randint(self.n_actions)
        q_values = self.model.predict(state[np.newaxis], verbose=0)[0]
        return np.argmax(q_values)
    
    @tf.function
    def _train_step(self, states, actions, rewards, next_states, dones):
        """Un pas de gradient."""
        # Target Q-values (depuis le target network pour la stabilité)
        next_q = self.target_model(next_states, training=False)
        max_next_q = tf.reduce_max(next_q, axis=1)
        targets = rewards + self.gamma * max_next_q * (1.0 - dones)
        
        with tf.GradientTape() as tape:
            q_values = self.model(states, training=True)
            # Sélectionner les Q-values pour les actions effectuées
            masks = tf.one_hot(actions, self.n_actions)
            q_selected = tf.reduce_sum(q_values * masks, axis=1)
            loss = tf.reduce_mean(tf.square(targets - q_selected))   # MSE
        
        grads = tape.gradient(loss, self.model.trainable_weights)
        self.optimizer.apply_gradients(zip(grads, self.model.trainable_weights))
        return loss
    
    def replay(self):
        """Entraîner sur un mini-batch depuis le replay buffer."""
        if len(self.memory) < self.batch_size:
            return None
        
        batch = random.sample(self.memory, self.batch_size)
        states      = np.array([t[0] for t in batch], dtype=np.float32)
        actions     = np.array([t[1] for t in batch], dtype=np.int32)
        rewards     = np.array([t[2] for t in batch], dtype=np.float32)
        next_states = np.array([t[3] for t in batch], dtype=np.float32)
        dones       = np.array([t[4] for t in batch], dtype=np.float32)
        
        loss = self._train_step(states, actions, rewards, next_states, dones)
        
        # Decay epsilon
        self.epsilon = max(self.epsilon_min, self.epsilon * self.epsilon_decay)
        return loss.numpy()

print("DQN Agent défini avec :")
print("  ✓ Experience Replay (replay buffer)")
print("  ✓ Target Network (stabilité)")
print("  ✓ ε-greedy avec decay")
```

### 5.5 CartPole avec Gymnasium

```python
# pip install gymnasium
try:
    import gymnasium as gym
    
    env = gym.make('CartPole-v1')
    state_dim  = env.observation_space.shape[0]   # 4 (position, vitesse, angle, vitesse angulaire)
    n_actions  = env.action_space.n               # 2 (gauche, droite)
    
    print(f"CartPole — State dim : {state_dim}, Actions : {n_actions}")
    
    agent_dqn = DQNAgent(state_dim=4, n_actions=2, lr=1e-3)
    
    # Boucle d'entraînement DQN
    n_episodes = 500
    target_update_freq = 10
    rewards_dqn = []
    
    for episode in range(n_episodes):
        state, _ = env.reset()
        total_reward = 0
        
        for step in range(500):   # max 500 pas
            action = agent_dqn.select_action(state)
            next_state, reward, terminated, truncated, _ = env.step(action)
            done = terminated or truncated
            
            agent_dqn.remember(state, action, reward, next_state, float(done))
            agent_dqn.replay()
            
            state = next_state
            total_reward += reward
            
            if done:
                break
        
        rewards_dqn.append(total_reward)
        
        # Mettre à jour le target network
        if (episode + 1) % target_update_freq == 0:
            agent_dqn.update_target_network()
        
        if (episode + 1) % 50 == 0:
            recent = np.mean(rewards_dqn[-50:])
            print(f"Episode {episode+1:4d} | Score moy. (50 ep.) : {recent:.1f} | ε : {agent_dqn.epsilon:.3f}")
            if recent >= 475:
                print("Environnement résolu !")
                break
    
    env.close()

except ImportError:
    print("gymnasium non installé : pip install gymnasium")
```

### 5.6 Policy Gradient — Introduction

```python
# Policy Gradient (REINFORCE) — apprend directement la politique π(a|s)
# Loss : -E[log π(a|s) * G_t]

print("Policy Gradient vs Q-Learning :")
print()
print("┌─────────────────────────────────────────────────────────────┐")
print("│         Value-Based           Policy-Based                  │")
print("│         (Q-Learning, DQN)     (REINFORCE, PPO, A3C)         │")
print("├─────────────────────────────────────────────────────────────┤")
print("│ Apprend Q(s,a)                Apprend π(a|s) directement    │")
print("│ Politique implicite           Politique paramétrique        │")
print("│ Espaces d'actions discrets    Espaces d'actions continus     │")
print("│ Plus stable                   Meilleur pour actions continues│")
print("│ Moins sample-efficient        État de l'art robotique/jeux   │")
print("└─────────────────────────────────────────────────────────────┘")
print()
print("Algorithmes modernes SOTA :")
print("  • PPO (Proximal Policy Optimization) — OpenAI Five, ChatGPT RLHF")
print("  • SAC (Soft Actor-Critic) — robotique")
print("  • TD3 (Twin Delayed DDPG) — contrôle continu")
print("  • AlphaZero / MuZero — jeux de plateau")
```

---

## 6. Récapitulatif — Choisir la Bonne Approche

| Situation | Algorithme Recommandé |
|-----------|----------------------|
| Clusters sphériques, $k$ connu | K-Means |
| Clusters de forme arbitraire | DBSCAN |
| Pas de $k$, hiérarchie intéressante | Hierarchical Clustering |
| Réduction pour visualisation | t-SNE ou UMAP |
| Réduction pour preprocessing | PCA |
| Compression avec reconstruction | Autoencoder |
| Génération de nouvelles données | VAE ou GAN |
| Actions discrètes dans environnement simple | Q-Learning |
| Actions discrètes, état complexe | DQN |
| Actions continues | SAC ou PPO |

---

## 7. Exercices Pratiques

> [!tip] Exercice 1 — Clustering
> Sur le dataset `make_blobs` avec 6 centres et des variances différentes, appliquez K-Means, DBSCAN et Hierarchical Clustering. Comparez les silhouette scores. Tracez les résultats côte à côte. Pour DBSCAN, optimisez `eps` en testant 5 valeurs différentes.

> [!tip] Exercice 2 — PCA et Reconstruction
> Sur le dataset Digits (8x8), appliquez PCA avec 2, 4, 8, 16, 32 et 64 composantes. Pour chaque $k$, calculez le MSE de reconstruction. Tracez le MSE en fonction du nombre de composantes. À partir de combien de composantes la reconstruction est-elle visuellement acceptable ?

> [!tip] Exercice 3 — Autoencoder Convolutionnel
> Implémentez un autoencoder avec des couches Conv2D (et Conv2DTranspose pour le decoder) sur MNIST ou Fashion-MNIST. Utilisez un espace latent de dimension 8. Évaluez la reconstruction. Puis entraînez un **denoising autoencoder** en ajoutant du bruit gaussien (σ=0.5).

> [!tip] Exercice 4 — Q-Learning Avancé
> Sur le même FrozenLake, implémentez les améliorations suivantes : (1) initialisation optimiste des Q-values (+1.0 partout), (2) learning rate décroissant, (3) epsilon decay basé sur les performances (réduire ε seulement quand le score moyen augmente). Comparez les courbes d'apprentissage.

> [!tip] Exercice 5 — DQN CartPole
> Entraînez le DQN sur CartPole-v1 jusqu'à résoudre l'environnement (score moyen ≥ 475 sur 100 épisodes). Implémentez ensuite **Double DQN** : utiliser le réseau principal pour choisir l'action, et le réseau cible pour évaluer la Q-value. Comparez la stabilité d'entraînement.

> [!tip] Exercice 6 — Visualisation GAN
> Entraînez le DCGAN sur MNIST pendant 30-50 epochs. Sauvegardez des images générées toutes les 5 epochs et créez une animation (GIF) montrant la progression de la qualité des images générées. Tracez aussi les courbes de loss G et D en parallèle.

---

## Liens

- [[02 - NumPy Pandas et Visualisation]] — outils de base pour les données
- [[04 - Reseaux de Neurones et Deep Learning]] — fondements des architectures
- [[05 - CNN Transformer et Computer Vision]] — architectures avancées
- [[03 - ML Supervise avec Scikit-Learn]] — K-Means et PCA Scikit-Learn

> [!info] Ressources
> - "Reinforcement Learning: An Introduction" — Sutton & Barto (référence RL)
> - "Generative Deep Learning" — David Foster (O'Reilly, GANs et VAE)
> - OpenAI Gymnasium : https://gymnasium.farama.org/
> - Stable Baselines 3 : RL en production — https://stable-baselines3.readthedocs.io/
> - Papers with Code — SOTA en clustering et génération : https://paperswithcode.com/
