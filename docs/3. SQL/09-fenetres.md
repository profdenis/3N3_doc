
# 9 - Autres fonctions fenêtrées essentielles en SQL

## Introduction

Après avoir maîtrisé `RANK()`, il est temps d’explorer d’autres fonctions fenêtrées puissantes qui vous permettront de
répondre à des besoins analytiques plus complexes : comparaisons temporelles, calculs cumulatifs, segmentation, et bien
plus encore.

Les fonctions fenêtrées ne se limitent pas au classement — elles permettent de **calculer des agrégats, des décalages,
des pourcentages, et des statistiques avancées** tout en conservant le détail ligne par ligne.

---

## 1. ROW_NUMBER() — Numérotation séquentielle

### À quoi ça sert ?

Attribue un numéro unique à chaque ligne dans la fenêtre, même en cas d’égalité.

### Syntaxe

```sql
ROW_NUMBER() OVER ([PARTITION BY colonne(s)] ORDER BY colonne(s)
)
```

### Exemple

```sql
SELECT employee_name,
       department_id,
       salary,
       ROW_NUMBER() OVER (
           PARTITION BY department_id
           ORDER BY salary DESC
           ) as row_num_in_dept
FROM employees;
```

→ Utile pour sélectionner **la première ou la dernière ligne** d’un groupe (ex. : top 1 par département).

---

## 2. DENSE_RANK() — Classement sans trous

### À quoi ça sert ?

Comme `RANK()`, mais **sans sauter de numéros** après les égalités.

### Exemple comparatif avec RANK()

| Salaire | RANK() | DENSE_RANK() | ROW_NUMBER() |
|---------|--------|--------------|--------------|
| 90000   | 1      | 1            | 1            |
| 85000   | 2      | 2            | 2            |
| 85000   | 2      | 2            | 3            |
| 80000   | 4      | 3            | 4            |

→ Idéal pour les classements “denses” comme les palmarès académiques ou sportifs.

---

## 3. NTILE(n) — Segmentation en groupes égaux

### À quoi ça sert ?

Divise les lignes en **n groupes de taille égale (ou presque)**, utile pour la segmentation (quartiles, déciles, etc.).

### Syntaxe

```sql
NTILE (nombre_groupes) OVER ([PARTITION BY ...] ORDER BY ...)
```

### Exemple : Diviser les employés en 4 quartiles de salaire

```sql
SELECT employee_name,
       salary,
       NTILE(4) OVER (ORDER BY salary DESC) as salary_quartile
FROM employees;
```

→ Résultat : chaque employé est assigné à un quartile (1 = top 25%, 4 = bas 25%).

---

## 4. LAG() et LEAD() — Accéder aux lignes précédentes ou suivantes

### À quoi ça sert ?

- `LAG()` : récupère la valeur de la **ligne précédente**
- `LEAD()` : récupère la valeur de la **ligne suivante**

Parfait pour les **comparaisons temporelles** (évolution mensuelle, différence d’une période à l’autre).

### Syntaxe

```sql
LAG(colonne [, offset [, valeur_par_défaut]]) OVER (ORDER BY ...)
LEAD(colonne [, offset [, valeur_par_défaut]]) OVER (ORDER BY ...)
```

### Exemple : Évolution des ventes mensuelles

```sql
SELECT month,
       sales,
       LAG(sales) OVER (ORDER BY month)         as previous_month_sales,
       sales - LAG(sales) OVER (ORDER BY month) as monthly_change
FROM monthly_sales;
```

→ Vous pouvez aussi utiliser `offset = 2` pour remonter de 2 lignes, ou définir une valeur par défaut si la ligne
n’existe pas.

---

## 5. FIRST_VALUE() et LAST_VALUE() — Valeurs extrêmes dans la fenêtre

### À quoi ça sert ?

- `FIRST_VALUE()` : renvoie la première valeur de la fenêtre
- `LAST_VALUE()` : renvoie la dernière valeur de la fenêtre (attention au cadrage !)

### Syntaxe

```sql
FIRST_VALUE(colonne) OVER ([PARTITION BY ...] ORDER BY ... [ROWS BETWEEN ...])

LAST_VALUE(colonne) OVER ([PARTITION BY ...] ORDER BY ... ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING  -- Important !)
```

### Exemple : Comparer chaque salaire au meilleur salaire du département

```sql
SELECT employee_name,
       department_id,
       salary,
       FIRST_VALUE(salary) OVER (PARTITION BY department_id ORDER BY salary DESC) as highest_salary_in_dept
FROM employees;
```

⚠️ **Attention avec LAST_VALUE** : par défaut, la fenêtre s’arrête à la ligne courante. Pour obtenir la vraie dernière
valeur, il faut spécifier :

```sql
ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
```

---

## 6. SUM(), AVG(), COUNT() … en mode fenêtré

