# Présentation et formatage des données

*Ce guide s'appuie sur les bases de COALESCE et CASE WHEN, en présentant des modèles avancés pour une présentation
professionnelle des données.*

---

## Fonction NULLIF

### Qu'est-ce que NULLIF ?

`NULLIF` convertit des valeurs spécifiques en NULL. C’est en quelque sorte l’« inverse » de COALESCE, idéal pour
nettoyer les données avant d’appliquer des valeurs par défaut.

### Syntaxe

```sql
NULLIF(expression1, expression2)
-- Renvoie NULL si expression1 est égale à expression2, sinon renvoie expression1
```

### Exemples de base

#### Nettoyage des chaînes vides

```sql
-- Convertir les chaînes vides en NULL, puis appliquer des valeurs par défaut
SELECT name,
       COALESCE(NULLIF(TRIM(phone), ''), 'Aucun téléphone fourni') AS clean_phone,
       COALESCE(NULLIF(TRIM(email), ''), 'Aucun email fourni')     AS clean_email
FROM Customer;
```

#### Gestion des valeurs nulles

```sql
-- Traiter les tarifs quotidiens à zéro comme NULL pour un meilleur affichage
SELECT license_plate,
       make,
       model,
       CASE
           WHEN NULLIF(daily_rate, 0) IS NULL THEN 'Prix sur demande'
           ELSE CONCAT('$', daily_rate, '/jour')
           END AS pricing_display
FROM Car;
```

---

## Fonctions de concaténation de chaînes

### CONCAT et CONCAT_WS

#### CONCAT - Concaténation de base

```sql
-- Construire des descriptions de voitures formatées
SELECT license_plate,
       CONCAT(
               COALESCE(CAST(year AS VARCHAR), 'Inconnue'), ' ',
               COALESCE(make, 'Inconnue'), ' ',
               COALESCE(model, 'Inconnu'),
               CASE
                   WHEN color IS NOT NULL THEN CONCAT(' (', color, ')')
                   ELSE ''
                   END
       ) AS full_description
FROM Car;
```

#### CONCAT_WS - Concaténer avec séparateur

```sql
-- Construire un résumé de contact avec gestion automatique des séparateurs
SELECT name,
       CONCAT_WS(' | ',
                 NULLIF(TRIM(email), ''),
                 NULLIF(TRIM(phone), ''),
                 CASE
                     WHEN driver_license IS NOT NULL
                         THEN CONCAT('Permis : ', driver_license)
                     ELSE NULL END
       ) AS contact_summary
FROM Customer;
```

### Modèles avancés de concaténation

#### Formatage intelligent des adresses

```sql
-- Formater intelligemment les adresses des agences
SELECT name,
       CONCAT_WS(', ',
                 NULLIF(TRIM(address), ''),
                 CASE
                     WHEN phone IS NOT NULL THEN CONCAT('Tél : ', phone)
                     ELSE NULL
                     END
       )                                                                     AS formatted_address,
       COALESCE(NULLIF(TRIM(operating_hours), ''), 'Horaires non spécifiés') AS hours_display
FROM RentalLocation;
```

---

## Nettoyage et formatage des chaînes

### Fonctions TRIM

#### Nettoyage de base

```sql
-- Nettoyer les descriptions d’entretien
SELECT maintenance_id,
       service_type,
       COALESCE(
               NULLIF(TRIM(description), ''),
               CONCAT('Entretien ', LOWER(service_type), ' standard')
       ) AS clean_description
FROM MaintenanceRecord;
```

### UPPER, LOWER et conversion de casse

#### Standardisation de la casse du texte

```sql
-- Standardiser l’affichage des marques et modèles de voitures
SELECT license_plate,
       UPPER(COALESCE(make, 'INCONNU')) AS make_display,
       CONCAT(
               UPPER(LEFT(COALESCE(model, 'inconnu'), 1)),
               LOWER(SUBSTRING(COALESCE(model, 'inconnu'), 2))
       )                                AS model_display,
       CASE
           WHEN color IS NOT NULL
               THEN CONCAT(UPPER(LEFT(color, 1)), LOWER(SUBSTRING(color, 2)))
           ELSE 'Couleur non spécifiée'
           END                          AS color_display
FROM Car;
```

### Fonction REPLACE

#### Standardisation des données

