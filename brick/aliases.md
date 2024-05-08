---
jupytext:
  formats: md:myst
  text_representation:
    extension: .md
    format_name: myst
kernelspec:
  display_name: Python 3
  language: python
  name: python3
---

Aliases and Equivalent Classes
==============================


```{note}
This feature is new in Brick v1.4
```

Over its history, Brick has accumulated several duplicate class names. For example: `brick:AHU`, `brick:Air_Handler_Unit` and `brick:Air_Handling_Unit` can all be found in the Brick 1.4 release.
Brick retains all of these classes for backwards-compatibility; however, in Brick 1.4 it became necessary to denote a *preferred* class among a set of interchangeable classes.
At the same time, downstream tools and consumers should remain agnostic to whichever class was chosen by a modeler.
Brick 1.4 handles both of these features through two mechanisms: *equivalent classes* and *aliases*.

## Aliases

Aliases identify the *preferred* class among a set of interchangeable (equivalent) classes.
Non-preferred classes will have a `brick:aliasOf` property which indicates the preferred class.

See below a snippet of Brick 1.4 containing definitions of `AHU`, `Air_Handler_Unit` and `Air_Handling_Unit`. Only `brick:Air_Handling_Unit` is missing the `brick:aliasOf` property, so it is the preferred class. The other classes have a `brick:aliasOf` property pointing to this preferred class.

```ttl
brick:AHU a owl:Class, sh:NodeShape ;
    rdfs:label "AHU" ;
    owl:equivalentClass brick:Air_Handling_Unit ;
    brick:aliasOf brick:Air_Handling_Unit .

brick:Air_Handler_Unit a owl:Class, sh:NodeShape ;
    rdfs:label "Air Handler Unit" ;
    owl:equivalentClass brick:Air_Handling_Unit ;
    brick:aliasOf brick:Air_Handling_Unit .

brick:Air_Handling_Unit a owl:Class, sh:NodeShape ;
    rdfs:label "Air Handling Unit" ;
    rdfs:subClassOf brick:HVAC_Equipment ;
    owl:equivalentClass brick:AHU,
        brick:Air_Handler_Unit ;
    brick:hasAssociatedTag tag:AHU,
        tag:Air,
        tag:Equipment,
        tag:Handler,
        tag:Handling,
        tag:Unit .
```

### Listing all Preferred Classes

This SPARQL query lists all preferred classes in Brick:

```sparql
PREFIX brick: <https://brickschema.org/schema/Brick#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?preferred WHERE {
    ?preferred a owl:Class ;
               rdfs:subClassOf* brick:Entity .
    FILTER NOT EXISTS { ?preferred brick:aliasOf ?alias }
}
```

Here is how to run this query on the latest Brick release:

```{code-cell}
from rdflib import Graph
brick = Graph()
brick.parse("https://github.com/BrickSchema/Brick/releases/download/nightly/Brick.ttl", format="ttl")
res = brick.query("""
PREFIX brick: <https://brickschema.org/schema/Brick#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?preferred WHERE {
    ?preferred a owl:Class ;
               rdfs:subClassOf* brick:Entity .
    FILTER NOT EXISTS { ?preferred brick:aliasOf ?alias }
}
LIMIT 10
""")
for row in res.bindings:
    print(row)
```

### Getting the Preferred Class for an Entity

This SPARQL query gets the preferred class for a specific Brick class:

```sparql
PREFIX brick: <https://brickschema.org/schema/Brick#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT ?preferred WHERE {
    { ?class brick:aliasOf ?preferred }
    UNION
    {
        BIND ( ?class AS ?preferred )
        FILTER NOT EXISTS { ?class brick:aliasOf ?other }
    }
}
```

Here is how to run this query on the latest Brick release. Adjust the `initBindings` paramete to change which class you are querying the alias for.

```{code-cell}
from rdflib import Graph, BRICK
brick = Graph()
brick.parse("https://github.com/BrickSchema/Brick/releases/download/nightly/Brick.ttl", format="ttl")

query = """
PREFIX brick: <https://brickschema.org/schema/Brick#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
SELECT ?preferred WHERE {
    { ?class brick:aliasOf ?preferred }
    UNION
    {
        BIND ( ?class AS ?preferred )
        FILTER NOT EXISTS { ?class brick:aliasOf ?other }
    }
}"""

# try this query for an alias
res = brick.query(query, initBindings={"class": BRICK.AHU})
for row in res.bindings:
    print(row)

# try this query for a preferred calss
res = brick.query(query, initBindings={"class": BRICK.Air_Handling_Unit})
for row in res.bindings:
    print(row)
```

## Equivalent Classes

Groups of duplicate classes (e.g., `AHU`, `Air_Handler_Unit` and `Air_Handling_Unit`) will also be marked as *equivalent* classes using the `owl:equivalentClass` property.
There are two SHACL rules in Brick (`bsh:OWLEquivalentClassRule1` and `bsh:OWLEquivalentClassRule2`) which handle the semantics of equivalent classes: when SHACL inference is run on a Brick model, these rules will add *all* the equivalent classes to each entity.

For example, consider a simple Brick model with a single AHU instance:

```ttl
@prefix brick: <https://brickschema.org/schema/Brick#> .
@prefix : <urn:bldg#> .

:myAHU a brick:AHU .
```

After SHACL inference, the model will contain:

```ttl
@prefix brick: <https://brickschema.org/schema/Brick#> .
@prefix : <urn:bldg#> .

:myAHU a brick:AHU, brick:Air_Handler_Unit, brick:Air_Handling_Unit .
```

This means that *after running SHACL inference*, one can use the preferred Brick classes to find an entity.
It is not necessary to know which (equivalent) class was used.