### À quoi ça sert ?

Calculer des **agrégats cumulatifs ou glissants** sans perdre le détail ligne par ligne.

### Syntaxe

```sql
SUM(colonne) OVER ([PARTITION BY ...] ORDER BY ... [ROWS BETWEEN ...])
```

### Exemple 1 : Cumul des ventes

```sql
SELECT month,
       sales,
       SUM(sales) OVER (ORDER BY month) as cumulative_sales
FROM monthly_sales;
```

### Exemple 2 : Moyenne mobile sur 3 mois

```sql
SELECT month,
       sales,
       AVG(sales) OVER (ORDER BY month ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) as moving_avg_3_months
FROM monthly_sales;
```

→ Très utile en analyse financière, prévisions, indicateurs de tendance.

---

## 7. PERCENT_RANK() et CUME_DIST() — Classement en pourcentage

### PERCENT_RANK()

Calcule le **rang relatif en pourcentage** (0 à 1) d’une ligne dans sa partition.

```sql
PERCENT_RANK() OVER ([PARTITION BY ...] ORDER BY ...)
```

→ 0 = pire, 1 = meilleur (mais jamais exactement 1 — formule : `(rank - 1) / (total_rows - 1)`)

### CUME_DIST()

Calcule la **distribution cumulative** — proportion de lignes avec une valeur ≤ à la ligne courante.

```sql
CUME_DIST() OVER ([PARTITION BY ...] ORDER BY ...)
```

→ Peut atteindre 1 — formule : `nombre de lignes <= valeur courante / total lignes`

### Exemple

```sql
SELECT student_name,
       score,
       PERCENT_RANK() OVER (ORDER BY score DESC) as pct_rank,
       CUME_DIST() OVER (ORDER BY score DESC)    as cume_dist
FROM exam_results;
```

→ Utile pour les analyses statistiques, benchmarks, normalisation.

---

## 8. Framing (cadrage de fenêtre) — Contrôler précisément la fenêtre

Vous pouvez définir **exactement quelles lignes** inclure dans le calcul avec `ROWS BETWEEN` ou `RANGE BETWEEN`.

### Options courantes :

- `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` → cumul depuis le début
- `ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING` → fenêtre de 3 lignes (précédente, courante, suivante)
- `ROWS BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING` → cumul jusqu’à la fin

### Exemple : Somme des 2 mois précédents + mois courant

```sql
SUM(sales) OVER (ORDER BY month ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) as rolling_3_months
```

→ Essentiel pour les calculs de fenêtres glissantes, cumuls partiels, etc.

---

## 9. Comparaison des fonctions fenêtrées

| Fonction       | Objectif principal            | Gère les égalités ? | Crée des trous ? | Partition ? | Ordre requis ? |
|----------------|-------------------------------|---------------------|------------------|-------------|----------------|
| ROW_NUMBER()   | Numérotation unique           | Non                 | Non              | Oui         | Oui            |
| RANK()         | Classement compétitif         | Oui                 | Oui              | Oui         | Oui            |
| DENSE_RANK()   | Classement dense              | Oui                 | Non              | Oui         | Oui            |
| NTILE(n)       | Segmentation en groupes       | Oui                 | Non              | Oui         | Oui            |
| LAG()/LEAD()   | Accès aux lignes voisines     | —                   | —                | Oui         | Oui            |
| FIRST_VALUE()  | Première valeur de la fenêtre | —                   | —                | Oui         | Oui            |
| LAST_VALUE()   | Dernière valeur de la fenêtre | —                   | —                | Oui         | Oui*           |
| SUM() OVER()   | Agrégats cumulatifs/glissants | —                   | —                | Oui         | Oui (souvent)  |
| PERCENT_RANK() | Rang en pourcentage (0-1)     | Oui                 | Oui              | Oui         | Oui            |
| CUME_DIST()    | Distribution cumulative       | Oui                 | Non              | Oui         | Oui            |

> *LAST_VALUE nécessite souvent un cadrage explicite pour fonctionner correctement.

---

## 10. Exemples avancés combinant plusieurs fonctions

### Exemple 1 : Tableau de bord analytique des ventes

```sql
SELECT month,
       sales,
       LAG(sales) OVER (ORDER BY month)    as prev_month_sales,
       ROUND(
               100.0 * (sales - LAG(sales) OVER (ORDER BY month))
                   / LAG(sales) OVER (ORDER BY month), 2
       )                                   as growth_percent,
       SUM(sales) OVER (ORDER BY month)    as cumulative_sales,
       AVG(sales) OVER (
           ORDER BY month
           ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
           )                               as moving_avg_3_months,
       NTILE(4) OVER (ORDER BY sales DESC) as sales_quartile
FROM monthly_sales
ORDER BY month;
```