```sql
-- Standardiser et nettoyer les méthodes de paiement
SELECT payment_id,
       amount,
       CASE
           WHEN UPPER(REPLACE(REPLACE(payment_method, '_', ' '), '-', ' ')) LIKE '%CREDIT%'
               THEN 'Carte de crédit'
           WHEN UPPER(REPLACE(REPLACE(payment_method, '_', ' '), '-', ' ')) LIKE '%DEBIT%'
               THEN 'Carte de débit'
           WHEN UPPER(REPLACE(REPLACE(payment_method, '_', ' '), '-', ' ')) LIKE '%CASH%'
               THEN 'Espèces'
           ELSE COALESCE(
                   NULLIF(TRIM(REPLACE(REPLACE(payment_method, '_', ' '), '-', ' ')), ''),
                   'Autre'
                )
           END AS standardized_payment_method
FROM Payment;
```

---

## Longueur des chaînes et troncature

### Fonction LENGTH/LEN

#### Troncature intelligente du texte

```sql
-- Tronquer les descriptions longues avec des points de suspension
SELECT report_id,
       CASE
           WHEN LENGTH(damage_description) > 50
               THEN CONCAT(LEFT(damage_description, 47), '...')
           ELSE damage_description
           END AS display_description,
       CASE
           WHEN LENGTH(damage_description) > 100 THEN 'Description longue'
           WHEN LENGTH(damage_description) > 50 THEN 'Description moyenne'
           ELSE 'Description courte'
           END AS description_category
FROM DamageReport
WHERE damage_description IS NOT NULL;
```

### Fonctions SUBSTRING/LEFT/RIGHT

#### Extraction et formatage des données

```sql
-- Créer des codes d’affichage et des résumés
SELECT rental_id,
       CONCAT('R-', RIGHT(CONCAT('0000', rental_id), 4)) AS rental_code,
       LEFT(COALESCE(
                    CONCAT(c.name, ' loue ', car.make, ' ', car.model),
                    'Enregistrement de location incomplet'
            ), 80)                                       AS rental_summary
FROM Rental r
         LEFT JOIN Customer c ON r.customer_id = c.customer_id
         LEFT JOIN Car car ON r.car_license_plate = car.license_plate;
```

---

## Formatage numérique et conversion

### ROUND, CEILING, FLOOR

#### Affichage numérique propre

```sql
-- Formater les coûts et tarifs pour l’affichage
SELECT license_plate,
       make,
       model,
       daily_rate AS raw_rate,
       CASE
           WHEN daily_rate IS NULL THEN 'Prix sur demande'
           WHEN daily_rate = FLOOR(daily_rate)
               THEN CONCAT('$', CAST(daily_rate AS INT))
           ELSE CONCAT('$', ROUND(daily_rate, 2))
           END    AS formatted_rate,
       CASE
           WHEN daily_rate < 25 THEN 'Économique'
           WHEN daily_rate < 75 THEN 'Standard'
           ELSE 'Premium'
           END    AS rate_category
FROM Car;
```

### CAST et CONVERT

#### Formatage des types de données

```sql
-- Formater les coûts d’entretien avec les bons types
SELECT car_license_plate,
       service_type,
       CASE
           WHEN cost IS NULL THEN 'Coût non enregistré'
           WHEN cost = 0 THEN 'Sans frais'
           ELSE CONCAT('$',
                       CASE
                           WHEN cost = FLOOR(cost) THEN CAST(cost AS VARCHAR)
                           ELSE CAST(ROUND(cost, 2) AS VARCHAR)
                           END
                )
           END AS formatted_cost,
       COALESCE(
               CAST(mileage AS VARCHAR),
               'Kilométrage non enregistré'
       )       AS mileage_display
FROM MaintenanceRecord;
```

---

## Correspondance de motifs avancée

### Opérateur LIKE dans les instructions CASE

#### Catégorisation basée sur les motifs

```sql
-- Catégoriser les types de couverture d’assurance
SELECT coverage_id,
       type,
       CASE
           WHEN UPPER(type) LIKE '%COMPREHENSIVE%' OR UPPER(type) LIKE '%FULL%'
               THEN 'Couverture complète'
           WHEN UPPER(type) LIKE '%LIABILITY%' OR UPPER(type) LIKE '%BASIC%'
               THEN 'Couverture de base'
           WHEN UPPER(type) LIKE '%COLLISION%'
               THEN 'Collision uniquement'
           WHEN UPPER(type) LIKE '%THEFT%' OR UPPER(type) LIKE '%SECURITY%'
               THEN 'Protection contre le vol'
           ELSE COALESCE(NULLIF(TRIM(type), ''), 'Autre couverture')
           END                          AS coverage_category,
       CONCAT('$', daily_cost, '/jour') AS cost_display
FROM InsuranceCoverage;
```

