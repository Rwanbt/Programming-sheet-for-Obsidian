# 03 - ML Supervisé avec Scikit-Learn

> [!info] Objectif du cours
> Maîtriser le workflow complet de Machine Learning supervisé avec Scikit-Learn : de la préparation des données à l'évaluation et au déploiement des modèles. Couvre la régression, la classification, et le tuning d'hyperparamètres.

---

## 1. Le Workflow ML Complet

```
Collecte → Exploration (EDA) → Nettoyage → Feature Engineering
→ Split Train/Test → Preprocessing → Entraînement → Évaluation
→ Tuning → Évaluation finale → Déploiement
```

> [!warning] Règle d'or
> Le **test set ne doit JAMAIS être vu** pendant l'entraînement, le preprocessing ou le tuning. Toute fuite d'information depuis le test set (data leakage) invalide l'évaluation.

```python
import numpy as np
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
from sklearn.model_selection import train_test_split, cross_val_score, StratifiedKFold
from sklearn.preprocessing import StandardScaler, MinMaxScaler, LabelEncoder, OneHotEncoder
from sklearn.pipeline import Pipeline
from sklearn.metrics import (accuracy_score, precision_score, recall_score,
                             f1_score, roc_auc_score, confusion_matrix,
                             mean_squared_error, mean_absolute_error, r2_score)
import warnings
warnings.filterwarnings('ignore')

# Dataset de travail
df = sns.load_dataset('titanic').dropna(subset=['age', 'embarked']).copy()
print(f"Dataset : {df.shape}")
```

---

## 2. Train/Test Split et Cross-Validation

### 2.1 Train/Test Split

```python
from sklearn.model_selection import train_test_split

# Features et cible
features = ['pclass', 'age', 'sibsp', 'parch', 'fare']
X = df[features].values
y = df['survived'].values

# Split 80% / 20%
X_train, X_test, y_train, y_test = train_test_split(
    X, y,
    test_size=0.2,
    random_state=42,
    stratify=y    # IMPORTANT pour classification : conserver les proportions de classes
)

print(f"Taille train  : {X_train.shape}  ({len(X_train)} exemples)")
print(f"Taille test   : {X_test.shape}   ({len(X_test)} exemples)")
print(f"Ratio de survivants train : {y_train.mean():.3f}")
print(f"Ratio de survivants test  : {y_test.mean():.3f}")
```

### 2.2 K-Fold Cross-Validation

La **cross-validation** permet d'évaluer la performance sans gaspiller des données en test set fixe.

```
Données   [=====|=====|=====|=====|=====]
Fold 1 :  [Test |Train|Train|Train|Train]
Fold 2 :  [Train|Test |Train|Train|Train]
Fold 3 :  [Train|Train|Test |Train|Train]
...
Score final = moyenne des k scores
```

```python
from sklearn.model_selection import cross_val_score, StratifiedKFold
from sklearn.linear_model import LogisticRegression

model = LogisticRegression(max_iter=1000, random_state=42)

# K-Fold simple
kfold = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
scores = cross_val_score(model, X_train, y_train,
                          cv=kfold, scoring='accuracy')

print(f"Scores K-Fold (5 folds) : {scores.round(4)}")
print(f"Moyenne  : {scores.mean():.4f}")
print(f"Std      : {scores.std():.4f}")
print(f"Intervalle de confiance : {scores.mean():.4f} ± {2*scores.std():.4f}")
```

---

## 3. Preprocessing

### 3.1 Scalers — Normalisation et Standardisation

