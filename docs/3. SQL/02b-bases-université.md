# 2b - Base de Données Université

Exécutez le
fichier [university_create.sql](../src/create/university_create.sql) pour
créer la base de données
Universitaire.

```sql
set search_path to university;
```

1- **Projection** : choisir les colonnes à inclure dans les résultats

```sql
select name, email
from student;
```

2- Supposons que l'université facture 200$ par crédit pour un cours. Trouvez le
prix pour chaque cours.

- `price` est un alias pour la colonne calculée

```sql
select code, name, credits * 200 as price
from course;
```

3- **Sélection** : filtrer les lignes à inclure dans les résultats

- Conserver uniquement les offres de cours pour le semestre d'hiver 2020

```sql
select *
from offering
where semester = 'W'
  and year = 2020;
```

4- **Tri** : ordonner les instructeurs par leurs noms

```sql
select iid, name, email
from instructor
order by name;

select iid, name, email
from instructor
order by 2;
```