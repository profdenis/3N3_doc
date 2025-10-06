# 7 - Requêtes SQL imbriquées

- Une *sous-requête* est une requête à l'intérieur d'une autre requête (ou d'une
  autre sous-requête).
- Nous pouvons avoir plusieurs niveaux de sous-requêtes.
- Parfois, nous faisons référence aux sous-requêtes comme des requêtes
  *internes* et aux requêtes englobantes comme des requêtes *externes*.

## Base de Données Universitaire

```sql
set search_path to university;
```

### Sous-requêtes Non Corrélées

- Aussi appelées sous-requêtes ***simples***.
- Les sous-requêtes *corrélées* seront abordées plus tard.
- Les sous-requêtes non corrélées sont indépendantes de leurs requêtes
  externes (englobantes).
- Elles peuvent s'exécuter seules.
- Elles ne dépendent pas de quelque chose défini dans les requêtes externes.

#### Exemples : avec 2 sous-requêtes non corrélées

1- Trouver les étudiants et les instructeurs avec une adresse
email `@example.com`

```sql
select name, email
from student
where email like '%@example.com'
union
select name, email
from instructor
where email like '%@example.com';
```

2- Vérifier s'il y a des étudiants et des instructeurs avec la même adresse
email

```sql
select email
from student
intersect
select email
from instructor;
```

3- Trouver les cours qui n'ont jamais été offerts

```sql
select cid
from course
except
select cid
from offering;

select c.cid
from course c
         left join offering o on c.cid = o.cid
where o.oid is null;
```

4- Trouver les étudiants non inscrits à un cours

```sql
select sid
from student
except
select sid
from enrollment;
```

5- Trouver les offres de cours dans lesquelles aucun étudiant n'est inscrit

```sql
select oid
from offering
except
select oid
from enrollment;
```

### Sous-requêtes Scalaires

- Les sous-requêtes scalaires sont le type le plus simple de sous-requêtes.
- Elles renvoient toujours exactement 1 ligne contenant exactement 1 colonne.
- Elles sont souvent (mais pas toujours) obtenues en calculant une fonction
  d'agrégat.

#### Exemples

1- Trouver le nombre d'étudiants avec une adresse email `@example.com`

```sql
select count(sid) as n_students
from student
where email like '%@example.com';
```

2- Trouver les cours qui ont été offerts plus souvent que le cours `DB`

a) Trouver le nombre de fois que `DB` a été offert

```sql
select count(o.cid)
from course c
         left join offering o on c.cid = o.cid
where c.code = 'DB';
```

b) Insérer le résultat précédent dans la clause `HAVING` d'une requête calculant
le nombre de fois que chaque cours a été offert. Il pourrait être préférable de
commencer avec un nombre fixe (par exemple 3) au lieu d'insérer immédiatement la
première requête dans la seconde

```sql
select c.cid, c.code
from course c
         left join offering o on c.cid = o.cid
group by c.cid, c.code
having count(o.oid) > 3;
```

```sql
select c.cid, c.code
from course c
         left join offering o on c.cid = o.cid
group by c.cid, c.code
having count(o.oid) > (select count(o.cid)
                       from course c
                                left join offering o on c.cid = o.cid
                       where c.code = 'DB');
```

3- Trouver le nombre moyen de fois que chaque cours a été offert. Commencer par
trouver le nombre de fois que chaque cours a été offert, puis prendre la moyenne

```sql
select c.cid, c.code, count(o.oid) as n_offerings
from course c
         left join offering o on c.cid = o.cid
group by c.cid;
```

```sql
-- ne fonctionnera pas
select c.cid, c.code, avg(count(o.oid)) as n_offerings
from course c
         left join offering o on c.cid = o.cid
group by c.cid;
```

