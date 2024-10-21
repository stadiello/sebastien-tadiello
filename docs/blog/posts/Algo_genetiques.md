---
title: "Introduction aux l'algorithmes génétiques"
date: 2024-10-15
author: "Sebastien Tadiello"
description: >
    Exposer les conceptes de bases de l'algorithmes génétiques
categories: ["C++","Optimisation","Mathematics","Machine Learning"]
draft: true
---

# Introduction aux Algorithmes Génétiques

Les algorithmes génétiques (AG) sont des méthodes d'optimisation inspirées des processus de sélection naturelle. Ils sont particulièrement utiles pour résoudre des problèmes complexes pour lesquels les méthodes traditionnelles sont inefficaces. Un AG commence par une population d'individus, puis procède par évaluations successives (appelées "générations") pour trouver une solution optimale.

Cet article introduit un exemple simple en C++, où nous essayons de maximiser une fonction simple: $f(x) = x^2$. Nous allons explorer les concepts fondamentaux de sélection, croisement, et mutation pour montrer comment un AG fonctionne en pratique.

# Mise en Place en C++

L'exemple suivant illustre un algorithme génétique simple écrit en C++. Le but est de trouver l'individu qui maximise la fonction $f(x) = x^2$.

## Explication des Concepts

Initialisation de la Population : La population est un ensemble d'individus générés aléatoirement, ici des entiers entre 0 et 99.

```C++
const int population_size = 10;
std::vector<int> population(population_size);
for (int &individual : population) {
    individual = rand() % 100;
}
```

Sélection : À chaque génération, les individus sont classés selon leur "fitness", c'est-à-dire leur valeur pour la fonction $f(x)$. Les meilleurs individus (ici, les 50% les plus aptes) sont sélectionnés pour continuer.
```C++
for (int generation = 0; generation < 100; ++generation) {
    std::sort(population.begin(), population.end(), [](int a, int b) {
        return fitness(a) > fitness(b);
    });
    population.resize(population_size / 2);

    ...
}
```

Croisement : Les individus sélectionnés sont croisés pour former de nouveaux individus. Le croisement consiste à prendre deux parents et à générer un enfant par une combinaison de leurs valeurs.
```C++	
while (population.size() < population_size) {
    int parent1 = population[rand() % (population_size / 2)];
    int parent2 = population[rand() % (population_size / 2)];
    int child = (parent1 + parent2) / 2; 
    population.push_back(child);
}
```

Mutation : Pour maintenir la diversité génétique, chaque individu a une petite chance (ici 5%) de subir une mutation, modifiant sa valeur aléatoirement.

```C++
for (int &individual : population) {
    if (rand() % 100 < 5) { // 5% de chance de mutation
        individual += rand() % 10 - 5;
    }
}
```