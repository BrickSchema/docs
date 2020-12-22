# Database Backends for Brick

In this section, we provide an incomplete list of available commercial and open-source databases which support Brick features.

## Feature List

- **RDF model storage**: does the database support the storage of RDF models (this should be *yes* for all of the below databases)
- **SPARQL support**: does the database support the execution of SPARQL queries? The relevant standards are:
    - [SPARQL 1.1 Query](https://www.w3.org/TR/sparql11-query/) (including property paths)
    - [SPARQL 1.1 Update](https://www.w3.org/TR/sparql11-update/) (the ability to write to the database using SPARQL)
- **Reasoning/Inference**: does the database support [reasoning and inference](../lifecycle/inference) on stored RDF models? Relevant features are:
    - what inference is supported: OWL-DL, OWL-RL, RDFS, OWL-Full, SHACL-AF
    - manual or automatic: is reasoning performed as a manual or batch processing task
- **SHACL support**: does the database support the validation of RDF models using SHACL shapes?
- **Multiple Graphs**: does the database support the storage of multiple graphs, and can those graphs be updated/queried independently?
- **Scaling and Performance**: what are the scaling and performance properties of the database? This includes horizontal scaling capabilities, query performance and storage requirements

## Database List

```{admonition} Info
:class: tip
If there is a missing database or our documentation is incomplete, please [file an issue](https://github.com/BrickSchema/docs/issues/new)
```

### RDFlib + Related Libraries

RDFlib and its related libraries are open-source, production-quality Python libraries for working with RDF data.

Links:
- [RDFlib GitHub](https://github.com/RDFLib/rdflib)
- [pySHACL GitHub](https://github.com/RDFLib/pySHACL)
- [OWL-RL GitHub](https://github.com/RDFLib/OWL-RL)

Features:
- **RDF model storage**: RDFlib supports the storage of RDF models in several [possible backends](https://rdflib.readthedocs.io/en/stable/persistence.html), including an in-memory store by default
- **SPARQL support**: RDFlib contains a complete implementation of SPARQL 1.1 (Query and Update).
- **Reasoning/Inference**: The OWL-RL library supports inference for the RDFS and OWL-RL ontology languages
- **SHACL support**: pySHACL provides a complete implementation of SHACL and SHACL Advanced Features
- **Multiple Graphs**: RDFlib can easily support multiple graphs via its incorporation in a larger Python program

- **Scaling and Performance**: scaling of storage can be provided by the different storage backends supported by RDFlib; additional backends can be added easily. RDFlib's implementation of SPARQL and OWL-RL are not heavily optimized so performance can suffer on larger graphs.

```{admonition} TODO
:class: warning
Does RDFlib provide an graph management API? Or is it all handled externally through Graph objects?
```

### HodDB

HodDB is an open-source, high-performance, research-quality query processor

Links:
- [HodDB GitHub](https://github.com/gtfierro/hoddb)
- [HodDB paper (BuildSys 2017)](http://people.eecs.berkeley.edu/~gtfierro/papers/hoddb.pdf)
- [Extended HodDB paper (TOSN 2018)](http://people.eecs.berkeley.edu/~gtfierro/papers/hoddb_tosn.pdf)

Features:
- **RDF model storage**: HodDB supports the storage of RDF models on a local filesystem, using BadgerDB
- **SPARQL support**: HodDB supports a subset of SPARQL 1.1, including property path queries and UNION (specifically *not* inserts). See the HodDB papers for a more precise list. HodDB also does not support the SPARQL API protocol, and instead requries use of a specific GRPC-based API
- **Reasoning/Inference**: HodDB supports some basic inference --- namely handling OWL inverse and transitive properties.
- **SHACL support**: HodDB does not support SHACL
- **Multiple Graphs**: HodDB supports the storage of multiple graphs which can be queried separately
- **Scaling and Performance**: Due to the restricted query and update model, HodDB provides excellent SPARQL query performance on graphs up to a few hundred thousand nodes. HodDB currently does not support distributed storage

### Allegrograph

Allegrograph is a proprietary, commercial graph database supporting RDF and related technologies. Allegrograph also provides a free version with limited features

Links:
- https://franz.com/agraph/allegrograph/

Features:
- **RDF model storage**: The paid version of Allegograph supports horizontally scalable storage of RDF models; the base version stores RDF models on a single node
- **SPARQL support**: Allegrograph supports full SPARQL 1.1
- **Reasoning/Inference**: Allegrograph supports both [OWL-RL](https://franz.com/agraph/support/documentation/current/materializer.html) and [RDFS](https://franz.com/agraph/support/documentation/current/reasoner-tutorial.html) languages
- **SHACL support**: Allegrograph supports [SHACL validation](https://franz.com/agraph/support/documentation/current/shacl.html) but does not seem to support SHACL advanced features such as inference
- **Multiple Graphs**: unknown
- **Scaling and Performance**: Allegrograph demonstrates good performance on graphs typical of Brick

### Oxigraph

### Apache Jena

### Blazegraph

### TopBraid
