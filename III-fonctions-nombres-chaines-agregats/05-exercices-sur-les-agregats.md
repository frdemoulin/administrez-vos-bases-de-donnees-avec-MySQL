# [05 - Exercices sur les agrégats](https://openclassrooms.com/fr/courses/1959476-administrez-vos-bases-de-donnees-avec-mysql/1967867-exercices-sur-les-agregats)

## 1. Du simple

1. Combien de races avons-nous dans la table Race ?

    ```sql
    SELECT COUNT(*) AS nb_races FROM Race;
    ```

1. De combien de chiens connaissons-nous le père ?

    ```sql
    SELECT nom, nom_courant, COUNT(pere_id) AS nb_chiens FROM Animal
    INNER JOIN Espece ON Espece.id = Animal.espece_id
    WHERE nom_courant = 'Chien';
    ```

## 2. Au compliqué