```sql
select round(avg(n_offerings), 2) as avg_n_offerings
from (select count(o.oid) as n_offerings
      from course c
               left join offering o on c.cid = o.cid
      group by c.cid) as T;

with T as (select count(o.oid) as n_offerings
           from course c
                    left join offering o on c.cid = o.cid
           group by c.cid)
select round(avg(n_offerings), 2) as avg_n_offerings
from T;
```

4- Trouver les cours qui ont été offerts plus souvent que la moyenne (nombre de
fois que chaque cours a été offert)

```sql
select c.cid, c.code
from course c
         left join offering o on c.cid = o.cid
group by c.cid
having count(o.oid) > (select avg(n_offerings) as avg_n_offerings
                       from (select count(o.oid) as n_offerings
                             from course c
                                      left join offering o on c.cid = o.cid
                             group by c.cid) as T);
```

### Format de Requête SQL `with`

- Afin d'exprimer plus facilement des requêtes avec plusieurs niveaux de
  sous-requêtes, nous pouvons utiliser le style de
  requête `with ... select ...`.
- Nous définissons (en quelque sorte) des tables temporaires avant que la
  requête principale `select` ne commence.
- Ensuite, nous utilisons les tables temporaires dans la requête
  principale `select` comme si elles étaient des tables stockées dans la base de
  données.

```sql
with T2 as (select avg(n_offerings) as avg_n_offerings
            from (select c.cid, c.code, count(o.oid) as n_offerings
                  from course c
                           left join offering o on c.cid = o.cid
                  group by c.cid) as T1)
select c.cid, c.code
from course c
         left join offering o on c.cid = o.cid
group by c.cid
having count(o.oid) > (select * from T2);

with n_offerings_per_course as (select c.cid,
                                       c.code,
                                       count(o.oid) as n_offerings
                                from course c
                                         left join offering o on c.cid = o.cid
                                group by c.cid)
select cid, code
from n_offerings_per_course
where n_offerings > (select avg(n_offerings) as avg_n_offerings
                     from n_offerings_per_course);
```

- Ne pas *sur-utiliser* la syntaxe `with`
- Par exemple, ne réécrivez pas cette requête

```sql
select name, email 
from student 
where email like '%@example.com'
union
select name, email 
from instructor 
where email like '%@example.com';
```

  comme

```sql
with students_example as (select name, email from student where email like '%@example.com'),
   instructor_example as (select name, email from instructor where email like '%@example.com')
select * from students_example
union
select * from instructor_example;
```

- Bien que cette requête soit techniquement correcte et équivalente à la requête
  originale, l'utilisation de `with` pour définir 2 tables temporaires est
  excessive ici et réduit en fait la lisibilité.
- Utilisez `with` uniquement lorsque les sous-requêtes sont compliquées ou
  lorsqu'il y a plusieurs niveaux de sous-requêtes.
- Nous pouvons également utiliser `with recursive` pour écrire des requêtes
  récursives.

### Valeurs `NULL` en SQL

- SQL utilise une *logique à 3 valeurs* au lieu de la logique booléenne (une
  logique à 2 valeurs).
- Les 3 valeurs sont `true`, `false` et `null` (ou `T`, `F` et `N` dans le
  tableau ci-dessous)...
- Les 2 premières valeurs ont le sens habituel, tandis que `null` peut avoir
  différentes significations :
    - *inconnu*
    - *non applicable*
    - *n'a pas d'importance*
- Les opérateurs logiques doivent être mis à jour pour tenir compte des
  valeurs `null`.

| A | B | NOT A | A OR B | A AND B |
|---|---|-------|--------|---------|
| T | T | F     | T      | T       |
| T | F | F     | T      | F       |
| T | N | F     | T      | N       |
| F | T | T     | T      | F       |
| F | F | T     | F      | F       |
| F | N | T     | N      | F       |
| N | T | N     | T      | N       |
| N | F | N     | N      | F       |
| N | N | N     | N      | N       |

### Sous-requêtes Non-Scalaires

- Si une (sous-)requête renvoie plus d'une ligne et/ou plus d'une colonne, alors
  ce n'est **pas** une sous-requête scalaire.
