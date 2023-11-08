
## Install

Container that puts data into /tmp/arangodb

```bash
podman run -d -p 8529:8529 -e ARANGO_ROOT_PASSWORD=openSesame \
    -v /tmp/arangodb:/var/lib/arangodb3 --name arangodb \
    arangodb/arangodb:3.11.4
```

## Basic Usage

`RETURN NEW` added to the end of a query shows the value handled in JSON

### Retrieve Document

```
RETURN DOCUMENT("users/9883")
```

### Insert Document

```
INSERT { _key: "10074", name: "James Hendrix", age: 69 } INTO users 
```

### Iterate and Sort Documents

```
FOR user IN users SORT user.age DESC RETURN user
```

### Modify Documents

Does a partial update on an existing record. Replacing whole doc instead uses `REPLACE`
```
UPDATE "9915" WITH { age: 40 } IN users
```

### Remove Documents

```
REMOVE "9883" IN users
```


## Graph Queries

For the example of Flights and Airports

Vertices: Airports
Edges: Flights between Airports

All Airports 1 airport stop away from LAX
```cypher
FOR airport IN 1..1 
  OUTBOUND
  'airports/LAX' flights
  RETURN DISTINCT airport.name
```

Return just 10 airplane flight numbers coming to BIS

```cypher
FOR airport, flight IN 
INBOUND
'airports/BIS' flights
  LIMIT 10
  RETURN flight.FlightNum
```

Find all connections which **depart from or land at** BIS on January 5th and 7th and return to the destination city and the arrival time in universal time (UTC)

```cypher
FOR airport, flight IN ANY 
'airports/BIS' flights
  FILTER flight.Month == 1
     AND flight.Day >= 5
     AND flight.Day <= 7
  RETURN { city: airport.city,
           time: flight.ArrTimeUTC }
```

Graph Query Syntax

```
FOR vertex[, edge[, path]]
  IN [min[..max]]
  OUTBOUND|INBOUND|ANY startVertex
  edgeCollection[, more…]
```


Default is DFS, can choose BFS instead like so

```cypher
FOR v IN 1..10 OUTBOUND 'verts/S' edges
    OPTIONS {bfs: true}
    FILTER v._key == 'F'
    LIMIT 1
    RETURN v
```

Shortest path from Bismark to JFK

```cypher
WITH airports
FOR v IN OUTBOUND
  SHORTEST_PATH 'airports/BIS'
  TO 'airports/JFK' flights
  RETURN v.name
```