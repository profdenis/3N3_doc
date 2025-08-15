# Configuration de PostgreSQL

## Pour Windows/Mac

1. Allez
   sur [Téléchargement PostgreSQL](https://www.enterprisedb.com/downloads/postgres-postgresql-downloads)
   pour télécharger le SGBD PostgreSQL.
2. Allez
   sur [Installation de PostgreSQL](https://www.enterprisedb.com/docs/supported-open-source/postgresql/installer/02_installing_postgresql_with_the_graphical_installation_wizard/01_invoking_the_graphical_installer/)
   et suivez les instructions pour l'installer.
3. Cela installera le serveur de base de données et quelques autres outils pour
   gérer les bases de données.
    - Le plus important pour l'instant, la commande `psql` et l'application
      *PgAdmin* seront installés en même temps que le serveur.
4. N'oubliez pas de saisir un mot de passe pour l'utilisateur postgres.
    - Si cela ne demande pas de mot de passe, il devra être défini depuis la
      ligne de commande.
    - Vous devrez utiliser la commande `psql` pour changer le mot de passe de
      l'utilisateur `postgres`.
    - Sinon, il faudra recommencer l'installation.

### Pour Linux

1. Suivez ce guide pour installer PostgreSQL sur
   Ubuntu : [PostgreSQL sur Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-postgresql-on-ubuntu-20-04-quickstart).
    - La procédure est très similaire pour d'autres distributions de Linux.
2. Installez *PgAdmin* depuis [ici](https://www.pgadmin.org/download/), ou par
   le gestionnaire de paquets de votre distribution.


-------

??? info "Utilisation de l'IA"
    Page rédigée en partie avec l'aide d'un assistant IA. L'IA a été utilisée pour générer des 
    explications, des exemples et/ou des suggestions de structure. Toutes les informations ont 
    été vérifiées, éditées et complétées par l'auteur.