- Tenter d'utiliser les opérateurs de comparaison ou arithmétiques normaux avec
  des sous-requêtes non-scalaires échouera s'il y a plus d'une ligne.
- Certains SGBD, comme PostgreSQL, permettent à certains opérateurs de
  fonctionner avec des sous-requêtes donnant exactement 1 ligne mais plusieurs
  colonnes.
- PostgreSQL est un ORDBMS, donc il est plus flexible avec les types de données.
- Il verra la seule ligne avec plusieurs colonnes comme un seul objet avec
  plusieurs champs.
- En général, nous devons utiliser des opérateurs spéciaux pour traiter les
  sous-requêtes non-scalaires :
    - `IN`, `NOT IN`, `EXISTS`, `NOT EXISTS`, `ANY`, `ALL`

#### `IN`

- `expression IN (sub-query)`
    - c'est la même chose que $\in$ en notation mathématique (sauf que nous
      devons gérer les valeurs `null`)
    - la sous-requête doit renvoyer exactement 1 colonne
    - `true` si l'expression est égale à 1 des lignes dans les résultats de la
      sous-requête
    - `false` si l'expression n'est pas `null` et qu'il n'y a pas de
      valeurs `null` dans la sous-requête et que l'expression n'est égale à
      aucune ligne de la sous-requête
    - `null` si l'expression est `null` ou si l'expression n'est égale à aucune
      ligne de la sous-requête et qu'il y a au moins 1 valeur `null` dans la
      sous-requête
- Parce que SQL utilise une logique à 3 valeurs, évaluer `IN` est plus compliqué

- Rappelez-vous que si nous voulons savoir si une valeur de colonne est `null`,
  nous ne pouvons pas utiliser l'opérateur d'égalité `=` car il renverra
  toujours `null`
    - `null` signifie *inconnu* dans ce cas, donc nous ne savons pas comment
      comparer des valeurs à une valeur inconnue
    - Nous devons donc utiliser `is null` au lieu de `= null`...
- L'opérateur `IN` compare les valeurs avec `=`, donc dès qu'il compare avec
  un `null`, il évaluera à `null`
    - Donc si l'expression est égale à `null`, `IN` évaluera à `null`
    - Si l'expression n'est pas `null`, alors elle comparera l'expression avec
      des valeurs non-nulles d'abord dans la sous-requête
        - si elle trouve une correspondance, alors la valeur de `IN` sera vraie
        - si nous ne trouvons pas de correspondance, alors elle vérifiera si la
          sous-requête contient des valeurs `null`
            - si non, alors nous savons avec certitude que l'expression n'est
              pas dans la sous-requête, donc la valeur de `IN` sera `false`
            - s'il y a des valeurs `null`, alors nous ne savons pas avec
              certitude si l'expression est dans la sous-requête parce que nous
              avons des valeurs *inconnues* (`null`), donc la valeur de `IN`
              est `null`

- Cet exemple fonctionne comme prévu

```sql
-- notez que (1, 2, 3) n'est pas vraiment une sous-requête, mais agit comme une sous-requête
-- il est utilisé pour simplifier l'exemple
select *
from course
where cid in (1, 2, 3);
```

```sql
-- notez que (2, 3, 4, null) n'est pas vraiment une sous-requête, mais agit comme une sous-requête
-- il est utilisé pour simplifier l'exemple
select *
from course
where cid in (2, 3, 4, null);
```

- Cet exemple est équivalent et montre comment les opérateurs `IN` sont évalués
  en interne

```sql
select * from course where cid = 2 or cid = 3 or cid = 4 or cid = null;
```

- Cela fonctionne pour les cours avec une valeur `cid` de 2, 3 ou 4 parce qu'au
  moins 1 des comparaisons sera vraie et nous obtiendrons quelque chose
  comme `T OR F OR F OR N`, ce qui est vrai
