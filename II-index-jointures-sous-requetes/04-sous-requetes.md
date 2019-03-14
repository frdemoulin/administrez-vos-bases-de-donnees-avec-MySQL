# [04 - Sous-requêtes](https://openclassrooms.com/fr/courses/1959476-administrez-vos-bases-de-donnees-avec-mysql/1964181-sous-requetes)

Une sous-requête est une requête à l'intérieur d'une autre requête. Avec le SQL, on peut construire des requêtes imbriquées sur autant de niveaux que désiré. On peut également mélanger jointures et sous-requêtes.

Une sous-requête peut être faite dans une requête de type `SELECT`, `INSERT`, `UPDATE` ou `DELETE` (et quelques autres).

Dans ce chapitre, on aborde les requêtes de sélection. Les jointures et sous-requêtes pour la modification, l'insertion et la suppression de données seront traitées dans le prochain chapitre.

:bulb: Pour optimiser les performances d'une application, utiliser plutôt des jointures lorsque c'est possible.

## 1. Sous-requêtes dans le FROM

Lors d'une requête de type `SELECT`, le résultat de la requête est envoyé sous forme de table. Et grâce aux sous-requêtes, il est tout à fait possible d'utiliser cette table et de refaire une recherche uniquement sur les lignes de celle-ci.

**Exemple :** sélectionner toutes les femelles parmi les perroquets et les tortues

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

**Exemple :** parmi les femelles perroquets et tortues précédentes, connaître la date de naissance de la plus âgée

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

:small_blue_diamond: **Parenthèses :** une sous-requête doit toujours être écrite entre parenthèses afin de clairement la définir

:small_blue_diamond: **Alias :** dans le cas de sous-requêtes dans le `FROM`, il est nécessaire de préciser un alias pour la table intermédiaire (le résultat de la sous-requête). Sinon, MySQL déclenchera une erreur. Dans l'exemple précédent, on l'a appelée `tortues_perroquets_F`.

Nommer la table intermédiaire permet de plus de s'y référer si l'on fait une jointure dessus ou si certains noms de colonnes sont ambigus et que le nom de la table doit être précisé.

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

**Explications :** `tortues_perroquets_F` n'a que deux colonnes : id et nom. Il est donc impossible de sélectionner la colonne `date_naissance` de cette table.

:small_blue_diamond: **Noms ambigus :** une table, même intermédiaire, ne peut pas avoir deux colonnes ayant le même nom (colonnes dites **ambiguës**). Si deux colonnes ont le même nom, il est nécessaire de renommer explicitement au moins l'une des deux.

**Exemple :** si l'on veut sélectionner la colonne `Espece.id` en plus dans la sous-requête, on peut procéder ainsi :

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

## 3. Sous-requêtes corrélées