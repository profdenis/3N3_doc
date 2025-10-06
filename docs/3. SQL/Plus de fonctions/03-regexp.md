# Expressions régulières

## Introduction aux expressions régulières

### Qu’est-ce qu’une expression régulière ?

Les expressions régulières (regex) sont des outils puissants de correspondance de motifs qui permettent de rechercher,
faire correspondre et manipuler du texte en fonction de motifs spécifiques. Elles offrent un moyen concis et flexible
d’identifier des chaînes de texte pertinentes, comme des caractères, mots ou motifs de caractères particuliers.

### Concepts fondamentaux des regex

#### Métacaractères de base

- `.` – Correspond à n’importe quel caractère unique (sauf un saut de ligne)
- `*` – Correspond à zéro ou plusieurs occurrences du caractère précédent
- `+` – Correspond à une ou plusieurs occurrences du caractère précédent
- `?` – Correspond à zéro ou une occurrence du caractère précédent
- `^` – Correspond au début d’une chaîne
- `$` – Correspond à la fin d’une chaîne
- `[]` – Classe de caractères (correspond à n’importe quel caractère entre crochets)
- `()` – Groupement (crée un groupe de capture)
- `|` – Alternative (opérateur OU)

#### Quantificateurs

- `{n}` – Exactement n occurrences
- `{n,}` – n occurrences ou plus
- `{n,m}` – Entre n et m occurrences

#### Classes de caractères

- `\d` – Tout chiffre (0-9)
- `\w` – Tout caractère alphanumérique (lettres, chiffres + underscore)
- `\s` – Tout caractère d’espacement
- `\D` – Tout caractère non-chiffre
- `\W` – Tout caractère non-alphanumérique
- `\S` – Tout caractère non-espacement

---

## Les regex dans différents systèmes

### Langages de programmation

#### JavaScript

```javascript
// Objet regex global
const pattern = /^[A-Z]{2}\d{4}$/i;
const isValid = pattern.test("AB1234");

// Méthodes sur les chaînes
const result = "ABC123".match(/\d+/);
const replaced = "ABC123".replace(/\d+/, "XXX");
```

#### Python

```python
import re

# Compilation du motif
pattern = re.compile(r'^[A-Z]{2}\d{4}$', re.IGNORECASE)
is_valid = pattern.match("AB1234")

# Utilisation directe
result = re.search(r'\d+', "ABC123")
replaced = re.sub(r'\d+', "XXX", "ABC123")
```

#### Java

```java
import java.util.regex.Pattern;
import java.util.regex.Matcher;

Pattern pattern = Pattern.compile("^[A-Z]{2}\\d{4}$", Pattern.CASE_INSENSITIVE);
Matcher matcher = pattern.matcher("AB1234");
boolean isValid = matcher.matches();
```

### Outils en ligne de commande

#### grep

```bash
# Trouver les lignes correspondant au motif
grep '^[A-Z]{2}[0-9]{4}$' filename.txt

# Recherche insensible à la casse
grep -i 'error\|warning' logfile.txt
```

#### sed

```bash
# Remplacement avec regex
sed 's/[0-9]\{4\}/XXXX/g' filename.txt
```

### Différences clés entre les systèmes

1. **Variations de syntaxe** : Les exigences d’échappement diffèrent (ex. : `\d` vs `[0-9]`)
2. **Drapeaux/modificateurs** : Différentes façons de spécifier la sensibilité à la casse, le mode multiligne
3. **Groupes de capture** : Syntaxe variable pour les références arrière
4. **Performance** : Les implémentations des moteurs varient considérablement
5. **Support des fonctionnalités** : Tous les systèmes ne prennent pas en charge toutes les fonctionnalités regex

---

## Expressions régulières dans les bases de données SQL

### Pourquoi utiliser les regex en SQL ?

La correspondance de motifs traditionnelle en SQL avec `LIKE` se limite aux caractères génériques simples (`%` et `_`).
Les expressions régulières offrent :

- Une correspondance de motifs complexes
- La validation des données
- L’extraction et la manipulation de texte
- Des capacités de recherche avancées
- Le nettoyage et la transformation des données

### Limitations courantes de la correspondance de motifs SQL

