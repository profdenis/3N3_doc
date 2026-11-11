# 4. BD Northwind (partie 2)

Ces exercices complètent la [partie 1](03-northwind.md) et couvrent les sujets SQL suivants : filtrage avec `OR` et
`NOT`, produits cartésiens et jointures externes droite et complète, opérateurs ensemblistes, `CASE WHEN`, `COALESCE`,
fonctions de fenêtrage (`ROW_NUMBER`, `RANK`, `DENSE_RANK`, `NTILE`), `LIKE` et `ILIKE`, ainsi que les types `DATE` et
`TIMESTAMP`.

Le schéma de la base est décrit dans le [script de création](../../src/create/northwind_create.sql).

### 1. Trouvez les produits non discontinués dont le prix est inférieur à 10 ou supérieur à 100.

Difficulté : 1
<details>
    <summary>Code</summary>

```sql
SELECT product_name, unit_price
FROM products
WHERE (unit_price < 10 OR unit_price > 100)
  AND discontinued <> 1
ORDER BY unit_price;
```

</details>
<br>

### 2. Listez les commandes qui n'ont pas encore été expédiées.

Difficulté : 1
<details>
    <summary>Code</summary>

```sql
SELECT order_id, order_date
FROM orders
WHERE shipped_date IS NULL
ORDER BY order_date;
```

</details>
<br>

### 3. Listez les clients dont la région est connue.

Difficulté : 1
<details>
    <summary>Code</summary>

```sql
SELECT company_name, region
FROM customers
WHERE region IS NOT NULL
ORDER BY company_name;
```

</details>
<br>

### 4. Listez les commandes passées en juillet 1996.

Difficulté : 1
<details>
    <summary>Code</summary>

```sql
SELECT order_id, order_date
FROM orders
WHERE order_date BETWEEN '1996-07-01'::date AND '1996-07-31'::date
ORDER BY order_date;
```

</details>
<br>

### 5. Produisez toutes les combinaisons possibles entre les catégories de produits et les transporteurs.

Difficulté : 2
<details>
    <summary>Code</summary>

```sql
SELECT c.category_name, s.company_name AS transporteur
FROM categories c,
     shippers s
ORDER BY c.category_name, s.company_name;
```

</details>
<br>

### 6. Listez tous les territoires avec l'identifiant de l'employé qui y est affecté, même si aucun employé n'y est affecté.

Difficulté : 2
<details>
    <summary>Code</summary>

```sql
SELECT t.territory_description, et.employee_id
FROM employee_territories et
         RIGHT OUTER JOIN territories t ON et.territory_id = t.territory_id
ORDER BY t.territory_description;
```

</details>
<br>

### 7. Comparez le nombre de clients et de fournisseurs par pays, en gardant les pays qui n'ont que l'un des deux.

Difficulté : 3
<details>
    <summary>Code</summary>

```sql
SELECT COALESCE(c.country, s.country) AS pays,
       c.nb_clients,
       s.nb_fournisseurs
FROM (SELECT country, COUNT(*) AS nb_clients
      FROM customers
      GROUP BY country) c
         FULL OUTER JOIN
     (SELECT country, COUNT(*) AS nb_fournisseurs
      FROM suppliers
      GROUP BY country) s ON c.country = s.country
ORDER BY pays;
```

</details>
<br>

### 8. Listez les villes des clients et des employés, sans doublons.

Difficulté : 2
<details>
    <summary>Code</summary>

```sql
SELECT city
FROM customers
UNION
SELECT city
FROM employees
ORDER BY city;
```

</details>
<br>

### 9. Listez les villes qui sont à la fois celles d'un client et celles d'un fournisseur.

Difficulté : 2
<details>
    <summary>Code</summary>

```sql
SELECT city
FROM customers
INTERSECT
SELECT city
FROM suppliers
ORDER BY city;
```

</details>
<br>

### 10. Listez les pays où il y a des clients mais aucun fournisseur.

Difficulté : 2
<details>
    <summary>Code</summary>

```sql
SELECT country
FROM customers
EXCEPT
SELECT country
FROM suppliers
ORDER BY country;
```

</details>
<br>

### 11. Comptez le nombre total de commandes, le nombre de commandes expédiées et le nombre de commandes non expédiées.

Difficulté : 2
<details>
    <summary>Code</summary>

```sql
SELECT COUNT(*)                       AS total_commandes,
       COUNT(shipped_date)            AS expediees,
       COUNT(*) - COUNT(shipped_date) AS non_expediees
FROM orders;
```

</details>
<br>

### 12. Trouvez les 5 produits dont le prix dépasse le plus le prix moyen de tous les produits.

Difficulté : 2
<details>
    <summary>Code</summary>

```sql
SELECT product_name,
       unit_price,
       ROUND((unit_price - (SELECT AVG(unit_price) FROM products))::numeric, 2) AS ecart_moyenne
FROM products
ORDER BY ecart_moyenne DESC
LIMIT 5;
```

