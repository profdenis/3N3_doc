# 5 - Survol des opérations faisant partie des requêtes SQL

Pour commencer à comprendre les opérations de base d'une requête SQL, examinons une projection simple à l'aide des
clauses `SELECT` et `FROM`. Ces clauses sont fondamentales pour extraire des données d'une base de données relationnelle
comme PostgreSQL. Nous allons illustrer cela avec une table de clients d'un magasin.

## Création d'une Table de Clients

Imaginons que nous avons une table nommée `clients` qui contient des informations sur les clients d'un magasin. Voici
comment cette table pourrait être définie et peuplée avec quelques lignes d'exemple :

```sql
CREATE TABLE clients
(
    id               SERIAL PRIMARY KEY,
    nom              VARCHAR(50),
    prenom           VARCHAR(50),
    email            VARCHAR(100),
    ville            VARCHAR(50),
    date_inscription DATE
);

INSERT INTO clients (nom, prenom, email, ville, date_inscription)
VALUES ('Dupont', 'Jean', 'jean.dupont@example.com', 'Paris', '2023-01-15'),
       ('Martin', 'Marie', 'marie.martin@example.com', 'Lyon', '2023-02-20'),
       ('Durand', 'Paul', 'paul.durand@example.com', 'Marseille', '2023-03-10'),
       ('Lefevre', 'Sophie', 'sophie.lefevre@example.com', 'Toulouse', '2023-04-05'),
       ('Moreau', 'Pierre', 'pierre.moreau@example.com', 'Nice', '2023-05-18'),
       ('Simon', 'Lucie', 'lucie.simon@example.com', 'Nantes', '2023-06-22'),
       ('Laurent', 'Julie', 'julie.laurent@example.com', 'Bordeaux', '2023-07-30'),
       ('Lemoine', 'Antoine', 'antoine.lemoine@example.com', 'Strasbourg', '2023-08-14'),
       ('Roux', 'Claire', 'claire.roux@example.com', 'Lille', '2023-09-01'),
       ('Petit', 'Nicolas', 'nicolas.petit@example.com', 'Rennes', '2023-10-10');
```

| id | Nom     | Prénom  | Email                       | Ville      | Date d'inscription |
|----|---------|---------|-----------------------------|------------|--------------------|
| 1  | Dupont  | Jean    | jean.dupont@example.com     | Paris      | 2023-01-15         |
| 2  | Martin  | Marie   | marie.martin@example.com    | Lyon       | 2023-02-20         |
| 3  | Durand  | Paul    | paul.durand@example.com     | Marseille  | 2023-03-10         |
| 4  | Lefevre | Sophie  | sophie.lefevre@example.com  | Toulouse   | 2023-04-05         |
| 5  | Moreau  | Pierre  | pierre.moreau@example.com   | Nice       | 2023-05-18         |
| 6  | Simon   | Lucie   | lucie.simon@example.com     | Nantes     | 2023-06-22         |
| 7  | Laurent | Julie   | julie.laurent@example.com   | Bordeaux   | 2023-07-30         |
| 8  | Lemoine | Antoine | antoine.lemoine@example.com | Strasbourg | 2023-08-14         |
| 9  | Roux    | Claire  | claire.roux@example.com     | Lille      | 2023-09-01         |
| 10 | Petit   | Nicolas | nicolas.petit@example.com   | Rennes     | 2023-10-10         |

## Utilisation de `SELECT` et `FROM`

La clause `SELECT` est utilisée pour spécifier les colonnes que vous souhaitez récupérer, tandis que la clause `FROM`
indique la table d'où proviennent ces données. Voici quelques exemples de requêtes SQL utilisant ces clauses :

### Exemple 1 : Sélectionner Toutes les Colonnes

Pour récupérer toutes les informations de la table `clients`, vous pouvez utiliser l'astérisque (`*`) pour indiquer que
vous voulez toutes les colonnes :

```sql
SELECT *
FROM clients;
```