```sql
-- LIKE est limité aux motifs simples
SELECT *
FROM Customer
WHERE phone LIKE '___-___-____'; -- Format fixe uniquement
SELECT *
FROM Customer
WHERE email LIKE '%@%.%';
-- Vérification très basique d'email

-- Les regex offrent une précision bien supérieure
-- Exemple PostgreSQL (nous le détaillerons plus bas)
SELECT *
FROM Customer
WHERE phone ~ '^\(\d{3}\)\s\d{3}-\d{4}$';
```

### Capacités générales des regex en SQL

La plupart des bases de données SQL modernes proposent des fonctionnalités regex pour :

1. **Correspondance de motifs** – Tester si un texte correspond à un motif
2. **Extraction de texte** – Extraire des portions de texte correspondant à des motifs
3. **Remplacement de texte** – Remplacer du texte selon des motifs
4. **Validation des données** – S’assurer que les données respectent des formats attendus
5. **Optimisation des recherches** – Trouver des enregistrements à l’aide de motifs textuels complexes

---

## Plongée approfondie dans les regex PostgreSQL

PostgreSQL dispose du support regex le plus complet et standardisé parmi les bases de données SQL, implémentant les
expressions régulières POSIX avec des extensions.

### Opérateurs regex PostgreSQL

#### Opérateurs de correspondance de base

```sql
-- ~ : Correspondance sensible à la casse
SELECT *
FROM Customer
WHERE email ~ '^[a-z]+@[a-z]+\.[a-z]{2,}$';

-- ~* : Correspondance insensible à la casse  
SELECT *
FROM Customer
WHERE email ~* '^[A-Z]+@[A-Z]+\.[A-Z]{2,}$';

-- !~ : Non-correspondance sensible à la casse
SELECT *
FROM Customer
WHERE phone !~ '^\d{10}$';

-- !~* : Non-correspondance insensible à la casse
SELECT *
FROM Customer
WHERE name !~* '^test.*';
```

#### Fonctions avancées

##### REGEXP_REPLACE

```sql
-- Remplacement de base
SELECT phone,
       REGEXP_REPLACE(phone, '[^0-9]', '', 'g') AS digits_only
FROM Customer;

-- Plusieurs remplacements avec drapeaux
SELECT name,
       REGEXP_REPLACE(name, '\s+', '_', 'g') AS url_friendly_name
FROM Customer;

-- Utilisation de groupes de capture
SELECT phone,
       REGEXP_REPLACE(phone, '(\d{3})(\d{3})(\d{4})', '(\1) \2-\3') AS formatted_phone
FROM Customer
WHERE phone ~ '^\d{10}$';
```

##### REGEXP_SPLIT_TO_TABLE

```sql
-- Diviser le texte en lignes
SELECT customer_id,
       REGEXP_SPLIT_TO_TABLE(name, '\s+') AS name_parts
FROM Customer;
```

##### REGEXP_SPLIT_TO_ARRAY

```sql
-- Diviser le texte en tableau
SELECT customer_id,
       name,
       REGEXP_SPLIT_TO_ARRAY(name, '\s+') AS name_array
FROM Customer;
```

##### REGEXP_MATCHES

```sql
-- Extraire toutes les correspondances (renvoie un tableau)
SELECT license_plate,
       REGEXP_MATCHES(license_plate, '([A-Z]+)(\d+)', 'g') AS plate_parts
FROM Car;

-- Extraire des groupes spécifiques
SELECT email,
       (REGEXP_MATCHES(email, '^([^@]+)@([^.]+)\.(.+)$'))[1] AS username,
       (REGEXP_MATCHES(email, '^([^@]+)@([^.]+)\.(.+)$'))[2] AS domain,
       (REGEXP_MATCHES(email, '^([^@]+)@([^.]+)\.(.+)$'))[3] AS tld
FROM Customer
WHERE email ~ '^[^@]+@[^.]+\..+$';
```

### Drapeaux regex PostgreSQL

```sql
-- 'i' - Insensible à la casse
SELECT *
FROM Customer
WHERE name ~* 'john|jane';
-- Équivalent à l'opérateur ~*

-- 'g' - Global (trouve toutes les correspondances, pas seulement la première)
SELECT REGEXP_REPLACE('abc123def456', '\d+', 'X', 'g');
-- Renvoie 'abcXdefX'

-- 'm' - Mode multiligne (^ et $ correspondent aux limites de ligne)
-- 's' - Le point correspond au saut de ligne
-- 'x' - Syntaxe étendue (ignore les espaces, autorise les commentaires)
-- 'n' - Correspondance sensible aux sauts de ligne
```

