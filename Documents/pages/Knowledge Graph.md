# What is a Knowledge Graph?**
	- In Logic Programming, KG is logically consistent collection of facts
	- > Data Integration + Concepts AND Relationships are first class citizens = Linked Data and Metadata encoded in a Graph
	- > Ontology + Instance Data = Knowledge Graph
	- KGs bridge the data-meaning gap between data producers and consumers.
- # Glossary
	- Taxonomy
		- Taxonomy is hierarchical
		- Multi-lingual taxonomy is used for searching data in English within documents that are in non-English
	- Ontology
- # What is in a Knowledge Graph?
	- A semantic graph database (scalable, secure, acid)
	- Ontologies and taxonomies
	- Reasoning and rule based processing
	- Smart integration of silos of information
	- Machine Learning and Advanced Analytics
	- Natural Language Processing and Text classification
- # Types of Knowledge Graphs
	- Document/text based KG
	- Entity/Event bsaed KG
- # Representations of Knowledge Graph
	- Symbolic representation - `s,p,o` (triplet), `p (s,o)`, `(s,p,o)`
		- `s` = subject, `p` = predicate, `o` = object
		- used in logical inference systems, DB and DB integration systems
	- Vector representation - `s,p,o ε `
		- used in NLP
- # Methodologies
	- Creata Data/Execute Queries
		- there are 2 approaches: Materialization (ETL) and Virtualization (No ETL)
		- __Materialization__: Create an ETL job to extract the source data from RDBMS, apply the mapping defined, then transform and load it to a graph DB.
		- __Virtualization__: Take SPARQL queries as input, apply the mapping logic and convert it into a SQL to extract the data. The results are then stored in a graph DB.
- # Tools
	- Data modelling: RDF, RDFS, OWL?
	- Query language
		- SPARQL
			- Complex query patterns
			- 8.5 million queries in wikidata daily
	- Ontologies
		- [schema.org](https://schema.org)
			- universal graph labels
			- labels for graph (open)
			- 31% of the internet uses this
		- Good Relations, FIBO, Gist
		- [VOWL](http://vowl.visualdataweb.org): Visual Notation for OWL Ontologies
		- [WebVOWL](http://vowl.visualdataweb.org/webvowl.html) - Web-based visualization of Ontologies
		- [Protégé](https://protege.stanford.edu) - Free, open-source Ontology editor - has VOWL plugin
	- Mapping from RDBMS to RDF graphs: R2RML
	- Visualization - [Grafo](https://gra.fo)
- # Examples
	- > Examples of public data sources using KG
	- Wikidata - 80 million nodes, 4800+ catalogs
	- Datacommons.org - Knowledge browser,
	- Library of Congress
	- data.gov
	- census.gov
	- NOAA
	- [The Linked Open Data Cloud](https://lod-cloud.net)
	- [data.world](https://data.world) - is a cloud-native enterprise data catalog, with data virtualization/federation, build on Knowledge Graph
- # Questions
	- How to get multiple, complex, disparate, heterogenous databases into a KG of meaningful data?
	- Difference between Data Lake and KG?
- # References
	- Videos - Knowledge Graph Seminar Session from Stanford 2020
		- [Session 1](https://www.youtube.com/watch?v=bvwjG-3qAmY&ab_channel=VinayKChaudhri), [Session 2](https://www.youtube.com/watch?v=ZWM-Dlw3VCM), [Session 3](https://www.youtube.com/watch?v=5RXVorAglVc)
	- Books
		- Linked Data
		- The Technology Fallacy
		- What is Semantic Web?
	- [History of Knowledge Graphs](https://knowledgegraph.today)
	- https://www.dagstuhl.de/18371
	- https://arxiv.org/abs/2003.02320