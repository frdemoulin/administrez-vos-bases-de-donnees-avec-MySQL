# [02 - Fonctions scalaires](https://openclassrooms.com/fr/courses/1959476-administrez-vos-bases-de-donnees-avec-mysql/1966680-fonctions-scalaires)

## 1. Manipulation de nombres

### 1.1. Arrondis

`CEIL()`

`FLOOR()`

`ROUND()`

`TRUNCATE()`

### 1.2. Exposants et racines

`POWER(n,e)`

`SQRT(n)`

### 1.3. Valeurs aléatoires

`RAND()`

### 1.4. Divers

`SIGN()`

`ABS()`

`MOD()`

## 2. Manipulation de chaînes de caractères

### 2.1. Longueur et comparaison

#### Longueur

`BIT_LENGTH(chaine)`

`CHAR_LENGTH(chaine)`

`LENGTH(chaine)`

#### Comparaison

`STRCMP(chaine1, chaine2)`

### 2.2. Retrait et ajout de caractères

`REPEAT(c, n)`

`LPAD()` et `RPAD()`

`TRIM()`

`SUBSTRING()`

### 2.3. Recherche et remplacement

#### Rechercher une chaîne de caractères

INSTR(), LOCATE()  et POSITION()

#### Changer la casse des chaînes

`LOWER(chaine)` et `LCASE(chaine)` mettent toutes les lettres de chaine en minuscules, tandis que `UPPER(chaine)` et `UCASE(chaine)` mettent toutes les lettres en majuscules.

#### Récupérer la partie gauche ou droite

LEFT(chaine, long)  retourne les long premiers caractères de chaine en partant de la gauche, et RIGHT(chaine, long)  fait la même chose en partant de la droite.

#### Inverser une chaîne

`REVERSE(chaine)` renvoie chaine en inversant les caractères.

#### Remplacer une partie par autre chose

`INSERT()` et `REPLACE()`

#### Concaténation

`CONCAT()` et `CONCAT_WS()`

#### Tri

`FIELD(rech, chaine1, chaine2, chaine3, ...)` recherche le premier argument (`rech`) parmi les arguments suivants (`chaine1`, `chaine2`, `chaine3`...) et retourne l'index auquel `rech` est trouvé (1 si `rech = chaine1`, 2 si `rech = chaine2`...). Si `rech` n'est pas trouvé parmi les arguments, 0 est renvoyé. On peut donc l’utiliser pour trier selon un ordre arbitraire de la manière suivante :

**Exemple :**

```sql
SELECT * FROM Animal ORDER BY FIELD(espece_id, 3, 1, 2);
```

Cette requête triera les lignes en donnant d’abord les animaux qui n’ont ni 3, ni 1, ni 2 comme espece_id (`FIELD()` renverra 0 pour ces lignes), suivis des animaux avec l’espece_id 3 (`FIELD()` renvoie 1), puis l’espece_id 1 et enfin l’espece_id 2.

#### Code ASCII

`ASCII()`

`CHAR()`

## 3. Exemples d'application et exercices

1. Afficher une phrase donnant le prix de l'espèce, pour chaque espèce (exemple : "un chat coûte 100 euros")

    ```sql
    SELECT CONCAT('Un(e) ', nom_courant, ' coûte ', prix, ' euros.') AS Solution FROM Espece;
    ```

1. Afficher les chats dont la deuxième lettre du nom est un "a"

    ```sql
    SELECT Animal.nom, Espece.nom_courant
    FROM Animal
    INNER JOIN Espece ON Espece.id = Animal.espece_id
    WHERE Espece.nom_courant = 'Chat'
    AND SUBSTRING(nom FROM 2 FOR 1) = 'a';
    ```

1. Afficher les noms des perroquets en remplaçant les "a" par "@" et les "e" par "3" pour en faire des perroquets Kikoolol

    ```sql
    SELECT REPLACE(REPLACE(Animal.nom, 'a', '@'), 'e', '3'), Espece.nom_courant
    FROM Animal
    INNER JOIN Espece ON Espece.id = Animal.espece_id
    WHERE Espece.nom_courant LIKE 'Perroquet%';
    ```

1. Afficher les chiens dont le nom a un nombre pair de lettres

    ```sql
    SELECT Animal.nom, CHAR_LENGTH(Animal.nom) AS nb_caracteres_nom, Espece.nom_courant
    FROM Animal
    INNER JOIN Espece ON Espece.id = Animal.espece_id
    WHERE Espece.nom_courant = 'Chien'
    AND MOD(CHAR_LENGTH(Animal.nom), 2) = 0;
    ```