### Opérateur IN pour conditions multiples

#### Simplification des conditions multiples

```sql
-- Catégoriser les employés selon les types de sites
SELECT e.employee_id,
       e.name,
       l.name  AS location_name,
       CASE
           WHEN UPPER(l.name) IN ('HEADQUARTERS', 'MAIN OFFICE', 'CORPORATE')
               THEN 'Personnel administratif'
           WHEN UPPER(l.name) LIKE '%AIRPORT%' OR l.name IN ('LAX', 'JFK', 'ORD')
               THEN 'Personnel aéroportuaire'
           WHEN UPPER(l.name) LIKE '%DOWNTOWN%' OR UPPER(l.name) LIKE '%CITY%'
               THEN 'Personnel urbain'
           ELSE 'Personnel régional'
           END AS staff_category,
       COALESCE(
               NULLIF(TRIM(e.email), ''),
               CONCAT(LOWER(REPLACE(e.name, ' ', '.')), '@company.com')
       )       AS email_display
FROM Employee e
         JOIN RentalLocation l ON e.location_id = l.location_id;
```

---

## Modèles combinés complexes

### Modèle 1 : Nettoyage multi-niveaux des données

```sql
-- Nettoyage et formatage complets des contacts clients
SELECT customer_id,
       COALESCE(NULLIF(TRIM(name), ''), 'Client anonyme') AS clean_name,
       CASE
           WHEN NULLIF(TRIM(email), '') IS NOT NULL
               AND NULLIF(TRIM(phone), '') IS NOT NULL
               THEN CONCAT(
                   TRIM(email),
                   ' (',
                   CASE
                       WHEN LENGTH(REPLACE(REPLACE(phone, '-', ''), ' ', '')) = 10
                           THEN CONCAT(
                               LEFT(REPLACE(REPLACE(phone, '-', ''), ' ', ''), 3), '-',
                               SUBSTRING(REPLACE(REPLACE(phone, '-', ''), ' ', ''), 4, 3), '-',
                               RIGHT(REPLACE(REPLACE(phone, '-', ''), ' ', ''), 4)
                                )
                       ELSE phone
                       END,
                   ')'
                    )
           WHEN NULLIF(TRIM(email), '') IS NOT NULL
               THEN TRIM(email)
           WHEN NULLIF(TRIM(phone), '') IS NOT NULL
               THEN CONCAT('Tél : ', TRIM(phone))
           ELSE 'Aucune information de contact'
           END                                            AS formatted_contact
FROM Customer;
```

### Modèle 2 : Affichage intelligent de la logique métier

```sql
-- Statut de location avec règles métier et formatage
SELECT r.rental_id,
       CONCAT('R-', RIGHT(CONCAT('0000', r.rental_id), 4))  AS rental_code,
       COALESCE(NULLIF(TRIM(c.name), ''), 'Client inconnu') AS customer_name,
       CONCAT_WS(' ',
                 COALESCE(CAST(car.year AS VARCHAR), ''),
                 COALESCE(car.make, ''),
                 COALESCE(car.model, '')
       )                                                    AS vehicle_description,
       CASE
           WHEN r.total_cost IS NULL THEN 'Coût en attente de calcul'
           WHEN r.total_cost = 0 THEN 'Location gratuite'
           WHEN r.total_cost = FLOOR(r.total_cost)
               THEN CONCAT('$', CAST(r.total_cost AS VARCHAR))
           ELSE CONCAT('$', CAST(ROUND(r.total_cost, 2) AS VARCHAR))
           END                                              AS cost_display,
       CASE
           WHEN p.payment_id IS NOT NULL THEN 'Payée'
           WHEN r.total_cost > 0 THEN 'Paiement en attente'
           ELSE 'Aucun paiement requis'
           END                                              AS payment_status
FROM Rental r
         LEFT JOIN Customer c ON r.customer_id = c.customer_id
         LEFT JOIN Car car ON r.car_license_plate = car.license_plate
         LEFT JOIN Payment p ON r.rental_id = p.rental_id;
```