| id | Nom     | Prénom  | Email                       | Ville      | Date d'inscription |
|----|---------|---------|-----------------------------|------------|--------------------|
| 1  | Dupont  | Jean    | jean.dupont@example.com     | Paris      | 2023-01-15         |
| 2  | Martin  | Marie   | marie.martin@example.com    | Lyon       | 2023-02-20         |
| 3  | Durand  | Paul    | paul.durand@example.com     | Marseille  | 2023-03-10         |
| 4  | Lefevre | Sophie  | sophie.lefevre@example.com  | Toulouse   | 2023-04-05         |
| 5  | Moreau  | Pierre  | pierre.moreau@example.com   | Nice       | 2023-05-18         |
| 6  | Simon   | Lucie   | lucie.simon@example.com     | Nantes     | 2023-06-22         |
| 7  | Laurent | Julie   | julie.laurent@example.com   | Bordeaux   | 2023-07-30         |
| 8  | Lemoine | Antoine | antoine.lemoine@example.com | Strasbourg | 2023-08-14         |
| 9  | Roux    | Claire  | claire.roux@example.com     | Lille      | 2023-09-01         |
| 10 | Petit   | Nicolas | nicolas.petit@example.com   | Rennes     | 2023-10-10         |

### Exemple 2 : Sélectionner des Colonnes Spécifiques

Si vous ne voulez que certaines colonnes, par exemple, le nom et l'email des clients, vous pouvez spécifier ces colonnes
dans la clause `SELECT` :

```sql
SELECT nom, email
FROM clients;
```

| Nom     | Email                       |
|---------|-----------------------------|
| Dupont  | jean.dupont@example.com     |
| Martin  | marie.martin@example.com    |
| Durand  | paul.durand@example.com     |
| Lefevre | sophie.lefevre@example.com  |
| Moreau  | pierre.moreau@example.com   |
| Simon   | lucie.simon@example.com     |
| Laurent | julie.laurent@example.com   |
| Lemoine | antoine.lemoine@example.com |
| Roux    | claire.roux@example.com     |
| Petit   | nicolas.petit@example.com   |

### Exemple 3 : Sélectionner avec un Alias

Vous pouvez également utiliser des alias pour renommer les colonnes dans le résultat. Cela peut être utile pour rendre
les résultats plus lisibles ou pour éviter des conflits de noms :

```sql
SELECT nom AS NomClient, email AS EmailClient
FROM clients;
```

### Résumé

- **`SELECT`** : Spécifie les colonnes à récupérer.
- **`FROM`** : Indique la table d'où proviennent les données.

Ces opérations de base constituent le fondement de la manipulation des données dans SQL. À mesure que vous progressez,
vous apprendrez à utiliser d'autres clauses et fonctions pour effectuer des opérations plus complexes, telles que le
filtrage avec `WHERE`, le tri avec `ORDER BY`, et bien plus encore.

## Utilisation de la Clause `WHERE`

La deuxième opération fondamentale dans une requête SQL est le filtrage des données à l'aide de la clause `WHERE`. Cette
clause permet de spécifier des conditions qui doivent être remplies pour qu'une ligne soit incluse dans le résultat de
la requête. Examinons quelques exemples simples de filtrage sur notre table `clients`.

La clause `WHERE` est utilisée pour extraire uniquement les enregistrements qui répondent à une condition spécifique.
Voici quelques exemples illustrant son utilisation :

### Exemple 1 : Filtrer par Ville

Supposons que nous souhaitons récupérer tous les clients qui habitent à Paris. Nous utiliserons la clause `WHERE` pour
spécifier cette condition :

```sql
SELECT *
FROM clients
WHERE ville = 'Paris';
```

| id | Nom    | Prénom | Email                   | Ville | Date d'inscription |
|----|--------|--------|-------------------------|-------|--------------------|
| 1  | Dupont | Jean   | jean.dupont@example.com | Paris | 2023-01-15         |

### Exemple 2 : Filtrer par Nom

Si nous voulons trouver tous les clients dont le nom de famille est "Durand", nous pouvons écrire la requête suivante :

```sql
SELECT *
FROM clients
WHERE nom = 'Durand';
```

### Exemple 3 : Filtrer par Email

Pour récupérer les clients dont l'adresse email se termine par "@example.com", nous pouvons utiliser l'opérateur `LIKE`
avec un motif de recherche :

```sql
SELECT *
FROM clients
WHERE email LIKE '%@example.com';
```

### Exemple 4 : Filtrer par Plusieurs Conditions

