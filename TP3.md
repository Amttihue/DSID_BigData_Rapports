Auteur : Matthieu Ramond - M1 MIAGE Classique

[Index](./README.md)

## HDFS

1. **Dans HDFS, créer en lignes de commande HDFS (hdfs dfs -??????) l'arborescence suivante** ``/data/common/raw/NOM_DATABASE_POSTGRES/NOM_TABLE_POST``

    Commande :
    ```bash
    hdfs dfs -mkdir -p /data/common/raw/NOM_DATABASE_POSTGRES/NOM_TABLE_POST
    ```
    Résultat : Ne renvoie rien 


2. **En lignes de commande HDFS, Créer un fichier ``studentM1_1.csv`` dans ce répertoire (ayant 3 colonnes firstName, lastName,email, avec vos données)**

    ```bash
    hdfs dfs -put studentM1_1.csv /data/common/raw/NOM_DATABASE_POSTGRES/NOM_TABLE_POST/
    ```
    Résultat : Ne renvoie rien

3. **Afficher le contenu en lignes de commande HDFS du fichier**

    Commande :
    ```bash
    hdfs dfs -cat /data/common/raw/NOM_DATABASE_POSTGRES/NOM_TABLE_POST/studentM1_1.csv
    ```
    Résultat : 
    ```
    firstName,lastName,email
    Matthieu,Ramond,matthieu.ramond@example.com
    Alice,Dupont,alice.dupont@example.com
    John,Smith,john.smith@example.com
    ```


## HIVE

1. **Créer une base de données DATABASE_M1 (celle que vous avez créée dans postgres)**

    Code HQL :
    ```SQL
    CREATE DATABASE DATABASE_M1;
    ```

2. **Avec HQL, créer une base de données DATABASE_M2**

    Code HQL :
    ```SQL
    CREATE DATABASE DATABASE_M2;
    ```

3. **Avec HQL, créer une table HIVE ETUDIANT_M1 dans la base de donnée DATABASE_M1 pointant sur le répertoire `data/common/raw/DATABASE_M1/ETUDIANT_M1`**

    Code HQL :
    ```SQL
    USE DATABASE_M1;

    CREATE EXTERNAL TABLE ETUDIANT_M1 (
        firstName STRING,
        lastName STRING,
        email STRING
    )
    STORED AS TEXTFILE
    LOCATION '/data/common/raw/DATABASE_M1/ETUDIANT_M1';
    ```

4. **Avec HQL, Afficher le contenu de la table ETUDIANT_M1**

    Code HQL :
    ```SQL
    SELECT * FROM ETUDIANT_M1;
    ```
    Résultat : Ne renvoie rien (car table vide)

5. **Avec HQL, Créer une table ETUDIANT_M1_PART dans la base de donnée DATABASE_M1 partitionnée sur le champ DateRecep (au format année mois, jour, heure, minute : AAAAMMJJHHmm) et pointant vers le répertoire /common/raw/DATABASE_M1/ETUDIANT_M1_PART**

    Code HQL :
    ```SQL
    CREATE EXTERNAL TABLE ETUDIANT_M1_PART (
        firstName STRING,
        lastName STRING,
        email STRING
    )
    PARTITIONED BY (DateRecept STRING)
    STORED AS TEXTFILE
    LOCATION '/common/raw/DATABASE_M1/ETUDIANT_M1_PART';
    ```

6. **Créer une table externe ETUDIANT_M2 dans la base de donnée DATABASE_M2**

    On suit le même schéma que pour la table ETUDIANT_M1.

    Code HQL :
    ```SQL
    USE DATABASE_M2;

    CREATE EXTERNAL TABLE ETUDIANT_M2 (
        firstName STRING,
        lastName STRING,
        email STRING
    )
    STORED AS TEXTFILE
    LOCATION '/data/common/raw/DATABASE_M2/ETUDIANT_M2';
    ```

## NIFI
1. **Dupliquer le TP2 en TP3**

    J'ai donc dupliqué le process group TP2-B, le process group dédié à la réception des données via Kafka et au stockage dans la base de données sql.

