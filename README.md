
# django_auth_project

## Plan d'Action

- Configuration d'un nouveau projet Django
- Création de l'application principale
- Configuration des modèles de données
- Configuration des vues et des templates
- Mise en place du système d'authentification
- Ajout de fonctionnalités avancées pour les utilisateurs
- Amélioration de l'affichage du site web
- Déploiement du projet

### 1. Configuration d'un nouveau projet Django

Commençons par créer une nouvelle application Django pour ce projet. Depuis le terminal, nous pouvons le faire en utilisant l'une des commandes suivantes :

```bash
django-admin startproject auth_project
```

ou

```bash
python3 -m django startproject auth_project
```

Cela créera notre projet intitulé 'auth_project'. Ensuite, accédez au dossier `auth_project` et créez notre application principale `main_app` :

```bash
cd auth_project
python manage.py startapp main_app
```

Maintenant, lançons notre serveur et accédons à `localhost:8000` pour nous assurer que tout fonctionne :

```bash
python manage.py runserver
```

### 2. Création de l'application principale

Créez une application principale qui gérera les fonctionnalités principales du site. Ajoutez `main_app` à la liste des `INSTALLED_APPS` dans le fichier `settings.py` de votre projet.

```python
INSTALLED_APPS = [
    ...
    'main_app',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

### 3. Configuration des modèles de données

Définissons les modèles de données pour les utilisateurs dans `main_app/models.py`.

```python
from django.db import models
from django.contrib.auth.models import AbstractUser

class CustomUser(AbstractUser):
    # Ajoutez des champs supplémentaires ici si nécessaire
    bio = models.TextField(blank=True)
```

Mettez à jour le fichier `settings.py` pour utiliser notre modèle utilisateur personnalisé.

```python
AUTH_USER_MODEL = 'main_app.CustomUser'
```

Exécutez les migrations pour créer les tables de base de données correspondantes :

```bash
python manage.py makemigrations
python manage.py migrate
```

### 4. Configuration des vues et des templates

Créez des vues pour l'inscription, la connexion et la déconnexion des utilisateurs. Modifiez `main_app/views.py` pour inclure les vues nécessaires :

```python
from django.shortcuts import render, redirect
from django.contrib.auth import login, authenticate, logout
from django.contrib.auth.forms import UserCreationForm, AuthenticationForm
from django.contrib.auth.decorators import login_required

def home(request):
    return render(request, 'main_app/home.html')

def register(request):
    if request.method == 'POST':
        form = UserCreationForm(request.POST)
        if form.is_valid():
            user = form.save()
            login(request, user)
            return redirect('home')
    else:
        form = UserCreationForm()
    return render(request, 'main_app/register.html', {'form': form})

def login_view(request):
    if request.method == 'POST':
        form = AuthenticationForm(request, data=request.POST)
        if form.is_valid():
            user = form.get_user()
            login(request, user)
            return redirect('home')
    else:
        form = AuthenticationForm()
    return render(request, 'main_app/login.html', {'form': form})

@login_required
def profile(request):
    return render(request, 'main_app/profile.html')

def logout_view(request):
    if request.method == 'POST':
        logout(request)
        return redirect('home')
```

Créez les templates correspondants dans `main_app/templates/main_app/` :

- **home.html**
    ```html
    <!DOCTYPE html>
    <html>
    <head>
        <title>Home</title>
        <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css">
    </head>
    <body>
        <nav class="navbar navbar-expand-lg navbar-light bg-light">
            <a class="navbar-brand" href="{% url 'home' %}">Django Auth</a>
            <div class="collapse navbar-collapse">
                <ul class="navbar-nav ml-auto">
                    {% if user.is_authenticated %}
                        <li class="nav-item"><a class="nav-link" href="{% url 'profile' %}">Profil</a></li>
                        <li class="nav-item">
                            <form method="post" action="{% url 'logout' %}">
                                {% csrf_token %}
                                <button class="btn btn-link nav-link" type="submit">Déconnexion</button>
                            </form>
                        </li>
                    {% else %}
                        <li class="nav-item"><a class="nav-link" href="{% url 'login' %}">Connexion</a></li>
                        <li class="nav-item"><a class="nav-link" href="{% url 'register' %}">Inscription</a></li>
                    {% endif %}
                </ul>
            </div>
        </nav>
        <div class="container">
            <h1>Bienvenue sur notre site!</h1>
            {% if user.is_authenticated %}
                <p>Bonjour, {{ user.username }}!</p>
                <p><a href="{% url 'profile' %}">Mon Profil</a></p>
            {% else %}
                <p><a href="{% url 'login' %}">Se Connecter</a></p>
                <p><a href="{% url 'register' %}">S'inscrire</a></p>
            {% endif %}
        </div>
    </body>
    </html>
    ```

- **register.html**
    ```html
    <!DOCTYPE html>
    <html>
    <head>
        <title>S'inscrire</title>
        <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css">
    </head>
    <body>
        <div class="container">
            <h1>Créer un compte</h1>
            <form method="post">
                {% csrf_token %}
                {{ form.as_p }}
                <button type="submit" class="btn btn-primary">S'inscrire</button>
            </form>
        </div>
    </body>
    </html>
    ```

- **login.html**
    ```html
    <!DOCTYPE html>
    <html>
    <head>
        <title>Se Connecter</title>
        <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css">
    </head>
    <body>
        <div class="container">
            <h1>Se Connecter</h1>
            <form method="post">
                {% csrf_token %}
                {{ form.as_p }}
                <button type="submit" class="btn btn-primary">Se Connecter</button>
            </form>
        </div>
    </body>
    </html>
    ```

- **profile.html**
    ```html
    <!DOCTYPE html>
    <html>
    <head>
        <title>Mon Profil</title>
        <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css">
    </head>
    <body>
        <div class="container">
            <h1>Profil de {{ user.username }}</h1>
            <p>Email: {{ user.email }}</p>
            <p>Bio: {{ user.bio }}</p>
            <p><a href="{% url 'home' %}" class="btn btn-primary">Retour à l'accueil</a></p>
        </div>
    </body>
    </html>
    ```

### 5. Mise en place du système d'authentification

Ajoutez les URL pour les vues d'authentification dans `main_app/urls.py`.

```python
from django.urls import path
from .views import register, login_view, logout_view, home, profile