```python
from sklearn.preprocessing import StandardScaler, MinMaxScaler, RobustScaler

data_ex = np.array([[100, 0.01],
                     [200, 0.02],
                     [300, 0.03],
                     [1000, 0.1]])   # features à des échelles très différentes

# StandardScaler : (x - mean) / std → N(0,1)
scaler_std = StandardScaler()
data_std = scaler_std.fit_transform(data_ex)
print("StandardScaler :")
print(f"  Avant : min={data_ex.min():.2f}, max={data_ex.max():.2f}")
print(f"  Après : mean={data_std.mean():.2f}, std={data_std.std():.2f}")

# MinMaxScaler : (x - min) / (max - min) → [0, 1]
scaler_mm = MinMaxScaler(feature_range=(0, 1))
data_mm = scaler_mm.fit_transform(data_ex)
print(f"\nMinMaxScaler : min={data_mm.min():.2f}, max={data_mm.max():.2f}")

# RobustScaler : (x - médiane) / IQR → résistant aux outliers
scaler_rob = RobustScaler()
data_rob = scaler_rob.fit_transform(data_ex)
print(f"RobustScaler : médiane≈{np.median(data_rob):.2f}")

# RÈGLE D'OR : fit() sur train, transform() sur train ET test
X_train_scaled = scaler_std.fit_transform(X_train)
X_test_scaled  = scaler_std.transform(X_test)    # PAS fit_transform !
```

### 3.2 Encodage des Variables Catégorielles

```python
from sklearn.preprocessing import LabelEncoder, OrdinalEncoder, OneHotEncoder

# Label Encoding — pour les variables ordinales ou les classes cibles
le = LabelEncoder()
sexe = np.array(['male', 'female', 'female', 'male', 'male'])
sexe_encoded = le.fit_transform(sexe)
print(f"Label Encoding : {dict(zip(le.classes_, le.transform(le.classes_)))}")
# female=0, male=1

# One-Hot Encoding — pour les variables nominales (pas d'ordre)
ohe = OneHotEncoder(sparse_output=False, handle_unknown='ignore', drop='first')
embarked = np.array([['C'], ['Q'], ['S'], ['C'], ['S']])
embarked_ohe = ohe.fit_transform(embarked)
print(f"\nOne-Hot Encoding embarked :\n{embarked_ohe}")
print(f"Catégories : {ohe.categories_}")
```

### 3.3 Pipeline Scikit-Learn

Le `Pipeline` enchaîne preprocessing et modèle — garantit l'absence de data leakage.

```python
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.impute import SimpleImputer
from sklearn.preprocessing import OneHotEncoder, StandardScaler

# Définir les colonnes par type
numerical_features = ['age', 'fare', 'sibsp', 'parch']
categorical_features = ['sex', 'embarked']

# Pipeline numérique : imputation → standardisation
numerical_pipeline = Pipeline([
    ('imputer', SimpleImputer(strategy='median')),
    ('scaler', StandardScaler())
])

# Pipeline catégoriel : imputation → one-hot
categorical_pipeline = Pipeline([
    ('imputer', SimpleImputer(strategy='most_frequent')),
    ('onehot', OneHotEncoder(handle_unknown='ignore', drop='first'))
])

# Combiner avec ColumnTransformer
preprocessor = ColumnTransformer([
    ('num', numerical_pipeline, numerical_features),
    ('cat', categorical_pipeline, categorical_features)
])

# Pipeline complet avec le modèle
from sklearn.ensemble import RandomForestClassifier

full_pipeline = Pipeline([
    ('preprocessor', preprocessor),
    ('classifier', RandomForestClassifier(n_estimators=100, random_state=42))
])

# Préparer les données
df_clean = df[numerical_features + categorical_features + ['survived']].dropna(subset=['embarked'])
X_full = df_clean[numerical_features + categorical_features]
y_full = df_clean['survived']

X_tr, X_te, y_tr, y_te = train_test_split(X_full, y_full, test_size=0.2,
                                            random_state=42, stratify=y_full)

# Entraîner — le pipeline gère tout automatiquement
full_pipeline.fit(X_tr, y_tr)
y_pred = full_pipeline.predict(X_te)
print(f"Accuracy Pipeline : {accuracy_score(y_te, y_pred):.4f}")
```

---

## 4. Régression

### 4.1 Régression Linéaire

$$\hat{y} = X\beta = \beta_0 + \beta_1 x_1 + \cdots + \beta_n x_n$$

La solution analytique (Ordinary Least Squares) :
$$\beta = (X^T X)^{-1} X^T y$$