</details>
<br>

Difficulté : 4
<details>
    <summary>Code</summary>

```sql
WITH produits_ecart AS (SELECT product_name,
                               unit_price,
                               ROUND((unit_price - (SELECT AVG(unit_price) FROM products))::numeric, 2) AS ecart_moyenne
                        FROM products),
     classement AS (SELECT product_name,
                           unit_price,
                           ecart_moyenne,
                           RANK() OVER (ORDER BY unit_price DESC) AS rang
                    FROM produits_ecart)
SELECT product_name,
       unit_price,
       ecart_moyenne
FROM classement
WHERE rang <= 5
ORDER BY ecart_moyenne DESC, product_name;
```

</details>
<br>

### 13. Listez les clients qui ont passé au moins une commande en 1998.

Difficulté : 3
<details>
    <summary>Code</summary>

```sql
SELECT company_name, country
FROM customers
WHERE customer_id IN (SELECT customer_id
                      FROM orders
                      WHERE EXTRACT(YEAR FROM order_date) = 1998)
ORDER BY company_name;
```

</details>
<br>

### 14. Affichez le statut de chaque produit : « Arrete » si le produit est discontinué, « Disponible » sinon.

Difficulté : 2
<details>
    <summary>Code</summary>

```sql
SELECT product_name,
       CASE discontinued
           WHEN 1 THEN 'Arrete'
           ELSE 'Disponible'
           END AS statut
FROM products
ORDER BY product_name;
```

</details>
<br>

### 15. Répartissez les produits en trois gammes de prix (« Bon marche » si le prix est inférieur à 10, « Moyen » s'il est inférieur ou égal à 50, « Cher » sinon) et comptez les produits de chaque gamme.

Difficulté : 2
<details>
    <summary>Code</summary>

```sql
SELECT CASE
           WHEN unit_price < 10 THEN 'Bon marche'
           WHEN unit_price <= 50 THEN 'Moyen'
           ELSE 'Cher'
           END  AS gamme,
       COUNT(*) AS nb_produits
FROM products
GROUP BY gamme
ORDER BY nb_produits DESC;
```

</details>
<br>

### 16. Affichez la région de chaque client, en remplaçant les régions absentes par « Inconnue ».

Difficulté : 1
<details>
    <summary>Code</summary>

```sql
SELECT company_name,
       COALESCE(region, 'Inconnue') AS region
FROM customers
ORDER BY company_name;
```

</details>
<br>

### 17. Pour chaque employé, comptez ses commandes par année (1996, 1997, 1998) dans des colonnes distinctes.

Difficulté : 4
<details>
    <summary>Code</summary>

```sql
SELECT e.last_name,
       SUM(CASE WHEN EXTRACT(YEAR FROM o.order_date) = 1996 THEN 1 ELSE 0 END) AS commandes_1996,
       SUM(CASE WHEN EXTRACT(YEAR FROM o.order_date) = 1997 THEN 1 ELSE 0 END) AS commandes_1997,
       SUM(CASE WHEN EXTRACT(YEAR FROM o.order_date) = 1998 THEN 1 ELSE 0 END) AS commandes_1998
FROM employees e
         LEFT JOIN orders o ON e.employee_id = o.employee_id
GROUP BY e.employee_id, e.last_name
ORDER BY e.last_name;
```

</details>
<br>

### 18. Classez les produits de chaque catégorie du plus cher au moins cher en plaçant les produits ayant le même prix au même rang, sans saut dans les numéros de rangs, et conservez le top 5 de chaque catégorie. N'utilisez pas `LIMIT`.

Difficulté : 4
<details>
    <summary>Code</summary>

```sql
WITH classement AS (SELECT c.category_name,
                           p.product_name,
                           p.unit_price,
                           DENSE_RANK() OVER (PARTITION BY c.category_id ORDER BY p.unit_price DESC) AS rang
                    FROM products p
                             JOIN categories c ON p.category_id = c.category_id)
SELECT category_name,
       product_name,
       unit_price,
       rang
FROM classement
WHERE rang <= 5
ORDER BY category_name, rang, product_name;
```

</details>
<br>

### 19. Classez les produits selon les plus chers avec `ROW_NUMBER`, `RANK` et `DENSE_RANK`.

Difficulté : 3
<details>
    <summary>Code</summary>

```sql
SELECT product_name,
       unit_price,
       ROW_NUMBER() OVER w AS row_number,
       RANK() OVER w       AS rang,
       DENSE_RANK() OVER w AS dense_rang
FROM products
WINDOW w AS (ORDER BY unit_price DESC)
ORDER BY unit_price DESC, product_name;
```

</details>
<br>

Difficulté : 3
<details>
    <summary>Code</summary>

