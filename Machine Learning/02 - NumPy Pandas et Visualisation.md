# 02 - NumPy, Pandas et Visualisation

> [!info] Objectif du cours
> Maîtriser les outils essentiels de l'écosystème Python scientifique : NumPy pour le calcul numérique, Pandas pour la manipulation de données, et Matplotlib/Seaborn pour la visualisation. Ces outils sont le socle de tout pipeline ML.

---

## 1. NumPy — Calcul Numérique Efficace

### 1.1 ndarray — Le Tableau NumPy

Un `ndarray` est un tableau multidimensionnel homogène (tous les éléments ont le même type). Contrairement aux listes Python, les opérations s'effectuent en C sous le capot — **10 à 100x plus rapide**.

```python
import numpy as np

# Création depuis une liste
arr1d = np.array([1, 2, 3, 4, 5])
arr2d = np.array([[1, 2, 3],
                  [4, 5, 6],
                  [7, 8, 9]])

print(f"Shape   : {arr2d.shape}")    # (3, 3)
print(f"Dtype   : {arr2d.dtype}")    # int64
print(f"Ndim    : {arr2d.ndim}")     # 2
print(f"Size    : {arr2d.size}")     # 9 éléments
print(f"Nbytes  : {arr2d.nbytes}")  # 72 octets

# Tableaux prédéfinis
zeros   = np.zeros((3, 4))           # rempli de 0.0
ones    = np.ones((2, 5))            # rempli de 1.0
eye     = np.eye(4)                  # matrice identité 4x4
full    = np.full((3, 3), 7.5)       # rempli de 7.5
empty   = np.empty((2, 3))           # non initialisé (rapide)
like    = np.zeros_like(arr2d)       # même shape que arr2d
```

### 1.2 Types de Données (dtypes)

```python
# Dtypes NumPy principaux
print(np.dtype('float32').itemsize)   # 4 octets
print(np.dtype('float64').itemsize)   # 8 octets (défaut)
print(np.dtype('int32').itemsize)     # 4 octets
print(np.dtype('bool').itemsize)      # 1 octet

# Conversion de dtype
arr = np.array([1, 2, 3], dtype=np.float32)
arr64 = arr.astype(np.float64)
arr_int = arr.astype(np.int8)

# En ML : float32 est préféré pour économiser la mémoire GPU
# Les poids de réseaux de neurones sont souvent en float32
X_train = np.random.randn(10000, 784).astype(np.float32)
print(f"Mémoire float64 : {X_train.astype(np.float64).nbytes / 1e6:.1f} MB")
print(f"Mémoire float32 : {X_train.nbytes / 1e6:.1f} MB")  # 2x moins
```

### 1.3 Broadcasting

Le **broadcasting** permet d'appliquer des opérations entre tableaux de formes différentes selon des règles précises. C'est l'une des fonctionnalités les plus puissantes de NumPy.

> [!info] Règles de Broadcasting
> 1. Si les tableaux n'ont pas le même nombre de dimensions, étendre la forme du plus petit à gauche avec des 1.
> 2. Les dimensions de taille 1 sont étirées pour correspondre à l'autre tableau.
> 3. Si les dimensions ne sont pas compatibles (ni identiques, ni 1), erreur.

```python
# Exemple 1 : scalaire + tableau
arr = np.array([[1, 2, 3],
                [4, 5, 6]])
print(arr + 10)           # 10 est broadcasté sur toutes les cellules

# Exemple 2 : (3,) broadcasté sur (4, 3)
ligne = np.array([1, 2, 3])         # shape (3,)
matrice = np.ones((4, 3))           # shape (4, 3)
print(matrice + ligne)               # ligne ajoutée à chaque ligne de matrice

# Exemple 3 : (3, 1) x (1, 3) → (3, 3) — produit externe
col = np.array([[1], [2], [3]])     # shape (3, 1)
row = np.array([[10, 20, 30]])      # shape (1, 3)
print(col * row)                     # produit externe — chaque combinaison

# Application ML : centrer un dataset (soustraction de la moyenne)
X = np.random.randn(100, 5)         # 100 exemples, 5 features
mean = X.mean(axis=0)               # shape (5,) — moyenne par feature
std  = X.std(axis=0)                # shape (5,)
X_norm = (X - mean) / std          # broadcasting : (100,5) - (5,) / (5,)
print(f"Après normalisation : mean≈{X_norm.mean():.6f}, std≈{X_norm.std():.6f}")
```

