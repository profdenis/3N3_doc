# Guide complet des sauvegardes de données

## Introduction : Pourquoi les sauvegardes de données sont essentielles

La sauvegarde de données consiste à créer des copies de vos données afin de les protéger contre les pertes causées par
diverses menaces, notamment les pannes matérielles, les erreurs humaines, les cyberattaques, les catastrophes naturelles
et la corruption logicielle. Selon les statistiques du secteur, 60 % des entreprises qui perdent leurs données cessent
leurs activités dans les six mois suivants, ce qui rend les stratégies de sauvegarde indispensables pour toute
organisation manipulant des données importantes.

**Règle fondamentale** : Si vos données n’existent pas dans au moins trois endroits différents, elles n’existent pas
vraiment.

## Types de sauvegardes

### 1. Sauvegardes complètes (Full Backups)

- **Définition** : Copie intégrale de toutes les données
- **Avantages** : Restauration la plus rapide, protection complète des données
- **Inconvénients** : Besoin d’espace de stockage important, durée de sauvegarde la plus longue
- **Quand l’utiliser** : Hebdomadairement ou mensuellement, comme référence de base

### 2. Sauvegardes incrémentielles (Incremental Backups)

- **Définition** : Seules les données modifiées depuis la dernière sauvegarde (de tout type) sont sauvegardées
- **Avantages** : Sauvegarde très rapide, consommation minimale d’espace
- **Inconvénients** : Restauration plus lente (nécessite la sauvegarde complète + toutes les sauvegardes
  incrémentielles)
- **Quand l’utiliser** : Sauvegardes quotidiennes entre deux sauvegardes complètes

### 3. Sauvegardes différentielles (Differential Backups)

- **Définition** : Toutes les données modifiées depuis la dernière sauvegarde complète
- **Avantages** : Restauration plus rapide qu’avec les sauvegardes incrémentielles (seule la sauvegarde complète + la
  dernière différentielle sont nécessaires)
- **Inconvénients** : Taille croissante entre deux sauvegardes complètes
- **Quand l’utiliser** : Bon compromis pour des sauvegardes hebdomadaires

### 4. Sauvegardes miroir / instantanés (Mirror/Snapshot Backups)

- **Définition** : Copie exacte des données à un instant précis
- **Avantages** : Restauration immédiate, efficacité d’espace grâce à la technique « copy-on-write »
- **Inconvénients** : Vulnérabilité aux mêmes catastrophes que les données d’origine
- **Quand l’utiliser** : Réplication en temps réel et scénarios nécessitant une restauration rapide

### 5. Réplication en temps réel (Real-time Replication)

- **Définition** : Synchronisation continue des modifications vers une ou plusieurs bases de données répliquées dès
  qu’elles surviennent
- **Fonctionnement** : Les modifications sont transmises immédiatement (ou presque) vers des serveurs de secours
- **Types** :
    - **Synchrone** : Le serveur principal attend la confirmation de la réplique avant de valider la transaction (plus
      lent mais garantit la cohérence)
    - **Asynchrone** : Le serveur principal valide immédiatement, les modifications sont répliquées ensuite (plus rapide
      mais risque mineur de perte de données)
- **Avantages** : Perte de données minimale (RPO proche de zéro), basculement rapide en cas de panne
- **Inconvénients** : Dépendance au réseau, complexité, impact potentiel sur les performances
- **Quand l’utiliser** : Systèmes haute disponibilité, reprise après sinistre, répartition de charge

## Stratégies et planification des sauvegardes

### La règle 3-2-1

- **3** copies des données importantes
- **2** types de supports de stockage différents
- **1** copie stockée hors site

### Fréquence recommandée des sauvegardes

- **Données critiques de l’entreprise** : Réplication en temps réel (pour la disponibilité) + sauvegardes quotidiennes (
  pour la récupération)
- **Bases de données importantes** : Sauvegardes quotidiennes complètes ou incrémentielles
- **Documents utilisateurs** : Sauvegardes quotidiennes ou hebdomadaires
- **Configurations système** : Sauvegardes hebdomadaires
- **Données archivées** : Sauvegardes mensuelles ou trimestrielles

**Remarque** : La réplication en temps réel assure une haute disponibilité mais ne remplace pas les sauvegardes
traditionnelles. Elle protège contre les pannes matérielles, mais pas contre la corruption des données, les erreurs
humaines ou les catastrophes affectant à la fois le serveur principal et sa réplique.

