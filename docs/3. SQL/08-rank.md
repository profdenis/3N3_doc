# 8 - Fonction SQL RANK

## Qu'est-ce que la fonction RANK ?

La fonction RANK est une **fonction fenêtrée** (aussi appelée *fonction analytique*) qui attribue un rang à chaque ligne
au sein d’un ensemble de résultats. Elle est particulièrement utile pour :

- Créer des classements et des palmarès
- Identifier les N meilleurs performeurs
- Analyser les distributions de données
- Générer des rapports comportant des données classées

**Caractéristiques principales :**

- Attribue des rangs entiers consécutifs à partir de 1
- Les lignes ayant des valeurs identiques reçoivent le même rang
- En cas d’égalité, le rang suivant « saute » des numéros (crée des trous)
- Renvoie le rang de chaque ligne par rapport aux autres lignes de la partition

## Syntaxe de base et concepts

### Syntaxe SQL standard

```sql
RANK() OVER
( [ PARTITION BY colonne1, colonne2, ...]
    ORDER BY colonne1 [ASC | DESC], colonne2 [ASC | DESC], ...)
```

### Composants clés

- **Clause OVER** : Obligatoire pour toutes les fonctions fenêtrées
    - Définit la « fenêtre » de lignes sur laquelle s’effectue le calcul
    - Contient les clauses PARTITION BY et/ou ORDER BY

- **PARTITION BY** : Optionnel
    - Divise l’ensemble de résultats en groupes (partitions)
    - La fonction RANK redémarre à 1 pour chaque nouvelle partition
    - Similaire à GROUP BY, mais sans agrégation des lignes

- **ORDER BY** : Obligatoire pour RANK
    - Détermine les critères de classement
    - Peut utiliser plusieurs colonnes pour un tri complexe
    - ASC (croissant) est la valeur par défaut, DESC pour décroissant

## RANK vs DENSE_RANK vs ROW_NUMBER

Comprendre les différences entre les fonctions de classement est essentiel :

| Fonction     | Gestion des égalités    | Trou après égalité | Cas d'utilisation                |
|--------------|-------------------------|--------------------|----------------------------------|
| ROW_NUMBER() | Numéros uniques         | Pas de trou        | Numérotation séquentielle unique |
| RANK()       | Même rang pour égalités | Crée des trous     | Classement type compétition      |
| DENSE_RANK() | Même rang pour égalités | Pas de trou        | Classement dense (sans trous)    |

### Exemple comparatif

```sql
-- Données exemple : Notes d'étudiants
-- Notes : 95, 87, 87, 82, 78

SELECT score,
       ROW_NUMBER() OVER (ORDER BY score DESC) as row_num,
       RANK() OVER (ORDER BY score DESC)       as rank_with_gaps,
       DENSE_RANK() OVER (ORDER BY score DESC) as dense_rank
FROM student_scores;
```

**Résultats :**

| score | row_num | rank_with_gaps | dense_rank |
|-------|---------|----------------|------------|
| 95    | 1       | 1              | 1          |
| 87    | 2       | 2              | 2          |
| 87    | 3       | 2              | 2          |
| 82    | 4       | 4              | 3          |
| 78    | 5       | 5              | 4          |

## Implémentation dans PostgreSQL

PostgreSQL offre un excellent support des fonctions fenêtrées, y compris RANK. Voici quelques détails spécifiques à
PostgreSQL :

### Exemple basique sous PostgreSQL

```sql
-- Créer une table exemple
CREATE TABLE sales_data
(
    salesperson_id   INTEGER,
    salesperson_name VARCHAR(50),
    region           VARCHAR(20),
    sales_amount     DECIMAL(10, 2)
);

-- Insérer des données exemple
INSERT INTO sales_data
VALUES (1, 'Alice Johnson', 'North', 125000.00),
       (2, 'Bob Smith', 'South', 98000.00),
       (3, 'Carol Davis', 'North', 134000.00),
       (4, 'David Wilson', 'South', 87000.00),
       (5, 'Eve Brown', 'East', 112000.00),
       (6, 'Frank Miller', 'East', 95000.00);

-- Classer les commerciaux par montant des ventes
SELECT salesperson_name,
       sales_amount,
       RANK() OVER (ORDER BY sales_amount DESC) as overall_rank
FROM sales_data
ORDER BY overall_rank;
```

### Fonctionnalités spécifiques à PostgreSQL

