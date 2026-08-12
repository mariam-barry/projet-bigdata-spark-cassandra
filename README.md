Traitement de retards de vols aériens — Apache Spark + Cassandra
Mini-projet Licence 3 Informatique — Introduction au Big Data
Sujet B : Retards de vols aériens
Description
Pipeline de traitement de données de bout en bout :
Ingestion d'un jeu de données de vols aériens avec Apache Spark
Nettoyage et calcul d'indicateurs de retard (par compagnie, par aéroport, par jour de la semaine)
Stockage des résultats agrégés dans Apache Cassandra (base NoSQL orientée colonnes)
Visualisation (bonus) des résultats via des graphiques matplotlib
Dataset
Source : Airline Delay Causes (Kaggle) — fichier `DelayedFlights.csv`
Volume : 1 936 758 lignes / 29 colonnes (vols aériens américains, données BTS)
 Le fichier CSV (~235 Mo) n'est pas inclus dans ce dépôt (dépasse la limite GitHub de 100 Mo).
Téléchargez-le depuis Kaggle et placez-le dans `data/DelayedFlights.csv` avant de lancer le projet.
Architecture
```
projet/
├── docker-compose.yml         # Environnement Spark (Jupyter) + Cassandra
├── data/                      # Dataset (non versionné, voir ci-dessus)
├── notebooks/
│   ├── 01_ingestion.ipynb   # Lecture CSV, échantillonnage, nettoyage
│   ├── 02_indicateur.ipynb    # Calcul des indicateurs avec Spark
│   ├── 03_ecriture.ipynb    # Écriture des résultats dans Cassandra
│   └── 04_visualisation.ipynb         # Lecture Cassandra + graphiques (bonus)
└── README.md
```
Technologies
Apache Spark (PySpark) — traitement et agrégation des données
Apache Cassandra 4.1 — stockage NoSQL orienté colonnes
Docker / Docker Compose — environnement reproductible
Spark-Cassandra Connector — intégration Spark ↔ Cassandra
matplotlib / cassandra-driver — visualisation (bonus)
Installation et lancement
Prérequis
Docker Desktop installé et lancé
1. Cloner le dépôt
```bash
git clone https://github.com/mariam-barry/projet-bigdata-spark-cassandra.git
cd projet-bigdata-spark-cassandra
```
2. Ajouter le dataset
Téléchargez `DelayedFlights.csv` depuis Kaggle (lien ci-dessus) et placez-le dans `data/`.
3. Lancer les conteneurs
```bash
docker compose up -d
```
Vérifiez que les deux services sont opérationnels :
```bash
docker compose ps
```
4. Ouvrir JupyterLab
Rendez-vous sur http://localhost:8888
5. Exécuter les notebooks dans l'ordre
`01` → `02` → `03` → `04`
6. Créer le keyspace et les tables Cassandra (avant le notebook 03)
```bash
docker exec -it cassandra cqlsh
```
Puis, dans `cqlsh` :
```sql
CREATE KEYSPACE IF NOT EXISTS vols
  WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1};

USE vols;

CREATE TABLE retards_par_compagnie (
    compagnie text PRIMARY KEY,
    nb_vols int, nb_retards int, retard_moyen float, taux_retard float
);

CREATE TABLE retards_par_aeroport (
    aeroport text PRIMARY KEY,
    nb_vols int, nb_retards int, retard_moyen float, taux_retard float
);

CREATE TABLE retards_par_jour (
    jour_semaine int PRIMARY KEY, nom_jour text,
    nb_vols int, nb_retards int, retard_moyen float, taux_retard float
);

CREATE TABLE retards_par_aeroport_jour (
    aeroport text, jour_semaine int, nom_jour text,
    nb_vols int, nb_retards int, retard_moyen float, taux_retard float,
    PRIMARY KEY ((aeroport), jour_semaine)
);
```
7. Arrêter l'environnement
```bash
docker compose down       # arrête sans supprimer les données Cassandra
docker compose down -v    # arrête et supprime tout (données incluses)
```
Indicateurs calculés
Seuil de retard retenu : ArrDelay > 15 minutes (convention standard FAA).
Indicateur	Table Cassandra	Clé de partition / clustering
Taux de retard par compagnie	`retards_par_compagnie`	`compagnie`
Taux de retard par aéroport	`retards_par_aeroport`	`aeroport`
Taux de retard par jour de la semaine	`retards_par_jour`	`jour_semaine`
Taux de retard par aéroport × jour (bonus)	`retards_par_aeroport_jour`	partition : `aeroport` / clustering : `jour_semaine`
Choix techniques principaux
Échantillonnage aléatoire (150 000 lignes, seed fixe) plutôt qu'une troncature, pour rester représentatif tout en gardant un temps de traitement raisonnable en local.
Format Parquet pour les échanges entre notebooks (préserve le typage, plus performant que le CSV).
Une table Cassandra par indicateur plutôt qu'un schéma unique, conformément à la logique query-first de Cassandra.
Auteur
Barry Mariama Bailo — Licence 3 Informatique
