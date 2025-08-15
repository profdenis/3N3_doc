# Présentation

Le modèle relationnel est un cadre conceptuel pour organiser et gérer des
données sous forme de tables, également appelées relations. Ce modèle a été
introduit par Edgar F. Codd en 1970 et est devenu la base des systèmes de
gestion de bases de données relationnelles (SGBDR), tels que PostgreSQL. Dans ce
modèle, les données sont représentées par des tuples (lignes) dans des
relations (tables), et les opérations sur les données sont effectuées à l'aide
d'un langage de requête structuré comme SQL (Structured Query Language).

## Lexique

1. **Relation (Table)** : Une relation est une table à deux dimensions composée
   de lignes et de colonnes. Chaque table représente une entité ou un concept du
   monde réel.

2. **Attribut (Colonne)** : Un attribut est une colonne dans une table. Chaque
   attribut a un nom et un type de données (par exemple, entier, texte, date).

3. **Tuple (Ligne)** : Un tuple est une ligne dans une table. Chaque tuple
   représente un enregistrement unique d'une entité.

4. **Domaine** : Un domaine est l'ensemble des valeurs possibles qu'un attribut
   peut prendre. Par exemple, le domaine d'un attribut "âge" pourrait être les
   entiers de 0 à 120.

5. **Clé Primaire** : Une clé primaire est un attribut ou un ensemble
   d'attributs qui identifie de manière unique chaque tuple dans une relation.
   Par exemple, un numéro de sécurité sociale peut être une clé primaire pour
   une table de personnes.

6. **Clé Étrangère** : Une clé étrangère est un attribut ou un ensemble
   d'attributs dans une table qui fait référence à la clé primaire d'une autre
   table. Cela crée une relation entre les deux tables.

7. **Schéma de Relation** : Le schéma de relation est la structure ou la
   définition d'une relation, incluant les noms des attributs et leurs types de
   données.

8. **Intégrité Référentielle** : L'intégrité référentielle est une contrainte
   qui assure que les valeurs des clés étrangères correspondent à des valeurs
   existantes de clés primaires dans les tables référencées.

## Exemples

### Exemple 1 : Table des Étudiants

| ID_Étudiant | Nom    | Prénom | Âge | Département   |
|-------------|--------|--------|-----|---------------|
| 1           | Dupont | Jean   | 20  | Informatique  |
| 2           | Martin | Sophie | 22  | Mathématiques |
| 3           | Durand | Pierre | 21  | Physique      |

- **Relation** : `Étudiants`
- **Attributs** : `ID_Étudiant`, `Nom`, `Prénom`, `Âge`, `Département`
- **Clé Primaire** : `ID_Étudiant`

### Exemple 2 : Table des Cours

| Code_Cours | Nom_Cours         | Crédit |
|------------|-------------------|--------|
| CS101      | Introduction à CS | 3      |
| MA101      | Calcul I          | 4      |
| PH101      | Physique Générale | 4      |

- **Relation** : `Cours`
- **Attributs** : `Code_Cours`, `Nom_Cours`, `Crédit`
- **Clé Primaire** : `Code_Cours`

### Exemple 3 : Table des Inscriptions

| ID_Étudiant | Code_Cours |
|-------------|------------|
| 1           | CS101      |
| 2           | MA101      |
| 3           | PH101      |
| 1           | MA101      |

- **Relation** : `Inscriptions`
- **Attributs** : `ID_Étudiant`, `Code_Cours`
- **Clé Primaire** : `(ID_Étudiant, Code_Cours)`
- **Clé Étrangère** : `ID_Étudiant` réfère
  à `Étudiants(ID_Étudiant)`, `Code_Cours` réfère à `Cours(Code_Cours)`

Ces exemples montrent comment les données peuvent être organisées et
interconnectées dans le modèle relationnel, facilitant ainsi la gestion et
l'interrogation des informations de manière efficace et cohérente.


-------

??? info "Utilisation de l'IA"
    Page rédigée en partie avec l'aide d'un assistant IA. L'IA a été utilisée pour générer des 
    explications, des exemples et/ou des suggestions de structure. Toutes les informations ont 
    été vérifiées, éditées et complétées par l'auteur.