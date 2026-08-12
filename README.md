#  Traitement des retards de vols — Apache Spark + Cassandra

##  Description

Mini-projet réalisé dans le cadre de la **Licence 3 Informatique — Introduction au Big Data**.

L'objectif est de mettre en place un pipeline de traitement de données de vols aériens permettant d'analyser les retards par **compagnie aérienne, aéroport et jour de la semaine**, puis de stocker les résultats dans **Apache Cassandra**.

##  Dataset

Le projet utilise le dataset **Airline Delay Causes** contenant :

* 1 936 758 lignes
* 29 colonnes
* des données sur les vols aériens américains

Le fichier `DelayedFlights.csv` doit être téléchargé depuis Kaggle et placé dans :

```text
data/DelayedFlights.csv
```

Le fichier n'est pas inclus dans le dépôt en raison de sa taille (~235 Mo).

##  Architecture

```text
projet/
├── docker-compose.yml
├── data/
│   └── DelayedFlights.csv
├── notebooks/
│   ├── 01_ingestion.ipynb
│   ├── 02_indicateur.ipynb
│   ├── 03_ecriture.ipynb
│   └── 04_visualisation.ipynb
└── README.md
```

##  Technologies

* **Apache Spark / PySpark** : traitement et agrégation
* **Apache Cassandra 4.1** : stockage NoSQL
* **Docker / Docker Compose** : environnement
* **Spark-Cassandra Connector** : connexion Spark ↔ Cassandra
* **Matplotlib** : visualisation (bonus)
* **JupyterLab** : exécution des notebooks

## Installation

### 1. Cloner le projet

```bash
git clone https://github.com/mariam-barry/projet-bigdata-spark-cassandra.git
cd projet-bigdata-spark-cassandra
```

### 2. Ajouter le dataset

Télécharger `DelayedFlights.csv` depuis Kaggle et le placer dans le dossier `data/`.

### 3. Lancer Docker

```bash
docker compose up -d
```

Vérifier les conteneurs :

```bash
docker compose ps
```

### 4. Ouvrir JupyterLab

Accéder à :

```text
http://localhost:8888
```

Exécuter les notebooks dans l'ordre :

```text
01 → 02 → 03 → 04
```

##  Cassandra

Avant le notebook `03_ecriture.ipynb`, créer le keyspace et les tables :

```bash
docker exec -it cassandra cqlsh
```

Puis :

```sql
CREATE KEYSPACE IF NOT EXISTS vols
WITH replication = {
    'class': 'SimpleStrategy',
    'replication_factor': 1
};

USE vols;

CREATE TABLE retards_par_compagnie (
    compagnie text PRIMARY KEY,
    nb_vols int,
    nb_retards int,
    retard_moyen float,
    taux_retard float
);

CREATE TABLE retards_par_aeroport (
    aeroport text PRIMARY KEY,
    nb_vols int,
    nb_retards int,
    retard_moyen float,
    taux_retard float
);

CREATE TABLE retards_par_jour (
    jour_semaine int PRIMARY KEY,
    nom_jour text,
    nb_vols int,
    nb_retards int,
    retard_moyen float,
    taux_retard float
);

CREATE TABLE retards_par_aeroport_jour (
    aeroport text,
    jour_semaine int,
    nom_jour text,
    nb_vols int,
    nb_retards int,
    retard_moyen float,
    taux_retard float,
    PRIMARY KEY ((aeroport), jour_semaine)
);
```

##  Indicateurs

Un vol est considéré comme retardé lorsque :

```text
ArrDelay > 15 minutes
```

Les indicateurs calculés sont :

* taux de retard par compagnie ;
* taux de retard par aéroport ;
* taux de retard par jour de la semaine ;
* taux de retard par aéroport et jour.

Pour chaque indicateur :

```text
Taux de retard = (Nombre de vols retardés / Nombre total de vols) × 100
```

## Choix techniques

* Échantillon aléatoire de **150 000 lignes** avec un seed fixe.
* Utilisation du format **Parquet** entre les notebooks.
* Une table Cassandra par indicateur, selon une approche **query-first**.

## 🛑 Arrêter Docker

```bash
docker compose down
```

Pour supprimer également les données Cassandra :

```bash
docker compose down -v
```

## 👤 Auteur

**Barry Mariama Bailo — Licence 3 Informatique**