```python
from sklearn.linear_model import LinearRegression, Ridge, Lasso

# Dataset de régression — prix de l'immobilier
from sklearn.datasets import fetch_california_housing
housing = fetch_california_housing(as_frame=True)
X_reg = housing.data
y_reg = housing.target   # prix en 100k$

X_tr, X_te, y_tr, y_te = train_test_split(X_reg, y_reg, test_size=0.2,
                                            random_state=42)
# Pipeline régression linéaire
pipeline_lr = Pipeline([
    ('scaler', StandardScaler()),
    ('model', LinearRegression())
])
pipeline_lr.fit(X_tr, y_tr)
y_pred_lr = pipeline_lr.predict(X_te)

# Métriques de régression
mse  = mean_squared_error(y_te, y_pred_lr)
rmse = np.sqrt(mse)
mae  = mean_absolute_error(y_te, y_pred_lr)
r2   = r2_score(y_te, y_pred_lr)

print("Régression Linéaire :")
print(f"  MSE  = {mse:.4f}")
print(f"  RMSE = {rmse:.4f}  (en 100k$, soit {rmse*1e5:.0f}$)")
print(f"  MAE  = {mae:.4f}")
print(f"  R²   = {r2:.4f}   ({r2*100:.1f}% de variance expliquée)")
```

### 4.2 Régression Ridge et Lasso (Régularisation)

| Méthode | Pénalité | Effet |
|---------|----------|-------|
| Ridge (L2) | $\lambda \sum \beta_j^2$ | Réduit tous les coefficients |
| Lasso (L1) | $\lambda \sum \|\beta_j\|$ | Annule certains coefficients (sélection de features) |
| Elastic Net | $\alpha L1 + (1-\alpha) L2$ | Combinaison Ridge + Lasso |

```python
from sklearn.linear_model import Ridge, Lasso, ElasticNet

alphas = [0.01, 0.1, 1.0, 10.0, 100.0]

print(f"{'Alpha':<10} {'Ridge R²':<12} {'Lasso R²':<12}")
print("-" * 36)
for alpha in alphas:
    ridge = Pipeline([('s', StandardScaler()), ('m', Ridge(alpha=alpha))])
    lasso = Pipeline([('s', StandardScaler()), ('m', Lasso(alpha=alpha, max_iter=5000))])
    
    r2_ridge = cross_val_score(ridge, X_tr, y_tr, cv=3, scoring='r2').mean()
    r2_lasso = cross_val_score(lasso, X_tr, y_tr, cv=3, scoring='r2').mean()
    print(f"{alpha:<10} {r2_ridge:<12.4f} {r2_lasso:<12.4f}")

# Coefficients Lasso — feature selection implicite
lasso_fit = Pipeline([('s', StandardScaler()), ('m', Lasso(alpha=0.1))])
lasso_fit.fit(X_tr, y_tr)
coefs = lasso_fit.named_steps['m'].coef_
print(f"\nCoefficients Lasso (alpha=0.1) :")
for fname, coef in zip(X_reg.columns, coefs):
    print(f"  {fname:<25} : {coef:+.4f}")
print(f"Features nullifiées : {(coefs == 0).sum()} / {len(coefs)}")
```

---

## 5. Classification

### 5.1 Régression Logistique

Malgré son nom, c'est un **classifieur**. Il modélise la probabilité via la fonction sigmoid :

$$P(y=1 | x) = \sigma(w^T x + b) = \frac{1}{1 + e^{-( w^T x + b)}}$$