```sql
WITH classement AS (SELECT product_name,
                           unit_price,
                           RANK() OVER (ORDER BY unit_price DESC) AS rang
                    FROM products)
SELECT product_name,
       unit_price,
       rang
FROM classement
WHERE rang <= 15
ORDER BY rang, product_name;
```

</details>
<br>

### 20. Pour chaque catégorie, trouvez les 2 produits qui ont généré le plus de chiffre d'affaires.

Difficulté : 5
<details>
    <summary>Code</summary>

```sql
WITH ventes AS (SELECT c.category_name,
                       p.product_id,
                       p.product_name,
                       SUM(od.quantity * od.unit_price * (1 - od.discount)) AS chiffre_affaires
                FROM categories c
                         JOIN products p ON c.category_id = p.category_id
                         JOIN order_details od ON p.product_id = od.product_id
                GROUP BY c.category_name, p.product_id, p.product_name),
     classement AS (SELECT category_name,
                           product_name,
                           chiffre_affaires,
                           RANK() OVER (PARTITION BY category_name ORDER BY chiffre_affaires DESC) AS rang
                    FROM ventes)
SELECT category_name,
       product_name,
       ROUND(chiffre_affaires::numeric, 2) AS chiffre_affaires,
       rang
FROM classement
WHERE rang <= 2
ORDER BY category_name, rang;
```

</details>
<br>

### 21. Répartissez les clients en 4 groupes (quartiles) selon leur nombre de commandes, du groupe 1 (le plus de commandes) au groupe 4.

Difficulté : 4
<details>
    <summary>Code</summary>

```sql
WITH nb AS (SELECT c.company_name,
                   COUNT(o.order_id) AS nb_commandes
            FROM customers c
                     LEFT JOIN orders o ON c.customer_id = o.customer_id
            GROUP BY c.customer_id, c.company_name)
SELECT company_name,
       nb_commandes,
       NTILE(4) OVER (ORDER BY nb_commandes DESC) AS quartile
FROM nb
ORDER BY quartile, nb_commandes DESC, company_name;
```

</details>
<br>

### 22. Pour chaque ligne de la commande 10248, calculez son montant ainsi que son pourcentage du montant total de la commande.

Difficulté : 5
<details>
    <summary>Code</summary>

```sql
SELECT od.order_id,
       p.product_name,
       ROUND((od.quantity * od.unit_price * (1 - od.discount))::numeric, 2) AS montant,
       ROUND((100 * (od.quantity * od.unit_price * (1 - od.discount))
           / SUM(od.quantity * od.unit_price * (1 - od.discount)) OVER (PARTITION BY od.order_id))::numeric,
             2)                                                             AS pourcentage_commande
FROM order_details od
         JOIN products p ON od.product_id = p.product_id
WHERE od.order_id = 10248
ORDER BY pourcentage_commande DESC;
```

</details>
<br>

### 23. Listez les clients dont le nom contient le mot « Store ».

Difficulté : 1
<details>
    <summary>Code</summary>

```sql
SELECT company_name, city
FROM customers
WHERE company_name LIKE '%Store%'
ORDER BY company_name;
```

</details>
<br>

### 24. Listez les produits contenant « choco » dans leur nom, sans tenir compte de la casse.

Difficulté : 1
<details>
    <summary>Code</summary>

```sql
SELECT product_name
FROM products
WHERE product_name ILIKE '%choco%'
ORDER BY product_name;
```

</details>
<br>

### 25. Listez les clients dont l'identifiant fait exactement 5 caractères et commence par la lettre « A ».

Difficulté : 1
<details>
    <summary>Code</summary>

```sql
SELECT customer_id, company_name
FROM customers
WHERE customer_id LIKE 'A____'
ORDER BY customer_id;
```

</details>
<br>

### 26. Calculez l'âge de chaque employé à son embauche, en années entières.

Difficulté : 2
<details>
    <summary>Code</summary>

```sql
SELECT first_name || ' ' || last_name                                 AS employe,
       CAST(EXTRACT(YEAR FROM AGE(hire_date, birth_date)) AS integer) AS age_embauche
FROM employees
ORDER BY age_embauche, employe;
```

</details>
<br>

### 27. Calculez l'heure d'échéance de la commande 10248, obtenue en ajoutant 2 heures 30 minutes à sa date de commande convertie en `TIMESTAMP`.

Difficulté : 2
<details>
    <summary>Code</summary>

```sql
SELECT order_id,
       order_date::timestamp + INTERVAL '2 hours 30 minutes' AS echeance
FROM orders
WHERE order_id = 10248;
```

</details>
<br>


-------

??? info "Utilisation de l'IA"
    Page rédigée en partie avec l'aide d'un assistant IA. L'IA a été utilisée pour générer des 
    explications, des exemples et/ou des suggestions de structure. Toutes les informations ont 
    été vérifiées, éditées et complétées par l'auteur.
