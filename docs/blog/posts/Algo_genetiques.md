---
title: "Introduction aux l'algorithmes génétiques"
date: 2024-10-15
author: "Sebastien Tadiello"
description: >
    Exposer les concepts de bases de l'algorithmes génétiques
categories: ["C++","Optimisation","Mathematics","Machine Learning"]
# draft: true
---

# Introduction aux Algorithmes Génétiques

Les algorithmes génétiques (AG) sont des méthodes d'optimisation inspirées des processus de sélection naturelle (la biologie évolutive). Ils sont particulièrement utiles pour résoudre des problèmes complexes pour lesquels les méthodes traditionnelles sont inefficaces. Un AG commence par une population d'individus, puis procède par évaluations successives (appelées "générations") pour trouver une solution optimale. Bien évidemment avec cette méthode on ne trouve pas 'LA' solution optimale, mais une solution très approchante.

Dans cet article j'introduis un exemple très simple en C++, où nous essayons de maximiser une fonction simple: $f(x) = x^2$. Nous allons explorer les concepts fondamentaux de sélection, croisement, et mutation pour montrer comment un AG fonctionne en pratique. L'objectif étant de comprendre les bases de cette optimisation et de s'entrainer à la mettre en oeuvre. 

# Mise en Place en C++

Donc ici notre but est de trouver l'individu qui maximise la fonction $f(x) = x^2$.

Juste avant cela je veux clarifier les conditions nécessaires pour appliquer un algorithme génétique : 

- une fonction de fitness clairement définie (permet de comparer les individus entre eux), 
- un espace de recherche vaste ou non linéaire, 
- une absence de dérivabilité de la fonction (pas un point bloquant en soit mais c'est là quelles sont le plus utile!), 
- des mécanismes de variation (croisement et mutation), 
- des critères de sélection bien définis,
- une capacité à gérer le coût computationnel élevé.


## Explication des Concepts

Initialisation de la Population : La population est un ensemble d'individus générés aléatoirement, ici des entiers entre 0 et 99.

```C++
const int population_size = 10;
std::vector<int> population(population_size);
for (int &individual : population) {
    individual = rand() % 100;
}
```

Sélection : À chaque génération, les individus sont classés selon leur "fitness", c'est-à-dire leur valeur pour la fonction $f(x)$. Les meilleurs individus (ici, les 50% les plus aptes) sont sélectionnés pour continuer. (D'ailleur, on peut jouer sur ces critères pour faire varier les temps de résolution.)
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
    if (rand() % 100 < 5) {
        individual += rand() % 10 - 5;
    }
}
```

Dans notre exemple, l'algorithme génétique explore différentes valeurs aléatoires pour maximiser f(x) = x^2. Une approche par descente de gradient, quant à elle, ajusterait continuellement la valeur de x en fonction de la dérivée de f(x), permettant ainsi une convergence plus directe vers le maximum. Mais comme je l'ai préciser plus haut, les AG sont surtout utiliser qaunt les foonctions sont non différentiables. Ici dans ce cas précis, pour une fonction aussi simple, la descente de gradient pourrait être plus efficace, mais pour des problèmes plus complexes avec de nombreux points de décisions (où la descente de gradient risquerait de se bloquer dans des optima locaux), les AG présentent un avantage indéniable.

## Résultats : Différences entre Algorithmes Génétiques et Descente de Gradient

En termes de résultats, la descente de gradient tend à converger beaucoup plus rapidement vers une solution optimale que les algorithmes génétiques, surtout pour des fonctions simples et différentiables comme **f(x) = x^2**. La descente de gradient suit une trajectoire définie par la pente de la fonction, ce qui lui permet d'atteindre un maximum ou un minimum rapidement.

Cependant, cette rapidité peut aussi être un inconvénient car elle risque de se bloquer dans un **optimum local**, sans garantie d'avoir atteint le **maximum global** (je parle ici d'une decente de gradient traditionnelle et pas d'une de ses variante plus robuste). En revanche, les AG, bien que plus lents, explorent davantage l'espace de recherche grâce à leurs mécanismes de mutation et de croisement, ce qui leur permet de mieux éviter les optima locaux et de trouver des solutions plus proches de l'optimum global.

Dans notre exemple, la descente de gradient pourrait trouver le maximum beaucoup plus rapidement, mais l'algorithme génétique est plus robuste face aux irrégularités de la fonction et peut offrir de meilleures solutions si la fonction est complexe ou non convexe.

Lors de l'exécution des deux algorithmes sur la fonction $f(x) = x^2$, nous avons obtenu les résultats suivants :
- **Algorithme Génétique** : Convergence vers une valeur de **121** avec une fitness de **14641** après plusieurs générations.
- **Descente de Gradient** : Convergence vers une valeur de **120** avec une fitness de **14400**.

Ces résultats montrent que les deux méthodes convergent vers des solutions similaires, mais avec des valeurs légèrement différentes. L'algorithme génétique a atteint une solution légèrement meilleure (121 contre 120), mais la différence est relativement mineure. Cela illustre la capacité des AG à explorer l'espace de recherche plus largement, alors que la descente de gradient est plus rapide mais peut manquer certaines variations qui mèneraient à un meilleur résultat.

## Conclusion

Les algorithmes génétiques sont des outils puissants pour explorer de grands espaces de solutions et trouver des réponses optimales là où d'autres méthodes seraient trop coûteuses en calcul. Cet exemple de code en C++ montre les bases du fonctionnement d'un AG : sélection, croisement et mutation. Comparés à la descente de gradient, les AG offrent une exploration plus globale, mais peuvent être plus lents à converger.

N'hésitez pas à essayer d'ajuster les paramètres, comme la taille de la population ou le taux de mutation, pour observer leur impact sur les résultats. Les algorithmes génétiques peuvent être appliqués à une grande variété de problèmes d'optimisation, des jeux jusqu'à la conception industrielle.

[Code sur GitHub](https://github.com/sebDtSci/genetic_optimization)