### 1.4 Indexing Avancé et Fancy Indexing

```python
arr = np.arange(20).reshape(4, 5)
print(arr)
# [[ 0  1  2  3  4]
#  [ 5  6  7  8  9]
#  [10 11 12 13 14]
#  [15 16 17 18 19]]

# Slicing standard
print(arr[1:3, 2:4])     # lignes 1-2, colonnes 2-3
print(arr[:, 2])          # toute la colonne 2
print(arr[::2, ::2])     # 1 ligne sur 2, 1 colonne sur 2

# Fancy indexing — index par tableaux
rows = np.array([0, 2, 3])
cols = np.array([1, 3, 4])
print(arr[rows, cols])    # arr[0,1], arr[2,3], arr[3,4]

# Masques booléens — très utilisés en filtrage
data = np.array([15, 3, 42, 7, 28, 5, 19])
masque = data > 10
print(f"Masque   : {masque}")
print(f"Filtrés  : {data[masque]}")    # [15, 42, 28, 19]
print(f"Filtrés  : {data[data > 10]}") # syntaxe directe

# Modifier par masque
data[data < 5] = 0   # mettre à 0 les valeurs < 5
print(f"Après masquage : {data}")
```

### 1.5 Reshape, Squeeze, Expand_dims

```python
arr = np.arange(24)

# reshape — changer la forme sans copier les données
a = arr.reshape(4, 6)
b = arr.reshape(2, 3, 4)
c = arr.reshape(-1, 8)      # -1 : NumPy calcule la dimension automatiquement

print(f"Original : {arr.shape}")  # (24,)
print(f"(4,6)    : {a.shape}")
print(f"(2,3,4)  : {b.shape}")
print(f"(-1,8)   : {c.shape}")   # (3, 8)

# Aplatir un tableau
flat = b.flatten()    # toujours une copie
ravel = b.ravel()     # vue si possible, copie si nécessaire

# expand_dims — ajouter une dimension (pour le batching)
vecteur = np.array([1, 2, 3])              # shape (3,)
mat_ligne = np.expand_dims(vecteur, 0)     # shape (1, 3) — 1 batch
mat_col  = np.expand_dims(vecteur, 1)      # shape (3, 1)
print(f"expand_dims(v, 0) : {mat_ligne.shape}")
print(f"expand_dims(v, 1) : {mat_col.shape}")

# squeeze — supprimer les dimensions de taille 1
batch_of_one = np.random.randn(1, 224, 224, 3)  # (1, H, W, C)
image = batch_of_one.squeeze(0)                  # (224, 224, 3)
print(f"Avant squeeze : {batch_of_one.shape}")
print(f"Après squeeze : {image.shape}")
```

### 1.6 Universal Functions (ufuncs) et Opérations Vectorisées

```python
x = np.linspace(-np.pi, np.pi, 1000)

# Fonctions mathématiques vectorisées
sin_x = np.sin(x)
cos_x = np.cos(x)
exp_x = np.exp(np.linspace(-3, 3, 1000))
log_x = np.log(np.linspace(0.01, 10, 1000))

# Réductions
arr = np.random.randn(100, 50)
print(f"Somme totale   : {arr.sum():.4f}")
print(f"Somme par col  : {arr.sum(axis=0).shape}")    # (50,)
print(f"Somme par ligne: {arr.sum(axis=1).shape}")    # (100,)
print(f"Max global     : {arr.max():.4f}")
print(f"ArgMax         : {arr.argmax()}")              # indice aplati
print(f"ArgMax 2D      : {np.unravel_index(arr.argmax(), arr.shape)}")

# Comparaison performance : Python pur vs NumPy
import time
n = 10_000_000

start = time.time()
py_sum = sum([i**2 for i in range(n)])
t_python = time.time() - start

start = time.time()
np_sum = np.arange(n, dtype=np.float64).__pow__(2).sum()
t_numpy = time.time() - start

print(f"\nPython pur : {t_python:.3f}s")
print(f"NumPy      : {t_numpy:.3f}s")
print(f"Accélération : {t_python/t_numpy:.1f}x")
```

