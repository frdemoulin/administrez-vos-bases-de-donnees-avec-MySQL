# [03 - Jointures](https://openclassrooms.com/fr/courses/1959476-administrez-vos-bases-de-donnees-avec-mysql/1963612-jointures)

Les jointures permettent de jongler avec plusieurs tables dans la même requête.

## 1. Principe des jointures et notion d'alias

### 1.1. Principe des jointures

Le principe des jointures est de joindre plusieurs tables. Pour ce faire, on utilise les informations communes des tables.

Sans jointure, on peut être amené à devoir écrire plusieurs requêtes.

**Exemple :** afficher la description de l'espèce de Cartouche

```sql
-- Etape 1 : sélection de l'espece_id de Cartouche (renvoie espece_id = 1)
SELECT espece_id FROM Animal
WHERE nom = 'Cartouche';

-- Etape 2 : sélection de la description de l'espèce d'id trouvé précédemment
SELECT description FROM Espece
WHERE id = 1;
```

<details>
<summary><b>Résultat de la requête</b></summary>

![jointure_description](https://user-images.githubusercontent.com/1475600/53809591-5d347600-3f55-11e9-8895-4223cdf03678.PNG)
</details>
<br/>

Sans jointures, il est ainsi nécessaire d'écrire deux requêtes.

Avec une jointure, on peut procéder en une seule requête :

```sql
SELECT Espece.description FROM Espece
INNER JOIN Animal ON Espece.id = Animal.espece_id
WHERE Animal.nom = 'Cartouche'
```

<details>
<summary><b>Résultat de la requête</b></summary>

![jointure_description](https://user-images.githubusercontent.com/1475600/53809591-5d347600-3f55-11e9-8895-4223cdf03678.PNG)
</details>
<br/>

**Explications :** en faisant une jointure, on crée une table virtuelle et temporaire qui reprend les colonnes des tables liées comme l'illustre le schéma ci-dessous :

![table virtuelle créée lors d'une jointure](https://user.oc-static.com/files/376001_377000/376487.png)

Au départ, on a deux tables : `Animal`(id, sexe, nom, race_id, espece_id) et `Espece`(id, nom_courant, nom_latin).

Les deux premières lignes d'`Animal` correspondent à la première ligne d'`Espece`, et la troisième ligne d'`Animal` à la deuxième ligne d'`Espece`. Une fois les deux tables jointes, on obtient une table possédant toutes les colonnes d'`Animal` et toutes les colonnes d'`Espece`, avec les valeurs correspondantes de chaque table.

On peut voir que les cinquième et sixième colonnes de la table de jointure ont les mêmes valeurs.

Ensuite, de cette table virtuelle, on peut extraire ce que l'on veut. La colonne `nom_latin` pour la ligne ayant "Caribou" dans la colonne `nom`, par exemple.

### 1.2. Notion d'alias

Les alias sont des noms de remplacement donnés de manière temporaire (le temps d'une requête) à une colonne, une table, une donnée. Les alias sont introduits par le mot-clé `AS` (facultatif, mais préférable de toujours le mettre pour gagner en lisibilité).

**Exemple :** calculer le nombre de chiots de Cartouche qui a eu une première portée de 5 chiots et une seconde de 3 chiots en utilisant un alias

```sql
-- on utilise avec AS l'alias Chiots_cartouche pour labelliser la colonne de résultat
SELECT 5+3 AS Chiots_Cartouche;

-- sans utiliser AS
SELECT 5+3 Chiots_Cartouche;
```

## 2. Jointure interne

**Exemple précédent de requête avec jointure interne :**

```sql
SELECT Espece.description 
FROM Espece 
INNER JOIN Animal 
    ON Espece.id = Animal.espece_id 
WHERE Animal.nom = 'Cartouche';
```

**Explications :**

:small_blue_diamond: `SELECT Espece.description` : sélection de la colonne `description` de la table `Espece`.

:small_blue_diamond: `FROM Espece` : on travaille sur la table `Espece`.

:small_blue_diamond: `INNER JOIN Animal` : on la joint (avec une jointure interne) à la table `Animal`.

:small_blue_diamond: `ON Espece.id = Animal.espece_id` : la jointure se fait sur les colonnes `id` de la table `Espece` et `espece_id` de la table `Animal`, qui doivent donc correspondre.

:small_blue_diamond: `WHERE Animal.nom = 'Cartouche'` : dans la table résultant de la jointure, on sélectionne les lignes qui ont la valeur "Cartouche" dans la colonne `nom` venant de la table `Animal`.

### 2.1. Syntaxe

```sql
SELECT *                                   -- comme d'habitude, vous sélectionnez les colonnes que vous voulez
FROM nom_table1
[INNER] JOIN nom_table2                    -- INNER explicite le fait qu'il s'agit d'une jointure interne, mais c'est facultatif
    ON colonne_table1 = colonne_table2     -- sur quelles colonnes se fait la jointure
                                           -- vous pouvez mettre colonne_table2 = colonne_table1, l'ordre n'a pas d'importance

[WHERE ...]                               
[ORDER BY ...]                            -- les clauses habituelles sont bien sûr utilisables !
[LIMIT ...]
```

#### Condition de jointure

La clause `ON` sert à préciser la condition de la jointure. C'est-à-dire sur quel(s) critère(s) les deux tables doivent être jointes. Dans la plupart des cas, il s'agira d'une condition d'égalité simple, comme `ON Animal.espece_id = Espece.id`. Il est cependant tout à fait possible d'avoir plusieurs conditions à remplir pour lier les deux tables. On utilise alors les opérateurs logiques habituels.

**Exemple :** jointure se faisant sur plusieurs colonnes

```sql
SELECT *
FROM table1
INNER JOIN table2
   ON table1.colonneA = table2.colonneJ
      AND table1.colonneT = table2.colonneX
      [AND ...];
```

#### Expliciter le nom des colonnes

Il peut arriver que des colonnes portent le même nom dans des tables différentes (cas de la colonne id par exemple). Ces colonnes sont dites _amibguës_. Il est donc important de préciser de quelle colonne on parle dans ce cas-là.

Pour cela, on utilise l'opérateur `.` (`nom_table.nom_colonne`). Pour les colonnes ayant un nom non ambigu (qui n'existe dans aucune autre table de la jointure), il n'est pas obligatoire de préciser la table.

:bulb: En général, on précise la table quand il s'agit de grosses requêtes avec plusieurs jointures. En revanche, pour les petites jointures courantes, on ne précise pas la table.

**Exemple :** sélection du nom des animaux commençant par "Ch", ainsi que de l'id et la description de leur espèce

```sql
SELECT Espece.id, Espece.description, Animal.nom FROM Animal
INNER JOIN Espece ON Espece.id = Animal.espece_id
WHERE Animal.nom LIKE 'Ch%';
```

<details>
<summary><b>Résultat de la requête</b></summary>

![jointure_nom_explicite](https://user-images.githubusercontent.com/1475600/53811448-bb635800-3f59-11e9-93ca-4b291c123d26.PNG)
</details>

#### Utiliser les alias

**Première raison :** avec les jointures, le alias permettent de renommer les tables et ainsi d'écrire moins de code.

:pencil: **Exemple :** on renomme la table `Espece` "e" et la table `Animal` "a"

```sql
SELECT e.id,
       e.description,
       a.nom
FROM Espece AS e          -- on donne l'alias "e" à Espece
INNER JOIN Animal AS a    -- et l'alias "a" à Animal
    ON e.id = a.espece_id
WHERE a.nom LIKE 'Ch%';
```

**Deuxième raison :** avec les jointures, les alias permettent de renommer les colonnes pour que le résultat soit plus clair.

:pencil: **Exemple :** on donne des alias aux colonnes (`id_espece` pour `id` de la table `Espece`, `description_espece` pour `Espece.description` et `nom_bestiole` pour `Animal.nom`).

```sql
SELECT Espece.id AS id_espece,
       Espece.description AS description_espece,
       Animal.nom AS nom_bestiole
FROM Espece   
INNER JOIN Animal
     ON Espece.id = Animal.espece_id
WHERE Animal.nom LIKE 'Ch%';
```

<details>
<summary><b>Résultat de la requête</b></summary>

![jointure_alias_colonne](https://user-images.githubusercontent.com/1475600/53822689-83b3da80-3f70-11e9-8326-d11935e92ca3.PNG)
</details>

### 2.2. Pourquoi interne ?

`INNER JOIN` permet de faire une jointure interne sur deux tables. _Interne_ signifie que l'on exige qu'il y ait des données de part et d'autre de la jointure.

**Principe :** si l'on fait une jointure sur la colonne a de la table A et la colonne b de la table B, ceci retournera uniquement les lignes pour lesquelles A.a et B.b correspondent :

```sql
SELECT * 
FROM A
INNER JOIN B 
    ON A.a = B.b
```

:pencil: **Exemple :** on veut connaître la race des chats

```sql
SELECT Animal.nom as nom_animal, Race.nom as race FROM Animal
INNER JOIN Race ON Race.id = Animal.race_id
INNER JOIN Espece ON Espece.id = Animal.espece_id
WHERE Espece.nom_courant = 'Chat'
ORDER BY Race.nom, Animal.nom;
```

<details>
<summary><b>Résultat de la requête</b></summary>

![jointure_interne](https://user-images.githubusercontent.com/1475600/53823841-d55d6480-3f72-11e9-98a5-c31426b555db.PNG)
</details>
<br/>

**Explications :** les chats Choupi et Roucky pour lesquels on n'a pas d'information sur la race (`race_id` est `NULL`) ne sont pas repris dans les résultats car la jointure est **interne**. Pour les inclure, il faudrait utiliser une jointure **externe**.

## 3. Jointure externe

Une **jointure externe** permet de sélectionner également les lignes pour lesquelles il n'y a pas de correspondance dans une des tables jointes. MySQL permet deux types de jointures externes : les jointures par la gauche et les jointures par la droite.

### 3.1. Jointures par la gauche

Une **jointure par la gauche** se fait grâce aux mots-clés `LEFT JOIN` ou `LEFT OUTER JOIN`. Cela signifie que l'on veut toutes les lignes de la table de gauche (sauf restrictions dans une clause `WHERE`, bien sûr), même si certaines n'ont pas de correspondance avec une ligne de la table de droite.

:warning: La table de gauche est la première mentionnée dans la requête !!! C'est en général la table donnée dans la clause `FROM`.

:pencil: **Exemple :** on veut connaître la race des chats en affichant les chats n'ayant éventuellement pas de race

```sql
SELECT Animal.nom as nom_animal, Race.nom as race FROM Animal -- table de gauche
LEFT JOIN Race ON Race.id = Animal.race_id -- table de droite
INNER JOIN Espece ON Espece.id = Animal.espece_id
WHERE Espece.nom_courant = 'Chat'
ORDER BY Race.nom, Animal.nom;
```

<details>
<summary><b>Résultat de la requête</b></summary>

![jointure_externe_gauche](https://user-images.githubusercontent.com/1475600/53824668-c7a8de80-3f74-11e9-909d-1575bae07573.PNG)
</details>
<br/>

**Explications :** on ne connaît pas la race de Choupi ni de Roucky, ils font cependant bien partie des lignes sélectionnées, car la jointure est externe. En effet, les colonnes qui viennent de la table `Race` (la colonne `Race.nom AS race` ici) sont `NULL` pour les lignes qui n'ont pas de correspondance (les lignes de Choupi et de Roucky ici).

### 3.2. Jointures par la droite

Dans une **jointure par la droite** (`RIGHT JOIN` ou `RIGHT OUTER JOIN`), toutes les lignes de la table de droite sont sélectionnées, même s'il n'y a pas de correspondance dans la table de gauche. C'est donc le même principe que pour les jointures par la gauche.

:pencil: **Exemple :** on veut connaître la race des chats en affichant les races correspondant à aucun chat dans l'élevage

```sql
SELECT Animal.nom AS nom_animal, Race.nom AS race
FROM Animal                                                -- table de gauche
RIGHT JOIN Race                                            -- table de droite
    ON Animal.race_id = Race.id
WHERE Race.espece_id = 2
ORDER BY Race.nom, Animal.nom;
```

<details>
<summary><b>Résultat de la requête</b></summary>

![jointure_externe_droite](https://user-images.githubusercontent.com/1475600/53826580-3425dc80-3f79-11e9-95a5-2b4b7de2d134.PNG)
</details>
<br/>

**Explications :** on a bien une ligne avec la race "Sphynx", bien qu'aucun sphynx ne soit présent dans la table `Animal`.

:bulb: Toutes les jointures par la droite peuvent être faites grâce à une jointure par la gauche (et vice versa).

:pencil: **Exemple :** équivalent de la requête précédente mais avec une jointure par la gauche

```sql
SELECT Animal.nom AS nom_animal, Race.nom AS race
FROM Race                                               -- table de gauche
LEFT JOIN Animal                                        -- table de droite
    ON Animal.race_id = Race.id
WHERE Race.espece_id = 2
ORDER BY Race.nom, Animal.nom;
```

## 4. Syntaxes alternatives

@TODO

## 5. Exemples d'application et exercices

A.1. Obtenir la liste des races de chiens qui sont des chiens de berger

```sql
SELECT Race.nom FROM Race
INNER JOIN Espece ON Espece.id = Race.espece_id
WHERE Espece.nom_courant = 'Chien' AND Race.nom LIKE 'berger%';
```

<details>
<summary><b>Résultat de la requête</b></summary>

![jointure_exemple_1](https://user-images.githubusercontent.com/1475600/53872602-e2279a00-3ffe-11e9-8798-c7f2e74d9606.PNG)
</details>

**Explications :** ne pas oublier la condition `Espece.nom_courant = 'chien'` car une race de chat (ou autre) pourrait très bien contenir "berger" or on a explicitement demandé les chiens

A.2. Obtenir la liste des animaux (leur nom, date de naissance et race) pour lesquels on n'a aucune information sur leur pelage

:warning: Dans la description des races, "pelage", "poil" ou "robe" caractérise le pelage d'un animal

```sql
SELECT Animal.nom, Animal.date_naissance, Race.nom
FROM Animal         -- table de gauche
LEFT JOIN Race      -- table de droite
    ON Race.id = Animal.race_id
WHERE (Race.description NOT LIKE '%pelage%'
    AND Race.description NOT LIKE '%poil%'
    AND Race.description NOT LIKE 'robe'
    )
    OR Race.id IS NULL;
```

<details>
<summary><b>Résultat de la requête</b></summary>

![jointure_exemple_2](https://user-images.githubusercontent.com/1475600/53874724-56643c80-4003-11e9-86c4-b105819d1700.PNG)
</details>

**Explications :** les animaux à sélectionner sont ceux dont :

a. On ne connaît pas la race (a fortiori, on n'a alors aucune information sur le pelage de la race) ;

b. On connaît la race, mais dont la description de celle-ci ne contient pas les mots-clés liés au pelage.

**Détails :**

**a. On ne connaît pas la race.** Les animaux dont on ne connaît pas la race ont `NULL` dans la colonne `race_id`. Mais vu que l'on fait une jointure sur cette colonne, il ne faut pas oublier de faire une __jointure externe__, sinon tous ces animaux sans race seront éliminés.

Une fois que c'est fait, pour les sélectionner, il suffit de mettre la condition `Animal.race_id IS NULL`, par exemple, ou simplement `Race.#n'importe quelle colonne# IS NULL`, vu qu'il n'y a pas de correspondance avec `Race`.

**b. Pas d'information sur le pelage dans la description de la race.** On sélectionne les races pour lesquelles la description ne contient pas les mots-clés. D'où l'utilisation de `NOT LIKE`.

**En résumé :** il fallait faire une jointure externe des tables `Animal` et `Race`, et ensuite sélectionner les animaux qui répondaient à l'une ou l'autre des conditions (opérateur logique `OR`).

B.1. Obtenir la liste des chats et des perroquets amazones, avec leur sexe, leur espèce (nom latin) et leur race s'ils en ont une. Regroupez les chats ensemble, les perroquets ensemble et, au sein de l'espèce, regroupez les races

```sql
SELECT Animal.nom AS nom_animal, Animal.sexe, Espece.nom_courant AS nom_espece, Espece.nom_latin, Race.nom AS nom_race
FROM Animal
INNER JOIN Espece ON Espece.id = Animal.espece_id
LEFT JOIN Race ON Race.id = Animal.race_id
WHERE Espece.nom_courant IN('Chat', 'Perroquet amazone')
ORDER BY Espece.nom_latin, Race.nom;
```

<details>
<summary><b>Résultat de la requête</b></summary>

![jointure_exemple_b1](https://user-images.githubusercontent.com/1475600/53875950-033fb900-4006-11e9-8c77-c0c8afe09eae.PNG)
</details>

:bulb: Il est possible de mélanger jointures internes et externes. L'ordre dans lequel les jointures sont faites n'est pas important.

**Explications :**

:small_blue_diamond: D'abord, on fait la jointure d'`Animal` et d'`Espece`. On se retrouve alors avec une grosse table qui possède toutes les colonnes d'`Animal` et toutes les colonnes d'`Espece`. Ensuite, à cette grosse table (à la fois virtuelle et intermédiaire), on joint la table `Race`, grâce à la colonne `Animal.race_id`.

:small_blue_diamond: Concernant la clause `ORDER BY`, on trie par ordre alphabétique (possible de trier sur les id de l'espèce et de la race). L'important ici était de trier d'abord sur une colonne d'`Espece`, ensuite sur une colonne de `Race`.

B.2. Obtenir la liste des chiennes dont on connaît la race, qui sont en âge de procréer (c'est-à-dire nées avant juillet 2010). Affichez leurs nom, date de naissance et race.

```sql
SELECT Animal.nom AS nom_animal, Animal.sexe, Animal.date_naissance, Espece.nom_courant AS nom_espece, Race.nom AS nom_race
FROM Animal
INNER JOIN Espece ON Espece.id = Animal.espece_id
INNER JOIN Race ON Race.id = Animal.race_id
WHERE (Espece.nom_courant = 'Chien'
    AND Animal.sexe = 'F'
    AND Animal.date_naissance < '2010-07-01'
    )
```

<details>
<summary><b>Résultat de la requête</b></summary>

![jointure_exemple_b2](https://user-images.githubusercontent.com/1475600/53876983-4a2eae00-4008-11e9-9ebb-89a79c97a1ee.PNG)
</details>

**Explications :** faire une jointure interne avec `Race`, puisque l'on veut que la race soit connue

C.1. Obtenir la liste des chats dont on connaît les parents, ainsi que le nom de ces parents

```sql

```

<details>
<summary><b>Résultat de la requête</b></summary>


</details>

**Explications :** 