```python
from sklearn.linear_model import LogisticRegression

df_clf = df[['pclass', 'age', 'fare', 'sibsp', 'parch', 'sex', 'survived']].dropna()
df_clf['sex_num'] = (df_clf['sex'] == 'female').astype(int)
X_clf = df_clf[['pclass', 'age', 'fare', 'sibsp', 'parch', 'sex_num']].values
y_clf = df_clf['survived'].values

X_tr, X_te, y_tr, y_te = train_test_split(X_clf, y_clf, test_size=0.2,
                                            random_state=42, stratify=y_clf)

pipe_lr = Pipeline([
    ('scaler', StandardScaler()),
    ('model', LogisticRegression(C=1.0, max_iter=1000, random_state=42))
])
pipe_lr.fit(X_tr, y_tr)
y_pred = pipe_lr.predict(X_te)
y_proba = pipe_lr.predict_proba(X_te)[:, 1]  # probabilité de la classe 1

print("Régression Logistique :")
print(f"  Accuracy  : {accuracy_score(y_te, y_pred):.4f}")
print(f"  Precision : {precision_score(y_te, y_pred):.4f}")
print(f"  Recall    : {recall_score(y_te, y_pred):.4f}")
print(f"  F1-Score  : {f1_score(y_te, y_pred):.4f}")
print(f"  ROC-AUC   : {roc_auc_score(y_te, y_proba):.4f}")
```

### 5.2 Métriques de Classification en Détail

```python
from sklearn.metrics import confusion_matrix, classification_report

cm = confusion_matrix(y_te, y_pred)
print("Matrice de Confusion :")
print(f"[[TN={cm[0,0]}, FP={cm[0,1]}]")
print(f" [FN={cm[1,0]}, TP={cm[1,1]}]]")
print()
print("Rapport :")
print(classification_report(y_te, y_pred, target_names=['Décédé', 'Survivant']))
```

> [!info] Métriques expliquées
> | Métrique | Formule | Quand l'utiliser |
> |----------|---------|-----------------|
> | **Accuracy** | (TP+TN) / Total | Classes équilibrées |
> | **Precision** | TP / (TP+FP) | Éviter les faux positifs |
> | **Recall** | TP / (TP+FN) | Éviter les faux négatifs (maladies) |
> | **F1-Score** | 2 × (P×R)/(P+R) | Compromis precision/recall |
> | **ROC-AUC** | Aire sous la courbe ROC | Évaluation globale du classifieur |

### 5.3 K-Nearest Neighbors (KNN)

```python
from sklearn.neighbors import KNeighborsClassifier

# Test de différentes valeurs de K
k_range = range(1, 31)
cv_scores = []

for k in k_range:
    knn = Pipeline([
        ('scaler', StandardScaler()),
        ('model', KNeighborsClassifier(n_neighbors=k))
    ])
    scores = cross_val_score(knn, X_tr, y_tr, cv=5, scoring='accuracy')
    cv_scores.append(scores.mean())

best_k = k_range[np.argmax(cv_scores)]
print(f"Meilleur K : {best_k} (accuracy : {max(cv_scores):.4f})")

knn_best = Pipeline([
    ('scaler', StandardScaler()),
    ('model', KNeighborsClassifier(n_neighbors=best_k))
])
knn_best.fit(X_tr, y_tr)
print(f"Test accuracy (K={best_k}) : {accuracy_score(y_te, knn_best.predict(X_te)):.4f}")
```

### 5.4 Support Vector Machine (SVM)

```python
from sklearn.svm import SVC

svm_pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('model', SVC(kernel='rbf', C=1.0, gamma='scale',
                  probability=True, random_state=42))
])
svm_pipe.fit(X_tr, y_tr)
y_pred_svm = svm_pipe.predict(X_te)
y_proba_svm = svm_pipe.predict_proba(X_te)[:, 1]

print("SVM (RBF kernel) :")
print(f"  Accuracy : {accuracy_score(y_te, y_pred_svm):.4f}")
print(f"  ROC-AUC  : {roc_auc_score(y_te, y_proba_svm):.4f}")
```

### 5.5 Decision Tree et Random Forest

