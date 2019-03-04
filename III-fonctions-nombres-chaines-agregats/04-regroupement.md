# [04 - Regroupement](https://openclassrooms.com/fr/courses/1959476-administrez-vos-bases-de-donnees-avec-mysql/1967834-regroupement)

Les fonctions d'agrégation groupent plusieurs lignes ensemble. Il est possible de regrouper les lignes en fonction d'un critère et d'obtenir ainsi plusieurs groupes distincts.

:pencil: **Exemple :** avec la fonction `COUNT(*)`, on peut compter le nombre de lignes dans la table `Animal`. On peut faire également des groupes par espèce et ainsi déterminer en une seule requête les nombres de chats, chiens, etc.

## 1. Regroupement sur un critère

### 1.1. L'utilisation de `GROUP BY`

`GROUP BY` permet de grouper les lignes selon un critère. Il se place après l'éventuelle clause `WHERE` (sinon directement après `FROM`), suivi du nom de la colonne à utiliser comme critère de regroupement.

**Syntaxe :**

```sql
SELECT ...
FROM nom_table
[WHERE condition]
GROUP BY nom_colonne;
```

:pencil: **Exemple 1 :** compter les lignes dans la table Animal en regroupant sur le critère de l'espèce (donc avec la colonne espece_id)

```sql
SELECT COUNT(*) AS nb_animaux FROM Animal
GROUP BY espece_id;
```

:pencil: **Exemple 2 :** compter les lignes dans la table Animal en regroupant sur le critère de l'espèce et en ne prenant en compte que les mâles

```sql
SELECT COUNT(*) AS nb_animaux FROM Animal
WHERE sexe = 'M'
GROUP BY espece_id;
```

### 1.2. Voir d'autres colonnes

:pencil: **Exemple 3 :** afficher l'espece_id à côté du nombre d'animaux de chaque groupement.

```sql
SELECT espece_id, COUNT(*) AS nb_animaux FROM Animal
GROUP BY espece_id;
```

:pencil: **Exemple 4 :** afficher le nom de chaque espèce à côté du nombre d'animaux de chaque groupement.

```sql
SELECT nom_courant, COUNT(*) AS nb_animaux FROM Animal
INNER JOIN Espece ON Espece.id = Animal.espece_id
GROUP BY nom_courant;
```

### 1.3. Colonnes sélectionnées

#### La règle SQL

Lors d'un groupement dans une requête avec `GROUP BY`, on ne peut sélectionner que deux types d'éléments dans la clause `SELECT` :

* une ou des colonnes ayant servi de critère pour le regroupement ;
* une fonction d'agrégation (agissant sur n'importe quelle colonne).

:pencil: **Exemple :** une espèce n'ayant pas de date de naissance, la requête suivante n'a pas de sens. En effet, chaque ligne représente une espèce. Il en est de même pour les colonnes sexe ou commentaires.

```sql
SELECT nom_courant, COUNT(*) AS nb_animaux, date_naissance
FROM Animal
INNER JOIN Espece ON Animal.espece_id = Espece.id
GROUP BY nom_courant;
```

Afin d'éviter les erreurs précédentes, par sécurité, **la sélection de colonnes n'étant pas dans les critères de groupement est interdite**.

:pencil: **Exemple :** afficher l'id de l'espèce, son nom et son nom latin à côté du nombre d'animaux de chaque groupement. Afin de respecter la règle précédente, on groupe sur 3 colonnes

```sql
SELECT Espece.id, nom_courant, Espece.nom_latin, COUNT(*) AS nb_animaux FROM Animal
INNER JOIN Espece ON Espece.id = Animal.espece_id
GROUP BY nom_courant, Espece.id, Espece.nom_latin;
```

#### Le cas de MySQL

MySQL est un SGBD extrêmement permissif. Concernant `GROUP BY`, MySQL ne génère pas d'erreurs si l'on sélectionne une colonne qui n'est pas dans les critères de regroupement.

### 1.4. Tri des données

De la même façon, il n'est possible de faire un tri des données qu'à partir d'une colonne faisant partie des critères de regroupement ou à partir d'une fonction d'agrégation. Un tri se fait avec la commande `ORDER BY` (par défaut, tri `ASC`).

:pencil: **Exemple :** afficher l'id de l'espèce, son nom et son nom latin à côté du nombre d'animaux, trier par nombre d'animaux croissant

```sql
SELECT Espece.id, nom_courant, Espece.nom_latin, COUNT(*) AS nb_animaux FROM Animal
INNER JOIN Espece ON Espece.id = Animal.espece_id
GROUP BY nom_courant, Espece.id, Espece.nom_latin
ORDER BY nb_animaux;
```

:warning: La norme SQL veut que l'on n'utilise pas d'expressions (fonction, opération mathématique...) dans `GROUP BY` ni dans `ORDER BY`

#### Tenir compte des champs à NULL dans un groupement

On effectue une jointure externe, puisqu'il faut tenir compte de toutes les espèces, même de celles n'ayant pas de correspondance dans la table `Animal`.

:pencil: **Exemple :** afficher le nombre d'animaux pour chaque espèce même pour celles n'étant pas possédées par l'élevage

```sql
SELECT Espece.nom_courant, COUNT(Animal.espece_id) AS nb_animaux
FROM Animal
RIGHT JOIN Espece ON Animal.espece_id = Espece.id --RIGHT puisque la table Espece est à droite
GROUP BY nom_courant;
```

## 2. Regroupement sur plusieurs critères

Il est possible de grouper sur plusieurs colonnes, mais jusqu'à présent, cela n'a servi qu'à pouvoir afficher correctement les colonnes voulues sans que cela n'influe sur les groupes. On n'avait donc qu'un seul critère représenté par plusieurs colonnes. Voyons un exemple avec deux critères différents (qui ne créent pas les mêmes groupes).

:pencil: **Exemple :** savoir combien d'animaux de chaque espèce sont présents dans la table Animal, ainsi que les nombres de mâles et de femelles, toutes espèces confondues.

```sql
-- nombre d'animaux regroupés par espèce
SELECT nom_courant, COUNT(*) AS nb_animaux FROM Animal
INNER JOIN Espece ON Espece.id = Animal.espece_id
GROUP BY nom_courant;

-- nombre d'animaux regroupés par genre
SELECT sexe, COUNT(*) AS nb_animaux FROM Animal
GROUP BY sexe;
```

En faisant un regroupement multicritère, il est possible de déterminer le nombre de mâles et de femelles par espèce dans la table Animal

:warning: L'ordre des critères a son importance !

On regroupe d'abord sur l'espèce, puis sur le sexe :

```sql
SELECT nom_courant, sexe, COUNT(*) AS nb_animaux FROM Animal
INNER JOIN Espece ON Espece.id = Animal.espece_id
GROUP BY nom_courant, sexe;
```

![capture03](https://user-images.githubusercontent.com/1475600/53732193-6c47f500-3e7d-11e9-8bfc-cbaf8196b6da.PNG)

En regroupant d'abord sur le sexe, puis sur l'espèce :

```sql
SELECT nom_courant, sexe, COUNT(*) as nb_animaux
FROM Animal
INNER JOIN Espece ON Espece.id = Animal.espece_id
GROUP BY sexe,nom_courant;
```

![capture04](https://user-images.githubusercontent.com/1475600/53732286-c8127e00-3e7d-11e9-91d6-ca0636a21cee.PNG)

## 3. Super-agrégats

## 4. Conditions sur les fonctions d'agrégation