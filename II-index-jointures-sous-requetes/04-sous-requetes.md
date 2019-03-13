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

## 2. Sous-requêtes dans les conditions

## 3. Sous-requêtes corrélées