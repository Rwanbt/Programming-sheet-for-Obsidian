# Django Framework Complet

> [!info] Django — "The web framework for perfectionists with deadlines"
> Django est un framework web Python full-stack, batteries incluses. Il fournit ORM, admin, authentification, templates, formulaires et sécurité — tout intégré, prêt en production.

## Table des matières
1. [[#Django vs les alternatives]]
2. [[#Installation et structure projet]]
3. [[#Models — ORM Django]]
4. [[#Views — Vues et logique]]
5. [[#URLs — Routage]]
6. [[#Templates]]
7. [[#Forms — Formulaires]]
8. [[#Admin Django]]
9. [[#Authentification et permissions]]
10. [[#Django REST Framework (DRF)]]
11. [[#ORM Avancé]]
12. [[#Middleware]]
13. [[#Signaux]]
14. [[#Testing Django]]
15. [[#Déploiement]]
16. [[#Exercices Pratiques]]

---

## Django vs les alternatives

| Critère | Django | Flask | FastAPI |
|---------|--------|-------|---------|
| **Philosophie** | Batteries included | Micro-framework | API-first, async |
| **ORM intégré** | Oui (puissant) | Non | Non |
| **Admin auto** | Oui (impressionnant) | Non | Non |
| **Auth intégrée** | Oui | Non | Non |
| **Templates** | Oui (DTL) | Jinja2 | Non natif |
| **REST** | Via DRF | Via Flask-RESTful | Natif |
| **Async** | Partiel (ASGI depuis 3.1) | Non natif | Natif |
| **Performances** | Moyen | Bon | Excellent |
| **Courbe d'apprentissage** | Élevée | Faible | Moyenne |
| **Idéal pour** | CRUD apps, dashboards, MVP | APIs simples, microservices | APIs haute perf, ML serving |

> [!tip] Quand choisir Django
> - Application avec interface admin (backoffice, CMS, outil interne)
> - Projet où la vitesse de développement prime sur les performances brutes
> - Équipe qui veut des conventions fortes (moins de décisions à prendre)
> - Application avec authentification utilisateur complexe

---

## Installation et structure projet

### Installation

```bash
# Créer un environnement virtuel
python -m venv venv
source venv/bin/activate  # Linux/Mac
venv\Scripts\activate     # Windows

# Installer Django
pip install django

# Vérifier la version
python -m django --version

# Créer un projet
django-admin startproject monprojet .
# Le point . crée les fichiers dans le dossier courant

# Créer une application
python manage.py startapp blog
```

### Anatomie d'un projet Django

```
monprojet/
├── manage.py                  ← Commandes de gestion
├── monprojet/
│   ├── __init__.py
│   ├── settings.py            ← Configuration globale
│   ├── urls.py                ← URLs racine
│   ├── asgi.py                ← Point d'entrée ASGI (async)
│   └── wsgi.py                ← Point d'entrée WSGI (sync)
├── blog/                      ← Application "blog"
│   ├── __init__.py
│   ├── admin.py               ← Enregistrement admin
│   ├── apps.py                ← Config de l'app
│   ├── models.py              ← Modèles de données
│   ├── views.py               ← Vues (logique)
│   ├── urls.py                ← URLs de l'app
│   ├── forms.py               ← Formulaires
│   ├── serializers.py         ← Sérialiseurs DRF
│   ├── tests.py               ← Tests
│   ├── migrations/            ← Historique des migrations DB
│   │   └── 0001_initial.py
│   └── templates/
│       └── blog/
│           ├── base.html
│           └── post_list.html
├── static/                    ← Fichiers statiques
├── media/                     ← Uploads utilisateurs
└── requirements.txt
```

### Settings essentiels

```python
# settings.py
import os
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent

SECRET_KEY = os.environ.get('DJANGO_SECRET_KEY', 'dev-only-insecure-key')

DEBUG = os.environ.get('DJANGO_DEBUG', 'True') == 'True'

ALLOWED_HOSTS = os.environ.get('ALLOWED_HOSTS', 'localhost').split(',')

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # Applications tierces
    'rest_framework',
    'corsheaders',
    # Nos applications
    'blog',
    'users',
]

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('DB_NAME', 'monprojet'),
        'USER': os.environ.get('DB_USER', 'postgres'),
        'PASSWORD': os.environ.get('DB_PASSWORD', ''),
        'HOST': os.environ.get('DB_HOST', 'localhost'),
        'PORT': os.environ.get('DB_PORT', '5432'),
    }
}

STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'staticfiles'  # pour collectstatic
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'

# Internationalisation
LANGUAGE_CODE = 'fr-fr'
TIME_ZONE = 'Europe/Paris'
USE_I18N = True
USE_TZ = True
```

---

## Models — ORM Django

### Types de champs courants

```python
# blog/models.py
from django.db import models
from django.contrib.auth import get_user_model
from django.urls import reverse

User = get_user_model()

class Categorie(models.Model):
    nom = models.CharField(max_length=100)
    slug = models.SlugField(unique=True)
    description = models.TextField(blank=True)
    
    class Meta:
        verbose_name = "Catégorie"
        verbose_name_plural = "Catégories"
        ordering = ['nom']
    
    def __str__(self):
        return self.nom


class Article(models.Model):
    
    class Statut(models.TextChoices):
        BROUILLON = 'draft', 'Brouillon'
        PUBLIE = 'published', 'Publié'
        ARCHIVE = 'archived', 'Archivé'
    
    # Champs texte
    titre = models.CharField(max_length=200)
    slug = models.SlugField(unique=True, max_length=200)
    contenu = models.TextField()
    extrait = models.CharField(max_length=500, blank=True)
    
    # Champs numériques
    vues = models.PositiveIntegerField(default=0)
    note = models.DecimalField(max_digits=3, decimal_places=1, null=True, blank=True)
    
    # Champs booléens
    est_a_la_une = models.BooleanField(default=False)
    
    # Champs date
    date_creation = models.DateTimeField(auto_now_add=True)  # Défini à la création
    date_modification = models.DateTimeField(auto_now=True)  # Mis à jour à chaque save
    date_publication = models.DateTimeField(null=True, blank=True)
    
    # Champs de choix
    statut = models.CharField(
        max_length=10,
        choices=Statut.choices,
        default=Statut.BROUILLON
    )
    
    # Fichiers
    image_couverture = models.ImageField(
        upload_to='articles/covers/%Y/%m/',
        null=True,
        blank=True
    )
    
    # Relations
    auteur = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='articles'
    )
    categorie = models.ForeignKey(
        Categorie,
        on_delete=models.SET_NULL,
        null=True,
        blank=True,
        related_name='articles'
    )
    tags = models.ManyToManyField('Tag', blank=True, related_name='articles')
    
    class Meta:
        ordering = ['-date_creation']
        indexes = [
            models.Index(fields=['statut', '-date_creation']),
            models.Index(fields=['slug']),
        ]
        verbose_name = "Article"
    
    def __str__(self):
        return self.titre
    
    def get_absolute_url(self):
        return reverse('blog:article-detail', kwargs={'slug': self.slug})
    
    @property
    def est_publie(self) -> bool:
        return self.statut == self.Statut.PUBLIE


class Tag(models.Model):
    nom = models.CharField(max_length=50, unique=True)
    slug = models.SlugField(unique=True)
    
    def __str__(self):
        return self.nom


class Commentaire(models.Model):
    article = models.ForeignKey(
        Article,
        on_delete=models.CASCADE,
        related_name='commentaires'
    )
    auteur = models.ForeignKey(User, on_delete=models.CASCADE)
    contenu = models.TextField(max_length=1000)
    date_creation = models.DateTimeField(auto_now_add=True)
    approuve = models.BooleanField(default=False)
    
    class Meta:
        ordering = ['date_creation']
    
    def __str__(self):
        return f"Commentaire de {self.auteur} sur {self.article}"
```

### Options `on_delete` pour ForeignKey

| Option | Comportement |
|--------|-------------|
| `CASCADE` | Supprime les objets liés |
| `SET_NULL` | Met null (nécessite `null=True`) |
| `SET_DEFAULT` | Met la valeur par défaut |
| `PROTECT` | Empêche la suppression |
| `RESTRICT` | Comme PROTECT mais plus strict |
| `DO_NOTHING` | Ne fait rien (risque d'intégrité) |

### Migrations

```bash
# Créer les migrations (analyse les models.py)
python manage.py makemigrations

# Appliquer les migrations
python manage.py migrate

# Voir le SQL généré sans l'appliquer
python manage.py sqlmigrate blog 0001

# État des migrations
python manage.py showmigrations
```

### QuerySet API

```python
from blog.models import Article
from django.utils import timezone

# --- Lecture ---
# Tous les articles
Article.objects.all()

# Filtrer
Article.objects.filter(statut='published')
Article.objects.filter(auteur__username='alice')  # traversée de FK
Article.objects.filter(vues__gte=100)             # >=
Article.objects.filter(titre__icontains='django') # LIKE %django% insensible

# Exclure
Article.objects.exclude(statut='archived')

# Obtenir un seul (lève DoesNotExist ou MultipleObjectsReturned)
article = Article.objects.get(slug='mon-article')

# first() / last() — retournent None si vide
Article.objects.filter(statut='published').first()

# --- Tri ---
Article.objects.order_by('-date_creation')       # DESC
Article.objects.order_by('categorie__nom', '-vues')

# --- Limiter ---
Article.objects.all()[:10]        # LIMIT 10
Article.objects.all()[10:20]      # OFFSET 10 LIMIT 10

# --- Valeurs ---
Article.objects.values('titre', 'slug')          # dicts
Article.objects.values_list('titre', flat=True)  # liste de valeurs

# --- Agrégation ---
from django.db.models import Count, Avg, Sum, Max, Min

Article.objects.aggregate(
    total=Count('id'),
    moyenne_vues=Avg('vues')
)

# --- Annotation (ajouter un champ calculé) ---
from django.db.models import Count

categories = Categorie.objects.annotate(
    nb_articles=Count('articles')
).order_by('-nb_articles')

# Accès : categorie.nb_articles

# --- Vérifier existence ---
Article.objects.filter(slug='test').exists()

# --- Mise à jour en masse ---
Article.objects.filter(statut='draft').update(statut='published')

# --- Suppression en masse ---
Article.objects.filter(date_creation__year=2020).delete()
```

---

## Views — Vues et logique

### Function-Based Views (FBV)

```python
# blog/views.py
from django.shortcuts import render, get_object_or_404, redirect
from django.http import HttpRequest, HttpResponse, JsonResponse
from django.contrib.auth.decorators import login_required
from django.views.decorators.http import require_POST
from django.contrib import messages

from .models import Article, Categorie
from .forms import CommentaireForm


def article_list(request: HttpRequest) -> HttpResponse:
    """Liste des articles publiés avec filtrage."""
    articles = Article.objects.filter(
        statut='published'
    ).select_related('auteur', 'categorie')
    
    # Filtrage par catégorie
    categorie_slug = request.GET.get('categorie')
    if categorie_slug:
        articles = articles.filter(categorie__slug=categorie_slug)
    
    # Recherche
    q = request.GET.get('q')
    if q:
        articles = articles.filter(titre__icontains=q)
    
    categories = Categorie.objects.annotate(
        nb=Count('articles')
    ).filter(nb__gt=0)
    
    context = {
        'articles': articles,
        'categories': categories,
        'q': q or '',
    }
    return render(request, 'blog/article_list.html', context)


def article_detail(request: HttpRequest, slug: str) -> HttpResponse:
    """Détail d'un article avec incrémentation des vues."""
    article = get_object_or_404(Article, slug=slug, statut='published')
    
    # Incrémenter les vues
    Article.objects.filter(pk=article.pk).update(vues=F('vues') + 1)
    
    commentaires = article.commentaires.filter(approuve=True)
    
    if request.method == 'POST':
        form = CommentaireForm(request.POST)
        if form.is_valid():
            commentaire = form.save(commit=False)
            commentaire.article = article
            commentaire.auteur = request.user
            commentaire.save()
            messages.success(request, "Commentaire soumis pour approbation.")
            return redirect(article)
    else:
        form = CommentaireForm()
    
    return render(request, 'blog/article_detail.html', {
        'article': article,
        'commentaires': commentaires,
        'form': form,
    })


@login_required
def article_create(request: HttpRequest) -> HttpResponse:
    from .forms import ArticleForm
    if request.method == 'POST':
        form = ArticleForm(request.POST, request.FILES)
        if form.is_valid():
            article = form.save(commit=False)
            article.auteur = request.user
            article.save()
            form.save_m2m()  # Sauvegarder les ManyToMany
            return redirect(article)
    else:
        form = ArticleForm()
    return render(request, 'blog/article_form.html', {'form': form})
```

### Class-Based Views (CBV)

```python
from django.views.generic import (
    ListView, DetailView, CreateView, UpdateView, DeleteView
)
from django.contrib.auth.mixins import LoginRequiredMixin, UserPassesTestMixin
from django.urls import reverse_lazy


class ArticleListView(ListView):
    model = Article
    template_name = 'blog/article_list.html'
    context_object_name = 'articles'
    paginate_by = 10
    
    def get_queryset(self):
        return Article.objects.filter(
            statut='published'
        ).select_related('auteur', 'categorie')
    
    def get_context_data(self, **kwargs):
        context = super().get_context_data(**kwargs)
        context['categories'] = Categorie.objects.all()
        return context


class ArticleDetailView(DetailView):
    model = Article
    template_name = 'blog/article_detail.html'
    context_object_name = 'article'
    slug_field = 'slug'
    
    def get_queryset(self):
        return Article.objects.filter(statut='published')


class ArticleCreateView(LoginRequiredMixin, CreateView):
    model = Article
    fields = ['titre', 'contenu', 'categorie', 'tags', 'statut']
    template_name = 'blog/article_form.html'
    
    def form_valid(self, form):
        form.instance.auteur = self.request.user
        return super().form_valid(form)


class ArticleUpdateView(LoginRequiredMixin, UserPassesTestMixin, UpdateView):
    model = Article
    fields = ['titre', 'contenu', 'categorie', 'tags', 'statut']
    template_name = 'blog/article_form.html'
    
    def test_func(self):
        article = self.get_object()
        return self.request.user == article.auteur


class ArticleDeleteView(LoginRequiredMixin, UserPassesTestMixin, DeleteView):
    model = Article
    template_name = 'blog/article_confirm_delete.html'
    success_url = reverse_lazy('blog:article-list')
    
    def test_func(self):
        return self.request.user == self.get_object().auteur
```

---

## URLs — Routage

```python
# monprojet/urls.py
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('blog.urls', namespace='blog')),
    path('api/', include('blog.api_urls', namespace='api')),
    path('accounts/', include('django.contrib.auth.urls')),
]

# Servir les médias en développement
if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

```python
# blog/urls.py
from django.urls import path
from . import views

app_name = 'blog'

urlpatterns = [
    path('', views.ArticleListView.as_view(), name='article-list'),
    path('article/<slug:slug>/', views.ArticleDetailView.as_view(), name='article-detail'),
    path('article/nouveau/', views.ArticleCreateView.as_view(), name='article-create'),
    path('article/<slug:slug>/modifier/', views.ArticleUpdateView.as_view(), name='article-update'),
    path('article/<slug:slug>/supprimer/', views.ArticleDeleteView.as_view(), name='article-delete'),
    path('categorie/<slug:slug>/', views.CategorieDetailView.as_view(), name='categorie-detail'),
]
```

```python
# Dans les templates — utilisation de reverse
{% url 'blog:article-detail' slug=article.slug %}
{% url 'blog:article-list' %}

# Dans le code Python
from django.urls import reverse, reverse_lazy

url = reverse('blog:article-detail', kwargs={'slug': 'mon-article'})
url_lazy = reverse_lazy('blog:article-list')  # pour les attributs de classe CBV
```

---

## Templates

### Langage de templates Django

```html
<!-- templates/blog/base.html -->
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <title>{% block title %}Mon Blog{% endblock %}</title>
    {% load static %}
    <link rel="stylesheet" href="{% static 'css/main.css' %}">
</head>
<body>
    <nav>
        <a href="{% url 'blog:article-list' %}">Accueil</a>
        {% if user.is_authenticated %}
            <span>Bonjour {{ user.get_full_name|default:user.username }}</span>
            <a href="{% url 'logout' %}">Déconnexion</a>
        {% else %}
            <a href="{% url 'login' %}">Connexion</a>
        {% endif %}
    </nav>
    
    {% if messages %}
        {% for message in messages %}
            <div class="alert alert-{{ message.tags }}">{{ message }}</div>
        {% endfor %}
    {% endif %}
    
    <main>
        {% block content %}{% endblock %}
    </main>
    
    {% block scripts %}{% endblock %}
</body>
</html>
```

```html
<!-- templates/blog/article_list.html -->
{% extends 'blog/base.html' %}
{% load humanize %}

{% block title %}Articles — Mon Blog{% endblock %}

{% block content %}
<h1>Articles</h1>

<!-- Filtres -->
<form method="get">
    <input type="search" name="q" value="{{ q }}" placeholder="Rechercher...">
    <button type="submit">Chercher</button>
</form>

<!-- Liste -->
{% for article in articles %}
    <article>
        <h2>
            <a href="{{ article.get_absolute_url }}">{{ article.titre }}</a>
        </h2>
        <p>
            Par {{ article.auteur.get_full_name }}
            le {{ article.date_creation|date:"d F Y" }}
            — {{ article.vues|intcomma }} vues
        </p>
        <p>{{ article.extrait|truncatewords:30 }}</p>
        
        {% if article.tags.exists %}
            <div>
                {% for tag in article.tags.all %}
                    <span class="tag">{{ tag.nom }}</span>
                {% endfor %}
            </div>
        {% endif %}
    </article>
{% empty %}
    <p>Aucun article trouvé.</p>
{% endfor %}

<!-- Pagination -->
{% if is_paginated %}
    <nav>
        {% if page_obj.has_previous %}
            <a href="?page={{ page_obj.previous_page_number }}">Précédent</a>
        {% endif %}
        
        <span>Page {{ page_obj.number }} / {{ page_obj.paginator.num_pages }}</span>
        
        {% if page_obj.has_next %}
            <a href="?page={{ page_obj.next_page_number }}">Suivant</a>
        {% endif %}
    </nav>
{% endif %}
{% endblock %}
```

### Filtres utiles

| Filtre | Exemple | Résultat |
|--------|---------|----------|
| `date` | `{{ dt\|date:"d/m/Y" }}` | `28/05/2026` |
| `truncatewords` | `{{ texte\|truncatewords:20 }}` | 20 premiers mots |
| `upper` / `lower` | `{{ nom\|upper }}` | Majuscules |
| `default` | `{{ val\|default:"N/A" }}` | N/A si vide |
| `length` | `{{ liste\|length }}` | Longueur |
| `linebreaks` | `{{ texte\|linebreaks }}` | `<p>` sur les sauts |
| `safe` | `{{ html\|safe }}` | Désactive l'échappement |
| `slugify` | `{{ titre\|slugify }}` | `mon-titre` |

> [!warning] Sécurité XSS
> Django échappe automatiquement toutes les variables `{{ var }}`. Utiliser `{{ var|safe }}` uniquement quand le contenu est **certifié sûr** (HTML généré par votre code, jamais par l'utilisateur).

---

## Forms — Formulaires

```python
# blog/forms.py
from django import forms
from django.core.exceptions import ValidationError
from .models import Article, Commentaire


class CommentaireForm(forms.ModelForm):
    class Meta:
        model = Commentaire
        fields = ['contenu']
        widgets = {
            'contenu': forms.Textarea(attrs={
                'rows': 4,
                'placeholder': 'Votre commentaire...',
                'class': 'form-control',
            })
        }
        labels = {
            'contenu': 'Commentaire'
        }
    
    def clean_contenu(self):
        contenu = self.cleaned_data['contenu']
        if len(contenu) < 10:
            raise ValidationError("Le commentaire doit faire au moins 10 caractères.")
        # Filtrage spam basique
        mots_interdits = ['spam', 'promo']
        for mot in mots_interdits:
            if mot.lower() in contenu.lower():
                raise ValidationError("Contenu non autorisé.")
        return contenu


class ArticleForm(forms.ModelForm):
    class Meta:
        model = Article
        fields = ['titre', 'contenu', 'extrait', 'categorie', 'tags',
                  'statut', 'est_a_la_une', 'image_couverture']
        widgets = {
            'contenu': forms.Textarea(attrs={'rows': 15}),
            'tags': forms.CheckboxSelectMultiple(),
        }
    
    def clean_titre(self):
        titre = self.cleaned_data['titre']
        if len(titre) < 5:
            raise ValidationError("Le titre doit faire au moins 5 caractères.")
        return titre.strip()
    
    def clean(self):
        """Validation croisée entre plusieurs champs."""
        cleaned_data = super().clean()
        statut = cleaned_data.get('statut')
        extrait = cleaned_data.get('extrait')
        
        if statut == 'published' and not extrait:
            raise ValidationError(
                "Un extrait est requis pour publier un article."
            )
        return cleaned_data
```

```html
<!-- Template formulaire avec CSRF -->
<form method="post" enctype="multipart/form-data">
    {% csrf_token %}
    
    {{ form.as_p }}
    
    <!-- Ou champ par champ : -->
    <div class="form-group">
        <label for="{{ form.titre.id_for_label }}">{{ form.titre.label }}</label>
        {{ form.titre }}
        {% if form.titre.errors %}
            <div class="error">{{ form.titre.errors }}</div>
        {% endif %}
    </div>
    
    <button type="submit">Enregistrer</button>
</form>
```

---

## Admin Django

```python
# blog/admin.py
from django.contrib import admin
from django.utils.html import format_html
from .models import Article, Categorie, Tag, Commentaire


@admin.register(Categorie)
class CategorieAdmin(admin.ModelAdmin):
    list_display = ['nom', 'slug', 'nb_articles']
    search_fields = ['nom']
    prepopulated_fields = {'slug': ('nom',)}
    
    def nb_articles(self, obj):
        return obj.articles.count()
    nb_articles.short_description = "Nombre d'articles"


class CommentaireInline(admin.TabularInline):
    model = Commentaire
    extra = 0
    fields = ['auteur', 'contenu', 'approuve', 'date_creation']
    readonly_fields = ['date_creation']


@admin.register(Article)
class ArticleAdmin(admin.ModelAdmin):
    list_display = ['titre', 'auteur', 'categorie', 'statut', 
                    'est_a_la_une', 'vues', 'date_creation', 'apercu_image']
    list_filter = ['statut', 'est_a_la_une', 'categorie', 'date_creation']
    search_fields = ['titre', 'contenu', 'auteur__username']
    prepopulated_fields = {'slug': ('titre',)}
    date_hierarchy = 'date_creation'
    ordering = ['-date_creation']
    
    filter_horizontal = ['tags']  # widget ManyToMany amélioré
    
    fieldsets = (
        ('Contenu', {
            'fields': ('titre', 'slug', 'contenu', 'extrait', 'image_couverture')
        }),
        ('Métadonnées', {
            'fields': ('auteur', 'categorie', 'tags', 'statut', 'est_a_la_une'),
        }),
        ('Statistiques', {
            'fields': ('vues',),
            'classes': ('collapse',),
        }),
    )
    
    readonly_fields = ['vues', 'date_creation', 'date_modification']
    inlines = [CommentaireInline]
    
    # Action personnalisée
    actions = ['publier_articles', 'archiver_articles']
    
    @admin.action(description="Publier les articles sélectionnés")
    def publier_articles(self, request, queryset):
        updated = queryset.update(statut='published')
        self.message_user(request, f"{updated} article(s) publié(s).")
    
    @admin.action(description="Archiver les articles sélectionnés")
    def archiver_articles(self, request, queryset):
        queryset.update(statut='archived')
    
    def apercu_image(self, obj):
        if obj.image_couverture:
            return format_html(
                '<img src="{}" style="max-height:50px;"/>',
                obj.image_couverture.url
            )
        return "—"
    apercu_image.short_description = "Image"
```

---

## Authentification et permissions

```python
# Vues d'authentification personnalisées
from django.contrib.auth import login, logout, authenticate
from django.contrib.auth.forms import AuthenticationForm
from django.contrib.auth.decorators import login_required, permission_required

@login_required(login_url='/connexion/')
def mon_profil(request):
    return render(request, 'users/profil.html', {'user': request.user})

@permission_required('blog.add_article', raise_exception=True)
def creer_article(request):
    pass

# Vérifier dans une vue
def ma_vue(request):
    if not request.user.is_authenticated:
        return redirect('login')
    if not request.user.has_perm('blog.change_article'):
        return HttpResponseForbidden()
```

### Modèle utilisateur personnalisé

```python
# users/models.py
from django.contrib.auth.models import AbstractUser
from django.db import models

class CustomUser(AbstractUser):
    bio = models.TextField(blank=True)
    avatar = models.ImageField(upload_to='avatars/', null=True, blank=True)
    site_web = models.URLField(blank=True)
    
    class Meta:
        verbose_name = "Utilisateur"

# settings.py
AUTH_USER_MODEL = 'users.CustomUser'
```

> [!warning] AUTH_USER_MODEL
> Définir `AUTH_USER_MODEL` **avant la première migration**. Changer après coup nécessite une migration complexe. Créer un modèle utilisateur personnalisé est une bonne pratique même si vous n'ajoutez aucun champ au départ.

---

## Django REST Framework (DRF)

```bash
pip install djangorestframework djangorestframework-simplejwt
```

### Serializers

```python
# blog/serializers.py
from rest_framework import serializers
from .models import Article, Categorie, Tag, Commentaire


class TagSerializer(serializers.ModelSerializer):
    class Meta:
        model = Tag
        fields = ['id', 'nom', 'slug']


class CategorieSerializer(serializers.ModelSerializer):
    nb_articles = serializers.IntegerField(read_only=True)
    
    class Meta:
        model = Categorie
        fields = ['id', 'nom', 'slug', 'nb_articles']


class ArticleListSerializer(serializers.ModelSerializer):
    """Sérialiseur léger pour les listes."""
    auteur_nom = serializers.CharField(source='auteur.get_full_name', read_only=True)
    categorie = CategorieSerializer(read_only=True)
    
    class Meta:
        model = Article
        fields = ['id', 'titre', 'slug', 'extrait', 'auteur_nom',
                  'categorie', 'vues', 'date_creation', 'statut']


class ArticleDetailSerializer(serializers.ModelSerializer):
    """Sérialiseur complet pour le détail."""
    auteur_nom = serializers.CharField(source='auteur.get_full_name', read_only=True)
    categorie = CategorieSerializer(read_only=True)
    tags = TagSerializer(many=True, read_only=True)
    tag_ids = serializers.PrimaryKeyRelatedField(
        queryset=Tag.objects.all(),
        many=True,
        write_only=True,
        source='tags'
    )
    
    class Meta:
        model = Article
        fields = [
            'id', 'titre', 'slug', 'contenu', 'extrait',
            'auteur_nom', 'categorie', 'tags', 'tag_ids',
            'vues', 'date_creation', 'date_modification', 'statut'
        ]
        read_only_fields = ['slug', 'vues', 'date_creation', 'date_modification']
    
    def validate_titre(self, value):
        if len(value) < 5:
            raise serializers.ValidationError("Titre trop court (min 5 chars).")
        return value
```

### ViewSets et Routers

```python
# blog/api_views.py
from rest_framework import viewsets, permissions, filters, status
from rest_framework.decorators import action
from rest_framework.response import Response
from django_filters.rest_framework import DjangoFilterBackend

from .models import Article
from .serializers import ArticleListSerializer, ArticleDetailSerializer


class ArticleViewSet(viewsets.ModelViewSet):
    queryset = Article.objects.select_related('auteur', 'categorie').prefetch_related('tags')
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]
    filter_backends = [DjangoFilterBackend, filters.SearchFilter, filters.OrderingFilter]
    filterset_fields = ['statut', 'categorie__slug']
    search_fields = ['titre', 'contenu']
    ordering_fields = ['date_creation', 'vues']
    ordering = ['-date_creation']
    
    def get_serializer_class(self):
        if self.action == 'list':
            return ArticleListSerializer
        return ArticleDetailSerializer
    
    def get_queryset(self):
        qs = super().get_queryset()
        if self.action == 'list':
            return qs.filter(statut='published')
        return qs
    
    def perform_create(self, serializer):
        serializer.save(auteur=self.request.user)
    
    @action(detail=True, methods=['post'])
    def publier(self, request, pk=None):
        """Action custom : publier un article."""
        article = self.get_object()
        if article.auteur != request.user:
            return Response(
                {'error': 'Non autorisé'},
                status=status.HTTP_403_FORBIDDEN
            )
        article.statut = 'published'
        article.save()
        return Response({'statut': 'publié'})
```

```python
# blog/api_urls.py
from rest_framework.routers import DefaultRouter
from .api_views import ArticleViewSet

router = DefaultRouter()
router.register('articles', ArticleViewSet, basename='article')

urlpatterns = router.urls
# Génère automatiquement :
# GET  /api/articles/            → list
# POST /api/articles/            → create
# GET  /api/articles/{id}/       → retrieve
# PUT  /api/articles/{id}/       → update
# PATCH /api/articles/{id}/      → partial_update
# DELETE /api/articles/{id}/     → destroy
# POST /api/articles/{id}/publier/ → publier (action custom)
```

---

## ORM Avancé

### Le problème N+1 et sa solution

```python
# ❌ N+1 : 1 requête pour les articles + N requêtes pour les auteurs
articles = Article.objects.filter(statut='published')
for article in articles:
    print(article.auteur.nom)  # Requête SQL à chaque itération !

# ✅ select_related : 1 seule requête avec JOIN (FK, OneToOne)
articles = Article.objects.filter(statut='published').select_related('auteur', 'categorie')

# ✅ prefetch_related : 2 requêtes (ManyToMany, reverse FK)
articles = Article.objects.filter(statut='published').prefetch_related('tags', 'commentaires')

# ✅ Combinaison
articles = Article.objects.filter(statut='published') \
    .select_related('auteur', 'categorie') \
    .prefetch_related('tags')
```

### Q objects — requêtes complexes

```python
from django.db.models import Q

# OR
Article.objects.filter(Q(titre__icontains='django') | Q(contenu__icontains='django'))

# AND avec NOT
Article.objects.filter(
    Q(statut='published') & ~Q(categorie__slug='archive')
)
```

### F expressions — comparaison de champs

```python
from django.db.models import F

# Tous les articles dont les vues dépassent les commentaires
Article.objects.filter(vues__gt=F('commentaires__count'))

# Incrémenter sans race condition
Article.objects.filter(pk=article.pk).update(vues=F('vues') + 1)
```

### Transactions

```python
from django.db import transaction

@transaction.atomic
def transferer_argent(compte_src, compte_dst, montant):
    compte_src.solde -= montant
    compte_src.save()
    
    if compte_dst.est_bloque:
        raise ValueError("Compte destination bloqué")  # Rollback automatique
    
    compte_dst.solde += montant
    compte_dst.save()

# Savepoint manuel
def operation_complexe():
    with transaction.atomic():
        creer_commande()
        try:
            with transaction.atomic():
                envoyer_notification()  # Peut échouer
        except Exception:
            pass  # L'envoi échoue mais la commande est sauvée
```

---

## Middleware

```python
# blog/middleware.py
import time
import logging

logger = logging.getLogger(__name__)


class RequestTimingMiddleware:
    """Mesure le temps de traitement de chaque requête."""
    
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        start = time.monotonic()
        response = self.get_response(request)
        duration = time.monotonic() - start
        
        if duration > 1.0:  # Log si > 1 seconde
            logger.warning(
                f"Requête lente: {request.method} {request.path} "
                f"({duration:.3f}s)"
            )
        
        response['X-Response-Time'] = f"{duration:.3f}s"
        return response


# Ajouter dans settings.py
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'blog.middleware.RequestTimingMiddleware',  # Ajouter ici
    # ...
]
```

---

## Signaux

```python
# blog/signals.py
from django.db.models.signals import post_save, pre_delete
from django.dispatch import receiver
from django.core.mail import send_mail
from .models import Article, Commentaire


@receiver(post_save, sender=Article)
def notifier_publication(sender, instance, created, **kwargs):
    """Notifier les abonnés à la publication d'un article."""
    if not created and instance.statut == 'published':
        # Envoyer newsletter...
        pass


@receiver(post_save, sender=Commentaire)
def notifier_auteur_commentaire(sender, instance, created, **kwargs):
    if created:
        send_mail(
            subject=f'Nouveau commentaire sur "{instance.article.titre}"',
            message=f'{instance.auteur} a commenté votre article.',
            from_email='noreply@monblog.fr',
            recipient_list=[instance.article.auteur.email],
        )


# Connecter dans apps.py
# blog/apps.py
from django.apps import AppConfig

class BlogConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'blog'
    
    def ready(self):
        import blog.signals  # Importer pour enregistrer les signaux
```

---

## Testing Django

```python
# blog/tests.py
from django.test import TestCase, Client
from django.contrib.auth import get_user_model
from django.urls import reverse
from .models import Article, Categorie

User = get_user_model()


class ArticleModelTest(TestCase):
    def setUp(self):
        self.user = User.objects.create_user(
            username='alice', password='testpass123'
        )
        self.categorie = Categorie.objects.create(nom='Django', slug='django')
        self.article = Article.objects.create(
            titre='Introduction à Django',
            slug='introduction-django',
            contenu='Contenu de test.',
            auteur=self.user,
            statut='published',
        )
    
    def test_str_representation(self):
        self.assertEqual(str(self.article), 'Introduction à Django')
    
    def test_est_publie_property(self):
        self.assertTrue(self.article.est_publie)
        self.article.statut = 'draft'
        self.assertFalse(self.article.est_publie)
    
    def test_get_absolute_url(self):
        url = self.article.get_absolute_url()
        self.assertEqual(url, '/article/introduction-django/')


class ArticleViewTest(TestCase):
    def setUp(self):
        self.client = Client()
        self.user = User.objects.create_user(username='bob', password='testpass123')
        self.article = Article.objects.create(
            titre='Test Article',
            slug='test-article',
            contenu='Contenu.',
            auteur=self.user,
            statut='published',
        )
    
    def test_article_list_returns_200(self):
        response = self.client.get(reverse('blog:article-list'))
        self.assertEqual(response.status_code, 200)
    
    def test_article_list_contains_article(self):
        response = self.client.get(reverse('blog:article-list'))
        self.assertContains(response, 'Test Article')
    
    def test_article_detail_returns_200(self):
        response = self.client.get(
            reverse('blog:article-detail', kwargs={'slug': self.article.slug})
        )
        self.assertEqual(response.status_code, 200)
    
    def test_create_article_requires_login(self):
        response = self.client.get(reverse('blog:article-create'))
        self.assertRedirects(response, '/connexion/?next=/article/nouveau/')
    
    def test_create_article_logged_in(self):
        self.client.login(username='bob', password='testpass123')
        response = self.client.post(reverse('blog:article-create'), {
            'titre': 'Mon Article',
            'contenu': 'Contenu suffisamment long.',
            'statut': 'draft',
        })
        self.assertEqual(Article.objects.filter(titre='Mon Article').count(), 1)
```

---

## Déploiement

### gunicorn + nginx

```bash
pip install gunicorn whitenoise django-environ

# Collecter les statics
python manage.py collectstatic --noinput

# Lancer gunicorn
gunicorn monprojet.wsgi:application \
    --workers 4 \
    --bind 0.0.0.0:8000 \
    --log-level info
```

### Dockerfile

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

RUN python manage.py collectstatic --noinput

EXPOSE 8000
CMD ["gunicorn", "monprojet.wsgi:application", "--workers", "4", "--bind", "0.0.0.0:8000"]
```

### settings production

```python
# settings.py avec django-environ
import environ

env = environ.Env(DEBUG=(bool, False))
environ.Env.read_env('.env')

SECRET_KEY = env('DJANGO_SECRET_KEY')
DEBUG = env('DEBUG')
ALLOWED_HOSTS = env.list('ALLOWED_HOSTS')

# Sécurité
SECURE_HSTS_SECONDS = 31536000
SECURE_SSL_REDIRECT = True
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True

# Whitenoise pour les statics
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'whitenoise.middleware.WhiteNoiseMiddleware',  # Juste après Security
    # ...
]
STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'
```

---

## Exercices Pratiques

### Exercice 1 — Modèles : Système de forum

Créez les modèles pour un forum :
- `Forum` : titre, description, ordre d'affichage
- `Sujet` : titre, forum (FK), auteur, date_creation, est_epingle, est_ferme, nb_vues
- `Message` : sujet (FK), auteur, contenu, date_creation, est_solution

Ajouter les `Meta`, `__str__`, `get_absolute_url`. Créer les migrations.

### Exercice 2 — API REST avec DRF

Créez une API pour un système de tâches (To-Do) :
- Modèle `Tache` : titre, description, terminee, priorite (1-5), date_echeance, utilisateur
- `ModelSerializer` avec validation (date_echeance dans le futur)
- `ViewSet` avec filtre par `terminee` et tri par `priorite`
- Authentification JWT avec `simplejwt`
- Tests : créer, lister, mettre à jour, supprimer

### Exercice 3 — Admin avancé

Pour le modèle `Tache` de l'exercice 2 :
- `list_display` : titre, utilisateur, priorite, terminee, date_echeance
- `list_filter` : terminee, priorite
- `search_fields` : titre, description
- Action custom "Marquer comme terminées"
- Champ `indicateur_retard` calculé (rouge si date_echeance dépassée et non terminée)

### Exercice 4 — Django complet : Application de recettes

Construire une app de partage de recettes :
- Models : Recette, Ingredient, IngredientRecette (table pivot avec quantité), Categorie, Note
- CRUD complet avec CBV
- Formulaire avec validation (au moins 2 ingrédients)
- API DRF : endpoint de recherche full-text
- Signal : recalculer la note moyenne à chaque nouvelle Note
- Tests : au moins 5 tests couvrant les modèles et les vues

---

## Liens et Références

- [[08 - APIs REST avec Flask]] — Comparaison avec Flask
- [[09 - APIs REST avec FastAPI]] — Alternative async pour les APIs
- [[10 - Python et Bases de Donnees]] — ORM et bases de données en Python
- [[01 - Introduction au SQL]] — SQL que l'ORM Django génère