- Conformité complète au standard SQL
- Support des cadres de fenêtres complexes
- Intégration avec d'autres fonctions analytiques PostgreSQL
- Optimisation de performance excellente pour les fonctions fenêtrées

## Exemples pratiques

### Exemple 1 : Classement simple

```sql
-- Trouver le rang de chaque produit par prix
SELECT product_name,
       price,
       RANK() OVER (ORDER BY price DESC) as price_rank
FROM products
ORDER BY price_rank;
```

### Exemple 2 : Classement au sein de catégories

```sql
-- Classer les employés par salaire dans chaque département
SELECT department_id,
       employee_name,
       salary,
       RANK() OVER (
           PARTITION BY department_id
           ORDER BY salary DESC
           ) as dept_salary_rank
FROM employees
ORDER BY department_id, dept_salary_rank;
```

### Exemple 3 : Trouver les N premiers de chaque groupe

```sql
-- Trouver les 3 employés les mieux payés dans chaque département
SELECT *
FROM (SELECT department_id,
             employee_name,
             salary,
             RANK() OVER (
                 PARTITION BY department_id
                 ORDER BY salary DESC
                 ) as salary_rank
      FROM employees) ranked_employees
WHERE salary_rank <= 3
ORDER BY department_id, salary_rank;
```

### Exemple 4 : Critères de classement multiples

```sql
-- Classer les étudiants par GPA, puis par nombre d'heures de crédit
SELECT student_name,
       gpa,
       credit_hours,
       RANK() OVER (
           ORDER BY gpa DESC, credit_hours DESC
           ) as academic_rank
FROM students
ORDER BY academic_rank;
```

## Support par les systèmes de gestion de bases de données

### ✅ Bases de données prenant en charge la fonction RANK

| Base de données     | Version        | Notes                                                           |
|---------------------|----------------|-----------------------------------------------------------------|
| **PostgreSQL**      | 8.4+ (2009)    | Support complet des fonctions fenêtrées, excellente performance |
| **MySQL**           | 8.0+ (2018)    | Ajout des fonctions fenêtrées, y compris RANK                   |
| **MariaDB**         | 10.2+ (2017)   | Implémentation compatible MySQL                                 |
| **SQL Server**      | 2005+          | Implémentation Microsoft, bien documentée                       |
| **Oracle Database** | 8i+ (1999)     | L’un des premiers à supporter les fonctions fenêtrées           |
| **SQLite**          | 3.25.0+ (2018) | Ajout du support complet des fonctions fenêtrées                |

## Exemples sur la BD Pagila


### Niveau 1 : RANK - Débutant

#### Exercice 1.1 : Classement simple des films par durée

```sql
-- Classer tous les films par leur durée (du plus long au plus court)
SELECT 
    title,
    length,
    RANK() OVER (ORDER BY length DESC) as rang
FROM film
ORDER BY rang, title;
```


#### Exercice 1.2 : Classement des clients par nombre de locations

```sql
-- Classer les clients selon leur nombre de locations
SELECT 
    c.customer_id,
    c.first_name,
    c.last_name,
    COUNT(r.rental_id) as nb_locations,
    RANK() OVER (ORDER BY COUNT(r.rental_id) DESC) as rank_client
FROM customer c
LEFT JOIN rental r ON c.customer_id = r.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name
ORDER BY rank_client
LIMIT 15;
```


#### Exercice 1.3 : Films les moins chers à louer

```sql
-- Classer les films par tarif de location (du moins cher au plus cher)
SELECT 
    title,
    rental_rate,
    RANK() OVER (ORDER BY rental_rate ASC) as rank_prix
FROM film
ORDER BY rank_prix, title
LIMIT 20;
```


### Niveau 2 : RANK avec PARTITION BY - Intermédiaire

#### Exercice 2.1 : Meilleurs films par catégorie (selon la durée)

```sql
-- Classer les films par durée dans chaque catégorie
SELECT c.name                                                     as categorie,
       f.title,
       f.length,
       RANK() OVER ( PARTITION BY c.name ORDER BY f.length DESC ) as rank_dans_categorie
FROM film f
       JOIN film_category fc ON f.film_id = fc.film_id
       JOIN category c ON fc.category_id = c.category_id
WHERE f.length IS NOT NULL
ORDER BY categorie, rank_dans_categorie
LIMIT 30;
```


#### Exercice 2.2 : Top 3 des acteurs par nombre de films, par catégorie