### Classes de caractères POSIX dans PostgreSQL

```sql
-- [:alnum:] - Caractères alphanumériques
SELECT *
FROM Customer
WHERE driver_license ~ '^[[:alnum:]]{8,}$';

-- [:alpha:] - Caractères alphabétiques
SELECT *
FROM Customer
WHERE name ~ '^[[:alpha:]\s]+$';

-- [:digit:] - Chiffres numériques
SELECT *
FROM Car
WHERE license_plate ~ '[[:digit:]]{3,}';

-- [:lower:] - Lettres minuscules
-- [:upper:] - Lettres majuscules  
-- [:space:] - Caractères d’espacement
-- [:punct:] - Caractères de ponctuation
```

---

## Comparaison entre systèmes de gestion de bases de données

### PostgreSQL

**Points forts :**

- Support complet des regex POSIX
- Ensemble riche de fonctions regex (REGEXP_REPLACE, REGEXP_MATCHES, etc.)
- Plusieurs opérateurs (~, ~*, !~, !~*)
- Support complet des drapeaux
- Fonctions de division en tableaux et tables

**Exemple :**

```sql
-- PostgreSQL
SELECT REGEXP_REPLACE(phone, '(\d{3})(\d{3})(\d{4})', '(\1) \2-\3')
FROM Customer
WHERE phone ~ '^\d{10}$';
```

### MySQL

**Points forts :**

- Opérateurs REGEXP/RLIKE
- Fonctions REGEXP_REPLACE, REGEXP_SUBSTR, REGEXP_INSTR (8.0+)
- Bon support regex de base

**Limitations :**

- Moins complet que PostgreSQL
- Moins de fonctions regex dans les anciennes versions
- Support limité des drapeaux

**Exemple :**

```sql
-- MySQL 8.0+
SELECT REGEXP_REPLACE(phone, '([0-9]{3})([0-9]{3})([0-9]{4})', '(\\1) \\2-\\3')
FROM Customer
WHERE phone REGEXP '^[0-9]{10}$';

-- Anciennes versions MySQL
SELECT *
FROM Customer
WHERE phone REGEXP '^[0-9]{10}$';
```

### SQL Server

**Points forts :**

- Correspondance de motifs de base avec LIKE et caractères génériques
- Certaines fonctionnalités regex via CLR dans les nouvelles versions

**Limitations :**

- Pas d’opérateurs regex natifs avant SQL Server 2022
- Fonctionnalités regex limitées comparées à PostgreSQL/MySQL

**Exemple :**

```sql
-- SQL Server 2022+
SELECT *
FROM Customer
WHERE phone LIKE '[0-9][0-9][0-9][0-9][0-9][0-9][0-9][0-9][0-9][0-9]';

-- Ou en utilisant le nouveau LIKE avec classes de caractères (limité)
SELECT *
FROM Customer
WHERE email LIKE '%_@_%.__%';
```

### Oracle

**Points forts :**

- Fonctions REGEXP_LIKE, REGEXP_REPLACE, REGEXP_SUBSTR, REGEXP_INSTR
- Support des regex POSIX
- Bonne performance

**Limitations :**

- Syntaxe différente de PostgreSQL
- Nommage des opérateurs moins intuitif

**Exemple :**

```sql
-- Oracle
SELECT REGEXP_REPLACE(phone, '(\d{3})(\d{3})(\d{4})', '(\1) \2-\3')
FROM Customer
WHERE REGEXP_LIKE(phone, '^\d{10}$');
```

### SQLite

**Points forts :**

- Opérateur REGEXP disponible avec extension
- Léger et rapide

**Limitations :**

- Pas de support regex intégré
- Nécessite de charger une extension ou des fonctions personnalisées

**Exemple :**

```sql
-- SQLite (avec extension regex chargée)
SELECT *
FROM Customer
WHERE phone REGEXP '^[0-9]{10}$';
```

### Tableau comparatif des fonctionnalités