### 1.7 Génération Aléatoire et Reproductibilité

```python
# Seed pour reproductibilité — TOUJOURS définir en ML
rng = np.random.default_rng(seed=42)    # Générateur moderne (NumPy 1.17+)

# Distributions
uniform = rng.uniform(low=0, high=1, size=(3, 4))
normal  = rng.normal(loc=0, scale=1, size=1000)
ints    = rng.integers(low=0, high=100, size=50)
choice  = rng.choice(np.arange(20), size=5, replace=False)   # sans remise

# Mélange
arr = np.arange(10)
rng.shuffle(arr)     # en place
print(f"Mélangé : {arr}")

# linspace et arange
lin = np.linspace(0, 1, num=11)    # 11 points de 0 à 1 inclus
ar  = np.arange(0, 10, step=2)    # [0, 2, 4, 6, 8]
print(f"linspace : {lin}")
print(f"arange   : {ar}")
```

---

## 2. Pandas — Manipulation de Données

### 2.1 DataFrame et Series

```python
import pandas as pd

# Series — tableau 1D avec labels
s = pd.Series([10, 20, 30, 40], index=['a', 'b', 'c', 'd'])
print(s)
print(f"Type   : {type(s)}")
print(f"Valeur b : {s['b']}")
print(f"Dtype  : {s.dtype}")

# DataFrame — tableau 2D avec labels de lignes et colonnes
df = pd.DataFrame({
    'nom': ['Alice', 'Bob', 'Charlie', 'Diana'],
    'age': [25, 30, 22, 28],
    'salaire': [45000, 62000, 38000, 70000],
    'departement': ['IT', 'RH', 'IT', 'Finance']
})
print(df)
print(f"\nShape  : {df.shape}")     # (4, 4)
print(f"Colonnes : {df.columns.tolist()}")
print(f"Index : {df.index.tolist()}")
print(f"Dtypes :\n{df.dtypes}")
```

### 2.2 Lecture de Données

```python
# Lecture CSV (le plus courant)
# df = pd.read_csv('data.csv', sep=',', encoding='utf-8', index_col=0)

# Lecture avec options avancées
# df = pd.read_csv('data.csv',
#     usecols=['col1', 'col2', 'col3'],  # sélectionner colonnes
#     dtype={'age': np.int32},            # forcer le type
#     na_values=['NA', 'N/A', '-'],       # valeurs NaN supplémentaires
#     nrows=1000,                          # lire seulement 1000 lignes
#     parse_dates=['date_col'],            # parser les dates
# )

# Simulation avec le dataset Titanic intégré à Seaborn
import seaborn as sns
titanic = sns.load_dataset('titanic')
print(f"Dataset Titanic : {titanic.shape}")
print(titanic.head())

# Écriture
# df.to_csv('output.csv', index=False)
# df.to_json('output.json', orient='records')
# df.to_excel('output.xlsx', index=False)

# Lecture SQL
# import sqlite3
# conn = sqlite3.connect('database.db')
# df = pd.read_sql("SELECT * FROM users WHERE age > 30", conn)
```

### 2.3 Exploration et Description

```python
df = sns.load_dataset('titanic')

# Vue d'ensemble
print(df.info())           # types, valeurs non-null, mémoire
print(df.describe())       # stats pour colonnes numériques
print(df.describe(include='all'))  # toutes colonnes

# Valeurs manquantes
print("\nValeurs manquantes :")
print(df.isnull().sum())
print(f"% de NaN par colonne :\n{(df.isnull().sum() / len(df) * 100).round(1)}")

# Distribution d'une colonne catégorielle
print("\nDistribution 'survived' :")
print(df['survived'].value_counts())
print(df['survived'].value_counts(normalize=True).round(3))  # proportions
```

