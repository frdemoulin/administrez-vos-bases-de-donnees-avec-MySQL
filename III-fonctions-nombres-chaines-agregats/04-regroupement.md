# [04 - Regroupement](https://openclassrooms.com/fr/courses/1959476-administrez-vos-bases-de-donnees-avec-mysql/1967834-regroupement)

Les fonctions d'agrégation groupent plusieurs lignes ensemble. Il est possible de regrouper les lignes en fonction d'un critère et d'obtenir ainsi plusieurs groupes distincts.

:pencil: **Exemple :** avec la fonction `COUNT(*)`, on peut compter le nombre de lignes dans la table `Animal`. On peut faire également des groupes par espèce et ainsi déterminer en une seule requête le nombre de chats, de chiens, etc.

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

:pencil: **Exemple 1 :** compter les lignes dans la table Animal en regroupant sur le critère de l'espèce (donc avec la colonne `espece_id`)

```sql
SELECT COUNT(*) AS nb_animaux FROM Animal
GROUP BY espece_id;
```

<details>
<summary><b>Résultat de la requête</b></summary>

![capture05](https://user-images.githubusercontent.com/1475600/53732865-7b2fa700-3e7f-11e9-9f33-b074776ce44b.PNG)
</details>

<br />

:pencil: **Exemple 2 :** compter les lignes dans la table Animal en regroupant sur le critère de l'espèce et en ne prenant en compte que les mâles

```sql
SELECT COUNT(*) AS nb_males FROM Animal
WHERE sexe = 'M'
GROUP BY espece_id;
```

<details>
<summary><b>Résultat de la requête</b></summary>

![regroupement_male](https://user-images.githubusercontent.com/1475600/53732936-b205bd00-3e7f-11e9-9cd8-be722a50a7df.PNG)
</details>

### 1.2. Voir d'autres colonnes

:pencil: **Exemple 3 :** afficher l'`espece_id` à côté du nombre d'animaux de chaque groupement

```sql
SELECT espece_id, COUNT(*) AS nb_animaux FROM Animal
GROUP BY espece_id;
```

<details>
<summary><b>Résultat de la requête</b></summary>

![regroupement_espece_id_nb_animaux](https://user-images.githubusercontent.com/1475600/53733041-0315b100-3e80-11e9-90dd-0a9501893f82.PNG)
</details>

<br/>

:pencil: **Exemple 4 :** afficher le nom de chaque espèce à côté du nombre d'animaux de chaque groupement

```sql
SELECT nom_courant, COUNT(*) AS nb_animaux FROM Animal
INNER JOIN Espece ON Espece.id = Animal.espece_id
GROUP BY nom_courant;
```

<details>
<summary><b>Résultat de la requête</b></summary>

![regroupement_nom_courant_nb_animaux](https://user-images.githubusercontent.com/1475600/53733050-0d37af80-3e80-11e9-8d27-c85edfe3798a.PNG)
</details>

### 1.3. Colonnes sélectionnées

#### La règle SQL

Lors d'un groupement dans une requête avec `GROUP BY`, on ne peut sélectionner que deux types d'éléments dans la clause `SELECT` :

* une ou des colonnes ayant servi de critère pour le regroupement ;
* une fonction d'agrégation (agissant sur n'importe quelle colonne).

Une espèce n'ayant pas de date de naissance, la requête suivante n'a pas de sens. En effet, chaque ligne représente une espèce. Il en est de même pour les colonnes sexe ou commentaires.

```sql
SELECT nom_courant, COUNT(*) AS nb_animaux, date_naissance
FROM Animal
INNER JOIN Espece ON Animal.espece_id = Espece.id
GROUP BY nom_courant;
```

Afin d'éviter l'erreur précédente, par sécurité, **la sélection de colonnes n'étant pas dans les critères de groupement est interdite**.

:pencil: **Exemple :** afficher l'id de l'espèce, son nom et son nom latin à côté du nombre d'animaux de chaque groupement (afin de respecter la règle précédente, on groupe sur 3 colonnes)

```sql
SELECT Espece.id, nom_courant, Espece.nom_latin, COUNT(*) AS nb_animaux FROM Animal
INNER JOIN Espece ON Espece.id = Animal.espece_id
GROUP BY nom_courant, Espece.id, Espece.nom_latin;
```

<details>
<summary><b>Résultat de la requête</b></summary>

![regroupement_id_nom_courant_nom_latin_nb_animaux](https://user-images.githubusercontent.com/1475600/53733155-63a4ee00-3e80-11e9-92f0-0971d9657ee3.PNG)
</details>

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

<details>
<summary><b>Résultat de la requête</b></summary>

![regroupement_id_nom_courant_nom_latin_nb_animaux_order_by_nb_animaux](https://user-images.githubusercontent.com/1475600/53733257-c1d1d100-3e80-11e9-8d79-1a7416acf479.PNG)
</details>

<br/>

:warning: La norme SQL veut que l'on n'utilise pas d'expressions (fonction, opération mathématique...) dans `GROUP BY` ni dans `ORDER BY`

#### Tenir compte des champs à NULL dans un groupement

On effectue une jointure externe, puisqu'il faut tenir compte de toutes les espèces, même de celles n'ayant pas de correspondance dans la table `Animal`.

:pencil: **Exemple :** afficher le nombre d'animaux pour chaque espèce, même pour celles n'étant pas possédées par l'élevage

```sql
SELECT Espece.nom_courant, COUNT(Animal.espece_id) AS nb_animaux
FROM Animal
RIGHT JOIN Espece ON Animal.espece_id = Espece.id --RIGHT puisque la table Espece est à droite
GROUP BY nom_courant;
```

<details>
<summary><b>Résultat de la requête</b></summary>

![regroupement_nom_courant_nb_animaux_null](https://user-images.githubusercontent.com/1475600/53733351-0493a900-3e81-11e9-8f37-92d4b348a8f5.PNG)
</details>

## 2. Regroupement sur plusieurs critères

Il est possible de grouper sur plusieurs colonnes, mais jusqu'à présent, cela n'a servi qu'à pouvoir afficher correctement les colonnes voulues sans que cela n'influe sur les groupes. On n'avait donc qu'un seul critère représenté par plusieurs colonnes. Voyons un exemple avec deux critères différents (qui ne créent pas les mêmes groupes).

:pencil: **Exemple :** déterminer le nombre d'animaux de chaque espèce présents dans la table Animal, ainsi que les nombres de mâles et de femelles, toutes espèces confondues.

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

En regroupant d'abord sur l'espèce, puis sur le sexe :

```sql
SELECT nom_courant, sexe, COUNT(*) AS nb_animaux FROM Animal
INNER JOIN Espece ON Espece.id = Animal.espece_id
GROUP BY nom_courant, sexe;
```

<details>
<summary><b>Résultat de la requête</b></summary>

![capture03](https://user-images.githubusercontent.com/1475600/53732193-6c47f500-3e7d-11e9-8bfc-cbaf8196b6da.PNG)
</details>
<br />

En regroupant d'abord sur le sexe, puis sur l'espèce :

```sql
SELECT nom_courant, sexe, COUNT(*) as nb_animaux
FROM Animal
INNER JOIN Espece ON Espece.id = Animal.espece_id
GROUP BY sexe, nom_courant;
```

<details>
<summary><b>Résultat de la requête</b></summary>

![capture04](https://user-images.githubusercontent.com/1475600/53732286-c8127e00-3e7d-11e9-91d6-ca0636a21cee.PNG)
</details>

## 3. Super-agrégats

L'option `WITH ROLLUP` de `GROUP BY` affiche des lignes supplémentaires dans la table de résultats, représentant des "super-groupes" (ou super-agrégats), i.e. des "groupes de groupes".

### 3.1. Avec un critère de regroupement

:pencil: **Exemple :** afficher le nombre d'animaux par espèce, ainsi que le nombre total d'animaux

```sql
SELECT nom_courant, COUNT(*) AS nb_animaux FROM Animal
INNER JOIN Espece ON Espece.id = Animal.espece_id
GROUP BY nom_courant WITH ROLLUP;
```

<details>
<summary><b>Résultat de la requête</b></summary>

![regroupement_with_rollup](https://user-images.githubusercontent.com/1475600/53735674-e67d7700-3e87-11e9-84ce-a09e157ce0f3.PNG)
</details>
<br/>

La ligne supplémentaire `NULL 60` représente le regroupement des quatre groupes basé sur le critère `GROUP BY nom_courant` (60 correspond ainsi au nombre total d'animaux dans ces quatre groupes).

### 3.2. Avec deux critères de regroupement

:pencil: **Exemple :**

```sql
SELECT nom_courant, sexe, COUNT(*) AS nb_animaux FROM Animal
INNER JOIN Espece ON Espece.id = Animal.espece_id
WHERE sexe IS NOT NULL
GROUP BY nom_courant, sexe WITH ROLLUP;
```

<details>
<summary><b>Résultat de la requête</b></summary>

![regroupement_with_rollup_sexe_nom](https://user-images.githubusercontent.com/1475600/53735876-758a8f00-3e88-11e9-836f-965719597efe.PNG)
</details>
<br/>

**Explications :**

:small_blue_diamond: Les deux premières lignes correspondent aux nombres de chats mâles et femelles.

:small_blue_diamond: La troisième ligne est une ligne insérée par `WITH ROLLUP`, elle indique le nombre de chats (mâles et femelles).

>On a fait des groupes en séparant les espèces et les sexes, et `WITH ROLLUP` a créé des "super-groupes" en regroupant les sexes, mais en gardant les espèces séparées.

:small_blue_diamond: Le résultat de la requête indique également le nombre de chiens à la sixième ligne, de perroquets à la neuvième et de tortues à la douzième.

:small_blue_diamond: La toute dernière ligne est un "super-super-groupe" réunissant tous les groupes ensemble (nombre total d'animaux ayant un genre déterminé, i.e. `NOT NULL`).

:warning: L'ordre des critères est important ! En échangeant les critères `nom_courant` et `sexe`, les super-groupes ne correspondent pas aux espèces, mais aux sexes, c'est-à-dire au premier critère. Le regroupement se fait bien dans l'ordre donné par les critères.

```sql
SELECT nom_courant, sexe, COUNT(*) AS nb_animaux FROM Animal
INNER JOIN Espece ON Espece.id = Animal.espece_id
WHERE sexe IS NOT NULL
GROUP BY sexe, nom_courant WITH ROLLUP;
```

<details>
<summary><b>Résultat de la requête</b></summary>

![regroupement_with_rollup_sexe_nom_courant](https://user-images.githubusercontent.com/1475600/53739305-bb982080-3e91-11e9-9a2b-f48403648572.PNG)
</details>

### 3.3. NULL, c'est pas joli

Il est possible de ne pas afficher `NULL` dans les lignes des super-groupes en utilisant la fonction `COALESCE()`. Elle prend autant de paramètres que l'on veut et renvoie le premier paramètre `NOT NULL`.

:pencil: **Exemple :**

```sql
SELECT COALESCE(1, NULL, 3, 4); -- affiche 1
SELECT COALESCE(NULL, 2);       -- affiche 2
SELECT COALESCE(NULL, NULL, 3); -- affiche 3
```

Dans le cas des super-agrégats de l'élevage :

```sql
SELECT COALESCE(nom_courant, 'Total'), COUNT(*) AS nb_animaux FROM Animal
INNER JOIN Espece ON Espece.id = Animal.espece_id
GROUP BY nom_courant WITH ROLLUP;
```

<details>
<summary><b>Résultat de la requête</b></summary>

![regroupement_coalesce](https://user-images.githubusercontent.com/1475600/53740022-7e349280-3e93-11e9-8c8c-f0b18e5a5576.PNG)
</details>
<br/>

**Explications :**

:small_blue_diamond: Pour les groupes simples, `nom_courant` contient bien le nom de l'espèce, `COALESCE()` renvoie donc celui-ci.

:small_blue_diamond: Pour les super-groupes, la colonne `nom_courant` du résultat contient `NULL`, `COALESCE()` va donc renvoyer "Total".

>En utilisant `COALESCE()` dans ce genre de situation, il est impératif que les critères de regroupement ne contiennent pas `NULL` (ou que ces lignes-là soient éliminées). Sinon, "Total" s'affichera à des lignes qui ne sont pas des super-groupes.

## 4. Conditions sur les fonctions d'agrégation

Pour faire des conditions sur une fonction d'agrégation, on utilise la clause `HAVING` à placer juste après `GROUP BY`.

:warning: La clause `WHERE` ne s'applique pas à une fonction d'agrégation

:pencil: **Exemple :** afficher uniquement les espèces dont on possède plus de 15 individus

```sql
SELECT nom_courant, COUNT(*) AS nombre FROM Animal
INNER JOIN Espece ON Espece.id = Animal.espece_id
GROUP BY nom_courant
HAVING COUNT(*) > 15;
```

<details>
<summary><b>Résultat de la requête</b></summary>

![regroupement_having](https://user-images.githubusercontent.com/1475600/53741171-13388b00-3e96-11e9-86a2-71aaeba433f0.PNG)
</details>

### Optimisation

Les conditions données dans la clause `HAVING` ne doivent pas nécessairement comporter une fonction d'agrégation. Les deux requêtes suivantes donneront des résultats équivalents :

```sql
-- Affiche les animaux groupés par espèce, dont le nom courant commence par C et d'effectifs supérieurs à 6
SELECT nom_courant, COUNT(*) as nombre
FROM Animal
INNER JOIN Espece ON Espece.id = Animal.espece_id
GROUP BY nom_courant
HAVING nombre > 6 AND SUBSTRING(nom_courant, 1, 1) = 'C';
    -- Deux conditions dans HAVING

SELECT nom_courant, COUNT(*) as nombre
FROM Animal
INNER JOIN Espece ON Espece.id = Animal.espece_id
WHERE SUBSTRING(nom_courant, 1, 1) = 'C'
    -- Une condition dans WHERE
GROUP BY nom_courant
HAVING nombre > 6;
    -- Et une dans HAVING
```

:bulb: Il est cependant préférable d'utiliser la clause `WHERE` autant que possible pour toutes les conditions, sauf celles utilisant une fonction d'agrégation (les conditions `HAVING` ne sont absolument pas optimisées, à l'inverse des conditions `WHERE`)