| Fonctionnalité         | PostgreSQL | MySQL        | SQL Server       | Oracle          | SQLite       |
|------------------------|------------|--------------|------------------|-----------------|--------------|
| Correspondance de base | ✅ (~, ~*)  | ✅ (REGEXP)   | ❌ (jusqu’à 2022) | ✅ (REGEXP_LIKE) | ✅ (avec ext) |
| Insensible à la casse  | ✅ (~*)     | ✅ (drapeaux) | ❌                | ✅ (drapeaux)    | ✅            |
| Remplacement           | ✅          | ✅ (8.0+)     | ❌                | ✅               | ❌            |
| Extraction             | ✅          | ✅ (8.0+)     | ❌                | ✅               | ❌            |
| Division               | ✅          | ❌            | ❌                | ❌               | ❌            |
| Classes POSIX          | ✅          | ✅            | ❌                | ✅               | ✅            |
| Performance            | ✅          | ✅            | N/A              | ✅               | ✅            |

---

## Exemples pratiques avec la base de données de location de voitures

### Validation des données

#### Validation des emails

```sql
-- Validation complète des emails
SELECT customer_id,
       name,
       email,
       CASE
           WHEN email ~ '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$'
               THEN 'Valide'
           WHEN email IS NULL OR email = ''
               THEN 'Manquant'
           ELSE 'Invalide'
           END AS email_status
FROM Customer;

-- Trouver les clients avec des formats d'email invalides
SELECT *
FROM Customer
WHERE email IS NOT NULL
  AND email !~ '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$';
```

#### Validation des numéros de téléphone

```sql
-- Valider différents formats de numéros de téléphone
SELECT customer_id,
       name,
       phone,
       CASE
           WHEN phone ~ '^\(\d{3}\)\s\d{3}-\d{4}$' THEN 'Format : (123) 456-7890'
           WHEN phone ~ '^\d{3}-\d{3}-\d{4}$' THEN 'Format : 123-456-7890'
           WHEN phone ~ '^\d{10}$' THEN 'Format : 1234567890'
           WHEN phone ~ '^\+1\d{10}$' THEN 'Format : +11234567890'
           WHEN phone IS NULL OR phone = '' THEN 'Manquant'
           ELSE 'Format invalide'
           END AS phone_format
FROM Customer;
```

#### Validation des plaques d’immatriculation

```sql
-- Valider les formats de plaques (divers motifs)
SELECT license_plate,
       make,
       model,
       CASE
           WHEN license_plate ~ '^[A-Z]{3}\d{3}$' THEN 'Format État : ABC123'
           WHEN license_plate ~ '^\d{3}[A-Z]{3}$' THEN 'Format État : 123ABC'
           WHEN license_plate ~ '^[A-Z]{2}\d{4}$' THEN 'Format État : AB1234'
           WHEN license_plate ~ '^[A-Z]\d{2}[A-Z]\d{2}$' THEN 'Format État : A12B34'
           ELSE 'Format non standard'
           END AS plate_format,
       -- Extraire les composants
       CASE
           WHEN license_plate ~ '^([A-Z]+)(\d+)$' THEN
               (REGEXP_MATCHES(license_plate, '^([A-Z]+)(\d+)$'))[1]
           ELSE NULL
           END AS letter_part,
       CASE
           WHEN license_plate ~ '^([A-Z]+)(\d+)$' THEN
               (REGEXP_MATCHES(license_plate, '^([A-Z]+)(\d+)$'))[2]
           ELSE NULL
           END AS number_part
FROM Car;
```

### Nettoyage et standardisation des données

#### Standardisation des numéros de téléphone

```sql
-- Nettoyer et standardiser les numéros de téléphone
SELECT customer_id,
       phone   AS original_phone,
       CASE
           WHEN phone ~ '^\d{10}$' THEN
               REGEXP_REPLACE(phone, '(\d{3})(\d{3})(\d{4})', '(\1) \2-\3')
           WHEN phone ~ '^\d{3}-\d{3}-\d{4}$' THEN
               REGEXP_REPLACE(phone, '(\d{3})-(\d{3})-(\d{4})', '(\1) \2-\3')
           WHEN phone ~ '^\(\d{3}\)\s*\d{3}-\d{4}$' THEN
               phone -- Déjà dans le format préféré
           ELSE
               REGEXP_REPLACE(
                       REGEXP_REPLACE(phone, '[^0-9]', '', 'g'),
                       '(\d{3})(\d{3})(\d{4})',
                       '(\1) \2-\3'
               )
           END AS standardized_phone
FROM Customer
WHERE phone IS NOT NULL;
```

