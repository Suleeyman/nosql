# Présentation des bases de données NoSQL puis Redis

## NoSQL

Le terme NoSQL fait référence aux bases de données non relationnelles qui stockent des données dans un format non tabulaire, plutôt que dans des tables relationnelles basées sur des règles, comme le font les bases de données relationnelles (MySQL, Postgres, etc.). Les bases de données NoSQL utilisent un modèle de schéma flexible qui accepte une grande variété de données non structurées, telles que des documents, des clé-valeurs, des colonnes larges et des graphiques. Les BDs NoSQL sont conçus pour gérer de gros volumes de données distribuées sur plusieurs serveurs.

Il existe différents types de modèles de bases de données NoSQL :

**Clé-valeur**

Les données sont stockées sous forme de paires clé-valeur. Idéal pour des cas simples où chaque clé unique fait référence à une valeur spécifique. _e.g_ : Redis, Amazon DynamoDB.

**Documents**

Les données sont stockées sous forme de documents semi-structurés (JSON, BSON, XML, etc.). Idéal pour les applications web et mobiles. _e.g_ : MongoDB, Couchbase.

**Orientées colonnes**

Les données sont organisées en colonnes plutôt qu’en lignes. Convient pour les analyses rapides de grandes quantités de données. _e.g_ : Apache Cassandra, HBase.

**Graphe**

Les données sont représentées sous forme de graphes. _e.g_ : Neo4j, ArangoDB.

**Série temporelle**

Une base de données de séries temporelles (_Time series database_) est un type de base de données NoSQL conçus pour organiser des informations mesurées dans le temps. Particulièrement présent dans l'**IoT**. La BD la plus utilisé dans cette catégorie est _InfluxDB_ (utilisé par Tesla)

## Redis

Redis est une BD NoSQL clé/valeur en mémoire open source rapide. Redis propose un ensemble de structures de données en mémoire polyvalentes qui vous permet de créer facilement un large éventail d'applications personnalisées. Les principaux cas d'utilisation de Redis comprennent la mise en cache, la gestion des sessions. C'est la BD NoSQL clé/valeur le plus populaire à l'heure actuelle. Il est distribué sous licence BSD, écrit en code C optimisé. Redis est l'acronyme de _REmote DIctionary Server_. L'avantage n°1 de Redis est qu'il stocke les données dans la RAM de la machine. Ce qui rend l'accès aux données extrémement rapide mais non persisente (par défaut).

Pour installer Redis dans un environnement Linux, créer un script `i-redis.sh` :

```
$ touch i-redis.sh
```

Ajoutez-y le contenu suivant :

```
sudo apt-get install lsb-release curl gpg
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg
sudo chmod 644 /usr/share/keyrings/redis-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list
sudo apt-get update
sudo apt-get install redis
```

Puis rendez le exécutable avant de la lancer :

```
$ chmod +x i-redis.sh
$ ./i-redis.sh
```

Pour vérifier que votre installation fonctionne : `$ redis-cli` devrait vous ouvrir un shell pour communiquer avec le serveur redis configuré automatiquement après l'installation.

### Commandes utiles

| Commande | Syntax        | Exemple             | Output |
| -------- | ------------- | ------------------- | ------ |
| SET      | SET key value | `SET myKey "Hello"` | "OK"   |

Définit la clé pour qu'elle contienne la valeur de la chaîne de caractères. Si la clé contient déjà une valeur, celle-ci est écrasée, quel que soit son type : O(1)

---

| Commande | Syntax  | Exemple     | Output  |
| -------- | ------- | ----------- | ------- |
| GET      | GET key | `GET myKey` | "Hello" |

Obtenir la valeur de la chaîne de caractères de la clé. Si la clé n'existe pas, la valeur spéciale nil est renvoyée : O(1)

---

| Commande | Syntax             | Exemple          | Output              |
| -------- | ------------------ | ---------------- | ------------------- |
| MGET     | MGET key [key ...] | `MGET myKey idk` | 1) "Hello" 2) (nil) |

Renvoie les valeurs de toutes les clés spécifiées. Pour chaque clé qui ne contient pas de valeur de chaîne ou qui n'existe pas, la valeur spéciale nil est renvoyée : O(N)

---

| Commande | Syntax        | Exemple          | Output      |
| -------- | ------------- | ---------------- | ----------- |
| INCR     | INCR compteur | `INCR myCounter` | (integer) 1 |

Augmente d'une unité le nombre stocké dans la clé. Si la clé n'existe pas, elle est mise à 0 avant d'effectuer l'opération : O(1)

---

| Commande | Syntax       | Exemple    | Output                    |
| -------- | ------------ | ---------- | ------------------------- |
| KEYS     | KEYS pattern | `KEYS my*` | 1) "myKey" 2) "myCounter" |

Renvoie toutes les clés correspondant au motif.Complexité temporelle : O(N)

---

| Commande | Syntax               | Exemple        | Output      |
| -------- | -------------------- | -------------- | ----------- |
| EXISTS   | EXISTS key [key ...] | `EXISTS myKey` | (integer) 1 |

Vérifie si une ou plusieurs clés existent.Complexité temporelle : O(N)

---

| Commande | Syntax             | Exemple            | Output      |
| -------- | ------------------ | ------------------ | ----------- |
| EXPIRE   | EXPIRE key seconds | `EXPIRE myKey 120` | (integer) 1 |

Définir un délai d'attente pour une clé. Une fois le délai d'attente expiré, la clé sera automatiquement supprimée.

---

| Commande | Syntax  | Exemple     | Output        |
| -------- | ------- | ----------- | ------------- |
| TTL      | TTL key | `TTL myKey` | (integer) 113 |

Renvoie le temps restant à vivre d'une clé qui a un délai d'attente.

---

| Commande | Syntax      | Exemple         | Output      |
| -------- | ----------- | --------------- | ----------- |
| PERSIST  | PERSIST key | `PERSIST myKey` | (integer) 1 |

Supprime l'expiration d'une clé.

---

| Commande | Syntax           | Exemple     | Output      |
| -------- | ---------------- | ----------- | ----------- |
| DEL      | DEL key [key...] | `DEL myKey` | (integer) 1 |

Supprime la clé.

---

| Commande | Syntax         | Exemple                          | Output |
| -------- | -------------- | -------------------------------- | ------ |
| INFO     | INFO [section] | `INFO server` ou `INFO keyspace` | ...    |

Affiche les statistiques et informations à propos du serveur Redis.

---

| Commande | Syntax                       | Exemple              | Output      |
| -------- | ---------------------------- | -------------------- | ----------- |
| SADD     | SADD key member [member ...] | `SADD mySet "Hello"` | (integer) 1 |

Ajoute les membres spécifiés à l'ensemble stocké dans la clé.

---

| Commande | Syntax       | Exemple          | Output     |
| -------- | ------------ | ---------------- | ---------- |
| SMEMBERS | SMEMBERS key | `SMEMBERS mySet` | 1) "Hello" |

Renvoie tous les membres de la valeur de l'ensemble stockée dans la clé.

---