### Au lieu d'utiliser la base de données POSTGRES, utiliser HIVE :
2. Créer le repertoire /common/raw/NOM_DE_LA_DATABASE_POSTGRES/NOM_DE_LA_TABLE_POSTGRES

    Dans notre cas, la base de donnée s'appelle ``dsid`` et la table ``dsid_m_departements``

    Commande : 
    ```bash
    hdfs dfs -mkdir -p /common/raw/dsid/dsid_m_departements
    ```
    


3. **Créer la base de donnée HIVE ``dsid``**

    Code HQL :
    ```SQL
    CREATE DATABASE dsid;
    ```

4. **Créer la table externe `dsid_m_departements` dans la base de donnée HIVE ``dsid`` et qui pointe vers le repertoire HDFS ``/common/raw/dsid/dsid_m_departements``**

    Code HQL :
    ```SQL
    USE dsid;

    CREATE EXTERNAL TABLE dsid_m_departements (
        nom STRING,
        code STRING,
        codeRegion STRING,
        date_trt STRING
    )
    STORED AS TEXTFILE
    LOCATION '/common/raw/dsid/dsid_m_departements';
    ```

5. **En utilisant le processesor putHdfs, déposer les données récupérées depuis KAFKA dans le répertoire HDFS ``/common/raw/dsid/dsid_m_departements/DateRecep=20241017`` (20241017 est la date du jout et cette valeur doit être générée dynamiquement par nifi, (utiliser un attribut du flowfile avec une valeur date au format demandé ex : Variable_DateRecep avec la valeur DateRecep=${now():format('yyyyMMdd')}**



6. Faire un select sur la table , qu'est ce que vous remarquez?
7. Lancer la commande sql suivante Msck repair table NOM_DE_LA_DATABASE_POSTGRES.NOM_DE_LA_TABLE_POSTGRES;
8. Créer le repertoire /common/raw/NOM_DE_LA_DATABASE_POSTGRES_2/NOM_DE_LA_TABLE_POSTGRES_2
9.  Creér la base de donnée HIVE NOM_DE_LA_DATABASE_POSTGRES_2
10. Creér la table manage NOM_DE_LA_TABLE_POSTGRES_2 dans la base de donnée HIVE NOM_DE_LA_DATABASE_POSTGRES_2 et qui pointe vers le repertoire HDFS /common/raw/NOM_DE_LA_DATABASE_POSTGRES_2/NOM_DE_LA_TABLE_POSTGRES_2
11. En utilsant un processor Hive ou ExcuteSql, Copier les données (via une requête hql ) de la table NOM_DE_LA_DATABASE_POSTGRES dans la table NOM_DE_LA_DATABASE_POSTGRES_2 de façon à ne garder que la dernière version du fichier envoyé (utilisé le mot clé OVERWRITE et dans la clause where du select utiliser la valeur de la dernière partition (requête + copie d'écran du résultat)
12. supprimer les 2 tables NOM_DE_LA_DATABASE_POSTGRES.NOM_DE_LA_TABLE_POSTGRES et NOM_DE_LA_DATABASE_POSTGRES_2.NOM_DE_LA_TABLE_POSTGRES_2
13. Quel est votre constat?
	

	
	

### Exposer une API NIFI pour recevoir des données fichiers externes (utiliser les 2 HandleHttpRequest et HandleHttpResponse)
1. Envoyer, 10 fois, le fichier de données student.csv (joint au cours) à l'api nifi.
2. Convertir les données reçues avec le format CSV vers le format avro
3. Déposer les données dans le répertoire (utiliser le processesor putHdfs) HDFS /common/raw/DATABASE_M1/ETUDIANT_M1_PART/DateRecep=202210ddHHmm (cette valeur doit être générée dynamiquement par nifi, (utiliser un attribut du flowfile avec une valeur date au format demandé ex : Variable_DateRecep avec la valeur DateRecep=${now():format('yyyyMMddHHmm')}

4. Utiliser Kafka pour stocker les données CSV reçues par nifi puis consommer avec NIFI les données csv depuis KAFKA, les convertir vers le format avro et les déposer dans hdfs.

