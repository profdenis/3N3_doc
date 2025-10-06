# 6 - Utilisation de `date` et `timestamp`

Voici un guide sur les opérations et fonctions sur les dates et les `timestamp` dans PostgreSQL.

## Différences entre `date` et `timestamp`

Voici les principales différences entre les types `date` et `timestamp` dans PostgreSQL :

1. Contenu stocké :

    - `date` stocke uniquement une date (année, mois, jour)
    - `timestamp` stocke une date et une heure (année, mois, jour, heure, minute, seconde, et optionnellement des
      fractions de seconde)

2. Taille de stockage :

    - `date` occupe 4 octets
    - `timestamp` occupe 8 octets

3. Précision :

    - `date` a une précision d'un jour
    - `timestamp` a une précision d'une microseconde

4. Plage de valeurs :

    - `date` : de 4713 avant J.-C. à 5874897 après J.-C.
    - `timestamp` : de 4713 avant J.-C. à 294276 après J.-C.

5. Opérations temporelles :

    - `date` permet des opérations basiques sur les dates
    - `timestamp` permet des opérations plus précises incluant les heures, minutes et secondes

6. Fuseau horaire :

    - `date` n'inclut pas d'information de fuseau horaire
    - `timestamp` peut inclure ou non une information de fuseau horaire (selon qu'on utilise `timestamp with time zone`
      ou `timestamp without time zone`)

7. Utilisation :

    - `date` est utilisé quand seule la date est importante (ex: date de naissance)
    - `timestamp` est utilisé quand on a besoin de la date et de l'heure précise (ex: horodatage d'une transaction)

En général, `timestamp` offre plus de flexibilité et de précision, mais `date` peut être suffisant et plus efficace en
termes de stockage si seule la date est nécessaire.

### Conversion

Pour convertir un `timestamp` en `date`, ou vice-versa, on peut _caster_ dans l'un ou dans l'autre :

```sql
select (now()::date);
-- now() donne le timestamp courant
```

| now        |
|:-----------|
| 2024-09-27 |


```sql
select (CURRENT_DATE::timestamp);
-- CURRENT_DATE donne la date d'aujourd'hui
```

| current\_date              |
|:---------------------------|
| 2024-09-27 00:00:00.000000 |

Citations:

- [1] https://www.squash.io/how-to-compare-dates-in-postgresql-databases/
- [2] https://stackoverflow.com/questions/5876218/difference-between-timestamps-with-without-time-zone-in-postgresql
- [3] https://www.enterprisedb.com/blog/postgresql-interval-date-timestamp-and-time-data-types
- [4] https://neon.tech/docs/data-types/date-and-time
- [5] https://stackoverflow.com/questions/60825438/postgres-date-compare-with-iso-timestamp
- [6] https://www.sqlines.com/postgresql/how-to/datediff
- [7] https://www.postgresql.org/docs/current/datatype-datetime.html

## Comparaisons simples avec des dates en format ISO

PostgreSQL supporte nativement le format de date ISO 8601 (YYYY-MM-DD) pour les comparaisons :

- Table `users`

| id | name  | created                    |
|:---|:------|:---------------------------|
| 1  | denis | 2024-09-25 20:42:08.200074 |
| 3  | bob   | 2023-08-09 12:12:00.000000 |
| 2  | alice | 2024-07-25 02:42:08.200074 |

```sql
select *
from users
where created > '2024-09-15';
```

| id | name  | created                    |
|:---|:------|:---------------------------|
| 1  | denis | 2024-09-25 20:42:08.200074 |

```sql
-- Sélectionner les enregistrements entre deux dates
select *
from users
where created between '2023-01-01' and '2023-12-12';
```

| id | name | created                    |
|:---|:-----|:---------------------------|
| 3  | bob  | 2023-08-09 12:12:00.000000 |

```sql
-- Sélectionner les enregistrements entre deux dates
select *
from users
where created between '2023-01-01' and '2023-08-09';
```

| id | name | created |
|:---|:-----|:--------|

## Fonctions courantes sur les dates

### EXTRACT

Permet d'extraire une partie spécifique d'une date ou d'un timestamp[1].

```sql
select name, extract(year from created) as annee
from users;
```

| name  | annee |
|:------|:------|
| denis | 2024  |
| alice | 2024  |
| bob   | 2023  |

```sql
select name, extract(month from created) as mois
from users;
```

| name  | mois |
|:------|:-----|
| denis | 9    |
| alice | 7    |
| bob   | 8    |

### DATE_PART

Similaire à EXTRACT, permet d'obtenir une partie spécifique d'une date[2].

```sql
select name, date_part('year', created) as annee
from users;
```