- Mais pour les cours avec un `cid` non inclus dans l'ensemble fourni, nous
  obtiendrons `null` parce que `F OR F OR F OR N` est `N`
- Cela ne crée pas de problème car les lignes avec une condition `where` seront
  supprimées
- Mais si nous nions `IN` pour obtenir un opérateur `NOT IN`, nous aurons des
  ennuis

```sql
select *
from course
where cid not in (2, 3, 4, null);

select *
from course
where cid not in (select cid from offering);

select *
from course
except
select c.*
from course c
         inner join offering o on c.cid = o.cid;

select c.*
from course c
         left join offering o on c.cid = o.cid
where o.oid is null;

select *
from instructor
where iid not in (select iid from offering);
```

- Nous n'obtenons rien
- Mais le cours avec `cid = 1` n'est pas dans la sous-requête, alors pourquoi ne
  l'obtenons-nous pas ?
- C'est à cause de la valeur `null`
    - `1 in (2, 3, 4, null)` évalue à `null`
    - et `1 not in (2, 3, 4, null)` évalue à `not null`, ce qui est `null`
- Donc les requêtes `NOT IN` sont dangereuses à cause des valeurs `null`
- La requête suivante est correcte parce que nous savons avec certitude
  que `cid` dans course ne peut pas être `null`
    - Donc nous pouvons trouver les cours qui n'ont jamais été offerts de cette
      manière

```sql
--insert into course(name, code, credits) --values ('Data Structures', 'DS', 3);
-- delete from course where code = 'DS';
select *
from course
where cid not in (select cid from offering);
```

- Mais essayer de faire quelque chose de similaire pour les instructeurs créera
  des problèmes car `iid` dans offering peut être `null`
- Nous devons explicitement exclure les valeurs `null` dans la sous-requête pour
  que la requête renvoie les résultats corrects

```sql
--insert into instructor(name, email, department) --values ('John', 'john@bbb.com', 'ECE');
-- delete from instructor where name = 'John';
select *
from instructor
where iid not in (select iid from offering);
```

```sql
select *
from instructor
where iid not in (select iid
                  from offering
                  where iid is not null);
```

#### Recommandation : ne pas utiliser `NOT IN`

#### Recommandation : utiliser une `left join` à la place

- Non seulement la jointure gauche (ou les jointures externes en général) vous
  oblige à penser aux valeurs `null` (et à les gérer correctement), mais en
  termes de performance, les jointures gauches seront généralement plus
  efficaces.
- Utiliser des jointures gauches évite de traiter avec la logique à 3 valeurs de
  SQL.

```sql
select i.*
from instructor i
         left join offering o on i.iid = o.iid
where o.iid is null;
```

#### `ANY` et `ALL`

- `ANY` et `ALL` sont utilisés comme modificateurs d'opérateurs (généralement
  des opérateurs de comparaison)
    - `expression operator ANY (sub-query)`
        - `true` lorsqu'il existe une ligne $r$ dans la sous-requête telle
          que `expression operator r` est vrai
        - `false` lorsque pour toutes les lignes r dans la
          sous-requête, `expression operator r` est faux et qu'il n'y a pas de
          valeurs `null` dans la sous-requête...
        - `null` lorsque pour toutes les lignes r dans la
          sous-requête, `expression operator r` est faux et qu'il y a au moins 1
          valeur `null` dans la sous-requête
    - `IN` est équivalent à `=ANY`
    - `expression operator ALL (sub-query)`
        - `true` lorsque pour toutes les lignes $r$ dans la
          sous-requête, `expression operator r` est vrai
        - `false` lorsque `expression operator r` est faux pour au moins 1 ligne
          dans la sous-requête
        - `null` lorsque pour toutes les lignes $r$ dans la
          sous-requête, `expression operator r` n'est pas faux et qu'il y a au
          moins 1 valeur `null` dans la sous-requête
    - `NOT IN` est équivalent à `<> ALL`