Il est également possible de combiner plusieurs conditions à l'aide des opérateurs logiques `AND` et `OR`. Par exemple,
pour sélectionner les clients qui habitent à Lyon ou à Marseille, nous utiliserons `OR` :

```sql
SELECT *
FROM clients
WHERE ville = 'Lyon'
   OR ville = 'Marseille';
```

Pour sélectionner les clients qui habitent à Paris et dont le nom de famille est "Dupont", nous utiliserons `AND` :

```sql
SELECT *
FROM clients
WHERE ville = 'Paris'
  AND nom = 'Dupont';
```

### Exemple 5 : Filtrer par Conditions Numériques

Bien que notre table `clients` ne contienne pas de données numériques dans cet exemple, si elle avait une colonne comme
`age`, nous pourrions filtrer les clients de plus de 30 ans comme suit :

```sql
SELECT *
FROM clients
WHERE age > 30;
```

### Résumé

- **`WHERE`** : Utilisée pour filtrer les lignes en fonction de conditions spécifiques.
- **Opérateurs Logiques** : `AND`, `OR` pour combiner plusieurs conditions.
- **Opérateurs de Comparaison** : `=`, `>`, `<`, `>=`, `<=`, `<>` (différent).
- **`LIKE`** : Utilisé pour les recherches de motifs dans les chaînes de caractères.

La clause `WHERE` est essentielle pour extraire des données pertinentes et précises d'une base de données, en fonction
des besoins spécifiques de l'utilisateur ou de l'application.

## Utilisation de `ORDER BY`

Le tri des résultats dans une requête SQL est effectué à l'aide de la clause `ORDER BY`. Cette clause vous permet de
spécifier l'ordre dans lequel les lignes doivent être retournées, que ce soit en ordre croissant ou décroissant. Vous
pouvez trier les résultats en fonction d'une ou plusieurs colonnes.

### Exemple 1 : Tri Simple par Nom

Pour trier les résultats par le nom des clients en ordre croissant (par défaut), vous pouvez utiliser la requête
suivante :

```sql
SELECT *
FROM clients
ORDER BY nom;
```

### Exemple 2 : Tri en Ordre Décroissant

Pour trier les résultats par le nom des clients en ordre décroissant, utilisez le mot clé `DESC` :

```sql
SELECT *
FROM clients
ORDER BY nom DESC;
```

### Exemple 3 : Tri par Plusieurs Colonnes

Vous pouvez trier les résultats par plusieurs colonnes. Par exemple, pour trier d'abord par ville en ordre croissant,
puis par nom en ordre décroissant :

```sql
SELECT *
FROM clients
ORDER BY ville, nom DESC;
```

### Exemple 4 : Tri avec une Clause `WHERE`

Vous pouvez combiner `ORDER BY` avec une clause `WHERE` pour filtrer et trier les résultats. Par exemple, pour obtenir
les clients de Paris triés par date d'inscription :

```sql
SELECT *
FROM clients
WHERE ville = 'Paris'
ORDER BY date_inscription;
```

### Résumé

- **`ORDER BY`** : Utilisé pour spécifier l'ordre des résultats.
- **Ordre Croissant** : Par défaut, les résultats sont triés en ordre croissant (`ASC`).
- **Ordre Décroissant** : Utilisez `DESC` pour trier en ordre décroissant.
- **Tri Multiple** : Vous pouvez trier par plusieurs colonnes en les séparant par des virgules.
- **Combinaison avec `WHERE`** : `ORDER BY` peut être utilisé après `WHERE` pour trier les résultats filtrés.

Le tri des résultats est une fonctionnalité puissante qui vous permet de présenter les données de manière organisée et
lisible, en fonction de vos besoins spécifiques.

## Fonctions Scalaires

Dans SQL, les fonctions peuvent être utilisées dans la clause `SELECT` pour effectuer diverses opérations sur les
données. Ces fonctions peuvent être des fonctions d'agrégation, qui opèrent sur un ensemble de valeurs pour en retourner
une seule, ou des fonctions scalaires, qui opèrent sur une seule valeur et retournent une nouvelle valeur.

Les fonctions scalaires sont appliquées à chaque ligne individuellement. Voici quelques exemples :

### Exemple 1 : Utilisation de `UPPER` pour Convertir en Majuscules

Supposons que nous voulons afficher les noms de famille en majuscules :

