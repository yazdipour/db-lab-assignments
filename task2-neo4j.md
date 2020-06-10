# Task 2 - Lab Training Database Systems SS20

## Shahriar Yazdipour [matr. No:62366]

The data that is used here can be loaded from <https://openflights.org/data.html>.

a) Load the three data files into Neo4J. You can manually download and copy them into the
Docker container, or use their direct website links in Neo4J to load them. Do not forget to
rename the attributes according to their description on the website, else it might be hard to
query them later. In addition, create relationships between them where necessary – e.g. a relationship for the Airline ID attribute of Routes, referencing to the ID attribute of Airlines (obviously).

```sh
docker pull neo4j
docker run \
    --publish=7474:7474 --publish=7687:7687 \
    --volume=$HOME/neo4j/data:/data \
    --env=NEO4J_AUTH=none \
    neo4j
```

Airport.dat

```sql
LOAD CSV FROM "https://raw.githubusercontent.com/jpatokal/openflights/master/data/airports.dat" AS row
CREATE (:Airport{
    AirportID:row[0],
    Name:row[1],
    City:row[2],
    Country:row[3],
    IATA:row[4],
    ICAO:row[5],
    Latitude:row[6],
    Longitude:row[7],
    Altitide:row[8],
    Timezone:row[9],
    DST:row[10],
    TzDB :row[11],
    Type:row[12],
    Source:row[13]});
CREATE INDEX ON :Airport(AirportID)
```

Airline.dat

```sql
LOAD CSV FROM "https://raw.githubusercontent.com/jpatokal/openflights/master/data/airlines.dat" AS row
CREATE (:Airline{
    AirlineID:row[0],
    Name:row[1],
    Alias:row[2],
    IATA:row[3],
    ICAO:row[4],
    Callsign:row[5],
    Country:row[6],
    Active:row[7]});
CREATE INDEX ON :Airline(AirlineID)
```

Route.dat

```sql
LOAD CSV FROM "https://raw.githubusercontent.com/jpatokal/openflights/master/data/routes.dat" AS row
CREATE (:Route{
    Airline:row[0],
    AirlineID:row[1],
    SourceAirport:row[2],
    SourceAirportID:row[3],
    DestinationAirport:row[4],
    DestinationAirportID:row[5],
    Codeshare:row[6],
    Stops:row[7],
    Equipment:row[8]});
CREATE INDEX ON :Route(AirlineID);
CREATE INDEX ON :Route(AirportID)
```

AirlineID--[FLIGHT]-> Route Relation:

```sql
MATCH (a:Airline), (r:Route{AirlineID:a.AirlineID}) MERGE (r)-[:FLIGHT]->(a);
```

AirportID--[SRC/DEST]-> Route Relation:

```sql
MATCH (a:Airport), (r:Route{SourceAirportID:a.AirportID}) MERGE (a)-[:SRC]->(r);
MATCH (a:Airport), (r:Route{DestinationAirportID:a.AirportID}) MERGE (r)-[:DEST]->(a);
```

b) Answer the eight given queries in Cypher for your task.

### Queries

a) How many airlines belong to Germany?

```sql
MATCH (a:Airline) WHERE a.Country = 'Germany' RETURN count(a)
-- OR
MATCH (a:Airline{Country:'Germany'}) RETURN count(a)
```

![result1](https://raw.githubusercontent.com/yazdipour/db-lab-assignments/master/tast2-result1.jpg?token=AB6QV52CSQUM2FQEJX4MQUS65HVEM)

b) How many flights are leaving from Frankfurt am Main Airport?

```sql
MATCH (:Route)<-[:SRC]-(:Airport{Name:'Frankfurt am Main Airport'}) RETURN COUNT(*)

╒══════════╕
│"COUNT(*)"│
╞══════════╡
│497       │
└──────────┘
```

Which of them are heading to an airport in Canada?

```sql
MATCH (r:Route)-[:SRC]-(:Airport{Name:'Frankfurt am Main Airport'})
MATCH (r)-[:DEST]-(a:Airport{Country:'Canada'}) RETURN DISTINCT a.Name

╒═════════════════════════════════════════════════════════╕
│"a.Name"                                                 │
╞═════════════════════════════════════════════════════════╡
│"Halifax / Stanfield International Airport"              │
├─────────────────────────────────────────────────────────┤
│"Ottawa Macdonald-Cartier International Airport"         │
├─────────────────────────────────────────────────────────┤
│"Montreal / Pierre Elliott Trudeau International Airport"│
├─────────────────────────────────────────────────────────┤
│"Vancouver International Airport"                        │
├─────────────────────────────────────────────────────────┤
│"Calgary International Airport"                          │
├─────────────────────────────────────────────────────────┤
│"Lester B. Pearson International Airport"                │
└─────────────────────────────────────────────────────────┘
```

c) Which airlines are flying **from or to** Frankfurt am Main Airport? We are looking for the
names of the airlines.

```sql
MATCH (r:Route) WHERE (r)-[:DEST]-(:Airport{City:"Frankfurt"}) OR (r)-[:SRC]-(:Airport{City:"Frankfurt"})
MATCH (r)-[FLIGHT]-(a:Airline) RETURN DISTINCT a.Name LIMIT 10

╒═══════════════════════════╕
│"a.Name"                   │
╞═══════════════════════════╡
│"American Airlines"        │
├───────────────────────────┤
│"Asiana Airlines"          │
├───────────────────────────┤
│"Adria Airways"            │
├───────────────────────────┤
│"Air Europa"               │
├───────────────────────────┤
│"Aegean Airlines"          │
├───────────────────────────┤
│"Aeroflot Russian Airlines"│
├───────────────────────────┤
│"Air France"               │
├───────────────────────────┤
│"Air Namibia"              │
├───────────────────────────┤
│"Azerbaijan Airlines"      │
├───────────────────────────┤
│"Air Berlin"               │
└───────────────────────────┘
```