### 2.4 Filtrage et Sélection

```python
df = sns.load_dataset('titanic')

# Sélection de colonnes
ages = df['age']               # Series
sous_df = df[['age', 'fare', 'survived']]  # DataFrame

# loc — par labels
ligne_0 = df.loc[0]            # ligne d'index 0
sous = df.loc[0:5, 'age':'survived']  # slice par labels

# iloc — par position
premier = df.iloc[0]           # première ligne
bloc = df.iloc[0:5, 2:5]       # lignes 0-4, colonnes 2-4

# Filtrage conditionnel
survivants = df[df['survived'] == 1]
adultes_1ere = df[(df['age'] >= 18) & (df['pclass'] == 1)]
femmes_ou_enfants = df[(df['sex'] == 'female') | (df['age'] < 12)]

print(f"Survivants      : {len(survivants)}")
print(f"Adultes 1ère    : {len(adultes_1ere)}")
print(f"Femmes|Enfants  : {len(femmes_ou_enfants)}")

# isin — membership test
classes_luxe = df[df['pclass'].isin([1, 2])]
print(f"Classes 1 et 2  : {len(classes_luxe)}")

# query — syntaxe SQL-like
riches = df.query("fare > 100 and pclass == 1")
print(f"Riches          : {len(riches)}")
```

### 2.5 GroupBy et Agrégation

```python
df = sns.load_dataset('titanic')

# GroupBy simple
survie_par_classe = df.groupby('pclass')['survived'].mean()
print("Taux de survie par classe :")
print(survie_par_classe.round(3))

# Agrégations multiples
stats = df.groupby('pclass').agg({
    'survived': ['mean', 'sum', 'count'],
    'age': ['mean', 'median'],
    'fare': ['mean', 'max']
})
print(f"\nStats par classe :\n{stats.round(2)}")

# GroupBy avec plusieurs colonnes
taux = df.groupby(['pclass', 'sex'])['survived'].mean().unstack()
print(f"\nTaux de survie par classe et sexe :\n{taux.round(3)}")

# Transform — conserver la taille originale
df['fare_vs_classe'] = df.groupby('pclass')['fare'].transform(
    lambda x: x - x.mean()
)
print(f"\nfare_vs_classe (5 premières) : {df['fare_vs_classe'].head().values}")
```

### 2.6 Merge, Join et Concat

```python
# Données de démonstration
df_notes = pd.DataFrame({
    'etudiant_id': [1, 2, 3, 4, 5],
    'note': [85, 92, 78, 65, 88]
})
df_etudiants = pd.DataFrame({
    'id': [1, 2, 3, 6, 7],
    'nom': ['Alice', 'Bob', 'Charlie', 'Diana', 'Eve']
})

# INNER JOIN — seulement les correspondances
inner = pd.merge(df_notes, df_etudiants,
                 left_on='etudiant_id', right_on='id', how='inner')
print(f"Inner join : {len(inner)} lignes")
print(inner)

# LEFT JOIN — toutes les notes, NaN si étudiant inconnu
left = pd.merge(df_notes, df_etudiants,
                left_on='etudiant_id', right_on='id', how='left')
print(f"\nLeft join : {len(left)} lignes")
print(left)

# Concatenation verticale
df_a = pd.DataFrame({'x': [1, 2, 3], 'y': ['a', 'b', 'c']})
df_b = pd.DataFrame({'x': [4, 5, 6], 'y': ['d', 'e', 'f']})
concat = pd.concat([df_a, df_b], ignore_index=True)
print(f"\nConcat vertical :\n{concat}")

# Concatenation horizontale
concat_h = pd.concat([df_a, df_b.rename(columns={'x': 'x2', 'y': 'y2'})], axis=1)
print(f"\nConcat horizontal :\n{concat_h}")
```

### 2.7 Gestion des Valeurs Manquantes