```sql
SELECT UPPER(nom) AS NomMajuscule, prenom
FROM clients;
```

| NomMajuscule | Prénom  |
|--------------|---------|
| DUPONT       | Jean    |
| MARTIN       | Marie   |
| DURAND       | Paul    |
| LEFEVRE      | Sophie  |
| MOREAU       | Pierre  |
| SIMON        | Lucie   |
| LAURENT      | Julie   |
| LEMOINE      | Antoine |
| ROUX         | Claire  |
| PETIT        | Nicolas |

### Exemple 2 : Utilisation de `LENGTH` pour Obtenir la Longueur d'une Chaîne

Pour obtenir la longueur des adresses email :

```sql
SELECT nom, LENGTH(email) AS LongueurEmail
FROM clients;
```

| Nom     | LongueurEmail |
|---------|---------------|
| Dupont  | 23            |
| Martin  | 24            |
| Durand  | 23            |
| Lefevre | 26            |
| Moreau  | 25            |
| Simon   | 22            |
| Laurent | 25            |
| Lemoine | 28            |
| Roux    | 23            |
| Petit   | 25            |

### Exemple 3 : Utilisation de Fonctions sur les Dates

Imaginons que nous avons ajouté une colonne `date_inscription` à notre table `clients` et que nous voulons extraire
l'année d'inscription :

```sql
ALTER TABLE clients
    ADD COLUMN date_inscription DATE;

UPDATE clients
SET date_inscription = '2023-01-15'
WHERE id = 1;
-- (Ajoutez des dates pour les autres clients selon vos besoins)

SELECT nom, EXTRACT(YEAR FROM date_inscription) AS AnneeInscription
FROM clients;
```

| Nom     | AnneeInscription |
|---------|------------------|
| Dupont  | 2023             |
| Martin  | 2023             |
| Durand  | 2023             |
| Lefevre | 2023             |
| Moreau  | 2023             |
| Simon   | 2023             |
| Laurent | 2023             |
| Lemoine | 2023             |
| Roux    | 2023             |
| Petit   | 2023             |

## Fonctions d'Agrégation

Les fonctions d'agrégation calculent une valeur unique à partir d'un ensemble de valeurs. Voici quelques exemples :

### Exemple 4 : Utilisation de `COUNT` pour Compter le Nombre de Clients

Pour compter le nombre total de clients dans la table :

```sql
SELECT COUNT(*) AS NombreClients
FROM clients;
```

### Exemple 5 : Utilisation de `AVG` pour Calculer une Moyenne

Si nous avions une colonne `age` et nous voulions calculer l'âge moyen des clients :

```sql
ALTER TABLE clients
    ADD COLUMN age INT;

UPDATE clients
SET age = 30
WHERE id = 1;
-- (Ajoutez des âges pour les autres clients selon vos besoins)

SELECT AVG(age) AS AgeMoyen
FROM clients;
```

### Exemple 6 : Utilisation de `MIN` et `MAX` pour Trouver les Valeurs Extrêmes

Pour trouver l'âge minimum et maximum :

```sql
SELECT MIN(age) AS AgeMinimum, MAX(age) AS AgeMaximum
FROM clients;
```

### Résumé

- **Fonctions Scalaires** : Opèrent sur chaque ligne individuellement (ex. `UPPER`, `LENGTH`, `EXTRACT`).
- **Fonctions d'Agrégation** : Opèrent sur un ensemble de lignes pour retourner une valeur unique (ex. `COUNT`, `AVG`,
  `MIN`, `MAX`).
- **Fonctions sur les Dates** : Utilisées pour manipuler et extraire des informations de date (ex. `EXTRACT`).

Ces fonctions enrichissent les capacités de SQL en permettant des transformations et des calculs avancés sur les
données.

## Jointure Interne (`INNER JOIN`)

Une jointure interne (INNER JOIN) est une opération qui combine des lignes de deux tables basées sur une condition
commune, généralement une clé étrangère. Elle retourne uniquement les lignes avec lesquelles il y a une correspondance dans les deux
tables.

### Création de la Table commandes

Pour créer une table `commandes` qui enregistre les commandes placées par les clients, nous allons définir une clé
étrangère reliant chaque commande à un client spécifique dans la table `clients`. Voici comment vous pouvez créer cette
table, insérer des données, et la représenter en format Markdown.

