# Introduction à PostgreSQL

## Historique de PostgreSQL

PostgreSQL, souvent appelé Postgres, est un système de gestion de base de
données relationnelle et objet-relationnelle
open-source. Son développement a commencé en 1986 à l'Université de Californie,
Berkeley, sous le projet POSTGRES dirigé
par le professeur Michael Stonebraker. Le projet a évolué pour devenir
PostgreSQL en 1996, avec l'ajout du support SQL.

## Modèle de données utilisé

PostgreSQL utilise un modèle de données relationnel et objet-relationnel. Cela
signifie qu'il supporte non seulement les
fonctionnalités des bases de données relationnelles traditionnelles, comme les
tables, les requêtes SQL et les
transactions, mais aussi des fonctionnalités avancées comme les types de données
personnalisés, les objets, et les
héritages de tables.

### Comparatif des modèles relationnel et objet-relationnel

| Caractéristique           | Modèle Relationnel                             | Modèle Objet-Relationnel                                             |
|---------------------------|------------------------------------------------|----------------------------------------------------------------------|
| **Structure des données** | Tables et colonnes                             | Tables, colonnes, types de données personnalisés, héritage de tables |
| **Langage de requête**    | SQL standard                                   | SQL avec extensions pour les objets                                  |
| **Types de données**      | Types de données standard (INT, VARCHAR, etc.) | Types de données standard + types personnalisés                      |
| **Support des objets**    | Non                                            | Oui                                                                  |
| **Héritage de tables**    | Non                                            | Oui                                                                  |

## Importance et support des grandes compagnies

PostgreSQL est largement utilisé par de nombreuses grandes entreprises et
organisations à travers le monde, notamment :

- **Apple**
- **Cisco**
- **Red Hat**
- **Debian**
- **Instagram**
- **Spotify**

Ces entreprises choisissent PostgreSQL pour sa robustesse, sa flexibilité et son
adhésion aux standards SQL.

## Particularités de PostgreSQL comparé aux autres SGBD

1. **Extensibilité** : PostgreSQL permet aux utilisateurs de définir leurs
   propres types de données, opérateurs et
   fonctions.
2. **Support des transactions ACID** : Assure la fiabilité des transactions avec
   des propriétés d'atomicité, de
   cohérence, d'isolation et de durabilité.
3. **Conformité aux standards SQL** : PostgreSQL est très conforme aux standards
   SQL, ce qui facilite la portabilité des
   applications.
4. **Support des requêtes complexes** : PostgreSQL supporte les jointures
   complexes, les sous-requêtes, les vues
   matérialisées, et plus encore.
5. **Indexation avancée** : Offre plusieurs types d'index (B-tree, Hash, GiST,
   SP-GiST, GIN, BRIN) pour améliorer les
   performances des requêtes.
6. **Sécurité** : Inclut des fonctionnalités de sécurité avancées comme
   l'authentification SSL, le chiffrement des
   données et le contrôle d'accès basé sur les rôles.

## Licence

PostgreSQL est distribué sous la licence PostgreSQL, une licence open-source
permissive similaire à la licence MIT. Cela
permet une utilisation, une modification et une distribution libres, y compris
pour des applications commerciales.

## Téléchargement et installation

### Liens pour le téléchargement

- [Site officiel de PostgreSQL](https://www.postgresql.org/download/)

### Guides d'installation

- [Guide d'installation pour Windows](https://www.postgresqltutorial.com/postgresql-getting-started/install-postgresql/)
- [Guide d'installation pour macOS](https://www.postgresqltutorial.com/postgresql-getting-started/install-postgresql-macos/)
- [Guide d'installation pour Linux](https://www.postgresqltutorial.com/postgresql-getting-started/install-postgresql-linux/)

## Conclusion

PostgreSQL est un SGBD puissant et flexible, adapté à une large gamme
d'applications. Son modèle de données
objet-relationnel, sa conformité aux standards SQL, et ses fonctionnalités
avancées en font un choix privilégié pour de
nombreuses entreprises et développeurs. Sa licence open-source et son large
support communautaire contribuent également
à sa popularité et à son adoption continue.


-------

??? info "Utilisation de l'IA"
    Page rédigée en partie avec l'aide d'un assistant IA. L'IA a été utilisée pour générer des 
    explications, des exemples et/ou des suggestions de structure. Toutes les informations ont 
    été vérifiées, éditées et complétées par l'auteur.