# 4b - Groupement et Agrégats (Université)

```sql
set search_path to university;
```

1- Trouver le nombre d'offres de cours pour chaque année

```sql
select year, count(oid)
from offering
group by year;
```

2- Même résultat car `oid` ne peut pas être nul

```sql
select year, count(*)
from offering
group by year;
```

3- Similaire, mais résultats différents à cause des valeurs nulles

```sql
select year, count(iid)
from offering
group by year;
```

4- Trouver le nombre d'offres de cours pour chaque instructeur

```sql
select iid, count(*)
from offering
group by iid;
```

5- Obtenir le nombre d'étudiants inscrits dans chaque offre de cours

```sql
select oid, count(sid)
from enrollment
group by oid;
```

6- Obtenir le nombre d'étudiants inscrits dans chaque offre de cours

- avec les codes de cours, les noms et les sections

```sql
select e.oid, code, name, section, count(sid) as n_students
from enrollment e
         inner join offering o on e.oid = o.oid
         inner join course c on o.cid = c.cid
group by e.oid, code, name, section;
```

7- Qu'en est-il des offres sans inscriptions ?

```sql
select o.oid, code, name, section, count(sid) as n_students
from enrollment e
         right join offering o on e.oid = o.oid
         inner join course c on o.cid = c.cid
group by o.oid, code, name, section
order by n_students desc;
```

8- Obtenir le nombre d'étudiants inscrits dans chaque offre de cours

- avec les codes de cours, les noms et les sections, mais seulement pour les
  offres de cours avec moins de 3 inscriptions

```sql
select o.oid, code, name, section, count(sid) as n_students
from enrollment e
         right join offering o on e.oid = o.oid
         inner join course c on o.cid = c.cid
group by o.oid, code, name, section
having count(sid) < 3;
```

9- Même chose, mais avec moins d'une inscription

```sql
select o.oid, code, name, section, count(sid) as n_students
from enrollment e
         right join offering o on e.oid = o.oid
         inner join course c on o.cid = c.cid
group by o.oid, code, name, section
having count(sid) < 1;
```

```sql
-- meilleure réponse
select o.oid, code, name, section
from enrollment e
         right join offering o on e.oid = o.oid
         inner join course c on o.cid = c.cid
where e.oid is null;
```

## Exercices

1. Obtenir le nombre d'offres de cours pour chaque instructeur
2. Obtenir le nombre d'offres de cours pour l'instructeur avec l'ID 1, pour
   chaque année
3. Obtenir le nombre d'offres de cours pour chaque instructeur pour chaque année
    - que faire des instructeurs sans aucune offre de cours ?
4. Obtenir le nombre d'offres de cours pour chaque instructeur pour chaque
   semestre et chaque année
    - que faire des semestres sans aucune offre de cours ?
    - que faire des semestres sans offres de cours pour certains instructeurs ?
        - par exemple, de nombreux instructeurs enseignent uniquement aux
          semestres d'automne et d'hiver, mais pas aux semestres d'été ;
          devons-nous lister ces semestres sans cours pour chaque instructeur
          avec un compte de 0 ?