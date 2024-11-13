# TP Neo4j

## Introduction à Cypher 

### 1. Afficher les nœuds de type "Film"

Affiche tous les nœuds de type Movie sous forme de graphe.

```cypher
MATCH (m:Movie) 
RETURN m
```

### 2. Afficher tous les titres de Films

Affiche les titres de tous les films présents dans la base.

```cypher
MATCH (m:Movie) 
RETURN m.title
```

### 3. Limiter la liste des Titres

Affiche les trois premiers titres de films, triés par ordre alphabétique.

```cypher
MATCH (m:Movie) 
RETURN m.title 
ORDER BY m.title 
LIMIT 3
```

### 4. Films avec Kevin Bacon

Recherche et affiche tous les films dans lesquels Kevin Bacon a joué.

```cypher
MATCH (a:Person {name: "Kevin Bacon"})-[:ACTED_IN]->(m:Movie) 
RETURN m
```

### 5. Acteurs ayant joué avec Tom Cruise

Liste les noms des acteurs ayant joué avec Tom Cruise.

```cypher
MATCH (a:Person {name: "Tom Cruise"})-[:ACTED_IN]->(m:Movie)<-[:ACTED_IN]-(coActor:Person)
RETURN coActor.name
```

### 6. Acteurs dans plusieurs Films

Identifie les acteurs ayant joué dans au moins deux films et affiche leur nombre de films.

```cypher
MATCH (a:Person)-[:ACTED_IN]->(m:Movie)
WITH a, count(m) AS movies_count
WHERE movies_count >= 2
RETURN a.name, movies_count
```

### 7. Acteurs nés après 2000

Liste les acteurs nés après l'année 2000.

```cypher
MATCH (a:Person)
WHERE a.born > 2000
RETURN a.name, a.born
```

### 8. Acteurs non suivis

Trouve les acteurs qui ne sont suivis par aucun autre acteur.

```cypher
MATCH (a:Person)
WHERE NOT (a)<-[:FOLLOWS]-()
RETURN a.name
```

## Mise à jour du graphe 
Ajout du film Darkstar avec les acteurs

```cypher
CREATE (d:Movie {title: "Darkstar", year: 1974})

// Ajout des acteurs 
CREATE (a1:Person {name: "Brian Narelle"}),
       (a2:Person {name: "Drexler D. R. H."}),
       (a3:Person {name: "Cynthia Catania"})

// Rélations
CREATE (a1)-[:ACTED_IN]->(d),
       (a2)-[:ACTED_IN]->(d),
       (a3)-[:ACTED_IN]->(d)

RETURN d, a1, a2, a3
```

Supression du film Darkstar

```cypher
MATCH (d:Movie {title: "Darkstar"})
DETACH DELETE d
```


## Analyse du graphe 

### 9. Degree Centrality

```cypher
MATCH (p:Person)-[r]->()
WITH p, COUNT(r) AS degree
RETURN p.name, degree
ORDER BY degree DESC
LIMIT 10

```

MATCH (p:Person)-[r]->() : Recherche tous les nœuds de type Person (les acteurs) et leurs relations sortantes.

COUNT(r) AS degree : Compte le nombre de relations sortantes pour chaque acteur, ce qui donne la centralité de degré.

RETURN p.name, degree : Retourne le nom de l'acteur et sa centralité de degré.

ORDER BY degree DESC : Trie ordre decroissant

LIMIT 10 : Limite le résultat aux 10 acteurs les plus centraux.


### 10. Betweeness Centrality

```cypher
MATCH p = allShortestPaths((p1:Person)-[:ACTED_IN*..4]-(p2:Person))
WHERE id(p1) < id(p2) AND length(p) > 1
UNWIND nodes(p)[1..-1] AS n
RETURN n, count(*) AS Betweenness
ORDER BY Betweenness DESC
LIMIT 10

```

Ici j'ai restreint la profondeur de recherche avec un nombre maximum de sauts à 4. Cela rend la requête plus rapide, mêm si ça reduit légèrement la précision.

### 11. Page Rank
En utilisant 

```cypher
//Creation d'un graphe 

CALL gds.graph.project(
  'movieGraph',                   // Nom du graphe 
  ['Person'],                      // Nœuds de type Person (acteurs)
  ['ACTED_IN']                     // Relations de type ACTED_IN (films dans lesquels les acteurs ont joué)
)

// Page Rank

CALL gds.pageRank.stream(
  'movieGraph', 
  // Projections                 
  {
    nodeLabels: ['Person'],      
    relationshipTypes: ['ACTED_IN'], 
    maxIterations: 20,           
    dampingFactor: 0.85,         // Facteur d'amortissement
    relationshipWeightProperty: null 
  }
)
YIELD nodeId, score
MATCH (p:Person) WHERE id(p) = nodeId
RETURN p.name AS actor, score
ORDER BY score DESC


```


### 12. Import CSV

Extrait de mon jeu de données

```csv
person_id,person_name,book_id,book_title,book_author,book_year
14,Ivy Walker,11,Lord of the Rings,James Joyce,1981
46,Olivia Adams,29,The Chronicles of Narnia,Rudyard Kipling,1837
6,Alice Black,36,The Handmaid's Tale,J.R.R. Tolkien,1905
35,Diana King,19,The Shining,Paulo Coelho,1913
14,Ivy Walker,15,The Picture of Dorian Gray,Agatha Christie,1953
6,Alice Black,2,To Kill a Mockingbird,S.E. Hinton,1931
4,James Brown,11,Lord of the Rings,Herman Melville,1949

```


### Chargement du jeu de données et création des noeuds

```cypher
// Charger les personnes (Person)
LOAD CSV WITH HEADERS FROM 'file:///people_books.csv' AS row
MERGE (p:Person {id: toInteger(row.person_id), name: row.person_name});

// Charger les livres (Book)
LOAD CSV WITH HEADERS FROM 'file:///people_books.csv' AS row
MERGE (b:Book {id: toInteger(row.book_id), title: row.book_title, author: row.book_author, year: toInteger(row.book_year)});

```

### Captures

- [Graph des livres](https://github.com/kouroumapaul/Tp_Neo4j/blob/master/books.svg)
- [Graph des personnes](https://github.com/kouroumapaul/Tp_Neo4j/blob/master/persons.svg)



### Création de la rélation READS

```cypher
// Créer les relations READS entre Person et Book
LOAD CSV WITH HEADERS FROM 'file:///people_books.csv' AS row
MATCH (p:Person {id: toInteger(row.person_id)}), (b:Book {id: toInteger(row.book_id)})
MERGE (p)-[:READS]->(b);

```

### Captures


- [Graph des personnes](https://github.com/kouroumapaul/Tp_Neo4j/blob/master/graph.svg?raw)



## Auteur

- Paul KOUROUMA

