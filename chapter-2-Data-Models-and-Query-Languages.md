# Chapter 2. Data Models and Query Languages

## Summary
> Overview of data models and their different purposes. Three main types of datastores: document, relational, and graph.
---
## Notes
1. We strive for effective representation of data models across layers of other data models. How one layer of data model is seen by another layer, and what information is transferred across layers.

2. 1970s SQL dominated and was the standard for the next 25-30 years. Then NoSQL came in. NoSQL adoption was driven by:
   1. Scalability that relational databases didn't have then.
   2. Preferences for open source software over commercial database products.
   3. Specialised query operations.
   4. More dynamic and expressive data models.

3. Concept of impedance mismatch: the disconnnect and extra translation layer required between data models. Happens when dealing with objects in OOP code and relational databases.

4. Most data models are framed by the types of relationships they are handled in: Many-to-One, Many-to-Many, One-to-Many.
   1. Many to one relationships (industry or region subsets) would be great to be sorted into IDs instead of plain-text strings using a drop-down list or autocompleter. Benefits are:
      1. Consistent style and spelling
      2. Avoiding ambiguity (many locations with the same name)
      3. Ease of updating name related to id
      4. Localisation support for different languages
      5. Better search. Klang is in Selangor but this is not apparent in a "Klang" string.

5. There exists many hybrids of models as relational and document databases become more similar.

6. Imperative and declarative query language for data.
   1. Imperative language is like programming languages.
      1. hard to parallelise across multiple cores and machines because of instructions follow order.
   2. Declarative languages is like SQL where you say what you want, hides implementation details, and let the system's query optimiser to decide on execution method.
      1. Easier to parallelise

7. MapReduce is a query language for processing large amounts of data. Not declarative nor imperative. MongoDB use this then after did their own aggregation pipeline (declarative).

### Models

Network Model (CODASYL model)

1. Links between records in the network model were like pointers in C.
2. Access record from root using an access path, like a linked list. Many access paths to reach one record so need to remember them.
3. Back then limited hardware capabilities so manual access path was efficient. But complicated code and inflexible data.

Relational Model

1. Everything laid out in a collection of rows.
2. Better support for joins.
3. Access path here is an automatic query optimiser, not developer code.
4. Query optimisers are complicated and a lot of R&D effort put in. But only need to build once and benefit all applications. General-purpose solution like this won in the long run.
5. Declarative query language

Document Model

1. Benefits are:
   1. Schema flexibility
   2. Locality performance benefits
   3. Closer to the data structure in application.
2. Schema-on-read - data structure implicit and interpreted only when read. Not really schemaless because some structure is usually assumed.

Graph Models

1. Ideal for data with common many to many relationships. EG social graphs, road networks etc.
2. Objects in graph are vertices(nodes) and edges(relationships).
3. Property graph models (neo4j, titan, infinitegraph)
   1. Vertex has identifier, ourgoing and incoming edges, key value pairs.
   2. Edge has identifier, head and tail vertex where edge starts and ends, label for relationship between vertices, key value pairs.
   3. neo4j uses cypher query language
4. Triple-store model (datomic, allegrograph)
   1. All info stored in three-part statements: subject, predicate, object.
   2. SPARQL query language (sparkle) predates cypher.
5. Datalog query language was a foundation for query languages. Used in datomic. Similar to triple-store but written differently. 
   1. Requires a different kind of thinking because of various predicates.
   2. Build things one by one but can cope with complex data.

---
## Things to Think About

Go through all the data models in all the layers of your application and think whether it is effective for the relationships associated with the data.