# COALESCE et CASE WHEN

## Fonction COALESCE

### Qu'est-ce que COALESCE ?

`COALESCE` renvoie la première valeur non nulle d'une liste d'expressions. Elle est idéale pour gérer les valeurs nulles
et fournir des valeurs par défaut ou des solutions de repli.

### Syntaxe

```sql
COALESCE
    (expression1, expression2, expression3, ...)
```

### Exemples de base

#### Gestion simple des valeurs nulles

```sql
-- Au lieu d'afficher NULL, afficher un message par défaut
SELECT name,
       COALESCE(phone, 'Aucun téléphone fourni') AS contact_phone
FROM Customer;
```

#### Plusieurs solutions de repli

```sql
-- Essayer plusieurs méthodes de contact selon un ordre de préférence
SELECT name,
       COALESCE(email, phone, 'Aucune information de contact') AS primary_contact
FROM Customer;
```

#### Valeurs numériques par défaut

```sql
-- Gérer les tarifs nuls avec une valeur par défaut
SELECT license_plate,
       make,
       model,
       COALESCE(daily_rate, 0.00) AS display_rate
FROM Car;
```

---

## Instructions CASE WHEN

### Qu'est-ce que CASE WHEN ?

`CASE WHEN` permet d'implémenter une logique conditionnelle en SQL, similaire aux instructions if-else dans les langages
de programmation.

### Deux formes de CASE

#### 1. CASE simple (comparaison d'une seule expression)

```sql
CASE expression
    WHEN valeur1 THEN résultat1
    WHEN valeur2 THEN résultat2
    ELSE résultat_par_défaut
END
```

#### 2. CASE recherché (conditions multiples)

```sql
CASE 
    WHEN condition1 THEN résultat1
    WHEN condition2 THEN résultat2
    ELSE résultat_par_défaut
END
```

### Exemples avec CASE WHEN

#### Catégorisation des données

```sql
-- Catégoriser les voitures par âge
SELECT license_plate,
       make,
       model,
       year,
       CASE
           WHEN year >= 2020 THEN 'Neuve'
           WHEN year >= 2015 THEN 'Récente'
           WHEN year >= 2010 THEN 'Ancienne'
           ELSE 'Vintage'
           END AS age_category
FROM Car;
```

#### Traduction des statuts

```sql
-- Rendre le statut des rapports de dommage plus convivial
SELECT report_id,
       damage_description,
       CASE status
           WHEN 'P' THEN 'En attente de validation'
           WHEN 'I' THEN 'En cours'
           WHEN 'C' THEN 'Terminé'
           WHEN 'X' THEN 'Annulé'
           ELSE 'Statut inconnu'
           END AS status_description
FROM DamageReport;
```

#### Calculs conditionnels

```sql
-- Appliquer des tarifs différents selon la durée de location
SELECT rental_id,
       start_date,
       end_date,
       end_date - start_date AS rental_days,
       daily_rate,
       CASE
           WHEN end_date - start_date >= 7 THEN daily_rate * 0.85 -- Réduction hebdomadaire de 15 %
           WHEN end_date - start_date >= 3 THEN daily_rate * 0.95 -- Réduction de 5 % pour 3 jours ou plus
           ELSE daily_rate
           END                        AS effective_daily_rate
FROM Rental r
         JOIN Car c ON r.car_license_plate = c.license_plate;
```

---

## Combinaison de COALESCE et CASE WHEN

### Exemples avancés

#### Affichage intelligent des contacts

```sql
-- Créer un champ de contact intelligent avec formatage
SELECT name,
       CASE
           WHEN email IS NOT NULL AND phone IS NOT NULL
               THEN CONCAT(email, ' (', phone, ')')
           WHEN email IS NOT NULL
               THEN email
           WHEN phone IS NOT NULL
               THEN phone
           ELSE 'Aucune information de contact'
           END AS formatted_contact
FROM Customer;
```

#### Liste de voitures améliorée

```sql
-- Créer des descriptions de voitures conviviales
SELECT license_plate,
       CONCAT(
               COALESCE(year, 'Année inconnue'), ' ',
               COALESCE(make, 'Marque inconnue'), ' ',
               COALESCE(model, 'Modèle inconnu')
       )                                        AS car_description,
       COALESCE(color, 'Couleur non spécifiée') AS color_display,
       CASE
           WHEN daily_rate IS NULL THEN 'Prix sur demande'
           WHEN daily_rate = 0 THEN 'Location gratuite'
           WHEN daily_rate < 50 THEN CONCAT('$', daily_rate, '/jour (Économique)')
           WHEN daily_rate < 100 THEN CONCAT('$', daily_rate, '/jour (Standard)')
           ELSE CONCAT('$', daily_rate, '/jour (Premium)')
           END                                  AS pricing_display
FROM Car;
```

#### Tableau de bord des locations