```sql
CREATE TABLE commandes
(
    id            SERIAL PRIMARY KEY,
    client_id     INT REFERENCES clients (id),
    date_commande DATE,
    montant       DECIMAL(10, 2)
);

INSERT INTO commandes (client_id, date_commande, montant)
VALUES (1, '2023-01-20', 150.00),
       (1, '2023-02-15', 200.50),
       (2, '2023-03-05', 300.75),
       (3, '2023-03-15', 120.00),
       (4, '2023-04-10', 450.00),
       (4, '2023-04-15', 300.00),
       (4, '2023-04-20', 250.00),
       (6, '2023-06-30', 60.00),
       (8, '2023-08-20', 250.00),
       (9, '2023-09-10', 320.00);
```

| id | Client ID | Date de Commande | Montant |
|----|-----------|------------------|---------|
| 1  | 1         | 2023-01-20       | 150.00  |
| 2  | 1         | 2023-02-15       | 200.50  |
| 3  | 2         | 2023-03-05       | 300.75  |
| 4  | 3         | 2023-03-15       | 120.00  |
| 5  | 4         | 2023-04-10       | 450.00  |
| 6  | 4         | 2023-04-15       | 300.00  |
| 7  | 4         | 2023-04-20       | 250.00  |
| 8  | 6         | 2023-06-30       | 60.00   |
| 9  | 8         | 2023-08-20       | 250.00  |
| 10 | 9         | 2023-09-10       | 320.00  |

### Explications

- **`client_id`** : Cette colonne est une clé étrangère qui référence l'identifiant (`id`) de la table `clients`. Cela
  établit une relation entre les commandes et les clients, indiquant quel client a passé chaque commande.
- **`date_commande`** : Enregistre la date à laquelle la commande a été passée.
- **`montant`** : Indique le montant total de la commande.

Cette structure permet de modéliser une relation un-à-plusieurs entre les clients et les commandes, où un client peut
passer plusieurs commandes.

### Exemple 1 : Ajouter le Courriel de Chaque Client à la Liste des Commandes

Pour obtenir une liste de toutes les commandes avec le courriel du client associé, nous utilisons une jointure interne
entre la table `commandes` et la table `clients`. La condition de jointure est que l'identifiant du client dans la table
`commandes` (`client_id`) doit correspondre à l'identifiant du client dans la table `clients` (`id`).

```sql
SELECT commandes.id AS CommandeID, commandes.date_commande, commandes.montant, clients.email
FROM commandes
         INNER JOIN clients ON commandes.client_id = clients.id;
```

| CommandeID | Date de Commande | Montant | Email                       |
|------------|------------------|---------|-----------------------------|
| 1          | 2023-01-20       | 150.00  | jean.dupont@example.com     |
| 2          | 2023-02-15       | 200.50  | jean.dupont@example.com     |
| 3          | 2023-03-05       | 300.75  | marie.martin@example.com    |
| 4          | 2023-03-15       | 120.00  | paul.durand@example.com     |
| 5          | 2023-04-10       | 450.00  | sophie.lefevre@example.com  |
| 6          | 2023-04-15       | 300.00  | sophie.lefevre@example.com  |
| 7          | 2023-04-20       | 250.00  | sophie.lefevre@example.com  |
| 8          | 2023-06-30       | 60.00   | lucie.simon@example.com     |
| 9          | 2023-08-20       | 250.00  | antoine.lemoine@example.com |
| 10         | 2023-09-10       | 320.00  | claire.roux@example.com     |

#### Explication

- **`SELECT`** : Spécifie les colonnes à inclure dans le résultat. Ici, nous sélectionnons l'ID de la commande, la date
  de commande, le montant, et l'email du client.
- **`FROM commandes`** : Indique que la table principale pour la jointure est `commandes`.
- **`INNER JOIN clients ON commandes.client_id = clients.id`** : Effectue la jointure interne en liant les commandes aux
  clients en utilisant la clé étrangère `client_id` de `commandes` et la clé primaire `id` de `clients`.

### Exemple 2 : Lister Toutes les Commandes de Jean Dupont

Pour lister toutes les commandes passées par Jean Dupont, nous utilisons également une jointure interne, mais avec une
condition supplémentaire pour filtrer les résultats.