### Politiques de rétention


Exemple de calendrier de rétention :
- Sauvegardes quotidiennes : conserver 1 mois
- Sauvegardes hebdomadaires : conserver 3 mois  
- Sauvegardes mensuelles : conserver 1 an
- Sauvegardes annuelles : conserver 7 ans (ou selon les exigences réglementaires)


## Bonnes pratiques générales

### Considérations liées au stockage

- **Stockage local** : Restauration rapide, vulnérable aux catastrophes locales
- **Stockage réseau** : Ressources partagées, dépendance au réseau
- **Stockage cloud** : Protection hors site, évolutif, coûts récurrents
- **Stockage sur bande** : Archivage à long terme, accès plus lent

### Mesures de sécurité

- **Chiffrement** : Chiffrer les sauvegardes à la fois en transit et au repos
- **Contrôle d’accès** : Limiter les personnes autorisées à créer, modifier ou supprimer des sauvegardes
- **Gestion des versions** : Conserver plusieurs versions pour se prémunir contre les sauvegardes corrompues
- **Sauvegardes « air-gapped »** : Maintenir certaines sauvegardes complètement déconnectées des réseaux

### Automatisation et surveillance

- Automatiser les processus de sauvegarde pour réduire les erreurs humaines
- Surveiller l’exécution des sauvegardes et vérifier leur intégrité
- Configurer des alertes en cas d’échec de sauvegarde
- Documenter et tester régulièrement les procédures de restauration

## Stratégies spécifiques à PostgreSQL

### 1. Sauvegardes logiques avec pg_dump

**Sauvegarde de base de données complète :**

```bash
# Sauvegarde complète d’une base de données
pg_dump -h localhost -U username -d database_name > backup.sql

# Sauvegarde avec compression
pg_dump -h localhost -U username -d database_name | gzip > backup.sql.gz

# Format personnalisé (recommandé pour les grandes bases)
pg_dump -h localhost -U username -d database_name -Fc > backup.dump
```

**Sauvegarde de tables spécifiques :**

```bash
# Sauvegarde de tables spécifiques
pg_dump -h localhost -U username -d database_name -t table1 -t table2 > tables_backup.sql

# Sauvegarde du schéma uniquement (sans données)
pg_dump -h localhost -U username -d database_name --schema-only > schema_backup.sql

# Sauvegarde des données uniquement (sans schéma)
pg_dump -h localhost -U username -d database_name --data-only > data_backup.sql
```

**Sauvegarde de l’ensemble du cluster :**

```bash
# Sauvegarde de toutes les bases de données
pg_dumpall -h localhost -U postgres > all_databases.sql

# Sauvegarde uniquement des objets globaux (rôles, tablespaces)
pg_dumpall -h localhost -U postgres --globals-only > globals.sql
```

### 2. Sauvegardes physiques avec pg_basebackup

**Création d’une sauvegarde de base :**

```bash
# Créer une sauvegarde de base
pg_basebackup -h localhost -U replication_user -D /backup/base -Ft -z -P

# Explication des paramètres :
# -D : répertoire de sauvegarde
# -Ft : format tar
# -z : compression
# -P : afficher la progression
```

**Configuration de la récupération à un instant précis (PITR) :**

```bash
# 1. Activer l’archivage WAL dans postgresql.conf
wal_level = replica
archive_mode = on
archive_command = 'cp %p /backup/wal/%f'

# 2. Créer une sauvegarde de base
pg_basebackup -h localhost -U postgres -D /backup/base -Ft -z -P

# 3. Les fichiers WAL sont archivés en continu
```

### 3. Archivage continu et récupération à un instant précis (PITR)

**Configuration dans postgresql.conf :**

```
# Activer l’archivage WAL
wal_level = replica
archive_mode = on
archive_command = 'rsync %p backup_server:/backup/wal/%f'
archive_timeout = 300  # Forcer le changement de WAL toutes les 5 minutes

# Paramètres de point de sauvegarde pour les performances
checkpoint_timeout = 15min
checkpoint_completion_target = 0.9
```

**Exemple de script de sauvegarde automatisée :**