#### Analyse des domaines d’email

```sql
-- Extraire et analyser les domaines d’email
SELECT (REGEXP_MATCHES(email, '^[^@]+@([^.]+\..+)$'))[1] AS domain,
       COUNT(*)                                          AS customer_count,
       ARRAY_AGG(name)                                   AS customers
FROM Customer
WHERE email ~ '^[^@]+@[^.]+\..+$'
GROUP BY (REGEXP_MATCHES(email, '^[^@]+@([^.]+\..+)$'))[1]
ORDER BY customer_count DESC;
```

#### Traitement des noms

```sql
-- Diviser les noms de clients en composants
SELECT customer_id,
       name,
       CASE
           WHEN name ~ '^\s*(\S+)\s+(\S+)\s*$' THEN
               (REGEXP_MATCHES(name, '^\s*(\S+)\s+(\S+)\s*$'))[1]
           WHEN name ~ '^\s*(\S+)' THEN
               (REGEXP_MATCHES(name, '^\s*(\S+)'))[1]
           ELSE NULL
           END AS first_name,
       CASE
           WHEN name ~ '^\s*(\S+)\s+(\S+)\s*$' THEN
               (REGEXP_MATCHES(name, '^\s*(\S+)\s+(\S+)\s*$'))[2]
           ELSE NULL
           END AS last_name,
       CASE
           WHEN name ~ '^\s*(\S+)\s+(\S+)\s+(.+)$' THEN
               (REGEXP_MATCHES(name, '^\s*(\S+)\s+(\S+)\s+(.+)$'))[3]
           ELSE NULL
           END AS additional_names
FROM Customer;
```

### Recherche et filtrage

#### Recherche avancée de voitures

```sql
-- Recherche flexible de voitures avec regex
SELECT license_plate,
       make,
       model,
       year,
       color,
       daily_rate
FROM Car
WHERE
  -- Recherche flexible sur marque/modèle
    (make ~* 'toy|honda|ford' OR model ~* 'cam|acc|civic')
  AND
  -- Plage d’années avec regex
    CAST(year AS TEXT) ~ '^20(1[5-9]|2[0-5])$' -- 2015-2025
  AND
  -- Correspondance de couleurs (gère les variations)
    color ~* '^(red|blue|black|white|silver|gr[ae]y)';
```

#### Analyse des journaux de maintenance

```sql
-- Analyser les descriptions de maintenance pour détecter des motifs
SELECT maintenance_id,
       service_type,
       description,
       CASE
           WHEN description ~* 'oil.*change|change.*oil' THEN 'Changement d’huile'
           WHEN description ~* 'tire|wheel' THEN 'Service pneus'
           WHEN description ~* 'brake|pad' THEN 'Service freins'
           WHEN description ~* 'engine|motor' THEN 'Moteur'
           WHEN description ~* 'transmission|trans' THEN 'Transmission'
           WHEN description ~* 'battery|electrical' THEN 'Électrique'
           ELSE 'Autre'
           END AS service_category,
       -- Extraire les mentions de coût dans la description
       CASE
           WHEN description ~ '\$([0-9,]+(?:\.[0-9]{2})?)' THEN
               (REGEXP_MATCHES(description, '\$([0-9,]+(?:\.[0-9]{2})?)'))[1]
           ELSE NULL
           END AS mentioned_cost
FROM MaintenanceRecord
WHERE description IS NOT NULL;
```

---

## Modèles et techniques avancés

### Lookahead et Lookbehind (non supportés par PostgreSQL, mais présentation du concept)

```sql
-- PostgreSQL ne supporte pas lookahead/lookbehind, mais voici comment contourner

-- Objectif : Trouver les emails avec un domaine spécifique mais pas les sous-domaines
-- Au lieu de : email ~ '^[^@]+@(?!mail\.)example\.com$'  -- Non supporté
-- Utiliser : Conditions multiples
SELECT *
FROM Customer
WHERE email ~ '^[^@]+@example\.com$'
  AND email !~ '^[^@]+@mail\.example\.com$';
```

### Motifs de validation complexes

#### Validation de numéros de permis de conduire

