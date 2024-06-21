---
title: "Automatisation des Tests"
date: 2024-06-21
author: "Sebastien Tadiello"
description: >
    Setup Pytest with GitHub Actions
categories: ["DevOps", "Python", "Testing"]
---

# Automatisation des Tests

Vous le savez (enfin j'espère), les tests sont cruciaux pour tous les projets de développement. Mais les exécuter manuellement à chaque fois ? Pas très 2024 ! 🤖

Voici une astuce pour les développeurs soucieux d'efficacité : l'intégration de Pytest avec GitHub Actions.

Pytest est un framework de test pour vos applications Python. Et quand vous combinez cela avec la puissance des GitHub Actions, vous obtenez une suite de tests automatisés qui s'exécutent à chaque push ou pull request, assurant que vos modifications n'introduisent pas de régressions !

Pour configurer une action de base GitHub et exécuter vos tests Pytest, créez un fichier `.github/workflows/python-app.yml`​ dans votre repo et ajouter votre configuration comme sur mon exemple en photo.

![image](assets/unitest.jpeg)

🚀 Et voilà ! Vos tests se lancent automatiquement, vous offrant une tranquillité d'esprit.

Je suis persuadé que certains d'entre vous utilisent d'autres techniques ou des outils plus performants pour les tests, l'essentiel étant de toujours prévenir son code de toutes formes de régression !