urlpatterns = [
    path('', home, name='home'),
    path('register/', register, name='register'),
    path('login/', login_view, name='login'),
    path('logout/', logout_view, name='logout'),
    path('profile/', profile, name='profile'),
]
```

Ajoutez les URLs de `main_app` au fichier `auth_project/urls.py`.

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('main_app.urls')),
]
```

### 6. Ajout de fonctionnalités avancées pour les utilisateurs

Ajoutez des fonctionnalités comme la modification du profil utilisateur. Modifiez `main_app/views.py` pour inclure la vue de mise à jour du profil :

```python
from django.contrib.auth.forms import UserChangeForm

@login_required
def update_profile(request):
    if request.method == 'POST':
        form = UserChangeForm(request.POST, instance=request.user)
        if form.is_valid():
            form.save()
            return redirect('profile')
    else:
        form = UserChangeForm(instance=request.user)
    return render(request, 'main_app/update_profile.html', {'form': form})
```

Créez le template correspondant `main_app/templates/main_app/update_profile.html` :

```html
<!DOCTYPE html>
<html>
<head>
    <title>Modifier Profil</title>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css">
</head>
<body>
    <div class="container">
        <h1>Modifier Profil</h1>
        <form method="post">
            {% csrf_token %}
            {{ form.as_p }}
            <button type="submit" class="btn btn-primary">Mettre à Jour</button>
        </form>
    </div>
</body>
</html>
```

Ajoutez l'URL correspondante dans `main_app/urls.py` :

```python
from .views import update_profile

urlpatterns = [
    ...
    path('update_profile/', update_profile, name='update_profile'),
]
```

### 7. Amélioration de l'affichage du site web

Pour améliorer l'affichage de votre site, vous pouvez utiliser le framework CSS Bootstrap. Ajoutez le lien vers Bootstrap dans vos templates de base, par exemple dans `base.html` :

```html
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}Site Web{% endblock %}</title>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css">
</head>
<body>
    <nav class="navbar navbar-expand-lg navbar-light bg-light">
        <a class="navbar-brand" href="{% url 'home' %}">Django Auth</a>
        <div class="collapse navbar-collapse">
            <ul class="navbar-nav ml-auto">
                {% if user.is_authenticated %}
                    <li class="nav-item"><a class="nav-link" href="{% url 'profile' %}">Profil</a></li>
                    <li class="nav-item">
                        <form method="post" action="{% url 'logout' %}">
                            {% csrf_token %}
                            <button class="btn btn-link nav-link" type="submit">Déconnexion</button>
                        </form>
                    </li>
                {% else %}
                    <li class="nav-item"><a class="nav-link" href="{% url 'login' %}">Connexion</a></li>
                    <li class="nav-item"><a class="nav-link" href="{% url 'register' %}">Inscription</a></li>
                {% endif %}
            </ul>
        </div>
    </nav>
    <div class="container">
        {% block content %}{% endblock %}
    </div>
</body>
</html>
```

Ensuite, étendez ce template dans vos autres fichiers HTML :

```html
{% extends 'base.html' %}

{% block title %}Accueil{% endblock %}

{% block content %}
<h1>Bienvenue sur notre site!</h1>
{% if user.is_authenticated %}
    <p>Bonjour, {{ user.username }}!</p>
    <p><a href="{% url 'profile' %}">Mon Profil</a></p>
{% else %}
    <p><a href="{% url 'login' %}">Se Connecter</a></p>
    <p><a href="{% url 'register' %}">S'inscrire</a></p>
{% endif %}
{% endblock %}
```

### 8. Déploiement du projet

Préparez votre projet pour le déploiement en configurant les fichiers `settings.py` pour la production, en configurant la base de données, et en utilisant un service comme Heroku ou AWS pour héberger votre application.

```bash
pip install gunicorn
pip install psycopg2-binary
```

Créez un fichier `Procfile` pour Heroku :

```
web: gunicorn auth_project.wsgi --log-file -
```

Configurez les paramètres de votre base de données PostgreSQL dans `settings.py` et assurez-vous que tous les fichiers statiques sont collectés pour le déploiement.

### Conclusion

Ce guide vous a montré comment créer un site web simple avec un système d'authentification en utilisant Django. En suivant ces étapes, vous avez appris à configurer un projet Django, à créer des modèles de données, à définir des vues et des templates, à gérer les utilisateurs et à déployer votre application. Amusez-vous à explorer davantage ces concepts et à les intégrer dans vos projets !