d) How many airlines exist in the dataset?

```sql
MATCH (:Airline) RETURN COUNT(*)

╒══════════╕
│"COUNT(*)"│
╞══════════╡
│6162      │
└──────────┘
```

What is the percentage of airlines having no value for callsigns, compared to all of them?

```sql
MATCH (:Airline{Callsign:''}) WITH COUNT(*) AS c MATCH (a:Airline) RETURN 100.0*c/Count(a)

╒══════════════════╕
│"100.0*c/Count(a)"│
╞══════════════════╡
│13.112625770853619│
└──────────────────┘
```

e) How many airports are more than 150 times destination of a flight?

```sql
MATCH (:Route)-[:DEST]->(a:Airport) WITH COUNT(*) AS c, a WHERE c>150 RETURN COUNT(a)

╒═══════════════╕
│"COUNT(a)"     │
╞═══════════════╡
│104            │
└───────────────┘
```

How many airports are acting as source of flight more than 150 times?

```sql
MATCH (a:Airport)-[:SRC]->(:Route) WITH COUNT(*) AS c, a WHERE c>150 RETURN COUNT(a)

╒═══════════════╕
│"COUNT(a)"     │
╞═══════════════╡
│105            │
└───────────────┘
```

Are both numbers equal? **-NO-**

f) Which airlines are flying over 1000 routes? => names & number of routes they are flying.

```sql
MATCH (a:Airline)<-[:FLIGHT]-(:Route) WITH COUNT(*) AS count,a WHERE count>1000 RETURN a.Name, count

╒═════════════════════════╤═══════╕
│"a.Name"                 │"count"│
╞═════════════════════════╪═══════╡
│"American Airlines"      │2354   │
├─────────────────────────┼───────┤
│"Air France"             │1071   │
├─────────────────────────┼───────┤
│"Air China"              │1260   │
├─────────────────────────┼───────┤
│"China Eastern Airlines" │1263   │
├─────────────────────────┼───────┤
│"China Southern Airlines"│1454   │
├─────────────────────────┼───────┤
│"Delta Air Lines"        │1981   │
├─────────────────────────┼───────┤
│"easyJet"                │1130   │
├─────────────────────────┼───────┤
│"Ryanair"                │2484   │
├─────────────────────────┼───────┤
│"Southwest Airlines"     │1146   │
├─────────────────────────┼───────┤
│"United Airlines"        │2180   │
├─────────────────────────┼───────┤
│"US Airways"             │1960   │
└─────────────────────────┴───────┘
```

g) How many airports can be reached with a maximum of two flights starting from Frankfurt am Main Airport? Be aware of duplicates with respect to airport names; make sure that each airport is only counted as one!

```sql
MATCH (:Airport{Name:'Frankfurt am Main Airport'})-[:SRC]-(:Route)-[:DEST]-(:Airport)-[:SRC]-(:Route)-[:DEST]-(a:Airport)
 RETURN COUNT(DISTINCT a)

╒═══════════════════╕
│"COUNT(DISTINCT a)"│
╞═══════════════════╡
│1959               │
└───────────────────┘
```

h) If the source of a flight is Erfurt Airport, how many intermediate airports are necessary until there is a flight back to Erfurt Airport (e.g., Erfurt Airport→…→Erfurt Airport)?
What are the names of those airports allowing to travel to and from Erfurt? You can solve it with recursion.

```sql
MATCH (:Airport{Name:'Erfurt Airport'})-[:SRC *1..]-(:Route)-[:DEST *1..]-(:Airport{Name:'Erfurt Airport'}) RETURN COUNT(*)

╒═══════════════╕
│"COUNT(*)"     │
╞═══════════════╡
│0              │
└───────────────┘
```

### Two more queries that a user could pose to this graph database

i) What are the top 10 most crowded airports - sorted?

```sql
MATCH (a:Airport)-[:SRC]-(:Route) WITH COUNT(*) AS c, a RETURN a.Name,c ORDER BY c DESC LIMIT 10

╒══════════════════════════════════════════════════╤═══╕
│"a.Name"                                          │"c"│
╞══════════════════════════════════════════════════╪═══╡
│"Hartsfield Jackson Atlanta International Airport"│915│
├──────────────────────────────────────────────────┼───┤
│"Chicago O'Hare International Airport"            │558│
├──────────────────────────────────────────────────┼───┤
│"Beijing Capital International Airport"           │535│
├──────────────────────────────────────────────────┼───┤
│"London Heathrow Airport"                         │527│
├──────────────────────────────────────────────────┼───┤
│"Charles de Gaulle International Airport"         │524│
├──────────────────────────────────────────────────┼───┤
│"Frankfurt am Main Airport"                       │497│
├──────────────────────────────────────────────────┼───┤
│"Los Angeles International Airport"               │492│
├──────────────────────────────────────────────────┼───┤
│"Dallas Fort Worth International Airport"         │469│
├──────────────────────────────────────────────────┼───┤
│"John F Kennedy International Airport"            │456│
├──────────────────────────────────────────────────┼───┤
│"Amsterdam Airport Schiphol"                      │453│
└──────────────────────────────────────────────────┴───┘
```

j) What is the percentage of the routes without stops.

```sql
MATCH (r:Route) with Count(r) as c MATCH (r) WHERE r.stops="0" RETURN 100.0*COUNT(r)/c

╒═══════════════════╕
│"100.0*COUNT(rr)/c"│
╞═══════════════════╡
│99.98374296144127  │
└───────────────────┘
```