```sql
SELECT commandes.id AS CommandeID, commandes.date_commande, commandes.montant
FROM commandes
         INNER JOIN clients ON commandes.client_id = clients.id
WHERE clients.nom = 'Dupont'
  AND clients.prenom = 'Jean';
```

| CommandeID | Date de Commande | Montant |
|------------|------------------|---------|
| 1          | 2023-01-20       | 150.00  |
| 2          | 2023-02-15       | 200.50  |

#### Explication

- **`WHERE clients.nom = 'Dupont' AND clients.prenom = 'Jean'`** : Ajoute une condition pour filtrer les résultats afin
  de n'inclure que les commandes passées par Jean Dupont.
- La jointure relie les tables `commandes` et `clients` de la même manière que dans le premier exemple, mais le `WHERE`
  restreint les résultats à un client spécifique.

### Résumé

- **Jointure Interne** : Combine les lignes de deux tables avec lesquelles il y a une correspondance dans les colonnes spécifiées.
- **Utilisation** : Souvent utilisée pour combiner des données réparties sur plusieurs tables, comme associer des
  commandes à des clients.
- **Syntaxe** : `INNER JOIN` suivi de la condition de correspondance (`ON`).

Les jointures internes sont essentielles pour exploiter pleinement les relations entre les tables dans une base de
données relationnelle. Elles permettent de combiner et d'analyser des données provenant de différentes sources de
manière cohérente et efficace.

## Jointure externe gauche

Une jointure externe gauche (`LEFT JOIN`) est utilisée pour retourner toutes les lignes de la table de gauche (dans ce
cas, `clients`), ainsi que les lignes correspondantes de la table de droite (`commandes`). Si aucune correspondance
n'est trouvée dans la table de droite, les résultats contiendront des valeurs nulles pour les colonnes de la table de
droite.

### Requête : Trouver les Clients qui n'ont pas de Commandes

Pour identifier les clients qui n'ont pas passé de commandes, nous pouvons utiliser une jointure externe gauche et
filtrer les résultats pour inclure uniquement les lignes avec lesquelles il n'y a pas de correspondance dans la table `commandes`.

```sql
SELECT clients.id, clients.nom, clients.prenom, clients.email
FROM clients
         LEFT JOIN commandes ON clients.id = commandes.client_id
WHERE commandes.id IS NULL;
```

#### Explication

- **`LEFT JOIN`** : Retourne toutes les lignes de la table `clients` et les lignes correspondantes de `commandes`. Si un
  client n'a pas de commande, les colonnes de `commandes` contiendront des valeurs nulles.
- **`WHERE commandes.id IS NULL`** : Filtre les résultats pour inclure uniquement les clients qui n'ont pas de
  commandes, c'est-à-dire ceux pour lesquels `commandes.id` est `NULL`.

### Résultats de la Requête

Voici le tableau en format Markdown représentant les résultats de cette requête :

| id | Nom     | Prénom  | Email                     |
|----|---------|---------|---------------------------|
| 5  | Moreau  | Pierre  | pierre.moreau@example.com |
| 7  | Laurent | Julie   | julie.laurent@example.com |
| 10 | Petit   | Nicolas | nicolas.petit@example.com |

Ces résultats montrent les clients qui n'ont pas de commandes enregistrées dans la base de données. La jointure externe
gauche est particulièrement utile pour identifier les enregistrements dans une table qui ne correspondent pas à des
enregistrements dans une autre table.

## Comparaison des Jointures Externes

### 1. Jointure Gauche (`LEFT JOIN`)

- Retourne toutes les lignes de la table de gauche (première table mentionnée).
- Inclut les lignes correspondantes de la table de droite si elles existent.
- Si aucune correspondance n'est trouvée, les colonnes de la table de droite contiennent `NULL`.

```sql
SELECT *
FROM clients
         LEFT JOIN commandes ON clients.id = commandes.client_id;
```

### 2. Jointure Droite (`RIGHT JOIN`)

- Retourne toutes les lignes de la table de droite (deuxième table mentionnée).
- Inclut les lignes correspondantes de la table de gauche si elles existent.
- Si aucune correspondance n'est trouvée, les colonnes de la table de gauche contiennent NULL.