```python
df = sns.load_dataset('titanic').copy()

print(f"NaN avant traitement :\n{df.isnull().sum()[df.isnull().sum() > 0]}")

# Stratégie 1 : supprimer les lignes avec NaN
df_drop = df.dropna()
print(f"\nAprès dropna : {df_drop.shape[0]} lignes (sur {df.shape[0]})")

# Stratégie 2 : supprimer les colonnes avec trop de NaN (> 40%)
seuil = 0.4
cols_a_garder = df.columns[df.isnull().mean() < seuil]
df_cols = df[cols_a_garder]
print(f"Colonnes retenues : {df_cols.columns.tolist()}")

# Stratégie 3 : imputation
df_imputed = df.copy()
df_imputed['age'] = df_imputed['age'].fillna(df_imputed['age'].median())
df_imputed['embarked'] = df_imputed['embarked'].fillna(df_imputed['embarked'].mode()[0])
df_imputed = df_imputed.drop(columns=['deck'])  # trop de NaN

print(f"\nAprès imputation :\n{df_imputed.isnull().sum()[df_imputed.isnull().sum() > 0]}")

# Interpolation (pour séries temporelles)
ts = pd.Series([1.0, np.nan, np.nan, 4.0, np.nan, 6.0])
print(f"\nInterpolation linéaire : {ts.interpolate().values}")
```

### 2.8 Apply, Map et Fonctions Personnalisées

```python
df = sns.load_dataset('titanic').copy()

# apply sur une colonne
df['age_groupe'] = df['age'].apply(
    lambda x: 'enfant' if x < 18 else ('senior' if x >= 60 else 'adulte')
)
print(df[['age', 'age_groupe']].head(10))

# apply sur un DataFrame (axis=1 = par ligne)
def calculer_score(row):
    """Score basé sur plusieurs colonnes."""
    score = 0
    if row['pclass'] == 1: score += 30
    if row['sex'] == 'female': score += 20
    if row['age'] < 18: score += 10
    return score

df['score_priorite'] = df.apply(calculer_score, axis=1)

# map — remplacer des valeurs (Series)
df['sex_num'] = df['sex'].map({'male': 0, 'female': 1})
print(f"\nSex encodé : {df['sex_num'].value_counts().to_dict()}")

# pipe — enchaîner des transformations
def normalize_col(df, col):
    df[col] = (df[col] - df[col].min()) / (df[col].max() - df[col].min())
    return df

result = (df
    .dropna(subset=['age', 'fare'])
    .pipe(normalize_col, 'fare')
    .query("pclass == 1")
)
print(f"\nPipeline result shape : {result.shape}")
```

### 2.9 Dates et Séries Temporelles

```python
# Créer un index temporel
dates = pd.date_range(start='2024-01-01', periods=365, freq='D')
ts = pd.Series(np.random.randn(365).cumsum(), index=dates)

print(f"Premiers index : {ts.index[:3]}")
print(f"Type index     : {type(ts.index)}")

# Filtrage par date
jan_2024 = ts['2024-01']
q1_2024  = ts['2024-01-01':'2024-03-31']
print(f"Janvier 2024 : {len(jan_2024)} jours")

# Resampling
weekly  = ts.resample('W').mean()    # moyenne hebdomadaire
monthly = ts.resample('ME').sum()    # somme mensuelle
print(f"Points hebdomadaires  : {len(weekly)}")
print(f"Points mensuels       : {len(monthly)}")

# Rolling (fenêtre glissante)
rolling_7d  = ts.rolling(window=7).mean()
rolling_30d = ts.rolling(window=30).mean()
```

---

## 3. Visualisation

### 3.1 Matplotlib — Base