```sql
-- Trouver les 3 acteurs les plus prolifiques dans chaque catégorie
SELECT *
FROM (
    SELECT 
        cat.name as categorie,
        a.first_name || ' ' || a.last_name as acteur,
        COUNT(DISTINCT f.film_id) as nb_films,
        RANK() OVER (
            PARTITION BY cat.name 
            ORDER BY COUNT(DISTINCT f.film_id) DESC
        ) as rank_categorie
    FROM actor a
    JOIN film_actor fa ON a.actor_id = fa.actor_id
    JOIN film f ON fa.film_id = f.film_id
    JOIN film_category fc ON f.film_id = fc.film_id
    JOIN category cat ON fc.category_id = cat.category_id
    GROUP BY cat.name, a.actor_id, a.first_name, a.last_name
) ranked
WHERE rank_categorie <= 3
ORDER BY categorie, rank_categorie;
```


#### Exercice 2.3 : Classement des ventes par magasin et par mois

```sql
-- Classer les montants de paiement par mois pour chaque magasin
SELECT 
    s.store_id,
    TO_CHAR(p.payment_date, 'YYYY-MM') as mois,
    SUM(p.amount) as revenus_mensuels,
    RANK() OVER (
        PARTITION BY s.store_id 
        ORDER BY SUM(p.amount) DESC
    ) as rank_mois
FROM payment p
JOIN staff st ON p.staff_id = st.staff_id
JOIN store s ON st.store_id = s.store_id
GROUP BY s.store_id, TO_CHAR(p.payment_date, 'YYYY-MM')
ORDER BY s.store_id, rank_mois;
```


### Niveau 3 : Comparaison RANK, DENSE_RANK et ROW_NUMBER - Avancé

#### Exercice 3.1 : Comprendre les différences sur les revenus par film

```sql
-- Comparer les trois fonctions de classement sur les revenus par film
SELECT 
    f.title,
    SUM(p.amount) as revenus_totaux,
    ROW_NUMBER() OVER (ORDER BY SUM(p.amount) DESC) as row_num,
    RANK() OVER (ORDER BY SUM(p.amount) DESC) as rank_avec_trous,
    DENSE_RANK() OVER (ORDER BY SUM(p.amount) DESC) as rank_dense
FROM film f
JOIN inventory i ON f.film_id = i.film_id
JOIN rental r ON i.inventory_id = r.inventory_id
JOIN payment p ON r.rental_id = p.rental_id
GROUP BY f.film_id, f.title
ORDER BY revenus_totaux DESC
LIMIT 20;
```


#### Exercice 3.2 : Classement des clients VIP avec égalités

```sql
-- Identifier les clients VIP en utilisant les trois méthodes
WITH customer_spending AS (
    SELECT 
        c.customer_id,
        c.first_name || ' ' || c.last_name as nom_complet,
        SUM(p.amount) as total_depense
    FROM customer c
    JOIN payment p ON c.customer_id = p.customer_id
    GROUP BY c.customer_id, c.first_name, c.last_name
)
SELECT 
    nom_complet,
    ROUND(total_depense, 2) as depense,
    ROW_NUMBER() OVER (ORDER BY total_depense DESC) as numero_ligne,
    RANK() OVER (ORDER BY total_depense DESC) as rank_standard,
    DENSE_RANK() OVER (ORDER BY total_depense DESC) as rank_dense
FROM customer_spending
ORDER BY total_depense DESC
LIMIT 25;
```


#### Exercice 3.3 : Films les plus loués par catégorie avec DENSE_RANK

```sql
-- Top 5 des films les plus loués dans chaque catégorie (sans trous dans le classement)
SELECT *
FROM (
    SELECT 
        cat.name as categorie,
        f.title,
        COUNT(r.rental_id) as nb_locations,
        DENSE_RANK() OVER (
            PARTITION BY cat.name 
            ORDER BY COUNT(r.rental_id) DESC
        ) as rank_dense
    FROM film f
    JOIN film_category fc ON f.film_id = fc.film_id
    JOIN category cat ON fc.category_id = cat.category_id
    JOIN inventory i ON f.film_id = i.film_id
    JOIN rental r ON i.inventory_id = r.inventory_id
    GROUP BY cat.name, f.film_id, f.title
) ranked
WHERE rank_dense <= 5
ORDER BY categorie, rank_dense, nb_locations DESC;
```


### Niveau 4 : Requêtes complexes - Expert

