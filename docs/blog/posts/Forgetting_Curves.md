---
title: "Modélisation des Forgetting Curves avec des Réseaux Neuronaux Simples"
date: 2024-10-21
author: "Sebastien Tadiello"
description: >
    Modélisation des Forgetting Curves
categories: ["Python","Artificial Intelligence","Mathematics","Machine Learning","Memory"]
draft: true
---

De par mes ettudes en Science Cognitives, j'ai eu l'occasion de travailler sur la modélisation des fonctions cognitives. Aujourd'hui c'est ce que je vous propose dans cette article, modélisé la courbe de l'oubli!

# L'effet
La courbe de l'oubli (forgetting curve), introduite par Hermann Ebbinghaus à la fin du XIXe siècle, est une représentation graphique qui montre comment la mémoire se dégrade avec le temps en l'absence de répétition. On connais également ce phénomène sous le nom *"l'effet de primauté et l'effet de récence"*.
Ce phénomène est central dans la compréhension de la rétention d'information et peut être modélisé mathématiquement par des fonctions d'atténuation exponentielle. Dans cet article, nous allons examiner comment un modèle simple de réseau de neurones artificiel (perceptron multicouche, MLP) peut capturer les dynamiques de l'oubli à travers la modélisation de la probabilité de rappel en fonction du temps.

# Modélisation Mathématique
Le modèle théorique d'Ebbinghaus pour la courbe de l'oubli repose sur une décroissance exponentielle de la rétention d'information. Mathématiquement, on peut représenter cette courbe par la formule suivante :
$$
P(t) = P_O \cdot e^{-\lambda t}
$$
Où:
- $P(t)$ est la probabilité de rappel d'une information en temps $t$
- $P_O$ est la probabilité initiale de rappel (généralement égale à 1 au début)
- $\lambda$ est la décroissance exponentielle de la probabilité de rappel (une constante caractéristique de la vitesse d'oubli)


Dans notre modélisation, nous partons de cette base théorique pour générer des données simulées que nous allons utiliser pour entraîner un réseau de neurones simple. L'objectif est d'observer si ce réseau peut apprendre et reproduire les dynamiques de l'oubli en s'appuyant sur des données bruitées.