```bash
#!/bin/bash
# daily_backup.sh

BACKUP_DIR="/backups/postgres"
DATE=$(date +%Y%m%d_%H%M%S)
DB_NAME="production_db"

# Créer le répertoire de sauvegarde
mkdir -p $BACKUP_DIR/$DATE

# Effectuer une sauvegarde logique
pg_dump -h localhost -U backup_user -d $DB_NAME -Fc > $BACKUP_DIR/$DATE/logical_backup.dump

# Effectuer une sauvegarde de base (pour PITR)
pg_basebackup -h localhost -U replication_user -D $BACKUP_DIR/$DATE/base -Ft -z -P

# Nettoyer les anciennes sauvegardes (conserver les 30 derniers jours)
find $BACKUP_DIR -type d -mtime +30 -exec rm -rf {} \;

# Journaliser la fin de l’opération
echo "$(date): Sauvegarde terminée avec succès" >> /var/log/postgres_backup.log
```

### 4. Réplication en continu pour la haute disponibilité

**Configuration du serveur maître :**

```
# postgresql.conf
wal_level = replica
max_wal_senders = 3
wal_keep_segments = 64  # Pour les versions antérieures
wal_keep_size = 1GB     # Pour les versions récentes

# pg_hba.conf
host replication replicator replica_ip/32 md5
```

**Configuration du serveur de secours :**

```bash
# 1. Créer une sauvegarde de base sur le serveur de secours
pg_basebackup -h master_ip -U replicator -D /var/lib/postgresql/data -P -W

# 2. Créer le fichier de configuration de récupération
# Pour PostgreSQL 12+, créer le fichier standby.signal
touch /var/lib/postgresql/data/standby.signal

# 3. Configurer postgresql.conf sur le serveur de secours
primary_conninfo = 'host=master_ip port=5432 user=replicator'
```

## Procédures de récupération

### Récupération à partir d’une sauvegarde logique

**Restauration complète de la base de données :**

```bash
# Supprimer et recréer la base de données
dropdb database_name
createdb database_name

# Restaurer à partir d’un dump SQL
psql -h localhost -U username -d database_name < backup.sql

# Restaurer à partir d’un format personnalisé
pg_restore -h localhost -U username -d database_name backup.dump
```

**Récupération sélective :**

```bash
# Restaurer des tables spécifiques
pg_restore -h localhost -U username -d database_name -t table1 -t table2 backup.dump

# Restaurer avec transformation des données
pg_restore -h localhost -U username -d database_name --data-only --disable-triggers backup.dump
```

### Récupération à un instant précis (PITR)

```bash
# 1. Arrêter le service PostgreSQL
systemctl stop postgresql

# 2. Restaurer la sauvegarde de base
rm -rf /var/lib/postgresql/data/*
tar -xzf /backup/base/base.tar.gz -C /var/lib/postgresql/data/

# 3. Créer la configuration de récupération
# Pour PostgreSQL 12+
echo "restore_command = 'cp /backup/wal/%f %p'" > /var/lib/postgresql/data/postgresql.auto.conf
echo "recovery_target_time = '2024-01-15 14:30:00'" >> /var/lib/postgresql/data/postgresql.auto.conf
touch /var/lib/postgresql/data/recovery.signal

# 4. Démarrer PostgreSQL
systemctl start postgresql

# 5. Surveiller la progression de la récupération
tail -f /var/log/postgresql/postgresql.log
```

## Tests et vérification

### Procédures de test régulières

**1. Vérification de l’intégrité des sauvegardes :**

```bash
# Tester une sauvegarde logique
pg_restore --list backup.dump > /dev/null
echo "Code de sortie : $?"  # Doit être 0

# Tester une sauvegarde compressée
gunzip -t backup.sql.gz
echo "Code de sortie : $?"  # Doit être 0
```

**2. Tests de récupération :**

```bash
# Créer un environnement de test
createdb test_restore_db

# Effectuer une restauration de test
pg_restore -d test_restore_db backup.dump

# Vérifier l’intégrité des données
psql -d test_restore_db -c "SELECT COUNT(*) FROM critical_table;"

# Nettoyer
dropdb test_restore_db
```

**3. Script de test automatisé :**