```python
from sklearn.tree import DecisionTreeClassifier, export_text
from sklearn.ensemble import RandomForestClassifier

# Decision Tree
dt = DecisionTreeClassifier(max_depth=5, min_samples_leaf=10, random_state=42)
dt.fit(X_tr, y_tr)
print(f"Decision Tree (max_depth=5) :")
print(f"  Train accuracy : {accuracy_score(y_tr, dt.predict(X_tr)):.4f}")
print(f"  Test accuracy  : {accuracy_score(y_te, dt.predict(X_te)):.4f}")

# Random Forest — ensemble de Decision Trees
rf = RandomForestClassifier(
    n_estimators=200,
    max_depth=10,
    min_samples_leaf=5,
    max_features='sqrt',
    random_state=42,
    n_jobs=-1    # utiliser tous les cœurs CPU
)
rf.fit(X_tr, y_tr)
y_pred_rf = rf.predict(X_te)
y_proba_rf = rf.predict_proba(X_te)[:, 1]

print(f"\nRandom Forest (200 arbres) :")
print(f"  Accuracy : {accuracy_score(y_te, y_pred_rf):.4f}")
print(f"  ROC-AUC  : {roc_auc_score(y_te, y_proba_rf):.4f}")

# Feature Importance
feature_names = ['pclass', 'age', 'fare', 'sibsp', 'parch', 'sex_num']
importances = pd.Series(rf.feature_importances_, index=feature_names)
print(f"\nFeature Importance :")
print(importances.sort_values(ascending=False).round(4))
```

---

## 6. Gradient Boosting — XGBoost et LightGBM

> [!info] Principe du Boosting
> Le boosting entraîne des modèles **séquentiellement** : chaque nouveau modèle corrige les erreurs du précédent. Différent du Random Forest (arbres en parallèle, indépendants).

```python
# pip install xgboost lightgbm
try:
    import xgboost as xgb
    from lightgbm import LGBMClassifier

    # XGBoost
    xgb_model = xgb.XGBClassifier(
        n_estimators=300,
        max_depth=6,
        learning_rate=0.05,
        subsample=0.8,
        colsample_bytree=0.8,
        use_label_encoder=False,
        eval_metric='logloss',
        random_state=42,
        n_jobs=-1
    )
    xgb_model.fit(X_tr, y_tr, verbose=False)
    print(f"XGBoost   : {accuracy_score(y_te, xgb_model.predict(X_te)):.4f}")

    # LightGBM — plus rapide sur grands datasets
    lgbm_model = LGBMClassifier(
        n_estimators=300,
        max_depth=6,
        learning_rate=0.05,
        random_state=42,
        verbose=-1
    )
    lgbm_model.fit(X_tr, y_tr)
    print(f"LightGBM  : {accuracy_score(y_te, lgbm_model.predict(X_te)):.4f}")

except ImportError:
    print("XGBoost / LightGBM non installés : pip install xgboost lightgbm")
```

| Algorithme | Points Forts | Points Faibles |
|-----------|-------------|---------------|
| Decision Tree | Interprétable, rapide | Overfitting facile |
| Random Forest | Robuste, peu de tuning | Lent sur très grands datasets |
| XGBoost | Très performant, gère NaN | Beaucoup d'hyperparamètres |
| LightGBM | Très rapide, moins de mémoire | Sensible aux petits datasets |

---

## 7. Hyperparameter Tuning

### 7.1 GridSearchCV

```python
from sklearn.model_selection import GridSearchCV

# Définir la grille
param_grid = {
    'classifier__n_estimators': [50, 100, 200],
    'classifier__max_depth': [3, 5, 10, None],
    'classifier__min_samples_leaf': [1, 5, 10]
}

grid_pipeline = Pipeline([
    ('preprocessor', StandardScaler()),
    ('classifier', RandomForestClassifier(random_state=42, n_jobs=-1))
])

# ATTENTION : GridSearchCV sur X_tr uniquement, jamais X_te !
grid_search = GridSearchCV(
    grid_pipeline,
    param_grid,
    cv=5,
    scoring='roc_auc',
    n_jobs=-1,
    verbose=1
)
grid_search.fit(X_tr, y_tr)

print(f"Meilleurs paramètres : {grid_search.best_params_}")
print(f"Meilleur score CV    : {grid_search.best_score_:.4f}")
print(f"Score test final     : {roc_auc_score(y_te, grid_search.predict_proba(X_te)[:, 1]):.4f}")
```

### 7.2 RandomizedSearchCV — Plus Rapide sur de Grandes Grilles

