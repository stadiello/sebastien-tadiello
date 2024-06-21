---
title: "Sécurité de vos codes"
date: 2024-06-21
author: "Sebastien Tadiello"
description: >
    Setup Bandit with GitHub Actions
categories: ["DevOps", "Python", "Security"]
---
# Sécurité de vos codes

La sécurité avec Bandit, un outil d'analyse de sécurité de code pour Python.

🔍 Pourquoi Bandit ?

Analyse automatique : Bandit passe en revue votre code pour détecter les usages courants qui peuvent être dangereux pour la sécurité.

Intégration facile : S'intègre dans vos pipelines CI/CD pour des contrôles de sécurité continus.

Mettre en place Bandit est aussi simple que:

``` zsh
​pip install bandit
bandit -r your_code.py​
```

Ajouter à un pipeline GitHub Action, à la suite des tests unitaires:

``` yaml
​name: Security check​
​run: bandit -r your_code.py
```

Vous pouvez aussi l'intégrer dans un nouveau workflows en ajoutant un nouveau fichier .yml.

Jetez un œil à l'image jointe pour voir Bandit en action dans un workflow GitHub Actions. Aucune vulnérabilité trouvée, c'est exactement ce que nous voulons voir !

![image](assets/1709585808775.jfif)

Avec Bandit, votre code est surveillé pour éviter les erreurs courantes qui pourraient le compromettre et ce à chaque push ! C'est super pratique et cela doit devenir une habitude.