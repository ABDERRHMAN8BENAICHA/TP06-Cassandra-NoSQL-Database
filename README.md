# 📘 TP06 - Cassandra (NoSQL Database)

## 🧑‍🎓 Student

* Name: *BEN AICHA Abderrahmane*
* Module: Big Data
* TP: N°06 - Cassandra

---

# 📌 1. Introduction

Dans ce TP, nous avons utilisé Cassandra pour gérer une base de données contenant des restaurants et leurs inspections.

Objectifs :

* Créer une base de données
* Importer des données CSV
* Exécuter des requêtes
* Comprendre les limitations de Cassandra

---

# ⚙️ 2. Environnement

* OS: Linux Mint
* Docker
* Cassandra
* cqlsh

---

# 🚀 3. Lancement

```bash
docker run -d --name mon-cassandra -p 9042:9042 cassandra:latest
docker exec -it mon-cassandra cqlsh
```

---

# 🗄️ 4. Keyspace

```sql
CREATE KEYSPACE resto_NY
WITH REPLICATION = {
  'class': 'SimpleStrategy',
  'replication_factor': 1
};

USE resto_NY;
```

---

# 🧱 5. Tables

## Restaurant

```sql
CREATE TABLE Restaurant (
 id INT,
 name VARCHAR,
 borough VARCHAR,
 buildingnum VARCHAR,
 street VARCHAR,
 zipcode INT,
 phone TEXT,
 cuisinetype VARCHAR,
 PRIMARY KEY (id)
);
```

## Inspection

```sql
CREATE TABLE Inspection (
 idrestaurant INT,
 inspectiondate DATE,
 violationcode VARCHAR,
 violationdescription VARCHAR,
 criticalflag VARCHAR,
 score INT,
 grade VARCHAR,
 PRIMARY KEY (idrestaurant, inspectiondate)
);
```

---

# 📥 6. Import des données

```bash
docker cp restaurants.csv mon-cassandra:/
docker cp restaurants_inspections.csv mon-cassandra:/
```

```sql
COPY Restaurant (id, name, borough, buildingnum, street, zipcode, phone, cuisinetype)
FROM '/restaurants.csv' WITH DELIMITER=',';

COPY Inspection (idrestaurant, inspectiondate, violationcode, violationdescription, criticalflag, score, grade)
FROM '/restaurants_inspections.csv' WITH DELIMITER=',';
```

---

# 📊 7. Vérification

```sql
SELECT count(*) FROM Restaurant;
SELECT count(*) FROM Inspection;
```

---

# 🔍 8. Requêtes CQL

## ✅ 1. Liste de tous les restaurants

```sql
SELECT * FROM Restaurant LIMIT 10;
```

---

## ✅ 2. Liste des noms des restaurants

```sql
SELECT name FROM Restaurant;
```

---

## ✅ 3. Nom et borough pour un restaurant donné

```sql
SELECT name, borough 
FROM Restaurant 
WHERE id = 41569764;
```

---

## ✅ 4. Dates et grades des inspections

```sql
SELECT inspectiondate, grade 
FROM Inspection 
WHERE idrestaurant = 41569764;
```

---

## ❌ 5. Restaurants de cuisine française (Erreur)

```sql
SELECT name 
FROM Restaurant 
WHERE cuisinetype = 'French';
```

Erreur :

```
Cannot execute this query as it might involve data filtering
```

---

## ✅ Solution (Index)

```sql
CREATE INDEX ON Restaurant (cuisinetype);

SELECT name 
FROM Restaurant 
WHERE cuisinetype = 'French';
```

---

## ❌ 6. Restaurants à Brooklyn (Erreur)

```sql
SELECT name 
FROM Restaurant 
WHERE borough = 'BROOKLYN';
```

---

## ✅ Solution 1 (ALLOW FILTERING)

```sql
SELECT name 
FROM Restaurant 
WHERE borough = 'BROOKLYN' 
ALLOW FILTERING;
```

---

## ✅ Solution 2 (Index)

```sql
CREATE INDEX ON Restaurant (borough);

SELECT name 
FROM Restaurant 
WHERE borough = 'BROOKLYN';
```

---

## ⚠️ 7. Inspections avec score ≥ 10

```sql
SELECT grade, score 
FROM Inspection 
WHERE idrestaurant = 41569764 
AND score >= 10 
ALLOW FILTERING;
```

---

## ❌ 8. Score > 30 (Erreur grave)

```sql
SELECT grade 
FROM Inspection 
WHERE score > 30 
ALLOW FILTERING;
```

Erreur :

```
READ_TOO_MANY_TOMBSTONES
```

---

## ⚠️ Solution temporaire

```sql
SELECT grade 
FROM Inspection 
WHERE score > 30 
LIMIT 10 ALLOW FILTERING;
```

---

## ❌ 9. Count avec condition

```sql
SELECT count(*) 
FROM Inspection 
WHERE score > 30 
ALLOW FILTERING;
```

⚠️ Warning :

```
Aggregation query used without partition key
```

---

# 🧠 9. Analyse

Problèmes observés :

* Cassandra refuse les requêtes sans clé primaire
* ALLOW FILTERING est lent
* Les requêtes avec conditions (>, <) sont inefficaces
* COUNT(*) est coûteux

---

# 💡 10. Solution avancée (Redesign)

```sql
CREATE TABLE Inspection_by_score (
 score INT,
 idrestaurant INT,
 inspectiondate DATE,
 grade VARCHAR,
 PRIMARY KEY (score, idrestaurant, inspectiondate)
);
```

---

# 🏁 11. Conclusion

Ce TP montre que Cassandra nécessite :

* Un design basé sur les requêtes
* L’utilisation d’index avec prudence
* Éviter les requêtes globales

👉 Contrairement à SQL, Cassandra impose des contraintes mais offre une grande performance.

---