| name  | annee |
|:------|:------|
| denis | 2024  |
| alice | 2024  |
| bob   | 2023  |

#### Différences entre `EXTRACT` et `DATE_PART`

Voici les principales différences entre les fonctions `EXTRACT()` et `DATE_PART()` dans PostgreSQL :

1. Syntaxe :
    - `EXTRACT()` utilise la syntaxe : `EXTRACT(champ FROM source)`
    - `DATE_PART()` utilise la syntaxe : `DATE_PART('champ', source)`

2. Standard SQL :
    - `EXTRACT()` est conforme au standard SQL
    - `DATE_PART()` est spécifique à PostgreSQL

3. Type de retour :
    - `EXTRACT()` retourne un type numeric depuis PostgreSQL 14
    - `DATE_PART()` retourne toujours un double precision (float8)

4. Performance :
    - Avant PostgreSQL 14, `EXTRACT()` était réécrit en `DATE_PART()` en interne
    - Depuis PostgreSQL 14, les implémentations sont différentes

5. Précision :
    - `EXTRACT()` peut être plus précis pour certains calculs depuis PostgreSQL 14
    - `DATE_PART()` peut perdre en précision dans certains cas à cause du type float

6. Utilisation :
    - `EXTRACT()` est généralement préféré pour la compatibilité SQL
    - `DATE_PART()` reste utilisé par habitude ou pour des raisons de compatibilité avec d'anciennes versions

En pratique, les deux fonctions sont très similaires et interchangeables dans la plupart des cas. Le choix dépend
souvent des préférences personnelles ou des conventions de codage. Pour une meilleure portabilité et précision,
`EXTRACT()` est généralement recommandé.

Citations:

- [1] https://www.squash.io/how-to-compare-dates-in-postgresql-databases/
- [2] https://docs.getdbt.com/blog/extract-sql-love-letter
- [3] https://www.postgresqltutorial.com/postgresql-date-functions/postgresql-extract/
- [4] https://materialize.com/docs/sql/functions/date-part/
- [5] https://stackoverflow.com/questions/38442340/difference-between-extractyear-from-timestamp-function-and-date-partyear-t
- [6] https://www.postgresqltutorial.com/postgresql-date-functions/postgresql-date_part/
- [7] https://www.timescale.com/learn/postgresql-date-and-time-functions
- [8] https://www.commandprompt.com/education/date-time-functions-in-postgresql-with-examples/

### DATE_TRUNC

Tronque une date à une précision spécifiée[2].

```sql
select name, date_trunc('month', created)
from users;
```

| name  | date\_trunc                |
|:------|:---------------------------|
| denis | 2024-09-01 00:00:00.000000 |
| bob   | 2023-08-01 00:00:00.000000 |
| alice | 2024-07-01 00:00:00.000000 |

### `now` et `current_date`

- `now()` : Retourne la date et l'heure actuelles avec le fuseau horaire[2].
- `current_date` : Retourne la date actuelle[2].

```sql
select now();
```

| now                               |
|:----------------------------------|
| 2024-09-25 21:12:05.032204 +00:00 |

```sql
select current_date;
```

| current\_date |
|:--------------|
| 2024-09-25    |

### Différences entre `now()` et `current_date`

1. Type de retour :
    - `now()` retourne un timestamp with time zone (date et heure avec fuseau horaire)
    - `current_date` retourne uniquement une date (sans heure ni fuseau horaire)

2. Précision :
    - `now()` inclut l'heure, les minutes, les secondes et les microsecondes
    - `current_date` ne retourne que la date (année, mois, jour)

3. Utilisation dans les transactions :
    - `now()` renvoie la même valeur tout au long d'une transaction
    - `current_date` peut changer si la transaction chevauche minuit

4. Performance :
    - `current_date` est généralement plus rapide car il ne traite que la date[3]

5. Comportement avec les fuseaux horaires :
    - `now()` prend en compte le fuseau horaire de la session
    - `current_date` retourne toujours la date locale du serveur

6. Flexibilité :
    - `now()` peut être converti facilement en d'autres types temporels
    - `current_date` est limité aux opérations sur les dates

7. Constance :
    - `now()` reste constant dans une transaction, utile pour la cohérence des données
    - `current_date` peut changer, reflétant toujours la date actuelle

En résumé, `now()` offre plus de précision et de flexibilité, tandis que CURRENT_DATE est plus simple et plus rapide
lorsque seule la date est nécessaire[1][3][4].

Citations:

- [1] https://www.devart.com/dbforge/postgresql/studio/compare-dates.html
- [2] https://www.cybertec-postgresql.com/en/postgresql-now-vs-nowtimestamp-vs-clock_timestamp/
- [3] https://www.postgresql.org/message-id/48B1BE33.3060505@vieonet.com
- [4] https://stackoverflow.com/questions/57469642/difference-between-current-timestamp-and-current-date
- [5] https://stackoverflow.com/questions/5876218/difference-between-timestamps-with-without-time-zone-in-postgresql
- [6] https://www.squash.io/how-to-compare-dates-in-postgresql-databases/
- [7] https://www.timescale.com/learn/postgresql-date-and-time-functions
- [8] https://www.postgresql.org/docs/8.1/functions-datetime.html

## Opérations avec `interval`

L'utilisation d'`interval` permet d'effectuer des calculs sur les dates[5]:

```sql
-- Ajouter 1 mois à la date actuelle
select current_date + interval '1 month';
```

| ?column?                   |
|:---------------------------|
| 2024-10-25 00:00:00.000000 |

```sql
-- Soustraire 2 semaines à une date spécifique
select date '2023-01-15' - interval '2 weeks';
```

```sql
-- les utilisateurs créés dans le dernier mois
select name, users.created
from users
where created > current_date - interval '1 month';
```

| name  | created                    |
|:------|:---------------------------|
| denis | 2024-09-25 20:42:08.200074 |

## Exemples avancés

### Calculer l'âge

```sql
select name, age(created)
from users;
-- Retourne l'intervalle entre la date spécifiée et la date actuelle
```

| name  | age                                                     |
|:------|:--------------------------------------------------------|
| denis | 0 years 0 mons 0 days -20 hours -42 mins -8.200074 secs |
| bob   | 1 years 1 mons 15 days 11 hours 48 mins 0.0 secs        |
| alice | 0 years 1 mons 30 days 21 hours 17 mins 51.799926 secs  |

### Convertir une chaîne en date

```sql
select to_date('2023-09-21', 'yyyy-mm-dd');
-- Convertit la chaîne en date
```

### Formater une date

```sql
select to_char(current_date, 'DD/MM/YYYY');
-- Formate la date actuelle au format JJ/MM/AAAA
```

| to\_char   |
|:-----------|
| 25/09/2024 |


Ces exemples couvrent les opérations les plus courantes sur les dates et timestamps dans PostgreSQL. N'oubliez pas que
PostgreSQL offre de nombreuses autres fonctions et possibilités pour manipuler les dates et les heures selon vos besoins
spécifiques.

Citations:

- [1] https://www.postgresqltutorial.com/postgresql-date-functions/postgresql-extract/
- [2] https://www.postgresqltutorial.com/postgresql-date-functions/
- [3] https://www.timescale.com/learn/postgresql-date-and-time-functions
- [4] https://www.tutorialspoint.com/postgresql/postgresql_date_time.htm
- [5] https://www.postgresqltutorial.com/postgresql-tutorial/postgresql-interval/
- [6] https://www.commandprompt.com/education/postgresql-interval-data-type-with-examples/
- [7] https://www.commandprompt.com/education/date-time-functions-in-postgresql-with-examples/
- [8] https://www.geeksforgeeks.org/postgresql-interval-data-type/
- [9] https://www.tutorialsteacher.com/postgresql/interval-data-type
- [10] https://www.enterprisedb.com/blog/postgresql-interval-date-timestamp-and-time-data-types
- [11] https://www.squash.io/how-to-compare-dates-in-postgresql-databases/
- [12] https://stackoverflow.com/questions/60825438/postgres-date-compare-with-iso-timestamp
- [13] https://stackoverflow.com/questions/31123409/interval-days-in-postgresql-with-two-parameters
- [14] https://www.postgresql.org/docs/current/datatype-datetime.html

## Opérations sur les dates

1. Soustraction de dates :
    - Vous pouvez soustraire deux dates pour obtenir un intervalle.
    - Exemple : `date '2023-09-21' - date '2023-01-01'` retournera un intervalle représentant la différence entre ces
      deux dates[3].

2. Addition/Soustraction avec un intervalle :
    - Vous pouvez ajouter ou soustraire un intervalle à une date.
    - Exemple : `date '2023-09-21' + interval '1 month'` ajoutera un mois à la date spécifiée[2].

## Opérations sur les timestamps

1. Soustraction de timestamps :
    - La soustraction de deux timestamps retourne un intervalle.
    - Exemple : `timestamp '2023-09-21 10:00:00' - timestamp '2023-09-20 09:00:00'` donnera l'intervalle entre ces deux
      moments[4].

