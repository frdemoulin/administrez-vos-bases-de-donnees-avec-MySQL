# [01 - Rappels et introduction](https://openclassrooms.com/fr/courses/1959476-administrez-vos-bases-de-donnees-avec-mysql/1966033-rappels-et-introduction)

## 1. Rappels et manipulation simple de nombres

### Rappels

Avec la commande `SELECT`, on peut afficher des nombres ou des chaînes de caractères.

**Exemples :**

```sql
# affiche 3 - Bonjour !
SELECT 3, 'Bonjour !';
```

Il est également possible d'effectuer des opérations mathématiques simples.

**Exemple :**

```sql
# affiche 8 - 2.6667 - 11.0000 - 6.0000
SELECT 3+5, 8/3, 10+2/2, (10+2)/2;
```

Six opérateurs mathématiques sont utilisables avec MySQL :

![capture](https://user-images.githubusercontent.com/1475600/53686331-e1081b80-3d25-11e9-9afc-c4619a94f628.PNG)

On peut les combiner avec les données stockées en base.

**Exemple 1 :** connaître le prix à payer pour acheter 3 individus de la même espèce (sans considérer la race)

```sql
SELECT nom_courant, 3*prix AS prix_trio FROM espece;
```

**Exemple 2 :** augmenter le prix de toutes les races de 35

```sql
UPDATE Race SET prix = prix + 35;
```

## 2. Définition d'une fonction

### Définition

Une _fonction_ est un code qui effectue une série d'instructions bien précises et renvoie le résultat de ces instructions. Une fonction est définie par son nom et ses paramètres.

**Exemple :** la fonction `MIN()` détermine la valeur minimale d'une série de valeurs passées en paramètres

### Paramètres

Un _paramètre_ de fonction est une donnée (ou un ensemble de données) que l'on fournit à la fonction afin qu'elle puisse effectuer son action.

Une fonction peut avoir aucun, un ou plusieurs paramètres. Dans le cas d'une fonction ayant plusieurs paramètres, l'ordre dans lequel on donne ces paramètres est important.

On parle aussi des _arguments_ d'une fonction.

**Exemple :** pour `MIN()`, il faut passer un paramètre : les données parmi lesquelles on souhaite récupérer la valeur minimale  

### Appel d'une fonction

Pour appeler une fonction, il suffit de donner son nom suivi des paramètres éventuels entre parenthèses (lesquelles sont obligatoires, même s'il n'y a aucun paramètre).

:warning: Pas d'espace entre le nom de la fonction et la parenthèse ouvrante !

**Exemples :**

```sql
-- Fonction sans paramètre
SELECT PI();    -- renvoie le nombre pi avec 5 décimales

-- Fonction avec un paramètre
SELECT MIN(prix) AS minimum  -- il est bien sûr possible d'utiliser les alias !
FROM Espece;

-- Fonction avec plusieurs paramètres
SELECT REPEAT('fort ! Trop ', 4);  -- répète une chaîne (ici : 'fort ! Trop ', répété 4 fois)

-- Même chose qu'au-dessus, mais avec les paramètres dans le mauvais ordre
SELECT REPEAT(4, 'fort ! Trop '); -- la chaîne de caractères 'fort ! Trop ' va être convertie en entier par MySQL, ce qui donne 0. "4" va donc être répété zéro fois...
```

### Fonctions scalaires vs fonctions d'agrégation

On distingue deux types de fonctions : les fonctions scalaires et les fonctions d'agrégation (ou fonctions de groupement) :

* les fonctions scalaires s'appliquent à chaque ligne indépendamment ;
* les fonctions d'agrégation regroupent les lignes (par défaut, elles regroupent toutes les lignes en une seule).

**Exemple de fonction scalaire :** la fonction `ROUND(X)` arrondit X à l'entier le plus proche. Il s'agit d'une fonction scalaire.

```sql
# on veut connaître le prix de vente des chiens de race arrondi à l'entier
SELECT nom, prix, ROUND(prix) FROM Race;
```

Il y a sept races dans la table `Race`. Lorsqu'on applique la fonction `ROUND(X)` à la colonne `prix`, on récupère donc sept lignes.

![capture01](https://user-images.githubusercontent.com/1475600/53686781-0566f680-3d2c-11e9-8f00-6838c5ff372a.PNG)

**Exemple de fonction d'agrégation :** La fonction `MIN(X)` est une fonction d'agrégation. 

```sql
# on cherche le prix de vente minimal des chiens de race
SELECT MIN(prix) FROM Race;
```

![capture02](https://user-images.githubusercontent.com/1475600/53686783-17489980-3d2c-11e9-98dc-a1fd69bf9655.PNG)

On ne récupère qu'une seule ligne de résultat. Les sept races ont été regroupées (cela n'aurait pas de sens d'avoir une ligne par race).

Cette particularité des fonctions d'agrégation les rend un peu plus délicates à utiliser, mais offre des possibilités ntéressantes.

## 3. Quelques fonctions générales

### 3.1. Informations sur l'environnement actuel

#### Version de MySQL

`VERSION()` affiche la version de MySQL installée sur le serveur.

#### Utilisateurs

`CURRENT_USER()` : renvoie l'utilisateur (et l'hôte) qui a été utilisé lors de l'identification au serveur

`USER()` : renvoie l'utilisateur (et l'hôte) qui a été spécifié lors de l'identification au serveur

### 3.2. Informations sur la dernière requête

#### Dernier ID généré par auto-incrémentation

`LAST_INSERT_ID()` : renvoie le dernier id créé par auto-incrémentation, pour la connexion utilisée (donc si quelqu'un se connecte au même serveur, avec un autre client, il n'influera pas sur le `LAST_INSERT_ID()` que vous recevrez)

**Exemple :** ajouter Pipo le rottweiller dans la base. Pour ce faire, on insère une nouvelle race (rottweiller), puis un nouvel animal (Pipo) pour lequel on a besoin de l'id de la nouvelle race

```sql
# insertion de la race Rottweiler dans la table Race
INSERT INTO Race (nom, espece_id, description, prix) VALUES ('Rottweiller', 1, 'Chien d''apparence solide, bien musclé, à la robe noire avec des taches feu bien délimitées.', 600.00);

# insertion d'un nouvel animal (Pipo le rottweiller)
INSERT INTO Animal (sexe, date_naissance, nom, espece_id, race_id) VALUES ('M', '2010-11-05', 'Pipo', 1, LAST_INSERT_ID()); -- LAST_INSERT_ID() renverra ici l'id de la race Rottweiller
```

#### Nombre de lignes renvoyées par la requête

`FOUND_ROWS()` : affiche le nombre de lignes que la dernière requête a rapportées

```sql
SELECT id, nom, espece_id, prix 
FROM Race;

SELECT FOUND_ROWS();
```

option `SQL_CALC_FOUND_ROWS` avec `LIMIT` : placée juste après le mot-clé `SELECT`, donne le nombre de lignes que la requête aurait rapportées en l'absence de `LIMIT`

```sql
SELECT id, nom, espece_id, prix -- Sans option
FROM Race 
LIMIT 3;              
         
SELECT FOUND_ROWS() AS sans_option;

SELECT SQL_CALC_FOUND_ROWS id, nom, espece_id, prix -- Avec option
FROM Race 
LIMIT 3; 

SELECT FOUND_ROWS() AS avec_option;
```

### 3.3. Convertir le type de données

#### Conversions automatiques

MySQL est assez permissif au sujet des conversions de type de données.

**Exemples :**

```sql
# la colonne id est de type INT. Pourtant, la comparaison avec une chaîne de caractères renvoie bien un résultat
SELECT * FROM Espece WHERE id = '3';

# la colonne prix est de type DECIMAL, mais l'insertion d'une valeur sous forme de chaîne de caractères ne pose pas de problème
INSERT INTO Espece (nom_latin, nom_courant, description, prix)
VALUES ('Rattus norvegicus', 'Rat brun', 'Petite bestiole avec de longues moustaches et une longue queue sans poils', '10.00');
```

#### Conversions forcées

Fonction `CAST(expr AS type)` : `expr` représente la donnée à convertir et `type` le type vers lequel convertir la donnée.

Ce type peut être : `BINARY`, `CHAR`, `DATE`, `DATETIME`, `TIME`, `UNSIGNED` (sous-entendu `INT`), `SIGNED` (sous-entendu `INT`), `DECIMAL`.

**Exemple :** conversion d'une chaîne de caractères en date

```sql
SELECT CAST('870303' AS DATE);
```