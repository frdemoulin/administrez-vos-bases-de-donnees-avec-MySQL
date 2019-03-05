# [03 - Fonctions d'agrégation](https://openclassrooms.com/fr/courses/1959476-administrez-vos-bases-de-donnees-avec-mysql/1966846-fonctions-dagregation)

Les fonctions d'agrégation, ou de groupement, sont des fonctions qui regroupent les lignes. Elles agissent sur une colonne et renvoient un résultat unique pour toutes les lignes sélectionnées (ou pour chaque groupe de lignes).

Elles servent majoritairement à faire des statistiques (compter des lignes, connaître une moyenne, trouver la valeur maximale d'une colonne, etc).

<!-- Nous verrons ensuite la fonction GROUP_CONCAT()  qui, comme son nom l'indique, est une fonction de groupement qui sert à concaténer des valeurs. -->

## 1. Fonctions statistiques

## 1.1. Nombre de lignes

### `COUNT()`

`COUNT()` : permet de savoir combien de lignes sont sélectionnées par la requête

**Exemples :**

```sql
-- Afficher le nombre de races
SELECT COUNT(*) AS nb_races FROM Race;

-- Afficher le nombre de chiens
SELECT COUNT(*) AS nb_chiens FROM Animal
INNER JOIN Espece ON Espece.id = Animal.espece_id
WHERE Espece.nom_courant = 'Chien';
```

:bulb: `COUNT(*)` compte les lignes sans se soucier de ce qu'elles contiennent.

`COUNT(colonne)` compte seulement les lignes dont la valeur de `colonne` n'est pas `NULL`.

**Exemple :**

```sql
-- COUNT(race_id) compte les animaux ayant une race
-- COUNT(*) compte tous les animaux
SELECT COUNT(race_id), COUNT(*) FROM Animal;
```

### Doublons

Le mot-clé `DISTINCT` permet de ne pas prendre en compte les doublons avec la fonction `COUNT()` .

**Exemple :** compter le nombre de races différentes parmi les animaux dont la race est définie

```sql
SELECT COUNT(DISTINCT race_id) FROM Animal;
```

## 1.2. Minimum et maximum

`MIN(x)` et `MAX(x)` retournent respectivement la plus petite et la plus grande valeur de `x`.

**Exemples :** déterminer les prix mini et maxi des animaux ayant une race

```sql
SELECT MIN(prix), MAX(prix) FROM Race;
```

`MIN()` et `MAX()` s'appliquent également à des chaînes de caractère (par ordre alphabétique) ou à des dates (ordre chronologique).

**Exemple :**

```sql
SELECT MIN(nom), MAX(nom), MIN(date_naissance), MAX(date_naissance)
FROM Animal;
```

## 1.3. Somme et moyenne

`SUM(x)` renvoie la somme de `x`

**Exemple :**

```sql
SELECT SUM(prix) FROM Espece;
```

`AVG(x)` renvoie la valeur moyenne de `x`

**Exemple :**

```sql
SELECT AVG(prix) FROM Espece;
```

## 2. Concaténation

### 2.1. Principe

Avec les fonctions d'agrégation, on regroupe plusieurs lignes. Les fonctions statistiques permettent d'avoir des informations utiles sur le résultat d'une requête, mais il est intéressant d'avoir également les valeurs concernées.

La fonction `GROUP_CONCAT(nom_colonne)` concatène les valeurs de `nom_colonne` pour chaque groupement réalisé. Cela permet d'afficher les valeurs concernées par le résultat d'une fonction d'agrégation.

**Exemple :** récupérer la somme des prix de chaque espèce et afficher les espèces concernées

```sql
SELECT SUM(prix), GROUP_CONCAT(nom_courant) FROM Espece;
```

### 2.2. Syntaxe

```sql
GROUP_CONCAT(
              [DISTINCT] col1 [, col2, ...]
              [ORDER BY col [ASC | DESC]]
              [SEPARATOR sep]
            )
```

* `DISTINCT` : sert à éliminer les doublons.
* `col1` : nom de la colonne dont les valeurs doivent être concaténées (c'est le seul argument obligatoire)
* `col2, ...`  : éventuelles autres colonnes (ou chaînes de caractères) à concaténer.
* `ORDER BY` : détermine dans quel ordre les valeurs seront concaténées.
* `SEPARATOR` : spécifie une chaîne de caractères à utiliser pour séparer les différentes valeurs (défaut : virgule).

### 2.3. Exemples

```sql
-- --------------------------------------
-- CONCATENATION DE PLUSIEURS COLONNES --
-- --------------------------------------
SELECT SUM(Race.prix), GROUP_CONCAT(Race.nom, Espece.nom_courant)
FROM Race
INNER JOIN Espece ON Espece.id = Race.espece_id;

-- ---------------------------------------------------
-- CONCATENATION DE PLUSIEURS COLONNES EN PLUS JOLI --
-- ---------------------------------------------------
SELECT SUM(Race.prix), GROUP_CONCAT(Race.nom, ' (', Espece.nom_courant, ')')
FROM Race
INNER JOIN Espece ON Espece.id = Race.espece_id;

-- ---------------------------
-- ELIMINATION DES DOUBLONS --
-- ---------------------------
SELECT SUM(Espece.prix), GROUP_CONCAT(DISTINCT Espece.nom_courant) 
    -- Essayez sans le DISTINCT pour voir
FROM Espece
INNER JOIN Race ON Race.espece_id = Espece.id;

-- --------------------------
-- UTILISATION DE ORDER BY --
-- --------------------------
SELECT SUM(Race.prix), GROUP_CONCAT(Race.nom, ' (', Espece.nom_courant, ')' ORDER BY Race.nom DESC)
FROM Race
INNER JOIN Espece ON Espece.id = Race.espece_id;

-- ----------------------------
-- CHANGEMENT DE SEPARATEUR  --
-- ----------------------------
SELECT SUM(Race.prix), GROUP_CONCAT(Race.nom, ' (', Espece.nom_courant, ')' SEPARATOR ' - ')
FROM Race
INNER JOIN Espece ON Espece.id = Race.espece_id;
```