#### Exercice 4.1 : Analyse comparative des performances des magasins

```sql
-- Analyse détaillée avec classement multi-critères
WITH store_metrics AS (
    SELECT 
        s.store_id,
        TO_CHAR(p.payment_date, 'YYYY-MM') as mois,
        COUNT(DISTINCT p.customer_id) as nb_clients,
        COUNT(p.payment_id) as nb_transactions,
        SUM(p.amount) as revenus
    FROM store s
    JOIN staff st ON s.store_id = st.store_id
    JOIN payment p ON st.staff_id = p.staff_id
    GROUP BY s.store_id, TO_CHAR(p.payment_date, 'YYYY-MM')
)
SELECT 
    store_id,
    mois,
    nb_clients,
    nb_transactions,
    ROUND(revenus, 2) as revenus,
    RANK() OVER (PARTITION BY store_id ORDER BY revenus DESC) as rank_revenus_magasin,
    DENSE_RANK() OVER (ORDER BY revenus DESC) as rank_global,
    ROW_NUMBER() OVER (PARTITION BY store_id ORDER BY nb_clients DESC) as position_clients
FROM store_metrics
ORDER BY store_id, rank_revenus_magasin;
```


#### Exercice 4.2 : Acteurs polyvalents - Classement par diversité de catégories

```sql
-- Identifier les acteurs les plus polyvalents (jouant dans plusieurs catégories)
WITH actor_diversity AS (
    SELECT 
        a.actor_id,
        a.first_name || ' ' || a.last_name as acteur,
        COUNT(DISTINCT c.category_id) as nb_categories,
        COUNT(DISTINCT f.film_id) as nb_films,
        STRING_AGG(DISTINCT c.name, ', ' ORDER BY c.name) as categories
    FROM actor a
    JOIN film_actor fa ON a.actor_id = fa.actor_id
    JOIN film f ON fa.film_id = f.film_id
    JOIN film_category fc ON f.film_id = fc.film_id
    JOIN category c ON fc.category_id = c.category_id
    GROUP BY a.actor_id, a.first_name, a.last_name
)
SELECT 
    acteur,
    nb_categories,
    nb_films,
    categories,
    RANK() OVER (ORDER BY nb_categories DESC, nb_films DESC) as rank_polyvalence,
    DENSE_RANK() OVER (ORDER BY nb_categories DESC) as rank_categories
FROM actor_diversity
ORDER BY rank_polyvalence
LIMIT 20;
```


#### Exercice 4.3 : Analyse temporelle des locations avec fenêtres mobiles

```sql
-- Classement des films par popularité avec tendance temporelle
WITH monthly_rentals AS (
    SELECT 
        f.film_id,
        f.title,
        DATE_TRUNC('month', r.rental_date) as mois,
        COUNT(r.rental_id) as locations_mois
    FROM film f
    JOIN inventory i ON f.film_id = i.film_id
    JOIN rental r ON i.inventory_id = r.inventory_id
    GROUP BY f.film_id, f.title, DATE_TRUNC('month', r.rental_date)
)
SELECT 
    title,
    mois,
    locations_mois,
    ROW_NUMBER() OVER (PARTITION BY mois ORDER BY locations_mois DESC) as position_mois,
    RANK() OVER (PARTITION BY mois ORDER BY locations_mois DESC) as rank_mois,
    DENSE_RANK() OVER (ORDER BY locations_mois DESC) as rank_global
FROM monthly_rentals
WHERE mois IS NOT NULL
ORDER BY mois DESC, rank_mois
LIMIT 30;
```


## Points clés à retenir

1. **RANK()** : Crée des "trous" dans le classement en cas d'égalité (1, 2, 2, 4, 5...)
2. **DENSE_RANK()** : Pas de trous dans le classement (1, 2, 2, 3, 4...)
3. **ROW_NUMBER()** : Toujours unique, même en cas d'égalité (1, 2, 3, 4, 5...)
4. **PARTITION BY** : Permet de créer des sous-classements par groupe
5. **ORDER BY** : Détermine le critère de classement (obligatoire)


-----------

??? info "Utilisation de l'IA"
    Page rédigée en partie avec l'aide d'un assistant IA, principalement à l'aide de Perplexity AI. L'IA a été
    utilisée pour générer des explications, des exemples et/ou des suggestions de structure. Toutes les informations
    ont été vérifiées, éditées et complétées par l'auteur.