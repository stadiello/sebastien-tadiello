---
title: La documentation technique et Sphinx
date: 2024-04-21
author: Sebastien Tadiello
description: >
    Create a documentation with Sphinx
categories: ["DevOps", "Python"]
---


# <h1 class="dic_titre">La documentation technique et Sphinx</h1>



Ici, je ne vous parlerai pas du README (qui est un guide rapide), mais plutôt de la documentation technique.

Une documentation claire et accessible est aussi cruciale que le code lui-même. Elle guide les utilisateurs et les contributeurs, facilitant l'utilisation et la contribution au projet.

Voici comment structurer et publier une documentation d'un projet Python, en utilisant Sphinx pour la génération et Read the Docs pour l'hébergement.



1. Structuration du Projet 📂 

<p align="justify">
La première étape est d'organiser le projet en suivant les meilleures pratiques, avec des dossiers distincts pour les sources (src/), les tests (tests/), la documentation (docs/) et le script setup.py. Cette structure claire facilite la navigation et la maintenance du projet. (Je ferai un post sur toute cette partie.)
</p>


2. Génération de Documentation avec Sphinx 📖

Sphinx transforme les docstrings en une documentation complète et bien formatée.

Nous utilisons des extensions comme autodoc pour générer automatiquement la documentation à partir des docstrings.


1/ Nous installons Sphinx :

``` zsh
pip install sphinx
```

2/ Nous créons les dossiers d'initialisation :

``` zsh
sphinx-quickstart docs
```

3/ Nous nous déplaçons dans docs et nous créons la documentation :

``` zsh
cd docs
make html
```

Nous pouvons supprimer la documentation avec :
 ``` zsh
make clean
```
<!-- <div> -->
3. Configuration pour Read the Docs 🌐

<P align="justify">
Avec un fichier .readthedocs.yaml, nous configurons le processus de build sur Read the Docs, en spécifiant la version de Python et les dépendances nécessaires. Une fois cela fait, rendez-vous sur ReadTheDoc et sélectionnez le repo contenant votre projet. Cela assure que la documentation est automatiquement mise à jour et accessible en ligne à chaque commit. (Vous trouverez un exemple dans l'image de ce post.)
</p>


Le résultat ? Une documentation en ligne toujours à jour, facilement accessible par les utilisateurs et les contributeurs.
<p align="justify">
La documentation ne doit jamais être une réflexion après coup dans le développement logiciel. Elle est essentielle pour la transparence, l'accessibilité et la réussite à long terme d'un projet.
<p>
