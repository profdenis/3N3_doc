# 1 - Introduction à SQL

SQL (Structured Query Language) est un langage de programmation standardisé
utilisé pour gérer et manipuler des bases de données relationnelles. Il permet
d'effectuer diverses opérations telles que la création, la modification, la
suppression et la récupération de données dans une base de données. SQL est
essentiel pour interagir avec les systèmes de gestion de bases de données
relationnelles (SGBDR) comme PostgreSQL, MySQL, Oracle, et SQL Server.

## Historique de SQL

SQL a été développé dans les années 1970 par IBM dans le cadre du projet System
R, qui visait à démontrer la faisabilité des bases de données relationnelles. Le
langage a été initialement appelé SEQUEL (Structured English Query Language)
avant d'être renommé SQL. En 1986, l'American National Standards Institute 
(ANSI) a publié la première norme SQL, suivie par l'International Organization
for Standardization (ISO) en 1987. Depuis lors, SQL a évolué avec plusieurs
révisions et extensions pour répondre aux besoins croissants des bases de
données modernes.

## Survol des Standards ISO

Les standards ISO pour SQL définissent les spécifications et les fonctionnalités
du langage afin d'assurer l'interopérabilité entre différents systèmes de bases
de données. Voici quelques-unes des versions clés :

- **SQL-86** : La première norme SQL publiée par ANSI en 1986 et adoptée par ISO
  en 1987.
- **SQL-89** : Une révision mineure de SQL-86, introduisant quelques
  améliorations et corrections.
- **SQL-92** : Une version majeure qui a ajouté de nombreuses fonctionnalités, y
  compris les sous-requêtes, les jointures externes et les contraintes
  d'intégrité.
- **SQL:1999 (SQL3)** : Introduction des types de données utilisateur définis,
  des procédures stockées, et du langage de requête récursif.
- **SQL:2003** : Ajout des fonctions XML, des tables dérivées et des types de
  données fenêtrés.
- **SQL:2008** : Améliorations des fonctionnalités existantes et ajout de
  nouvelles, telles que les expressions de table communes (CTE).
- **SQL:2011** : Introduction des fonctionnalités de traitement des données
  temporelles.
- **SQL:2016** : Ajout de fonctionnalités JSON et de nouvelles extensions pour
  les types de données géospatiales.

## Principes d'un Langage Non-Procedural

SQL est un langage non-procédural, ce qui signifie que l'utilisateur spécifie
**ce qu'il veut** obtenir sans détailler **comment** le système doit l'obtenir.
Voici quelques principes clés :

1. **Déclaration des Intentions** : En SQL, vous déclarez vos intentions en
   termes de résultats souhaités. Par exemple, une requête SELECT indique
   quelles colonnes et quelles lignes vous voulez récupérer, mais pas comment
   les récupérer.

2. **Abstraction de l'Implémentation** : Les détails de l'implémentation, comme
   les algorithmes de tri ou les chemins d'accès aux données, sont gérés par le
   SGBDR. L'utilisateur n'a pas besoin de connaître ces détails.

3. **Optimisation Automatique** : Les SGBDR optimisent automatiquement les
   requêtes pour améliorer les performances. L'utilisateur se concentre sur la
   logique de la requête plutôt que sur l'optimisation.

4. **Facilité d'Utilisation** : Le langage non-procédural est généralement plus
   facile à apprendre et à utiliser pour les utilisateurs finaux, car il se
   concentre sur les résultats plutôt que sur les processus.

5. **Portabilité** : Les requêtes SQL sont portables entre différents systèmes
   de bases de données, tant qu'elles respectent les standards ISO, ce qui
   facilite la migration et l'interopérabilité.

## Exemple de Requête SQL

```sql
SELECT Nom, Prénom, Âge
FROM Étudiants
WHERE Département = 'Informatique';
```

Cette requête sélectionne les colonnes Nom, Prénom et Âge de la table Étudiants
pour les étudiants appartenant au département d'Informatique, sans spécifier
comment le SGBDR doit accéder aux données ou les trier.

En résumé, SQL est un langage puissant et flexible pour interagir avec les bases
de données relationnelles, offrant une abstraction qui permet aux utilisateurs
de se concentrer sur les résultats souhaités plutôt que sur les détails
d'implémentation.


-------

??? info "Utilisation de l'IA"
    Page rédigée en partie avec l'aide d'un assistant IA. L'IA a été utilisée pour générer des 
    explications, des exemples et/ou des suggestions de structure. Toutes les informations ont 
    été vérifiées, éditées et complétées par l'auteur.