```python
from sklearn.model_selection import RandomizedSearchCV
from scipy.stats import randint, uniform

# Distribution au lieu de liste discrète
param_dist = {
    'classifier__n_estimators': randint(50, 500),
    'classifier__max_depth': randint(3, 20),
    'classifier__min_samples_leaf': randint(1, 20),
    'classifier__max_features': uniform(0.1, 0.9)
}

random_search = RandomizedSearchCV(
    grid_pipeline,
    param_dist,
    n_iter=30,         # tester 30 combinaisons aléatoires
    cv=5,
    scoring='roc_auc',
    random_state=42,
    n_jobs=-1
)
random_search.fit(X_tr, y_tr)
print(f"RandomizedSearchCV meilleur score : {random_search.best_score_:.4f}")
```

---

## 8. Courbes d'Apprentissage — Underfitting vs Overfitting

```python
from sklearn.model_selection import learning_curve

def plot_learning_curve(estimator, X, y, title="Courbe d'apprentissage"):
    """Trace la courbe d'apprentissage train vs validation."""
    train_sizes, train_scores, val_scores = learning_curve(
        estimator, X, y,
        cv=5, scoring='accuracy',
        train_sizes=np.linspace(0.1, 1.0, 10),
        n_jobs=-1
    )
    
    train_mean = train_scores.mean(axis=1)
    train_std  = train_scores.std(axis=1)
    val_mean   = val_scores.mean(axis=1)
    val_std    = val_scores.std(axis=1)
    
    fig, ax = plt.subplots(figsize=(8, 5))
    ax.plot(train_sizes, train_mean, 'o-', color='blue', label='Train score')
    ax.fill_between(train_sizes, train_mean - train_std, train_mean + train_std,
                    alpha=0.1, color='blue')
    ax.plot(train_sizes, val_mean, 'o-', color='orange', label='Validation score')
    ax.fill_between(train_sizes, val_mean - val_std, val_mean + val_std,
                    alpha=0.1, color='orange')
    
    ax.set_xlabel("Taille du training set")
    ax.set_ylabel("Accuracy")
    ax.set_title(title)
    ax.legend()
    ax.grid(True)
    plt.tight_layout()
    plt.savefig(f'learning_curve_{title[:10].replace(" ","_")}.png', dpi=120)
    plt.close()
    
    return train_mean[-1], val_mean[-1]

# Comparer underfitting vs bon modèle vs overfitting
dt_under  = DecisionTreeClassifier(max_depth=1, random_state=42)
dt_good   = DecisionTreeClassifier(max_depth=5, random_state=42)
dt_over   = DecisionTreeClassifier(max_depth=None, random_state=42)  # pas de limite

for name, model in [('Underfitting (max_depth=1)', dt_under),
                     ('Bon compromis (max_depth=5)', dt_good),
                     ('Overfitting (pas de limite)', dt_over)]:
    train_sc, val_sc = plot_learning_curve(model, X_tr, y_tr, name)
    gap = train_sc - val_sc
    print(f"{name[:30]:30} : Train={train_sc:.3f}, Val={val_sc:.3f}, Gap={gap:.3f}")
```

> [!tip] Diagnostique
> - **Gap train - val très élevé** → Overfitting → Régularisation, moins de paramètres, plus de données
> - **Les deux scores faibles** → Underfitting → Modèle plus complexe, plus de features
> - **Les deux scores proches et élevés** → Bon compromis

---

## 9. Sauvegarder et Charger les Modèles

```python
import joblib
import os

# Sauvegarder le pipeline complet
best_model = grid_search.best_estimator_
model_path = 'titanic_model.joblib'
joblib.dump(best_model, model_path)
print(f"Modèle sauvegardé : {model_path} ({os.path.getsize(model_path)/1024:.1f} KB)")

# Charger le modèle
loaded_model = joblib.load(model_path)

# Prédiction avec le modèle chargé
X_nouveau = np.array([[1, 35, 85, 0, 0, 1],   # 1ère classe, femme, 35 ans
                       [3, 20, 8, 0, 0, 0]])   # 3ème classe, homme, 20 ans
predictions = loaded_model.predict(X_nouveau)
probabilities = loaded_model.predict_proba(X_nouveau)
print(f"\nPrédictions sur nouveaux passagers :")
for i, (pred, proba) in enumerate(zip(predictions, probabilities)):
    statut = 'Survivant' if pred == 1 else 'Décédé'
    print(f"  Passager {i+1} : {statut} (proba survie = {proba[1]:.3f})")
```

