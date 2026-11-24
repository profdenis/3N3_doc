# Modélisation Entité-Association

- *Modèle de données relationnel* (ou *modèle relationnel*)
- *Modèle Entité-Association* (ou *Modèle EA*)
- Démarche de modélisation (Conceptuel, Logique, Physique)
- Exemples de notations

!!! warning "Précision terminologique : « Relation » vs « Association » (*Relation* vs *Relationship*)"
    Dans le langage courant et dans la documentation traduite de l'anglais, il existe une confusion très fréquente entre les mots **relation** et **association**. Cette confusion provient des faux-amis entre le français et l'anglais :

    * **En anglais** :
        * ***Relation*** désigne formellement une **relation mathématique**, c'est-à-dire une **table** dans le modèle relationnel inventé par Edgar F. Codd.
        * ***Relationship*** désigne le **lien conceptuel ou métier** unissant deux entités.
    * **En français** :
        * On traduit ***relation*** par **relation** : dans les bases de données relationnelles, une *relation* est une **table** (un ensemble de lignes ou tuples avec des colonnes ou attributs).
        * On traduit ***relationship*** par **association** : le rapport existant entre deux entités (par exemple : un `Client` *passe* une `Commande`).

    C'est pour cette raison qu'il faut impérativement parler de **Modèle Entité-Association** (ou **Diagramme Entité-Association**, en abrégé **DEA**) et **non pas** de *modèle entité-relation* :
    
    * Dire *entité-relation* entretiendrait la confusion entre la table (relation au sens de Codd) et le lien métier (association).
    * Une **entité** modélise un objet du monde réel (ex. : `Étudiant`, `Cours`).
    * Une **association** modélise le lien sémantique entre deux entités (ex. : `suit`, `enseigne`).
    * Une **relation** (au sens relationnel) est la structure de données finale (la table SQL avec ses n-uplets).

## Modèle Relationnel

- **Relation (table)** : table à deux dimensions, ensemble (et non liste) de
  tuples
- **Attributs** : colonnes de la table, champs
- **Schéma** : `Movies(title, year, length, filmType)`
- **Tuples** : lignes dans la table, enregistrements
- **Domaines** : types

### Exemple de Table (Base de Données `Monde`)

#### Table : **country** (données d'exemple, triées par nom de pays)

![example table](../images/relational_table.png)

### Terminologie

- **Entité** : Quelque chose d'intérêt pour la communauté des utilisateurs de la
  base de données.
    - clients, pièces, localisations géographiques
- **Colonne** : Un élément de données individuel stocké dans une table.
- **Ligne** : Un ensemble de colonnes qui, ensemble, décrivent complètement une
  entité ou une action sur une entité. Aussi appelé un enregistrement.
- **Table** : Un ensemble de lignes, conservées soit en mémoire (non
  persistante) soit sur un stockage permanent (persistant).
- **Ensemble de résultats** : Un autre nom pour une table non persistante,
  généralement le résultat d'une requête SQL.
- **Clé primaire** : Une ou plusieurs colonnes pouvant être utilisées comme
  identifiant unique pour chaque ligne d'une table.
- **Clé étrangère** : Une ou plusieurs colonnes pouvant être utilisées ensemble
  pour identifier une ligne unique dans une autre table.

## Étapes pour créer une nouvelle base de données

1. **Modèle Conceptuel de Données (MCD)** : diagramme de travail (ou brouillon)
    - permet d'esquisser rapidement le domaine métier avec les parties prenantes
    - se concentre uniquement sur les grandes entités et les associations
    - ne contient ni types de données détaillés, ni clés étrangères

2. **Modèle Logique de Données (MLD / DEA final)** :
    - version finale, complète et validée du **Diagramme Entité-Association (DEA)**
    - définit précisément les entités, leurs attributs et leurs identifiants
    - conserve des entités (pas de tables) et des associations directes, y compris les associations plusieurs-à-plusieurs ($N:M$)
    - **ne contient aucune clé étrangère** (la clé étrangère n'apparaît qu'au niveau physique)
    - peut être représenté selon la notation traditionnelle du DEA ou selon la notation **UML** (voir [notations](02-notation.md))

3. **Modèle Physique de Données (MPD)** :
    - conversion du modèle logique en modèle de données relationnel
    - est composé de **tables** (et non plus d'entités) et de colonnes
    - chaque association plusieurs-à-plusieurs devient une table d'association (table de jonction)
    - les associations sont traduites par des **clés primaires (`PK`)** et des **clés étrangères (`FK`)**

4. **Création de la Base de Données (SQL)** :
    - génération des instructions SQL `CREATE TABLE` avec types de données SQL et contraintes d'intégrité référentielle
    - respect de l'ordre de création des tables (tri topologique selon les dépendances de clés étrangères)
    - remplissage initial et exploitation (requêtes `SELECT`, mises à jour, ...)

### Modèle Entité-Association (EA)

#### Notation Traditionnelle

- **Ensembles d'entités** : rectangles
- **Ensembles d'associations** : losanges
- **Attributs** : ovales
- **Flèches** pour connecter les ensembles d'associations aux ensembles
  d'entités :
    - *triangle noir plein* : au plus 1
    - *triangle rond ouvert* ou *triangle transparent* : exactement 1
    - pas de flèches : plusieurs
- **Clés primaires** : noms d'attributs soulignés

#### Notation UML

[notation.md](02-notation.md)

- **Entités** : rectangles
- **Associations** : lignes entre les entités, éventuellement avec des étiquettes
  et des flèches directionnelles à côté des étiquettes
- **Attributs** : dans les rectangles des entités, sous les noms des entités
- **Cardinalités** pour connecter les associations aux entités :
    - *au plus 1* : `0..1`
    - *exactement 1* : `1` ou `1..1`
    - *plusieurs* : `*`
    - *au moins 1* : `1..*`
- **Clés primaires** : attributs avec la contrainte `<<pk>>`
- **Clés étrangères** (dans les diagrammes physiques uniquement) : attributs
  avec la contrainte `<<fk>>`
- **Contrainte de non-nullité** : cercle avant le nom de l'attribut

#### Base de Données de Contacts

#### Notation Traditionnelle

![contacts original notation](../images/contacts_orig.png)

#### Autre Notation Traditionnelle

![contacts alternative notation](../images/contacts_alt.png)

#### Notation UML

![contacts_uml.png](../images/contacts_uml.png)