### Modèle 3 : Présentation complète des données

```sql
-- Affichage complet de l’inventaire automobile avec tout le formatage
SELECT c.license_plate,
       CONCAT_WS(' ',
                 CASE WHEN c.year IS NOT NULL THEN CAST(c.year AS VARCHAR) ELSE 'Année inconnue' END,
                 COALESCE(UPPER(c.make), 'MARQUE INCONNUE'),
                 COALESCE(
                         CONCAT(UPPER(LEFT(c.model, 1)), LOWER(SUBSTRING(c.model, 2))),
                         'Modèle inconnu'
                 )
       )                                       AS vehicle_name,
       COALESCE(
               CONCAT(UPPER(LEFT(c.color, 1)), LOWER(SUBSTRING(c.color, 2))),
               'Couleur non spécifiée'
       )                                       AS color_display,
       CASE
           WHEN NULLIF(c.daily_rate, 0) IS NULL THEN 'Prix sur demande'
           WHEN c.daily_rate = FLOOR(c.daily_rate)
               THEN CONCAT('$', CAST(c.daily_rate AS VARCHAR), '/jour')
           ELSE CONCAT('$', CAST(ROUND(c.daily_rate, 2) AS VARCHAR), '/jour')
           END                                 AS rate_display,
       COALESCE(cat.name, 'Non catégorisé')    AS category_name,
       COALESCE(loc.name, 'Site non attribué') AS location_name,
       CASE
           WHEN c.daily_rate IS NULL THEN 'Contacter pour tarif'
           WHEN c.daily_rate < 30 THEN 'Option économique'
           WHEN c.daily_rate < 60 THEN 'Option standard'
           WHEN c.daily_rate < 100 THEN 'Option premium'
           ELSE 'Option luxe'
           END                                 AS marketing_category
FROM Car c
         LEFT JOIN CarCategory cat ON c.category_id = cat.category_id
         LEFT JOIN RentalLocation loc ON c.location_id = loc.location_id;
```

---

## Conseils de performance

1. **Optimisation des fonctions de chaîne** :
    - Utilisez le motif `NULLIF(TRIM(field), '')` pour nettoyer efficacement les données
    - `CONCAT_WS` est plus efficace que plusieurs opérations CONCAT
    - Envisagez de créer des colonnes calculées pour les expressions complexes fréquemment utilisées

2. **Sensibilité à la casse** :
    - Utilisez `UPPER()` ou `LOWER()` dans les comparaisons LIKE pour assurer la cohérence
    - Tenez compte des paramètres de classement pour les comparaisons de chaînes

3. **Chaîne de gestion des NULL** :
    - Ordre recommandé : `COALESCE(NULLIF(TRIM(field), ''), 'valeur_par_défaut')`
    - Ce motif gère les valeurs NULL, les chaînes vides et les chaînes contenant uniquement des espaces

4. **Vérifications de longueur** :
    - Vérifiez toujours la longueur de la chaîne avant d’utiliser SUBSTRING/LEFT/RIGHT
    - Utilisez CASE WHEN avec des vérifications de longueur pour éviter les erreurs

---

## Résumé

Ces fonctions avancées fonctionnent conjointement avec COALESCE et CASE WHEN pour créer des présentations de données
professionnelles et conviviales :

- **NULLIF** : Nettoie les données avant d’appliquer des valeurs par défaut
- **CONCAT/CONCAT_WS** : Construit intelligemment des chaînes formatées
- **TRIM/UPPER/LOWER** : Standardise la présentation du texte
- **REPLACE** : Nettoie et standardise les valeurs des données
- **LENGTH/SUBSTRING** : Contrôle la longueur d’affichage du texte
- **ROUND/CAST** : Formate correctement les nombres
- **LIKE/IN** : Simplifie la correspondance de motifs complexes dans les conditions

L’essentiel est d’enchaîner ces fonctions pour gérer les cas limites et créer un rendu cohérent et professionnel, clair
et utile pour les utilisateurs finaux.

-------

??? info "Utilisation de l'IA"
      Page rédigée en partie avec l'aide d'un assistant IA, principalement à l'aide de Perplexity AI. L'IA a été 
      utilisée pour générer des explications, des exemples et/ou des suggestions de structure. Toutes les informations 
      ont été vérifiées, éditées et complétées par l'auteur.