```python
import matplotlib.pyplot as plt
import matplotlib.gridspec as gridspec

# Style global
plt.style.use('seaborn-v0_8-darkgrid')

# Figure et Axes — OOP API (recommandée)
fig, axes = plt.subplots(2, 2, figsize=(12, 10))
fig.suptitle('Démonstration Matplotlib', fontsize=16, fontweight='bold')

x = np.linspace(0, 2*np.pi, 100)

# Ligne + scatter
axes[0, 0].plot(x, np.sin(x), 'b-', label='sin(x)', linewidth=2)
axes[0, 0].plot(x, np.cos(x), 'r--', label='cos(x)', linewidth=2)
axes[0, 0].set_title('Fonctions trigonométriques')
axes[0, 0].legend()
axes[0, 0].set_xlabel('x (radians)')
axes[0, 0].set_ylabel('f(x)')

# Scatter plot
np.random.seed(42)
x_sc = np.random.randn(200)
y_sc = 2*x_sc + np.random.randn(200)
colors = np.abs(y_sc)
scatter = axes[0, 1].scatter(x_sc, y_sc, c=colors, cmap='viridis',
                              alpha=0.7, s=50)
axes[0, 1].set_title('Scatter Plot')
fig.colorbar(scatter, ax=axes[0, 1])

# Histogramme
data = np.concatenate([np.random.normal(-2, 1, 500),
                        np.random.normal(3, 0.8, 300)])
axes[1, 0].hist(data, bins=50, color='steelblue', alpha=0.8,
                edgecolor='white', linewidth=0.5, density=True)
axes[1, 0].set_title('Histogramme bimodal')

# Boxplot
data_box = [np.random.normal(i, 1, 100) for i in range(5)]
axes[1, 1].boxplot(data_box, labels=[f'G{i}' for i in range(5)],
                   patch_artist=True)
axes[1, 1].set_title('Boxplot par groupe')

plt.tight_layout()
plt.savefig('matplotlib_demo.png', dpi=150, bbox_inches='tight')
plt.close()
print("Figure sauvegardée : matplotlib_demo.png")
```

### 3.2 Heatmap avec Matplotlib

```python
# Heatmap de la matrice de corrélation
df = sns.load_dataset('titanic')
numerique = df.select_dtypes(include=np.number).dropna()
corr = numerique.corr()

fig, ax = plt.subplots(figsize=(8, 6))
im = ax.imshow(corr, cmap='RdYlBu_r', vmin=-1, vmax=1)
plt.colorbar(im, ax=ax)

# Annotations
for i in range(corr.shape[0]):
    for j in range(corr.shape[1]):
        ax.text(j, i, f'{corr.iloc[i, j]:.2f}',
                ha='center', va='center', fontsize=9)

ax.set_xticks(range(len(corr.columns)))
ax.set_yticks(range(len(corr.columns)))
ax.set_xticklabels(corr.columns, rotation=45, ha='right')
ax.set_yticklabels(corr.columns)
ax.set_title('Matrice de Corrélation — Titanic')
plt.tight_layout()
plt.savefig('heatmap_corr.png', dpi=150, bbox_inches='tight')
plt.close()
```

### 3.3 Seaborn — Visualisation Statistique

```python
import seaborn as sns

df = sns.load_dataset('titanic')
iris = sns.load_dataset('iris')

# 1. Distribution plots
fig, axes = plt.subplots(1, 3, figsize=(15, 5))

# Histogramme + KDE
sns.histplot(df['age'].dropna(), kde=True, ax=axes[0], color='steelblue')
axes[0].set_title('Distribution des Ages')

# Boxplot par catégorie
sns.boxplot(x='pclass', y='fare', data=df, palette='Set2', ax=axes[1])
axes[1].set_title('Fare par Classe')

# Violinplot
sns.violinplot(x='sex', y='age', hue='survived', data=df,
               split=True, palette='coolwarm', ax=axes[2])
axes[2].set_title('Age, Sexe et Survie')

plt.tight_layout()
plt.savefig('seaborn_dist.png', dpi=150)
plt.close()

# 2. Pairplot — relations entre toutes les paires de variables
# pairplot = sns.pairplot(iris, hue='species', diag_kind='kde')
# pairplot.savefig('pairplot_iris.png', dpi=120)

# 3. Heatmap Seaborn (plus simple)
fig, ax = plt.subplots(figsize=(8, 6))
numerique = df.select_dtypes(include=np.number).dropna()
sns.heatmap(numerique.corr(), annot=True, fmt='.2f', cmap='RdYlBu_r',
            center=0, ax=ax, linewidths=0.5)
ax.set_title('Corrélation Titanic — Seaborn')
plt.tight_layout()
plt.savefig('heatmap_seaborn.png', dpi=150)
plt.close()
```