### Exemple 2 : Analyse des performances étudiantes

```sql
SELECT student_name,
       subject,
       score,
       RANK() OVER (PARTITION BY subject ORDER BY score DESC)         as subject_rank,
       PERCENT_RANK() OVER (PARTITION BY subject ORDER BY score DESC) as subject_pct_rank,
       AVG(score) OVER (PARTITION BY subject)                         as subject_avg,
       score - AVG(score) OVER (PARTITION BY subject)                 as diff_from_avg,
       FIRST_VALUE(student_name) OVER (
           PARTITION BY subject
           ORDER BY score DESC
           )                                                          as top_student_in_subject
FROM student_grades;
```

---

## 11. Bonnes pratiques et pièges à éviter

### Bonnes pratiques

1. **Toujours spécifier ORDER BY** sauf si intentionnel (ex. : `COUNT(*) OVER()`).
2. **Utiliser PARTITION BY** pour des calculs par groupe.
3. **Documenter la logique de fenêtrage** dans les requêtes complexes.
4. **Tester les bords** (première/dernière ligne, valeurs NULL, égalités).
5. **Optimiser les performances** : indexer les colonnes de `ORDER BY` et `PARTITION BY`.

### Pièges courants

1. **Oublier le framing** avec `LAST_VALUE()` → résultat inattendu.
2. **Confondre ROWS et RANGE** → comportement différent avec valeurs égales.
3. **Calculer des agrégats sans ORDER BY** → résultat non déterministe.
4. **Ne pas gérer les NULL** dans `LAG/LEAD` → utiliser une valeur par défaut.
5. **Sur-utiliser les fenêtres** → parfois, une sous-requête ou un `JOIN` est plus lisible ou performant.

---

## 12. Exercices pratiques

### Exercice 1 : Évolution des prix boursiers

Créez une table `stock_prices` avec colonnes : `ticker`, `date`, `price`.
Calculez :

- Le prix de la veille (`LAG`)
- Le changement en % par rapport à la veille
- La moyenne mobile sur 5 jours

### Exercice 2 : Segmentation client

Avec une table `customers` (id, total_purchases), assignez chaque client à un décile d’achat (`NTILE(10)`).

### Exercice 3 : Palmarès par catégorie

Dans une table `products` (name, category, rating), affichez :

- Le rang par catégorie (`RANK`)
- Le meilleur produit de chaque catégorie (`FIRST_VALUE`)
- La différence de note avec le meilleur produit de la catégorie

### Exercice 4 : Analyse cumulative

Avec une table `orders` (order_date, amount), calculez :

- Le cumul des ventes par mois
- Le pourcentage du cumul par rapport au total annuel (`CUME_DIST` ou calcul manuel)

### Exercice avancé : Classement complexe

Créez un classement général des étudiants basé sur :

- 50% du rang en mathématiques
- 30% du rang en sciences
- 20% du rang en littérature
  (Utilisez `DENSE_RANK()` par matière, puis calculez un score pondéré)

---

## 13. Résumé

Les fonctions fenêtrées transforment SQL d’un langage de requêtes en un **langage d’analyse puissant**. En combinant :

- Classements (`RANK`, `DENSE_RANK`, `ROW_NUMBER`, `NTILE`)
- Comparaisons temporelles (`LAG`, `LEAD`)
- Agrégats contextuels (`SUM`, `AVG`, `COUNT` OVER)
- Statistiques avancées (`PERCENT_RANK`, `CUME_DIST`)
- Contrôle fin de la fenêtre (`ROWS BETWEEN`, `RANGE`)

… vous pouvez répondre à des questions analytiques complexes **sans quitter la base de données**, avec une grande
efficacité et lisibilité.

---

## 14. Prochaines étapes

Après ce chapitre, vous pouvez explorer :

1. **Fonctions conditionnelles fenêtrées** : `CASE` + fonctions fenêtrées
2. **Fenêtres nommées** (WINDOW clause) pour éviter la répétition
3. **Optimisation avancée** : index, materialized views, partition pruning
4. **Intégration avec BI tools** : comment ces fonctions se traduisent dans Tableau, Power BI, etc.
5. **Analyse temporelle avancée** : fonctions de série temporelle, prédictions simples

---

**Vous maîtrisez désormais l’essentiel des fonctions fenêtrées en SQL.**  
Ces outils sont utilisés quotidiennement par les analystes de données, data scientists, et ingénieurs en bases de
données pour transformer des données brutes en insights actionnables — directement dans la base.


-------

??? info "Utilisation de l'IA"
      Page rédigée en partie avec l'aide d'un assistant IA, principalement à l'aide de Perplexity AI. L'IA a été 
      utilisée pour générer des explications, des exemples et/ou des suggestions de structure. Toutes les informations 
      ont été vérifiées, éditées et complétées par l'auteur.