---

## 10. Comparaison Finale de Tous les Modèles

```python
from sklearn.svm import SVC

modeles = {
    'Logistic Regression': Pipeline([('s', StandardScaler()),
                                      ('m', LogisticRegression(max_iter=1000))]),
    'KNN (k=7)':           Pipeline([('s', StandardScaler()),
                                      ('m', KNeighborsClassifier(n_neighbors=7))]),
    'Decision Tree':       DecisionTreeClassifier(max_depth=5, random_state=42),
    'Random Forest':       RandomForestClassifier(n_estimators=200, random_state=42, n_jobs=-1),
    'SVM (RBF)':          Pipeline([('s', StandardScaler()),
                                     ('m', SVC(probability=True, random_state=42))])
}

print(f"{'Modèle':<25} {'Accuracy':>10} {'ROC-AUC':>10} {'F1':>10}")
print("-" * 58)
resultats = []
for nom, model in modeles.items():
    model.fit(X_tr, y_tr)
    y_p = model.predict(X_te)
    y_prob = model.predict_proba(X_te)[:, 1]
    
    acc  = accuracy_score(y_te, y_p)
    auc  = roc_auc_score(y_te, y_prob)
    f1   = f1_score(y_te, y_p)
    
    resultats.append({'Modèle': nom, 'Accuracy': acc, 'AUC': auc, 'F1': f1})
    print(f"{nom:<25} {acc:>10.4f} {auc:>10.4f} {f1:>10.4f}")

df_resultats = pd.DataFrame(resultats).sort_values('AUC', ascending=False)
print(f"\nMeilleur modèle (AUC) : {df_resultats.iloc[0]['Modèle']}")
```

---

## 11. Exercices Pratiques

> [!tip] Exercice 1 — Régression
> Sur le dataset California Housing (`fetch_california_housing`), entraînez une régression Ridge avec 5 valeurs d'alpha différentes. Utilisez GridSearchCV avec 5-fold CV. Comparez RMSE et R² sur le test set. Identifiez les features les plus importantes.

> [!tip] Exercice 2 — Classification multiclasse
> Chargez le dataset `wine` (`sklearn.datasets.load_wine`) — 3 classes. Entraînez un Random Forest et affichez la matrice de confusion, le rapport de classification. Calculez l'accuracy par classe.

> [!tip] Exercice 3 — Pipeline complet
> Sur le dataset Titanic, construisez un Pipeline complet avec `ColumnTransformer` (features numériques + catégorielles), un classifieur RandomForest, et un `GridSearchCV`. Obtenez un ROC-AUC > 0.86 sur le test set.

> [!tip] Exercice 4 — Courbe ROC
> Pour 3 modèles (Logistic Regression, Random Forest, SVM), tracez les courbes ROC sur le même graphique. Calculez l'AUC pour chacun. Quel modèle a la meilleure AUC ? Quand préfèrerait-on un modèle avec moins d'AUC mais plus de précision ?

> [!tip] Exercice 5 — Data Leakage
> Identifiez et corrigez le data leakage dans ce code : `scaler = StandardScaler(); X_scaled = scaler.fit_transform(X); X_tr, X_te = train_test_split(X_scaled, test_size=0.2)`. Quel est le problème exact ? Comment le corriger ?

---

## Liens

- [[01 - Mathematiques pour le ML]] — fondements mathématiques
- [[02 - NumPy Pandas et Visualisation]] — préparation des données
- [[04 - Reseaux de Neurones et Deep Learning]] — modèles plus profonds
- [[06 - ML Non Supervise et Reinforcement Learning]] — apprentissage sans labels

> [!info] Ressources
> - Documentation Scikit-Learn : https://scikit-learn.org/stable/
> - "Hands-On ML" — Aurélien Géron (O'Reilly)
> - Kaggle Learn : https://www.kaggle.com/learn