```sql
-- Créer une vue complète du statut des locations
SELECT r.rental_id,
       c.name                                          AS customer_name,
       CONCAT(car.year, ' ', car.make, ' ', car.model) AS vehicle,
       r.start_date,
       r.end_date,
       CASE
           WHEN r.start_date > now() THEN 'À venir'
           WHEN r.end_date < now() THEN 'Terminée'
           ELSE 'Active'
           END                                         AS rental_status,
       COALESCE(
               CASE
                   WHEN p.payment_id IS NOT NULL THEN 'Payée'
                   WHEN r.end_date < now() THEN 'Paiement en retard'
                   ELSE 'Paiement en attente'
                   END,
               'Statut de paiement inconnu'
       )                                               AS payment_status
FROM Rental r
         JOIN Customer c ON r.customer_id = c.customer_id
         JOIN Car car ON r.car_license_plate = car.license_plate
         LEFT JOIN Payment p ON r.rental_id = p.rental_id;
```

---

## Conseils pratiques

### 1. Gestion élégante des valeurs NULL

```sql
-- Au lieu d'afficher des valeurs NULL aux utilisateurs
SELECT name,
       COALESCE(driver_license, 'Vérification du permis en attente') AS license_status,
       COALESCE(email, 'Email non fourni')                           AS email_display
FROM Customer;
```

### 2. Création de rapports conviviaux

```sql
-- Analyse des coûts de maintenance avec catégories lisibles
SELECT car_license_plate,
       service_type,
       cost,
       CASE
           WHEN cost IS NULL THEN 'Coût non enregistré'
           WHEN cost = 0 THEN 'Sans frais'
           WHEN cost < 100 THEN 'Entretien mineur'
           WHEN cost < 500 THEN 'Maintenance standard'
           ELSE 'Réparation majeure'
           END                                              AS cost_category,
       COALESCE(description, 'Aucun détail supplémentaire') AS service_notes
FROM MaintenanceRecord;
```

### 3. Validation et nettoyage des données

```sql
-- Nettoyer et standardiser les descriptions des transactions de fidélité
SELECT transaction_id,
       customer_id,
       points_amount,
       CASE transaction_type
           WHEN 'E' THEN 'Gagnés'
           WHEN 'R' THEN 'Échangés'
           WHEN 'A' THEN 'Ajustés'
           WHEN 'B' THEN 'Bonus'
           ELSE COALESCE(transaction_type, 'Inconnu')
           END AS transaction_type_display,
       COALESCE(
               NULLIF(TRIM(description), ''), -- Convertir d'abord les chaînes vides en NULL
               'Transaction standard'
       )       AS description_clean
FROM LoyaltyTransaction;
```

---

## Modèles courants

### Modèle 1 : Concaténation sécurisée contre les NULL

```sql
-- Combiner en toute sécurité des champs pouvant être nuls
SELECT COALESCE(
               CASE
                   WHEN name IS NOT NULL AND email IS NOT NULL
                       THEN CONCAT(name, ' <', email, '>')
                   ELSE name
                   END,
               'Client anonyme'
       ) AS display_name
FROM Customer;
```

### Modèle 2 : Agrégation conditionnelle

```sql
-- Calculer différents totaux selon des conditions
SELECT location_id,
       COUNT(*)                                           AS total_cars,
       SUM(CASE WHEN daily_rate < 50 THEN 1 ELSE 0 END)   AS budget_cars,
       SUM(CASE WHEN daily_rate >= 100 THEN 1 ELSE 0 END) AS premium_cars,
       AVG(COALESCE(daily_rate, 0))                       AS average_rate
FROM Car
GROUP BY location_id;
```

### Modèle 3 : Tri dynamique

```sql
-- Créer des valeurs d'affichage triables
SELECT *,
       CASE
           WHEN status = 'Active' THEN 1
           WHEN status = 'Pending' THEN 2
           WHEN status = 'Completed' THEN 3
           ELSE 4
           END AS sort_priority
FROM (SELECT rental_id,
             customer_id,
             CASE
                 WHEN start_date > now() THEN 'Pending'
                 WHEN end_date >= now() THEN 'Active'
                 ELSE 'Completed'
                 END AS status
      FROM Rental) AS rental_with_status
ORDER BY sort_priority, start_date;
```

---

## Notes sur les performances

- `COALESCE` cesse d'évaluer les expressions dès qu'elle trouve une valeur non nulle
- `CASE WHEN` cesse d'évaluer les conditions dès qu'une correspondance est trouvée
- Les deux fonctions peuvent être utilisées dans les clauses WHERE, ORDER BY et les agrégations
- Pensez à indexer les colonnes utilisées dans les conditions CASE pour de meilleures performances

---

## Résumé

- **COALESCE** : Idéal pour fournir des valeurs par défaut et gérer les valeurs nulles
- **CASE WHEN** : Essentiel pour la logique conditionnelle et la transformation des données
- **Combinaison** : Permet de créer des présentations de données sophistiquées et conviviales
- **Bonne pratique** : Pensez toujours à la manière dont vos résultats seront perçus par les utilisateurs finaux, et
  gérez les cas limites avec élégance

-------

??? info "Utilisation de l'IA"
      Page rédigée en partie avec l'aide d'un assistant IA, principalement à l'aide de Perplexity AI. L'IA a été 
      utilisée pour générer des explications, des exemples et/ou des suggestions de structure. Toutes les informations 
      ont été vérifiées, éditées et complétées par l'auteur.