```bash
#!/bin/bash
# test_backup.sh

BACKUP_FILE="$1"
TEST_DB="backup_test_$(date +%s)"

# Créer une base de données de test
createdb $TEST_DB

# Tenter la restauration
if pg_restore -d $TEST_DB $BACKUP_FILE; then
    echo "✓ Restauration réussie"
    
    # Effectuer des vérifications d’intégrité basiques
    ROW_COUNT=$(psql -d $TEST_DB -t -c "SELECT COUNT(*) FROM pg_tables WHERE schemaname = 'public';")
    echo "✓ $ROW_COUNT tables trouvées dans la sauvegarde"
    
    # Nettoyer
    dropdb $TEST_DB
    echo "✓ Base de données de test supprimée"
else
    echo "✗ Échec de la restauration"
    dropdb $TEST_DB 2>/dev/null
    exit 1
fi
```

### Surveillance et alertes

**Requête de surveillance des sauvegardes :**

```sql
-- Vérifier l’ancienneté de la dernière sauvegarde
SELECT name,
       last_backup,
       EXTRACT(EPOCH FROM (NOW() - last_backup)) / 3600 as hours_since_backup
FROM backup_log
WHERE last_backup < NOW() - INTERVAL '25 hours'
ORDER BY last_backup;
```

**État de l’archivage WAL :**

```sql
-- Vérifier l’état de l’archivage WAL
SELECT archived_count,
       last_archived_wal,
       last_archived_time,
       failed_count,
       last_failed_wal,
       last_failed_time
FROM pg_stat_archiver;
```

## Erreurs courantes à éviter

### 1. Erreurs liées aux sauvegardes

- **Ne pas tester régulièrement les restaurations** : 37 % des sauvegardes échouent au moment où elles sont réellement
  nécessaires
- **Stocker les sauvegardes au même endroit que les données** : Vulnérabilité aux mêmes catastrophes
- **Ne pas chiffrer les données sensibles sauvegardées** : Risques de conformité et de sécurité
- **Ignorer la surveillance des sauvegardes** : Les échecs passent inaperçus
- **Ne pas documenter les procédures** : Retards lors de la récupération en cas d’urgence

### 2. Pièges spécifiques à PostgreSQL

- **Oublier de sauvegarder pg_hba.conf et postgresql.conf** : Ces fichiers de configuration ne sont pas inclus dans
  pg_dump
- **Ne pas coordonner avec les périodes d’indisponibilité applicative** : Sauvegardes incohérentes pendant les pics
  d’activité
- **Rétention WAL insuffisante** : Échec de la PITR à cause de fichiers WAL manquants
- **Permissions incorrectes de l’utilisateur de sauvegarde** : Échecs dus à des privilèges insuffisants
- **Ne pas tenir compte des objets volumineux** : pg_dump exclut les objets volumineux par défaut

### 3. Problèmes de planification de la récupération

- **Absence de procédures de récupération documentées** : Tentatives de récupération improvisées en situation de crise
- **Objectifs de temps de récupération non testés** : Attentes irréalistes en cas d’urgence
- **Documentation manquante des dépendances** : Échec des applications après la récupération de la base de données
- **Absence de plan de communication** : Parties prenantes non informées pendant les pannes

## Liste de vérification pour la mise en œuvre d’une stratégie de sauvegarde

### Configuration initiale

- [ ] Identifier les données critiques et les exigences de récupération
- [ ] Définir les objectifs RTO (Recovery Time Objective) et RPO (Recovery Point Objective)
- [ ] Choisir les types et la fréquence appropriés de sauvegardes
- [ ] Mettre en place des emplacements sécurisés pour le stockage des sauvegardes
- [ ] Configurer la surveillance et les alertes
- [ ] Documenter toutes les procédures

### Configuration PostgreSQL

- [ ] Configurer l’archivage WAL pour permettre la PITR
- [ ] Créer un utilisateur dédié aux sauvegardes avec les permissions appropriées
- [ ] Planifier les tâches de sauvegarde automatisées
- [ ] Configurer la réplication en continu si nécessaire
- [ ] Tester les procédures de sauvegarde et de récupération

### Maintenance continue

- [ ] Surveiller quotidiennement l’exécution des sauvegardes
- [ ] Tester les procédures de restauration mensuellement
- [ ] Réviser et mettre à jour les politiques de rétention trimestriellement
- [ ] Organiser des exercices complets de reprise après sinistre annuellement
- [ ] Mettre à jour la documentation à mesure que les systèmes évoluent

**N’oubliez pas** : une stratégie de sauvegarde n’est efficace que si votre dernière restauration testée a été un succès.
Les tests réguliers ne sont pas optionnels — ils sont essentiels pour garantir que votre stratégie de protection des
données fonctionnera au moment où vous en aurez le plus besoin.