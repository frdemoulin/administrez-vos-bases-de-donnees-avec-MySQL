# [05 - Exercices sur les agrégats](https://openclassrooms.com/fr/courses/1959476-administrez-vos-bases-de-donnees-avec-mysql/1967867-exercices-sur-les-agregats)

## 1. Du simple

1. Combien de races avons-nous dans la table Race ?

```sql
SELECT COUNT(*) AS nb_races FROM Race;
```

2. De combien de chiens connaissons-nous le père ?

```sql
SELECT nom, nom_courant, COUNT(pere_id) AS nb_chiens FROM Animal
INNER JOIN Espece ON Espece.id = Animal.espece_id
WHERE nom_courant = 'Chien';
```

**Rappel :** `COUNT(pere_id)` compte les animaux ayant un père identifié (i.e. un `pere_id` `not null`)

3. Quelle est la date de naissance de notre plus jeune femelle ?

```sql
-- méthode 1
SELECT nom, date_naissance FROM Animal
WHERE sexe = 'F'
ORDER BY date_naissance DESC
LIMIT 1;

-- méthode 2
SELECT MAX(date_naissance) FROM Animal
WHERE sexe = 'F';
```

4. En moyenne, quel est le prix d'un chien ou d'un chat de race, par espèce, et en général ?

```sql
SELECT COALESCE(nom_courant, 'Général') AS espece, AVG(Race.prix) AS prix_moyen FROM Race
INNER JOIN Espece ON Race.espece_id = Espece.id
WHERE nom_courant IN('Chat','Chien')
GROUP BY nom_courant WITH ROLLUP;
```

5. Quel est le nombre de perroquets mâles et femelles, et quels sont leurs noms (en une seule requête) ?

```sql
SELECT sexe, COUNT(*), GROUP_CONCAT(nom SEPARATOR ', ') AS nom
FROM Animal
INNER JOIN Espece ON Espece.id = Animal.espece_id
WHERE Espece.nom_courant = 'Perroquet amazone'
GROUP BY sexe;
```

**Rappel :** `GROUP_CONCAT(nom_colonne)` concatène les valeurs de `nom_colonne` pour chaque groupement réalisé. Cela permet d'afficher les valeurs concernées par le résultat d'une fonction d'agrégation

## 2. Au compliqué

1. Quelles sont les races dont nous ne possédons aucun individu ?

```sql
SELECT Race.nom, COUNT(Animal.race_id) AS nombre FROM Race
LEFT JOIN Animal ON Race.id = Animal.race_id
GROUP BY Race.nom
HAVING nombre = 0;
```

**Rappel :** Pour faire des conditions sur une fonction d'agrégation, on utilise la clause `HAVING` à placer juste après `GROUP BY`

2. Quelles sont les espèces (triées par ordre alphabétique du nom latin) dont nous possédons moins de cinq mâles ?

```sql
SELECT Espece.nom_latin, COUNT(espece_id) AS nombre
FROM Espece
LEFT JOIN Animal ON Espece.id = Animal.espece_id
WHERE sexe = 'M'
GROUP BY Espece.nom_latin
HAVING nombre < 5;
```

3. Combien de mâles et de femelles de chaque race avons-nous, avec un compte total intermédiaire pour les races (mâles et femelles confondues) et pour les espèces ? Afficher le nom de la race et le nom courant de l'espèce.

```sql

```

4. Quel serait le coût, par espèce et au total, de l'adoption de Parlotte, Spoutnik, Caribou, Cartouche, Cali, Canaille, Yoda, Zambo et Lulla ?

```sql

```