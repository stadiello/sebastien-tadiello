# Bases Mathématiques pour l'IA

Voici un aperçu des notions essentielles utilisées dans le domaine. C'est ce qu'on utilise 99% du temps.

## 1. Algèbre Linéaire

L'algèbre linéaire est au cœur des algorithmes d'IA, notamment pour la manipulation des données et l'entraînement des modèles.

### 1.1 Vecteurs

Un vecteur est une liste ordonnée de nombres.

```python
import numpy as np
v = np.array([1, 2, 3])  # Vecteur 3D
print(v)
```

### 1.2 Matrices

Une matrice est une collection bidimensionnelle de nombres.

```python
M = np.array([[1, 2], [3, 4]])  # Matrice 2x2
print(M)
```

### 1.3 Opérations

```python
v1 = np.array([1, 2, 3])
v2 = np.array([4, 5, 6])
print(v1 + v2)  # Addition
print(np.dot(v1, v2))  # Produit scalaire
```

## 2. Calcul Différentiel

Le calcul différentiel est essentiel pour l'optimisation des modèles d'IA, notamment dans l'entraînement des réseaux de neurones.

### 2.1 Dérivée

La dérivée mesure le taux de variation d'une fonction.

```python
import sympy as sp
x = sp.Symbol('x')
f = x**2
derivative = sp.diff(f, x)
print(derivative)  # Résultat : 2*x
```

### 2.2 Gradient

Le gradient est une généralisation de la dérivée pour les fonctions multivariables.

```python
x, y = sp.symbols('x y')
f = x**2 + y**2
grad = [sp.diff(f, var) for var in (x, y)]
print(grad)  # Résultat : [2*x, 2*y]
```

## 3. Probabilités et Statistiques

Les modèles d'IA reposent souvent sur des concepts probabilistes.

### 3.1 Loi des Probabilités

```python
from scipy.stats import norm
mean, std = 0, 1  # Moyenne et écart-type
p = norm.pdf(0, mean, std)  # Densité de probabilité en 0
print(p)
```

### 3.2 Espérance et Variance

```python
data = np.array([1, 2, 3, 4, 5])
print(np.mean(data))  # Espérance
print(np.var(data))   # Variance
```

## 4. Optimisation et Descente de Gradient

L'optimisation permet d'ajuster les paramètres d'un modèle pour minimiser une fonction de coût.

### 4.1 Fonction de Coût

```python
def cost_function(x):
    return (x - 3)**2  # Fonction quadratique
```

### 4.2 Algorithme de Descente de Gradient

```python
def gradient_descent(lr=0.1, epochs=10):
    x = 0  # Initialisation
    for _ in range(epochs):
        grad = 2 * (x - 3)  # Dérivée de la fonction
        x -= lr * grad  # Mise à jour
        print(x)

gradient_descent()
```

## Conclusion

Ces bases mathématiques sont essentielles pour comprendre et implémenter les modèles d'intelligence artificielle. Une maîtrise de ces concepts facilite l'analyse et l'optimisation des algorithmes d'apprentissage automatique.