```sql
SELECT *
FROM clients
         RIGHT JOIN commandes ON clients.id = commandes.client_id;
```

### 3. Jointure Externe Complète (`FULL JOIN`)

- Combine les résultats d'une jointure gauche et d'une jointure droite.
- Retourne toutes les lignes des deux tables.
- Si aucune correspondance n'est trouvée, les colonnes de la table non correspondante contiennent NULL.

```sql
SELECT *
FROM clients
         FULL OUTER JOIN commandes ON clients.id = commandes.client_id;
```

### Différences Clés

1. **LEFT JOIN** : Utile pour trouver tous les enregistrements de la table de gauche, même s'ils n'ont pas de
   correspondance dans la table de droite.

2. **RIGHT JOIN** : Similaire au LEFT JOIN, mais en se concentrant sur la table de droite. Moins couramment utilisé, car
   on peut généralement obtenir le même résultat en inversant l'ordre des tables et en utilisant un LEFT JOIN.

3. **FULL JOIN** : Combine les résultats des deux jointures précédentes. Utile pour voir toutes les données des
   deux tables, qu'il y ait correspondance ou non.

### Exemple d'Utilisation

- **LEFT JOIN** : Trouver tous les clients, qu'ils aient passé des commandes ou non.
- **RIGHT JOIN** : Trouver toutes les commandes, même si elles sont associées à des clients qui n'existent plus dans la
  table clients.
- **FULL JOIN** : Voir tous les clients et toutes les commandes, en identifiant les cas où il n'y a pas de
  correspondance dans l'une ou l'autre table.

### Note Importante

Le mot `OUTER` peut être ajouté à chacune de ces jointures (`LEFT OUTER JOIN`, `RIGHT OUTER JOIN` et `FULL OUTER JOIN`),
mais il est optionnel. L'utilisation de `OUTER` ne change pas le comportement de la jointure ; elle sert simplement à
rendre explicite le fait qu'il s'agit d'une jointure externe.

Ces jointures externes sont essentielles pour analyser des données qui peuvent ne pas avoir de correspondances parfaites
entre les tables, permettant ainsi une vue plus complète des données disponibles.

Les regroupements en SQL, réalisés avec la clause `GROUP BY`, permettent d'agréger des données en fonction de
critères spécifiques. Voici une explication détaillée avec des exemples utilisant nos tables `clients` et `commandes`.

## Regroupements avec `GROUP BY`

Les regroupements en SQL, réalisés avec la clause `GROUP BY`, permettent d'agréger des données en fonction de
critères spécifiques. Voici une explication détaillée avec des exemples utilisant nos tables `clients` et `commandes`.

La clause `GROUP BY` est utilisée pour regrouper des lignes qui ont les mêmes valeurs dans des colonnes spécifiées. Elle
est souvent utilisée avec des fonctions d'agrégation comme `COUNT()`, `SUM()`, `AVG()`, etc.

### Exemple 1 : Nombre de Commandes par Client

```sql
SELECT client_id, COUNT(*) AS nombre_commandes
FROM commandes
GROUP BY client_id;
```

Ce qui pourrait donner un résultat comme :

| client_id | nombre_commandes |
|-----------|------------------|
| 1         | 2                |
| 2         | 1                |
| 3         | 1                |
| 4         | 3                |
| 6         | 1                |
| 8         | 1                |
| 9         | 1                |

Le concept de regroupement dans SQL, particulièrement avec la clause `GROUP BY`, est essentiel pour comprendre comment
les données sont agrégées et résumées. Voici une explication détaillée du processus de formation des groupes :

### Concept de Regroupement

#### Définition des Groupes

- **Clé de Regroupement** : Les groupes sont définis par les valeurs distinctes d'une ou plusieurs colonnes spécifiées
  dans la clause `GROUP BY`. Dans notre exemple, la clé de regroupement est `client_id`. Cela signifie que chaque valeur
  unique de `client_id` dans la table `commandes` définit un groupe distinct.

#### Formation des Groupes

1. **Identification des Valeurs Uniques** : Le moteur SQL identifie toutes les valeurs uniques de la colonne `client_id`
   dans la table `commandes`. Chaque valeur unique devient la base d'un groupe.