### 3.4 Plotly — Graphes Interactifs

```python
# pip install plotly
import plotly.express as px
import plotly.graph_objects as go

df = sns.load_dataset('iris')

# Scatter 3D interactif
fig = px.scatter_3d(df, x='sepal_length', y='sepal_width',
                     z='petal_length', color='species',
                     title='Iris Dataset — Scatter 3D Interactif',
                     symbol='species')
fig.write_html('iris_3d.html')
print("Graphe interactif sauvegardé : iris_3d.html")

# Histogramme interactif
titanic = sns.load_dataset('titanic')
fig2 = px.histogram(titanic.dropna(subset=['age']),
                     x='age', color='survived',
                     marginal='box',
                     barmode='overlay',
                     title='Distribution Age — Titanic (Interactif)')
fig2.write_html('titanic_age.html')
```

---

## 4. Pipeline EDA Complet — Dataset Titanic

> [!info] EDA = Exploratory Data Analysis
> L'analyse exploratoire est la première étape de tout projet ML. Elle permet de comprendre les données avant de modéliser.

```python
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
from scipy import stats

# =============================================
# ÉTAPE 1 : Chargement et première inspection
# =============================================
df = sns.load_dataset('titanic')
print("=" * 60)
print("ÉTAPE 1 — Chargement")
print(f"Shape : {df.shape}")
print(f"Types :\n{df.dtypes}")
print(f"\nPremières lignes :\n{df.head()}")

# =============================================
# ÉTAPE 2 : Valeurs manquantes
# =============================================
print("\n" + "=" * 60)
print("ÉTAPE 2 — Valeurs manquantes")
nan_pct = df.isnull().mean() * 100
print(nan_pct[nan_pct > 0].round(1).to_string())

# =============================================
# ÉTAPE 3 : Distribution de la variable cible
# =============================================
print("\n" + "=" * 60)
print("ÉTAPE 3 — Variable cible (survived)")
print(df['survived'].value_counts(normalize=True).round(3))
print(f"→ Déséquilibre de classes : {df['survived'].mean()*100:.1f}% de survivants")

# =============================================
# ÉTAPE 4 : Statistiques descriptives
# =============================================
print("\n" + "=" * 60)
print("ÉTAPE 4 — Stats descriptives (variables numériques)")
print(df.describe().round(2))

# =============================================
# ÉTAPE 5 : Analyse univariée — catégories
# =============================================
print("\n" + "=" * 60)
print("ÉTAPE 5 — Variables catégorielles")
for col in ['pclass', 'sex', 'embarked']:
    print(f"\n{col} :\n{df.groupby(col)['survived'].mean().round(3)}")

# =============================================
# ÉTAPE 6 : Corrélations
# =============================================
print("\n" + "=" * 60)
print("ÉTAPE 6 — Corrélations avec 'survived'")
numerique = df.select_dtypes(include=np.number)
corr_survived = numerique.corr()['survived'].drop('survived').sort_values()
print(corr_survived.round(3))

# =============================================
# ÉTAPE 7 : Outliers (méthode IQR)
# =============================================
print("\n" + "=" * 60)
print("ÉTAPE 7 — Détection d'outliers (IQR) sur 'fare'")
Q1 = df['fare'].quantile(0.25)
Q3 = df['fare'].quantile(0.75)
IQR = Q3 - Q1
outliers = df[(df['fare'] < Q1 - 1.5*IQR) | (df['fare'] > Q3 + 1.5*IQR)]
print(f"Q1={Q1:.2f}, Q3={Q3:.2f}, IQR={IQR:.2f}")
print(f"Outliers : {len(outliers)} ({len(outliers)/len(df)*100:.1f}%)")

# =============================================
# ÉTAPE 8 : Feature Engineering simple
# =============================================
print("\n" + "=" * 60)
print("ÉTAPE 8 — Feature Engineering")
df['age_imputed'] = df['age'].fillna(df['age'].median())
df['family_size'] = df['sibsp'] + df['parch'] + 1
df['is_alone'] = (df['family_size'] == 1).astype(int)
df['fare_log'] = np.log1p(df['fare'])   # normaliser la distribution asymétrique

print("Nouvelles features créées :")
print(df[['family_size', 'is_alone', 'fare_log']].describe().round(3))

# =============================================
# ÉTAPE 9 : Rapport final
# =============================================
print("\n" + "=" * 60)
print("RAPPORT EDA — RÉSUMÉ")
print(f"  Observations       : {df.shape[0]}")
print(f"  Features           : {df.shape[1]}")
print(f"  Taux de survie     : {df['survived'].mean()*100:.1f}%")
print(f"  Meilleur prédicteur: 'sex' (r={df['sex'].map({'female':1,'male':0}).corr(df['survived']):.3f})")
print(f"  NaN 'age'          : {df['age'].isnull().sum()} ({df['age'].isnull().mean()*100:.1f}%)")
print(f"  Outliers 'fare'    : {len(outliers)} lignes")
```

