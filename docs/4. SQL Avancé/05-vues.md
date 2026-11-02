# Vues

Les vues sont essentiellement des **tables virtuelles** créées au-dessus d'une ou plusieurs tables réelles. Elles ne
stockent pas les données elles-mêmes (sauf dans le cas de vues matérialisées), mais exécutent dynamiquement la requête
SQL sous-jacente lorsqu'on les interroge.

## 1. Sécurité et Contrôle d'Accès

Combinées avec le système de privilèges (`GRANT` / `REVOKE`), les vues permettent d'assurer différents niveaux de
confidentialité et de sécurité (sécurité au niveau des colonnes et des lignes).

**Exemple :**

- Le département des ressources humaines (RH) a besoin d'accéder aux numéros d'assurance sociale (NAS/SIN) des employés
  pour la paie et les impôts.
- Les services de support informatique ou de marketing ne doivent pas avoir accès à ces informations sensibles.
- On peut donc accorder l'accès direct à la table `employee_priv` uniquement aux RH, et exposer une **vue** sans la
  colonne confidentielle aux autres services.

```sql
drop schema if exists company cascade;
create schema company;
set search_path to company;

-- Table de base contenant des données sensibles
create table employee_priv
(
    eid   integer generated always as identity primary key,
    sin   CHAR(9) not null unique,
    name  text    not null,
    phone text
);

insert into employee_priv(sin, name, phone)
values ('123456789', 'Denis', '123-456-7890'),
       ('987654321', 'Jane', '987-654-3210');

select *
from employee_priv;
```

Création de la vue publique/restreinte :

```sql
create or replace view employee AS
select eid, name, phone
from employee_priv;

select *
from employee;
```

Attribution des permissions :

```sql
create role hr;
create role itsupport;

-- Les RH ont accès complet à la table avec les NAS
grant select, insert, update, delete on employee_priv to hr;

-- Le support IT n'a accès qu'à la vue (sans NAS)
grant select on employee to itsupport;
```

## 2. Simplification des Requêtes

Les vues permettent d'encapsuler des requêtes complexes (notamment avec de multiples jointures ou agrégations) pour
faciliter l'écriture des requêtes ultérieures.

```sql
set search_path to university;

create or replace view studentcourse as
select s.sid,
       s.name as student_name,
       c.code,
       c.name as course_name,
       o.semester,
       o.year
from student s
         left join enrollment e on s.sid = e.sid
         inner join offering o on e.oid = o.oid
         inner join course c on o.cid = c.cid;

select *
from studentcourse;
```

On peut ensuite interroger et filtrer la vue simplement comme une table classique :

```sql
select *
from studentcourse
where sid = 3
  and semester = 'W'
  and year = 2020;
```

## 3. Indépendance Logique des Données

Les vues permettent de minimiser l'impact des changements de schéma dans la base de données :

- Supposons qu'un schéma de table doive être modifié (par exemple décomposition d'une table pour normaliser et éliminer
  des redondances).
- Normalement, toutes les applications et requêtes utilisant l'ancienne table devraient être réécrites.
- En créant une vue portant le nom et les colonnes de l'ancienne table, définie sur les nouvelles tables décomposées,
  les applications existantes continuent de fonctionner sans modification.

## 4. Les Vues sont-elles Modifiables ?

La réponse est : **cela dépend de la complexité de la vue**.

1. **Vues simples (sur une seule table)** :
    - Si nous mettons à jour le numéro de téléphone d'un employé via la vue `employee`, PostgreSQL peut facilement
      rediriger l'`UPDATE` vers la table sous-jacente `employee_priv`.
    - En revanche, insérer (`INSERT`) une ligne via `employee` échouera si la table sous-jacente a des colonnes
      obligatoires (`NOT NULL` sans valeur par défaut) qui ne font pas partie de la vue (comme le `sin`).

2. **Vues complexes (avec jointures, agrégations, DISTINCT)** :
    - Mettre à jour une vue comme `studentcourse` issue de la jointure de 4 tables pose problème car le SGBD ne peut pas
      déterminer de manière univoque quelle table ou ligne modifier.
    - Pour rendre des vues complexes modifiables dans PostgreSQL, il est possible d'utiliser des déclencheurs de type
      `INSTEAD OF` (abordés dans le chapitre suivant).

