# Introduction à NumPy

NumPy (Numerical Python) est une bibliothèque fondamentale pour le calcul scientifique en Python. Elle permet de manipuler des tableaux de données multidimensionnels et offre des fonctions mathématiques optimisées.

## Installation

Pour installer NumPy, utilisez la commande suivante :

```bash
pip install numpy
```

## Importation de NumPy

Avant d'utiliser NumPy, il faut l'importer :

```python
import numpy as np
```

## Création de tableaux (ndarray)

### A partir d'une liste

```python
arr = np.array([1, 2, 3, 4, 5])
print(arr)
```

### Matrice (tableau 2D)

```python
mat = np.array([[1, 2, 3], [4, 5, 6]])
print(mat)
```

### Tableaux de zéros, de uns et valeurs arbitraires

```python
zeros = np.zeros((3, 3))  # Matrice 3x3 remplie de 0
ones = np.ones((2, 2))    # Matrice 2x2 remplie de 1
rand = np.random.rand(3, 3)  # Matrice 3x3 avec valeurs aléatoires
```

### Espacements réguliers

```python
lin = np.linspace(0, 10, 5)  # 5 valeurs équidistantes entre 0 et 10
```

## Propriétés des tableaux

```python
print(arr.shape)  # Dimensions
print(arr.size)   # Nombre d'éléments
print(arr.dtype)  # Type des éléments
```

## Manipulation des tableaux

### Accès aux éléments

```python
print(arr[0])    # Premier élément
print(mat[1, 2]) # Élément à la ligne 1, colonne 2
```

### Slicing (extraction de sous-tableaux)

```python
print(arr[1:4])    # Éléments d'indice 1 à 3
print(mat[:, 1])   # Deuxième colonne
```

### Modification des valeurs

```python
arr[2] = 10
mat[0, 1] = 99
```

## Opérations mathématiques

```python
arr2 = np.array([10, 20, 30, 40, 50])
print(arr + arr2)  # Addition
print(arr * 2)     # Multiplication scalaire
print(np.sqrt(arr))  # Racine carrée
print(np.exp(arr))   # Exponentielle
```

## Fonctions utiles

```python
print(np.sum(arr))   # Somme des éléments
print(np.mean(arr))  # Moyenne
print(np.max(arr))   # Valeur max
print(np.min(arr))   # Valeur min
print(np.argmax(arr)) # Indice de la valeur max
print(np.argmin(arr)) # Indice de la valeur min
```

## Reshape et transposition

```python
reshaped = arr.reshape((5, 1))  # Changement de forme
transposed = mat.T  # Transposition
```

## Produit matriciel

```python
mat1 = np.array([[1, 2], [3, 4]])
mat2 = np.array([[5, 6], [7, 8]])
product = np.dot(mat1, mat2)  # Produit matriciel
```

## Conclusion

NumPy est une bibliothèque très puissante pour le calcul numérique. Son utilisation est essentielle en science des données, en apprentissage automatique et en analyse scientifique.

*\*Fait à 90% avec ChatGPT car il est beaucoup plus rapide que moi*