2. **Regroupement des Lignes** : Toutes les lignes (ou enregistrements) de la table qui partagent la même valeur de
   `client_id` sont regroupées ensemble. Cela signifie que chaque groupe contient toutes les commandes associées à un
   client particulier.

#### Processus de Regroupement

- **Extraction des Données** : Le moteur SQL parcourt la table `commandes` et extrait chaque ligne.
- **Association aux Groupes** : Pour chaque ligne, le moteur vérifie la valeur de `client_id` et l'associe au groupe
  correspondant. Si un groupe pour cette valeur de `client_id` n'existe pas encore, il est créé.
- **Agrégation des Données** : Une fois les lignes regroupées, des fonctions d'agrégation (comme `COUNT`, `SUM`, etc.)
  sont appliquées à chaque groupe pour calculer des statistiques ou des résumés.

#### Exemple Visuel

Imaginons une table simplifiée avec les `client_id` suivants : 1, 1, 2, 3, 4, 4, 4.

- **Groupes Formés** :
    - Groupe pour `client_id` 1 : [1, 1]
    - Groupe pour `client_id` 2 :
    - Groupe pour `client_id` 3 :
    - Groupe pour `client_id` 4 : [4, 4, 4]

Chaque groupe contient toutes les lignes avec lesquelles `client_id` a la même valeur. Les fonctions d'agrégation sont ensuite
appliquées à ces groupes pour obtenir des résultats comme le nombre total de commandes par client.

### Utilité du Regroupement

Le regroupement permet de transformer des données brutes en informations significatives en résumant et en analysant les
données selon des critères spécifiques. Cela est particulièrement utile pour générer des rapports, des statistiques, et
des analyses qui nécessitent une vue d'ensemble des données agrégées.

### Exemple 2 : Montant Total des Commandes par Ville

```sql
SELECT clients.ville, SUM(commandes.montant) AS montant_total
FROM clients
         JOIN commandes ON clients.id = commandes.client_id
GROUP BY clients.ville;
```

Résultat possible :

| ville      | montant_total |
|------------|---------------|
| Paris      | 350.50        |
| Lyon       | 300.75        |
| Marseille  | 120.00        |
| Toulouse   | 1000.00       |
| Nantes     | 60.00         |
| Strasbourg | 250.00        |
| Lille      | 320.00        |

### Utilisation de `HAVING`

La clause `HAVING` est utilisée avec `GROUP BY` pour filtrer les résultats des groupes. Contrairement à `WHERE` qui
filtre les lignes individuelles avant le regroupement, `HAVING` filtre les groupes après le regroupement.

### Exemple 3 : Clients avec Plus d'Une Commande

```sql
SELECT client_id, COUNT(*) AS nombre_commandes
FROM commandes
GROUP BY client_id
HAVING COUNT(*) > 1;
```

Résultat possible :

| client_id | nombre_commandes |
|-----------|------------------|
| 1         | 2                |
| 4         | 3                |

### Exemple 4 : Villes avec un Montant Total de Commandes Supérieur à 500

```sql
SELECT clients.ville, SUM(commandes.montant) AS montant_total
FROM clients
         JOIN commandes ON clients.id = commandes.client_id
GROUP BY clients.ville
HAVING SUM(commandes.montant) > 500;
```

Résultat possible :

```markdown
| ville     | montant_total |
|-----------|---------------|
| Toulouse  | 1000.00       |
```

### Résumé

- **`GROUP BY`** : Regroupe les lignes ayant les mêmes valeurs dans les colonnes spécifiées.
- Utilisé avec des fonctions d'agrégation (`COUNT()`, `SUM()`, `AVG()`, etc.) pour calculer des statistiques sur chaque
  groupe.
- **`HAVING`** : Filtre les résultats des groupes après le regroupement.
- `WHERE` filtre les lignes individuelles avant le regroupement, tandis que `HAVING` filtre les groupes après le
  regroupement.

Les regroupements sont essentiels pour l'analyse de données, permettant de résumer et d'agréger des informations de
manière significative à partir de grands ensembles de données.


-------

??? info "Utilisation de l'IA"
    Page rédigée en partie avec l'aide d'un assistant IA. L'IA a été utilisée pour générer des 
    explications, des exemples et/ou des suggestions de structure. Toutes les informations ont 
    été vérifiées, éditées et complétées par l'auteur.