```sql
-- Valider les numéros de permis (formats variés selon les États)
SELECT customer_id,
       driver_license,
       CASE
           -- Format Californie : 1 lettre + 7 chiffres
           WHEN driver_license ~ '^[A-Z]\d{7}$' THEN 'Format CA'
           -- Format New York : 3 lettres + 6 chiffres  
           WHEN driver_license ~ '^[A-Z]{3}\d{6}$' THEN 'Format NY'
           -- Format Texas : 8 chiffres
           WHEN driver_license ~ '^\d{8}$' THEN 'Format TX'
           -- Format Floride : 1 lettre + 12 chiffres
           WHEN driver_license ~ '^[A-Z]\d{12}$' THEN 'Format FL'
           ELSE 'Format inconnu'
           END                AS license_format,
       LENGTH(driver_license) AS license_length
FROM Customer
WHERE driver_license IS NOT NULL;
```

#### Validation de numéros VIN (si nous en avions)

```sql
-- Exemple de validation de VIN (17 caractères, motif spécifique)
-- Ceci serait dans une table hypothétique vehicle_details
/*
SELECT 
    vin,
    CASE 
        WHEN vin ~ '^[A-HJ-NPR-Z0-9]{17}$' THEN 'Format VIN valide'
        ELSE 'Format VIN invalide'
    END AS vin_status,
    -- Extraire le code fabricant (3 premiers caractères)
    LEFT(vin, 3) AS manufacturer_code,
    -- Extraire l’année du modèle (10e caractère)
    SUBSTRING(vin, 10, 1) AS model_year_code
FROM vehicle_details;
*/
```

### Exploration et analyse de texte

#### Analyse des descriptions

```sql
-- Analyser les descriptions de rapports de dommages pour identifier les problèmes courants
WITH damage_analysis AS (SELECT report_id,
                                damage_description,
                                -- Extraire les indicateurs de gravité
                                CASE
                                    WHEN damage_description ~* 'severe|major|extensive|total' THEN 'Élevée'
                                    WHEN damage_description ~* 'minor|small|light|slight' THEN 'Faible'
                                    WHEN damage_description ~* 'moderate|medium' THEN 'Moyenne'
                                    ELSE 'Inconnue'
                                    END AS severity,
                                -- Extraire les parties du véhicule mentionnées
                                ARRAY(
                                        SELECT unnest(
                                                       REGEXP_SPLIT_TO_ARRAY(
                                                               LOWER(damage_description),
                                                               '[^a-z]+'
                                                       )
                                               )
                                        WHERE unnest(
                                                      REGEXP_SPLIT_TO_ARRAY(
                                                              LOWER(damage_description),
                                                              '[^a-z]+'
                                                      )
                                              ) ~ '^(door|bumper|hood|trunk|window|tire|wheel|mirror|light)s?$'
                                )       AS affected_parts
                         FROM DamageReport
                         WHERE damage_description IS NOT NULL)
SELECT severity,
       COUNT(*)                                   AS report_count,
       ARRAY_AGG(DISTINCT unnest(affected_parts)) AS common_parts
FROM damage_analysis
GROUP BY severity;
```

### Modèles d’extraction de données

#### Extraction de données structurées depuis du texte

```sql
-- Extraire les informations de kilométrage des descriptions de maintenance
SELECT maintenance_id,
       description,
       -- Extraire les mentions de kilométrage
       CASE
           WHEN description ~ '(\d{1,6})\s*(mile|mi|k|km)' THEN
               (REGEXP_MATCHES(description, '(\d{1,6})\s*(mile|mi|k|km)', 'i'))[1]
           ELSE NULL
           END AS extracted_mileage,
       -- Extraire les dates mentionnées dans le texte
       CASE
           WHEN description ~ '(\d{1,2})[/-](\d{1,2})[/-](\d{2,4})' THEN
               REGEXP_MATCHES(description, '(\d{1,2})[/-](\d{1,2})[/-](\d{2,4})')
           ELSE NULL
           END AS mentioned_date_parts
FROM MaintenanceRecord
WHERE description IS NOT NULL;
```

---

## Considérations de performance

### Stratégies d’indexation

#### Index partiels avec regex