---

## 5. Récapitulatif — Cheat Sheet

| Opération | NumPy | Pandas |
|-----------|-------|--------|
| Sélection | `arr[1:3, ::2]` | `df.iloc[1:3]` / `df.loc[:, 'col']` |
| Filtrage | `arr[arr > 5]` | `df[df['col'] > 5]` |
| Agrégation | `arr.mean(axis=0)` | `df.groupby('g')['c'].mean()` |
| Jointure | `np.concatenate` | `pd.merge(df1, df2, on='key')` |
| NaN | `np.isnan(arr)` | `df.isnull().fillna()` |
| Shape | `arr.reshape(-1, 3)` | `df.shape` |
| Tri | `np.sort(arr)` | `df.sort_values('col')` |

---

## 6. Exercices Pratiques

> [!tip] Exercice 1 — Broadcasting NumPy
> Sans boucle `for`, créez une matrice de distances euclidiennes entre tous les points d'un tableau `X` de shape `(100, 2)`. Indice : utiliser `expand_dims` et le broadcasting.

> [!tip] Exercice 2 — Pandas EDA
> Chargez le dataset `'diamonds'` avec `sns.load_dataset('diamonds')`. Calculez le prix moyen par `cut` et `color`. Quelle combinaison a le prix moyen le plus élevé ? Visualisez avec un heatmap.

> [!tip] Exercice 3 — Gestion des NaN
> Sur le Titanic, implémentez une stratégie d'imputation avancée pour `age` : remplir les NaN par la **médiane du groupe** (par `pclass` et `sex`). Comparer avec l'imputation par la médiane globale — laquelle a la meilleure distribution ?

> [!tip] Exercice 4 — Feature Engineering
> Sur le Titanic, créez les features : `title` (extrait du nom : Mr, Mrs, Miss, etc.), `cabin_deck` (première lettre de `cabin`), `is_child` (age < 12). Mesurez leur corrélation avec `survived`.

> [!tip] Exercice 5 — Visualisation
> Créez une figure Matplotlib avec 4 subplots : (1) distribution des âges par classe, (2) courbe de survie par tranche d'âge (binning par 10 ans), (3) heatmap des corrélations, (4) scatter fare vs age coloré par survived.

---

## Liens

- [[01 - Mathematiques pour le ML]] — fondements mathématiques
- [[03 - ML Supervise avec Scikit-Learn]] — préprocessing et pipelines Scikit-Learn
- [[06 - ML Non Supervise et Reinforcement Learning]] — PCA et visualisation t-SNE

> [!info] Ressources
> - Documentation NumPy : https://numpy.org/doc/
> - Documentation Pandas : https://pandas.pydata.org/docs/
> - Galerie Matplotlib : https://matplotlib.org/stable/gallery/
> - Galerie Seaborn : https://seaborn.pydata.org/examples/
