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

Ontology Versioning
===================

Brick models (digital representations of buildings) are often developed against a specific version of the Brick ontology.
As the Brick ontology evolves, it is important to keep track of which version of the Brick ontology is required for a particular model.

This is accomplished through use of of (a) *ontology declarations*, and (b) *ontology imports*.

All Brick models should explicitly declare the graph which contains the statements modeling the building.
An ontology declaration is a triple of the form:

```ttl
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .

<graph name> rdf:type owl:Ontology .
```

`<graph name>` is an RDF IRI, and is *usually* the same as the namespace that is used to define the entities in the building:

```ttl
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix brick: <https://brickschema.org/schema/Brick#> .
@prefix bldg: <urn:my_building#> .

<urn:my_building> a owl:Ontology .
```

Since v1.2, all versions of the Brick ontology have the same namespace: `https://brickschema.org/schema/Brick#`.
This makes it easier to upgrade the Brick ontology used with a particular model without having to change all of the classes, relationships, etc used by that model to use a different namespace.
Instead, we use *ontology imports* to track the dependency on a particular version of Brick.

Ontology imports are properties of the `owl:Ontology` entity which point to other graphs via the `owl:imports` relationship.

```ttl
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix brick: <https://brickschema.org/schema/Brick#> .
@prefix bldg: <urn:my_building#> .

<urn:my_building> a owl:Ontology ;
    owl:imports <https://brickschema.org/schema/1.2/Brick#> .
```

The IRI of the *imported* graph has the version of the Brick ontology embedded in it.
This *versioned* IRI points to the version of the Brick ontology indicated in the IRI; for example, `https://brickschema.org/schema/1.2/Brick` is `v1.2` of Brick, `https://brickschema.org/schema/1.3/Brick` is `v1.3` of Brick, and so on.
To update Brick, it is only necessary to change the object of the `owl:imports` statement.
This is possible because the IRI that *imports* Brick will always resolve to a file that uses the *version-agnostic* Brick namespace `https://brickschema.org/schema/Brick#`.
