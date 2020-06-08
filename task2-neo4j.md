# Lab Training Database Systems SS20

## Shahriar Yazdipour [matr. No:62366]

## Task 2

The data that is used here can be loaded from <https://openflights.org/data.html>.

a) Load the three data files into Neo4J. You can manually download and copy them into the
Docker container, or use their direct website links in Neo4J to load them. Do not forget to
rename the attributes according to their description on the website, else it might be hard to
query them later.

In addition, create relationships between them where necessary – e.g. a relationship for the
Airline ID attribute of Routes, referencing to the ID attribute of Airlines (obviously).
If you are not sure if everything works correctly, feel free to check your design by using
only a few tuples/nodes and displaying it in your browser.

=> For the PDF: Add all used commands to load the data, to rename the attributes, and to
create the relationships.

b) Answer the eight given queries in Cypher for your task.

=> For the PDF: Add your solution to the queries as well as the result from execution. If the
result is too big (e.g. a thousand nodes), limit the output to 10 by adding LIMIT 10 to your
solution.

### Queries:

a) How many airlines belong to Germany?

b) How many flights are leaving from Frankfurt am Main Airport? Which of them are heading
to an airport in Canada?

c) Which airlines are flying from or to Frankfurt am Main Airport? We are looking for the
names of the airlines.

d) How many airlines exist in the dataset? What is the percentage of airlines having no value
for callsigns, compared to all of them?
Calculate that percentage directly in Neo4j!

e) How many airports are more than 150 times destination of a flight? How many airports are
acting as source of flight more than 150 times? Are both numbers equal?

f) Which airlines are flying over 1000 routes? We are looking for their names along with their
exact number of routes they are flying.

g) How many airports can be reached with a maximum of two flights starting from Frankfurt
am Main Airport (e.g., starting from Frankfurt am Main Airport, flying to Singapore Changi
Airport and then further to Sydney Kingsford Smith International Airport)?
Be aware of duplicates with respect to airport names; make sure that each airport is only
counted as one!

h) If the source of a flight is Erfurt Airport, how many intermediate airports are necessary until
there is a flight back to Erfurt Airport (e.g., Erfurt Airport→…→Erfurt Airport)?
What are the names of those airports allowing to travel to and from Erfurt?
You can solve it with recursion.

### Two more queries that a user could pose to this graph database

i) AAAA

j) BBBBB
