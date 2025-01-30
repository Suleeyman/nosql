# Base de donnée orienté documents MongoDB (architecture Client-Serveur)

## Informations utiles

MongoDB existe depuis 2007, est un système de gestion de base de données orienté documents faisant partie de la mouvance NoSQL, répartissable sur un nombre quelconque d'ordinateurs et ne nécessitant pas de schéma prédéfini des données. Il est écrit en C++.

MongoDB stocke les données dans des documents flexibles de type JSON, ce qui signifie que les champs peuvent varier d'un document à l'autre et que la structure des données peut être modifiée au fil du temps.

MongoDB est une base de données distribuée, la haute disponibilité, le dimensionnement horizontal, et la distribution géographique sont donc intégrés et faciles à utiliser

MongoDB propose des pilotes pour plus de 10 langages de programmation (Java, C++, JS, etc.)

## Installation via Docker :

```sh
$ docker pull mongodb/mongodb-community-server:latest
$ docker images # Cette commande est sensée vous afficher l'image docker que vous venez d'installer
$ docker run --name mongodb -p 27017:27017 -d mongodb/mongodb-community-server:latest
$ docker ps # Cette commande est sensée vous afficher le processus docker actuellement en marche
$ docker exec -u root -it ee1b38 /bin/bash # remplacer 'ee1b38' par l'id de votre processus docker
```

## Commandes mongos

Pour visualiser toutes les commandes mongo disponible, écrivez `mongo` dans votre terminal et appuyez deux fois de suites sur la touche _TAB_ (tabulation). Cela va vous afficher toutes les commandes commençant par _mongo_ et donc toutes les commandes mongo.

## Importer les données de teste

Copie le contenu du fichier `films.json`, nous allons le coller dans un fichier à l'intérieur de l'instance docker.

```sh
$ cat > fichiers.json # Coller le contenu avec CTRL+SHIFT+V et fermer l'intéraction avec CTRL+D
$ mongoimport --db lesfilms --collection films films.json --jsonArray
```

## Dans le shell mongo

Dans le terminal, tapez `mongosh` pour entrer dans le terminal

```sh
> help # Affiche les commandes disponible
> show dbs # Affiche les base de données disponible
> use lesfilms # Entre dans la BD 'my_bd'
> show collections # Affiche toutes les collections de la BD
```

## Quelques requêtes à la collection de films

1. Compter le nombre de documents dans la collection `films` : `db.films.countDocuments()`
2. Récupérer un film : `db.films.findOne()`
3. Récuper la liste des films d'actions : `db.films.find({ genre: "Action" }).pretty()` (pretty formatte le résultat)
4. Nombre de films d’action : `db.films.countDocuments({ genre: "Action" })`
5. Films d’action produits en France : `db.films.find({ genre: "Action", country: "France" }).pretty()`
6. Films d’action produits en France en 1963 : `db.films.find({ genre: "Action", country: "France", year: 1963 }).pretty()`
7. Films d’action réalisés en France sans les grades : `db.films.find({ genre: "Action", country: "France" }, { grades: 0 }).pretty()`
8. Films d’action réalisés en France sans les identifiants : `db.films.find({ genre: "Action", country: "France" }, { _id: 0 }).pretty()`
9. Titres et les grades des films d’action réalisés en France sans leurs identifiants :
   ```bash
   db.films.find(
   { genre: "Action", country: "France" },
   { title: 1, grades: 1, _id: 0 }
   ).pretty()
   ```
10. Titres et les notes des films d’action réalisés en France ayant obtenu une note supérieure à 10 :
    ```bash
    db.films.find(
    { genre: "Action", country: "France", "grades.note": { $gt: 10 } },
    { title: 1, "grades.note": 1, _id: 0 }
    ).pretty()
    ```
11. Films d’action réalisés en France ayant uniquement des notes supérieures à 10 :
    ```bash
    db.films.find(
    { genre: "Action", country: "France", "grades.note": { $not: { $lte: 10 } } },
    { title: 1, "grades.note": 1, _id: 0 }
    ).pretty()
    ```
12. Afficher les différents genres de la base lesfilms : `db.films.distinct("genre")`
13. Afficher les différents grades attribués : `db.films.distinct("grades.grade")`
14. Tous les films dans lesquels joue au moins un des artistes suivants ["artist:4", "artist:18", "artist:11"] : `db.films.find({ "actors._id": { $in: ["artist:4", "artist:18", "artist:11"] } }).pretty()`
15. Tous les films qui n’ont pas de résumé : `db.films.find({ summary: { $exists: false } }).pretty()`
16. Tous les films tournés avec Leonardo DiCaprio en 1997 : `db.films.find({ "actors.first_name": "Leonardo", "actors.last_name": "DiCaprio", year: 1997 }).pretty()`
17. Afficher les films tournés avec Leonardo DiCaprio ou en 1997 :
    ```bash
    db.films.find({
    $or: [
        { "actors.first_name": "Leonardo", "actors.last_name": "DiCaprio" },
        { year: 1997 }
    ]
    }).pretty()
    ```

## Exercices

Voici une liste de requêtes que tu peux tester dans mongosh pour interagir avec ta base MongoDB. La correction est en bas du document.

1. Afficher tous les films
2. Trouver un film par son titre
3. Trouver tous les films d'un genre spécifique (ex: Aventure)
4. Trouver tous les films sortis après 2000
5. Trouver un film réalisé par un certain réalisateur (ex: George Lucas)
6. Trouver les films où un acteur spécifique a joué (ex: Mark Hamill)
7. Afficher seulement certains champs (ex: titre et année)
8. Trier les films par année de sortie (du plus récent au plus ancien)
9. Compter le nombre de films dans la collection
10. Trouver les films avec une note minimale (ex: note > 70)

Correction

```bash
db.films.find().pretty() # 1
db.films.findOne({ title: "La Guerre des étoiles" }) # 2
db.films.find({ genre: "Aventure" }).pretty() # 3
db.films.find({ year: { $gt: 2000 } }).pretty() # 4
db.films.find({ "director.last_name": "Lucas", "director.first_name": "George" }).pretty() # 5
db.films.find({ "actors.last_name": "Hamill", "actors.first_name": "Mark" }).pretty() # 6
db.films.find({}, { title: 1, year: 1, \_id: 0 }).pretty() # 7
db.films.find().sort({ year: -1 }).pretty() # 8
db.films.countDocuments() # 9
db.films.find({ "grades.note": { $gt: 70 } }).pretty() # 10
```
