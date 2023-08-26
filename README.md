# SPARQLGEN

SPARQLGEN is a one-shot generative approach 
for generating SPARQL queries by augmenting LLMs with the relevant context 
within a single prompt. 
The prompt includes heterogeneous data sources: a question itself, an RDF subgraph required to answer the question, 
and an example of a correct SPARQL query for a different question.

##  Supplemental material
This repository contains supplementary material for the corresponding poster, 
accepted to SEMANTICS 2023 Conference, namely:

* BESTIARY KG and dataset
  * Dataset with 100 datapoints in QALD-format
  * Authentic RDF knowledge graph structuring the Dungeons & Dragons domain
* Augmented data for the datasets, used during the experiments:
  * QALD-9: https://github.com/danrd/sparqlgen/blob/main/normalized_qald9_test_with_entities_relations_graph_fragment_full_sparql_query.json
  * QALD-10: https://github.com/danrd/sparqlgen/blob/main/normalized_qald_10_entities_relations_with_graph_full_sparql_query.json
  * BESTIARY: https://github.com/danrd/sparqlgen/blob/main/normalized_beast_kg_queries_ver2_cleaned_full_sparql_query.json
* Error analysis:
  * detailed comparison of target and generated queries for QALD-10 with labeled error types  https://github.com/danrd/sparqlgen/blob/main/QALD10%20queries%20-%20WrongQueriesErrorAnalysisQALD10.csv
  * error types distribution across 3 datasets https://github.com/danrd/sparqlgen/blob/main/errors.png
* Images:
  * SPARQLGEN pipeline architecture and prompt elements https://github.com/danrd/sparqlgen/blob/main/sparqlgen_prompt_element-diagram.png
  * Subgraph extraction algorithm, that was designed for augmenting the original datasets https://github.com/danrd/sparqlgen/blob/main/subgraph_extraction.png
* Supplemental material for the paper:
  * Proposed error classification for LLM generated SPARQL queries
  * Description of subgraph extraction algorithm 

## BESTIARY KG and dataset
BESTIARY RDF knowledge graph https://github.com/danrd/sparqlgen/blob/main/beastiary_kg.rdf.zip

BESTIARY dataset in QALD-format https://github.com/danrd/sparqlgen/blob/main/beastiary_with_qald_format.json.zip

We release .zip versions of BESTIARY sources to avoid direct  parsing of these data for training LLMs.

The BESTIARY dataset was specifically designed to test the performance of LLM SPARQL generation 
and consists of 100 manually created queries related
to a custom BESTIARY knowledge graph. The knowledge graph contains diverse
information about creatures from the Dungeons &  Dragons fantasy role-playing
game. Specifically, a creature is characterized by an alignment type, a set of spoken
languages and a range of numerical attributes. 
Neither the knowledge graph, nor the dataset were ever presented to the language model. Thus it is possible to use them
to evaluate some of the LLM reasoning skills assuming that the model haven't memorized any snippets of these two reseources.
Each element of the dataset
comprises of the following: 
* (1) a natural language question; 
* (2) a corresponding
SPARQL query; 
* (3) a list of entities from the question; 
* (4) a list of relations from
the question; 
* (5) the SPARQL query where entities and relations are replaced
with placeholders < ENT1 >, . . . ,< ENTN > and < REL1 >, . . . ,< RELN >; 
* (6) a subgraph of BESTIARY knowledge graph which is sufficient to retrieve the
answer. 

The complexity of the questions varies with respect to several aspects.
Firstly, the questions differ in terms of the number of entities and relations. And
secondly, the dataset includes questions with multi-hop reasoning (e.g., ”What
values of experience points are related to creatures with the highest level?”), comparatives
(e.g., ”Is Sangudaemon faster than Eblis?”), superlatives (e.g., “What
alignment type is the most common for elementals?”), counts (e.g., ”How many
creatures with non-neutral alignment do speak treant language?”) and other
aggregators (e.g., ”What is the average attack parameter for robots?”).



## Proposed error classification for LLM generated SPARQL queries

Objective evaluation with F1-macro has shown quite diverse results across the 3
datasets. Therefore, we tried to categorize the errors in a small set of categories,
listed below, that cover all types of errors witnessed during LLM inference. Error
distribution is shown in Fig. 3. In all datasets the following classes of errors
appear:
1. **syntax errors** - this category includes all malformed queries that cannot
be executed due to the presence of syntax errors.
2. **wrong query type** - queries with the wrong query type, namely SELECT
instead of ASK.
3. **wrong query pattern** - misuse of matching alternatives, negation, property
paths, assignment or aggregates (BIND, UNION, FILTER, COUNT etc.).
We included here both missed and hallucinated options.
4. **KG structure ignored** - misused or hallucinated literals, entities and relations.
5. **incomplete query** - cut queries.
6. **missing query** - generated string does not contain a query (SELECT keyword).
7. **angle brackets missing** - missing brackets around URIs in generated query.
8. **incomplete triple** - generated query contains a triple with less than 3
elements.
9. **namespace mistake** - hallucanated prefixes or namespaces in the body of
a query.

## Description of subgraph extraction algorithm 

To generate SPARQL queries from natural language questions it is very useful
to have the structure of the underlying knowledge graph available. The same information can be conceptualized in different ways
and SPARQL queries for the same question will differ for different knowledge graphs. Thus, it makes sense
to introduce the KG to the LLM beforehand.
In order to investigate, whether adding sub-graph
information to the model improves the performance, we enriched each sample in
the applied test sets with a sub-graph of the source RDF graph of that dataset.

These sub-graphs contain all the triples that are required to answer the question
correctly, but no irrelevant triples.
First, we strip the ground truth SPARQL queries from any modifiers leaving
a simple SELECT * query. By doing that, we get all possible bindings
for the variables in the ground truth query. Second, we extract the individual
triples from the query. They contain entities, relations but also still the variables.
Third, we take the result bindings from the SELECT * query and replace the
variables in the extracted triples. The result is a set of triples, which is based
on the original ground truth query, representing the sub-graph that is sufficient
to answer the given query. Further, this sub-graph is considerably smaller than
taking all triples from all the given entities and relations of the ground truth
query. The disadvantage of this approach is that it will not work on inference,
since it requires the ground truth data. There, sub-graph extraction based on
n-hops around extracted entities and relations will be an option instead. We
still chose our approach for this paper in order to keep the sub-graph as small
as possible, so that it can fit into the GPT-3 context windows. This allows us
to evaluate, whether or not providing the sub-graph improves SPARQL query
generation and focus on improving sub-graph extraction in future work.
The subgraph extraction process is shown in https://github.com/danrd/sparqlgen/blob/main/subgraph_extraction.png.