```sql
-- Créer un index pour les motifs fréquemment recherchés
CREATE INDEX idx_customer_email_domain
    ON Customer ((REGEXP_MATCHES(email, '^[^@]+@([^.]+\..+)$'))[1])
    WHERE email ~ '^[^@]+@[^.]+\..+$';

-- Index pour les recherches de numéros de téléphone (après standardisation)
CREATE INDEX idx_customer_phone_digits
    ON Customer (REGEXP_REPLACE(phone, '[^0-9]', '', 'g'))
    WHERE phone IS NOT NULL;
```

#### Index fonctionnels

```sql
-- Index sur les données nettoyées/standardisées
CREATE INDEX idx_car_plate_clean
    ON Car (UPPER(REGEXP_REPLACE(license_plate, '[^A-Z0-9]', '', 'g')));
```

### Conseils d’optimisation des requêtes

1. **Utilisez d’abord des motifs simples**

```sql
-- Bon : Filtrer d’abord avec des conditions simples
SELECT *
FROM Customer
WHERE email LIKE '%@gmail.com' -- Filtre rapide
  AND email ~ '^[a-zA-Z0-9._%+-]+@gmail\.com$';
-- Regex précis

-- À éviter : Commencer par un regex complexe sur de grandes tables
```

2. **Limiter la portée du regex**

```sql
-- Bon : Limiter d’abord le jeu de données
SELECT *
FROM Customer
WHERE LENGTH(phone) = 10 -- Comparaison numérique rapide
  AND phone ~ '^\d{10}$';
-- Regex sur un sous-ensemble plus petit

-- Bon : Utiliser des clauses WHERE pour limiter l’application du regex
SELECT *
FROM MaintenanceRecord
WHERE service_type = 'Changement d’huile' -- Filtre compatible avec les index
  AND description ~* 'synthetic|conventional'; -- Regex sur un sous-ensemble
```

3. **Motifs compilés**
   Dans le code applicatif, compilez les motifs regex une fois et réutilisez-les plutôt que de les compiler pour chaque
   requête.

### Pièges courants de performance

1. **Évitez les caractères génériques en début de regex**

```sql
-- Lent : Motif qui ne peut pas utiliser efficacement les index
SELECT *
FROM Customer
WHERE name ~ '.*john.*';

-- Mieux : Motif plus spécifique
SELECT *
FROM Customer
WHERE name ~* '^john|john$|\sjohn\s';
```

2. **Soyez précis avec les quantificateurs**

```sql
-- Lent : Correspondance gourmande
SELECT *
FROM Description
WHERE text ~ '.*error.*details.*';

-- Mieux : Plus spécifique
SELECT *
FROM Description
WHERE text ~ 'error.{1,50}details';
```

---

## Résumé

Les expressions régulières en SQL, notamment dans PostgreSQL, offrent des capacités puissantes pour :

- **Validation des données** – Garantir la qualité et la conformité des formats
- **Traitement de texte** – Nettoyer, standardiser et transformer les données textuelles
- **Correspondance de motifs** – Trouver des motifs complexes que LIKE ne peut pas gérer
- **Extraction de données** – Extraire des informations spécifiques depuis du texte non structuré
- **Amélioration des recherches** – Offrir des capacités de recherche flexibles et précises

**Points clés à retenir :**

1. PostgreSQL propose le support regex SQL le plus complet
2. Pensez toujours aux implications de performance et utilisez judicieusement les index
3. Combinez les regex avec les filtres SQL traditionnels pour une performance optimale
4. Utilisez les regex pour la qualité des données et la validation dans les processus ETL
5. Documentez les motifs complexes pour faciliter la maintenance

**Bonnes pratiques :**

- Commencez par des motifs simples et ajoutez de la complexité au besoin
- Testez soigneusement les motifs regex avec des cas limites
- Utilisez les regex pour la validation mais stockez les données nettoyées séparément
- Envisagez de créer des fonctions utilitaires pour les motifs couramment utilisés
- Équilibrez la complexité des regex avec la performance des requêtes

Les expressions régulières transforment SQL d’un simple langage de requête en un outil puissant de traitement de texte,
permettant des analyses et manipulations de données sophistiquées impossibles avec le SQL standard seul.


-------

??? info "Utilisation de l'IA"
      Page rédigée en partie avec l'aide d'un assistant IA, principalement à l'aide de Perplexity AI. L'IA a été 
      utilisée pour générer des explications, des exemples et/ou des suggestions de structure. Toutes les informations 
      ont été vérifiées, éditées et complétées par l'auteur.