# [04 - Sous-requêtes](https://openclassrooms.com/fr/courses/1959476-administrez-vos-bases-de-donnees-avec-mysql/1964181-sous-requetes)

Une sous-requête est une requête à l'intérieur d'une autre requête. Avec le SQL, on peut construire des requêtes imbriquées sur autant de niveaux que désiré. On peut également mélanger jointures et sous-requêtes.

Une sous-requête peut être faite dans une requête de type `SELECT`, `INSERT`, `UPDATE` ou `DELETE` (et quelques autres).

Dans ce chapitre, on aborde les **requêtes de sélection** (les jointures et sous-requêtes pour la modification, l'insertion et la suppression de données seront traitées dans le prochain chapitre).

:bulb: Pour optimiser les performances d'une application, préférer les jointures aux requêtes lorsque c'est possible.

## 1. Sous-requêtes dans le FROM

Lors d'une requête de type `SELECT`, le résultat de la requête est envoyé sous forme de table. Et grâce aux sous-requêtes, il est tout à fait possible d'utiliser cette table et de refaire une recherche uniquement sur les lignes de celle-ci.

**Exemple :** en sélectionnant toutes les femelles parmi les perroquets et les tortues, on obtient un résultat sous forme de table

```sql
SELECT Animal.id, Animal.nom, Animal.sexe, Animal.date_naissance
FROM Animal
INNER JOIN Espece
    ON Espece.id = Animal.espece_id
WHERE sexe = 'F'
    AND Espece.nom_courant IN ('Perroquet amazone', 'Tortue d''Hermann');
```

<details>
<summary><b>Résultat de la requête</b></summary>

![capture_perroquet_tortue_femelle](https://user-images.githubusercontent.com/1475600/54316777-d54c0d00-45e1-11e9-83e3-ff8f41ab5381.JPG)
</details>
<br/>

**Exemple :** sous-requête utilisant la fonction d'agrégat `MIN`

Parmi les femelles perroquets et tortues précédentes, connaître la date de naissance de la plus âgée

```sql
SELECT MIN(date_naissance) FROM (
    SELECT Animal.id, Animal.nom, Animal.sexe, Animal.date_naissance
    FROM Animal
    INNER JOIN Espece
        ON Espece.id = Animal.espece_id
    WHERE sexe = 'F'
        AND Espece.nom_courant IN ('Perroquet amazone', 'Tortue d''Hermann')
) AS tortues_perroquets_F; -- on donne un alias à la table intermédiaire générée par la sous-requête
```

<details>
<summary><b>Résultat de la requête</b></summary>


</details>
<br/>

### Règles à respecter

:small_blue_diamond: **Parenthèses :** une sous-requête doit toujours être écrite entre parenthèses afin de clairement la définir.

:small_blue_diamond: **Alias :** dans le cas de sous-requêtes dans le `FROM`, il est nécessaire de préciser un alias pour la table intermédiaire (le résultat de la sous-requête). Sinon, MySQL déclenchera une erreur. Dans l'exemple précédent, on l'a appelée `tortues_perroquets_F`.

:bulb: Nommer la table intermédiaire permet également de s'y référer si l'on fait une jointure dessus ou si certains noms de colonnes sont ambigus et que le nom de la table doit être précisé.

:warning: Attention au fait qu'il ne s'agit pas de la table `Animal`, mais bien d'une table tirée d'`Animal`. Par conséquent, si l'on veut préciser le nom de la table dans le `SELECT` principal, on doit écrire `SELECT MIN(tortues_perroquets_F.date_naissance)` et non pas `SELECT MIN(Animal.date_naissance)`.

:small_blue_diamond: **Cohérence des colonnes :** les colonnes sélectionnées dans le `SELECT` "principal" doivent bien sûr être présentes dans la table intermédiaire.

**Exemple :** la requête suivante ne fonctionnera pas :

```sql
SELECT MIN(date_naissance)
FROM (
    SELECT Animal.id, Animal.nom
    FROM Animal
    INNER JOIN Espece
        ON Espece.id = Animal.espece_id
    WHERE sexe = 'F'
    AND Espece.nom_courant IN ('Tortue d''Hermann', 'Perroquet amazone')
) AS tortues_perroquets_F;
```

**Explications :** `tortues_perroquets_F` n'a que deux colonnes : `id` et `nom`. Il est donc impossible de sélectionner la colonne `date_naissance` de cette table.

:small_blue_diamond: **Noms ambigus :** une table, même intermédiaire, ne peut pas avoir deux colonnes ayant le même nom (colonnes dites **ambiguës**). Si deux colonnes ont le même nom, il est nécessaire de renommer explicitement au moins l'une des deux.

**Exemple :** pour sélectionner la colonne `Espece.id` en plus dans la sous-requête, on peut procéder ainsi :

```sql
SELECT MIN(date_naissance)
FROM (
    SELECT Animal.id, Animal.sexe, Animal.date_naissance, Animal.nom, Animal.espece_id, 
            Espece.id AS espece_espece_id         -- on renomme la colonne id de Espece, donc il n'y a plus de doublons
    FROM Animal                                   -- attention de ne pas la renommer espece_id, puisqu'on sélectionne aussi la colonne espece_id dans Animal
    INNER JOIN Espece
        ON Espece.id = Animal.espece_id
    WHERE sexe = 'F'
    AND Espece.nom_courant IN ('Tortue d''Hermann', 'Perroquet amazone')
) AS tortues_perroquets_F;
```

## 2. Sous-requêtes dans les conditions

Lors d'une requête `SELECT`, le résultat est sous forme de table. Ces tables de résultats peuvent avoir :
* plusieurs colonnes et plusieurs lignes ;
* plusieurs colonnes, mais une seule ligne ;
* plusieurs lignes, mais une seule colonne ;
* une seule ligne et une seule colonne (c'est-à-dire juste une valeur).

Les sous-requêtes renvoyant plusieurs lignes et plusieurs colonnes ne sont utilisées que dans les clauses `FROM`. On s'intéresse ici aux trois autres possibilités.

On peut utiliser des comparaisons avec des sous-requêtes qui donnent comme résultat soit une valeur (c'est-à-dire une seule ligne et une seule colonne), soit une ligne (plusieurs colonnes, mais une seule ligne).

<details>
<summary><b>Opérateurs de comparaison</b></summary>

![capture_operateurs_comparaison](https://user-images.githubusercontent.com/1475600/54370812-94530780-4678-11e9-94da-3e3dd08e89b7.JPG)
</details>

### 2.1. Sous-requêtes renvoyant une valeur

```sql
SELECT id, sexe, nom, commentaires, espece_id, race_id
FROM Animal
WHERE race_id = 
    (SELECT id FROM Race WHERE nom = 'Berger Allemand');  -- la sous-requête renvoie simplement 1
```

### 2.2. Sous-requêtes renvoyant une ligne

Seuls les opérateurs `=` et `!=` (ou `<>`) sont utilisables avec une sous-requête de ligne. Toutes les comparaisons de type "plus grand" ou "plus petit" ne sont pas supportées.

**Syntaxe d'une sous-requête dont le résultat est une ligne :**

```sql
SELECT *
FROM nom_table1
WHERE [ROW](colonne1, colonne2) = (    -- le ROW n'est pas obligatoire
    SELECT colonneX, colonneY
    FROM nom_table2
    WHERE...);                         -- condition qui ne retourne qu'UNE SEULE LIGNE
```

**Explications :** cette requête renvoie toutes les lignes de la table1 dont la colonne1 = la colonneX de la ligne résultat de la sous-requête ET la colonne2 = la colonneY de la ligne résultat de la sous-requête.

**Exemple :** sélectionner l'animal dont l'id vaut 7 et dont le race_id vaut l'espece_id de la race d'id 7

```sql
SELECT id, sexe, nom, espece_id, race_id 
FROM Animal
WHERE (id, race_id) = (
    SELECT id, espece_id
    FROM Race
    WHERE id = 7);
```

**Explications :**

:small_blue_diamond: La sous-requête renvoie une ligne de la table race, en l'occurrence l'entrée d'id 7 et d'espece_id 2.

<details>
<summary><b>Résultat de la sous-requête</b></summary>

![capture_ss_req_une_ligne](https://user-images.githubusercontent.com/1475600/54375566-8f468600-4681-11e9-8479-c193572f9ece.JPG)
</details>
<br/>

:small_blue_diamond: La requête principale renvoie donc l'entrée de la table Animal dont l'id vaut 7 et le race_id 2.

<details>
<summary><b>Résultat de la requête</b></summary>

![capture_req_une_ligne](https://user-images.githubusercontent.com/1475600/54375777-fc5a1b80-4681-11e9-9391-e5e7699720b1.JPG)
</details>

:warning: Il est impératif que la sous-requête ne renvoie qu'une seule ligne. Dans le cas contraire, la requête échouera.

## 2.3. Conditions avec `IN` et `NOT IN`

### Opérateur `IN`

L'opérateur `IN` compare une colonne avec une liste de valeurs. 

**Exemple :** sélectionner les tortues et les perroquets (afficher leur id, leur nom et l'id de leur espece)

```sql
-- avec une jointure
SELECT Animal.id, Animal.nom, Animal.espece_id
FROM Animal
INNER JOIN Espece
    ON Animal.espece_id = Espece.id
WHERE Espece.nom_courant IN ('Tortue d''Hermann', 'Perroquet amazone');

-- avec une sous-requête
SELECT Animal.id, Animal.nom, Animal.espece_id
FROM Animal
WHERE espece_id IN (
    SELECT id
    FROM Espece
    WHERE Espece.nom_courant IN ('Tortue d''Hermann', 'Perroquet amazone')
);
```

<details>
<summary><b>Résultat de la requête</b></summary>

![capture_ss_req_in](https://user-images.githubusercontent.com/1475600/54377073-b488c380-4684-11e9-88ac-f140a3bf62e5.JPG)
</details>

**Explications :**

:small_blue_diamond: La sous-requête renvoie la colonne id contenant les valeurs 3 et 4 (les id des espèces Tortue d'Hermann et Perroquet amazone).

:small_blue_diamond: La requête principale sélectionne les lignes qui ont un espece_id parmi ceux renvoyés par la sous-requête, i.e. 3 ou 4.

### Opérateur `NOT IN`

L'opérateur `NOT IN` exclut les lignes qui correspondent au résultat de la sous-requête.

**Exemple :** sélectionner les animaux dont l'espece_id n'est pas 3 ou 4.

```sql
SELECT Animal.id, Animal.nom, Animal.espece_id
FROM Animal
WHERE espece_id NOT IN (
    SELECT id FROM Espece
    WHERE Espece.nom_courant IN ('Tortue d''Hermann', 'Perroquet amazone')
);
```

### Conditions avec ANY, SOME et ALL

Les conditions avec `IN` et `NOT IN` se limitent aux comparaisons  de type "est égal" ou "est différent". Avec `ANY` et `ALL`, on peut utiliser les autres comparateurs (plus grand, plus petit, etc).

:warning: Comme pour `IN`, `ANY`, `SOME` et `ALL` s'utilisent avec des sous-requêtes dont le résultat est soit une valeur, soit une colonne.

* `ANY` : veut dire "au moins une des valeurs" ;
* `SOME` : est un synonyme de ANY ;
* `ALL` : signifie "toutes les valeurs".

### ANY (ou SOME)

**Exemple :** sélectionner les lignes de la table `Animal` dont l'espece_id est inférieur à au moins une des valeurs sélectionnées dans la sous-requête (en l'occurrence, un espece_id inférieur à au moins l'une des valeurs 3 ou 4, i.e. 1, 2 ou 3)

```sql
SELECT *
FROM Animal
WHERE espece_id < ANY (
    SELECT id FROM Espece
    WHERE Espece.nom_courant IN ('Tortue d''Hermann', 'Perroquet amazone')
);
```

### ALL

En reprenant l'exemple précédent avec `ALL` plutôt qu'avec `ANY`, on sélectionne les lignes de la table `Animal` dont l'espece_id est inférieur à toutes les valeurs sélectionnées dans la sous-requête, i.e. inférieur à 3 et à 4. Ainsi, on n'aura que les lignes dont l'espece_id vaut 1 ou 2.

**Exemple :** 

```sql
SELECT *
FROM Animal
WHERE espece_id < ALL (
    SELECT id FROM Espece
    WHERE Espece.nom_courant IN ('Tortue d''Hermann', 'Perroquet amazone')
);
```

:warning: `= ANY` est l'équivalent de `IN`, tandis que `<> ALL` est l'équivalent de `NOT IN`. Attention, cependant `ANY` et `ALL` (et `SOME`) ne peuvent s'utiliser qu'avec des sous-requêtes et non avec des valeurs, comme on peut le faire avec `IN`.

## 3. Sous-requêtes corrélées

Une sous-requête corrélée est une sous-requête qui fait référence à une colonne (ou une table) qui n'est pas définie dans sa clause `FROM`, mais bien ailleurs dans la requête dont elle fait partie.

**Exemple :** requête avec sous-requête corrélée

```sql
SELECT colonne1 
FROM tableA
WHERE colonne2 IN (
    SELECT colonne3
    FROM tableB
    WHERE tableB.colonne4 = tableA.colonne5
    );
```

La sous-requête seule ne peut pas s'exécuter :

```sql
SELECT colonne3
FROM tableB
WHERE tableB.colonne4 = tableA.colonne5
```

**Explications :** seule la tableB est sélectionnée dans la clause `FROM`, il n'y a pas de jointure avec la tableA et pourtant, on utilise la tableA dans la condition.

Par contre, aucun problème pour l'utiliser comme sous-requête, puisque la clause `FROM` de la requête principale sélectionne la tableA. La sous-requête est donc corrélée à la requête principale.

@TODO