2. Addition/Soustraction avec un intervalle :
    - Comme pour les dates, vous pouvez ajouter ou soustraire des intervalles aux timestamps.
    - Exemple : `timestamp '2023-09-21 10:00:00' - interval '2 hours'` soustraira 2 heures au timestamp[6].

## Points importants à noter

1. Précision : Les opérations sur les timestamps sont plus précises car elles prennent en compte les heures, minutes et
   secondes[1].

2. Fuseaux horaires : Pour les `timestamp with time zone`, PostgreSQL gère automatiquement les ajustements de fuseau
   horaire lors des opérations[7].

3. Intervalles complexes : Vous pouvez utiliser des intervalles complexes comme `interval '1 year 2 months 3 days'` pour
   des calculs plus élaborés[2].

4. Fonction AGE() : Pour calculer la différence entre deux dates ou timestamps de manière plus lisible, vous pouvez
   utiliser la fonction `age()`[5].

Ces opérations sont très utiles pour effectuer des calculs temporels dans vos requêtes, comme trouver la durée entre
deux événements ou calculer des dates futures ou passées basées sur une date de référence.

Citations:

- [1] https://docs.postgresql.fr/16/functions-datetime.html
- [2] https://learnsql.fr/blog/fonctions-de-date-de-postgresql/
- [3] https://forums.postgresql.fr/viewtopic.php?id=1067
- [4] https://postgresql.developpez.com/documentation/francais/10beta4/functions-datetime.html
- [5] https://sql.sh/fonctions/date-heure
- [6] https://postgresql.developpez.com/documentation/francais/8.2.5/functions-datetime.html
- [7] https://www.postgresql.org/docs/8.1/functions-datetime.html

## Conversion d'un intervalle en nombre d'heures (float)

La méthode la plus courante pour convertir un intervalle en nombre est de le transformer en secondes, puis de le diviser
pour obtenir l'unité souhaitée. Pour obtenir un nombre d'heures en float, vous pouvez utiliser la fonction EXTRACT avec
l'unité 'epoch' :

```sql
select extract(epoch from interval '4 hours 30 minutes') / 3600.0 as heures;
```

Cette requête retournera 4.5, représentant 4 heures et 30 minutes en format décimal[1].

## Conversion en numeric

Pour une conversion en type numeric, qui offre une précision arbitraire, vous pouvez utiliser une approche similaire :

```sql
select cast(extract(epoch from interval '4 hours 30 minutes') / 3600.0 as numeric(10, 2)) as heures;
```

Cette requête convertira l'intervalle en un nombre de type numeric avec 2 décimales[6].

## Points importants à noter

1. _Précision_ : La conversion d'un intervalle en nombre peut entraîner une perte de précision, surtout pour les
   intervalles incluant des années ou des mois, car ces unités n'ont pas une durée fixe en secondes[5].

2. _Unités variables_ : Les années et les mois sont traités de manière spéciale dans les intervalles PostgreSQL. Par
   exemple, '1 year' n'est pas toujours égal à 365 jours, ce qui peut conduire à des résultats inattendus lors de la
   conversion[3].

3. _Choix de l'unité_ : Vous pouvez ajuster le diviseur (3600.0 dans les exemples ci-dessus) pour obtenir d'autres unités.
   Par exemple, utilisez 86400.0 pour obtenir des jours, ou 60.0 pour des minutes.

4. _Fonction `to_char()`_ : Pour des besoins de formatage plus complexes, vous pouvez utiliser la fonction to_char() qui
   permet de formater divers types de données, y compris les intervalles[2].

En conclusion, bien que PostgreSQL ne fournisse pas de conversion directe d'un intervalle en float ou numeric, il est
possible d'effectuer cette conversion en utilisant la fonction EXTRACT avec l'unité 'epoch', suivie d'une division et
éventuellement d'un CAST si nécessaire.

Citations:

- [1] https://stackoverflow.com/questions/952493/how-do-i-convert-an-interval-into-a-number-of-hours-with-postgres
- [2] https://www.postgresql.org/docs/8.1/functions-formatting.html
- [3] https://www.alibabacloud.com/blog/how-to-convert-a-postgresql-time-interval-to-a-numeric-value_598990
- [4] https://www.postgresql.org/message-id/20180914134014.18a9cf7d%40wrkhors.com
- [5] https://github.com/cockroachdb/cockroach/issues/43272
- [6] https://www.postgresql.org/docs/9.3/datatype-numeric.html



-------

??? info "Utilisation de l'IA"
   Page rédigée en partie avec l'aide d'un assistant IA. L'IA a été utilisée pour générer des
   explications, des exemples et/ou des suggestions de structure. Toutes les informations ont
   été vérifiées, éditées et complétées par l'auteur.