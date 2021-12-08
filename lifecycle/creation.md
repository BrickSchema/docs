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

Creating a Brick Model
======================

A Brick model must be created, derived or otherwise produced in order to enable data-driven applications and deliver value.
There are several ways of creating a Brick model for a building.
The most appropriate choice of method or tool depends on a number of factors:
- whether there is existing metadata or an existing digital representation of the building
- the format or standard that defines any digital representation
- how standardized, regular or predictable that digital representation is

The rest of this section describes common ways of creating a Brick model, organized by what form of source information is available.


## From Scratch

A Brick model can be created from scratch by explicitly listing the contents of the graph.
This is often helpful when creating [small examples](https://github.com/BrickSchema/Brick/tree/master/examples), when no existing structured information is available, or when the building is best described programatically.

There is no prescribed methodology for creating a Brick model from scratch.
A common pattern is to use data structures to represent collections of similar information (e.g. all rooms in a building, all VAVs and their points, etc) and then to traverse those data structures, creating triples in a graph along the way.

### Software

- [brickschema](https://brickschema.readthedocs.io/) (Python): Python package over RDFlib that provides Brick-specific features and APIs for working with Brick models
- [RDFlib](https://rdflib.readthedocs.io/en/stable/) (Python): Python package for working with RDF graphs

### Example

The following example uses the [brickschema](https://brickschema.readthedocs.io/) package to create a simple Brick model:


```python
import brickschema
from brickschema.namespaces import A, BRICK, UNIT
from rdflib import Namespace, Literal

# create a namespace for the building
BLDG = Namespace("urn:my-building-name#")

# create a graph object to store the Brick model
g = brickschema.Graph()
g.bind("bldg", BLDG)

# create a datastructure for floors + rooms
rooms_and_floors = {
    "Floor1": ["Room1", "Room2", "Room3"],
    "Floor2": ["Room4"],
}

for floor, room_list in rooms_and_floors.items():
    # Use the strings in the datastructure to refer to entities in the Brick model.
    # By putting "BLDG[floor]" into the graph, we implicitly create the entity.
    g.add((BLDG[floor], A, BRICK.Floor))
    for room in room_list:
        g.add((BLDG[room], A, BRICK.Room))
        g.add((BLDG[room, BRICK.isPartOf, BLDG[floor]))

# save the file to disk
g.serialize("my-building.ttl", format="ttl")
```

Also see the Brick [examples folder](https://github.com/BrickSchema/Brick/tree/master/examples) which contains a few examples of creating sample models from scratch.

## From Structured Tabular Sources

Structured tabular sources like CSV files, schedules and COBie sheets are often good sources of metadata.
The Brick ecosystem contains several tools that help describe how this structured data can be transformed into a Brick model.
Most of these tools operate by
- use a mapping between the terms/types in the source data and their corresponding Brick classes to create entities
- use the relationships between columns in the source to define the relationships between those entities

Generally, this means that each *row* of the input data results in the creation of a subgraph containing some entities, their types, and the relationships between them.

### Software
- [brickify](https://brickschema.readthedocs.io/en/latest/brickify/index.html): a robust tool for describing transformations to Brick from structured sources like tables (spreadsheets, CSV files)
- [brick-builder](https://github.com/gtfierro/brick-builder): like `brickify` but simpler and with fewer features. Uses templates to describe how CSV rows should be turned into a Brick model.
- [rule-based-model-builder](https://github.com/gtfierro/rule-based-model-builder): experimental framework for expressing complex transformations of structured data to a Brick model; includes COBie translation.

### Example

#### Brickify

Assume a spreadsheet with the following data

VAV name | temperature sensor | temperature setpoint | has_reheat
---------|--------------------|----------------------|-----------
A | A_ts | A_sp | false
B | B_ts | B_sp | true

The following `brickify` config file expresses how to create VAV objects in Brick from that spreadsheet, with their corresponding sensors. It optionally creates the VAV with Reheat type when prompted. Notice that the types of the entities in the rows are determined by the column name and that this is baked into the config file.

```yaml
---
namespace_prefixes:
  brick: "https://brickschema.org/schema/Brick#"
  bldg: "http://mybuildings.com/mybuilding#"
  rdf: "http://www.w3.org/1999/02/22-rdf-syntax-ns#"
operations:
  -
    data: |-
      bldg:{VAV name} rdf:type brick:VAV ;
                      brick:hasPoint bldg:{temperature sensor} ;
                      brick:hasPoint bldg:{temperature setpoint} .
      bldg:{temperature sensor} rdf:type brick:Temperature_Sensor .
      bldg:{temperature setpoint} rdf:type brick:Temperature_Setpoint .
  -
    conditions:
      - |
        '{has_reheat}'
    data: |-
      bldg:{VAV name} rdf:type brick:RVAV .
```

This creates the following Brick model

```turtle
@prefix brick: <https://brickschema.org/schema/Brick#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix mybldg: <http://mybuildings.com/mybuilding#> .

bldg:A rdf:type brick:VAV ;
                brick:hasPoint bldg:A_ts ;
                brick:hasPoint bldg:A_sp .
bldg:A_ts rdf:type brick:Temperature_Sensor .
bldg:A_sp rdf:type brick:Temperature_Setpoint .

bldg:B rdf:type brick:VAV, brick:RVAV ;
                brick:hasPoint bldg:B_ts ;
                brick:hasPoint bldg:B_sp .
bldg:B_ts rdf:type brick:Temperature_Sensor .
bldg:B_sp rdf:type brick:Temperature_Setpoint .
```


### Tutorials
- [Creating a Brick model with Brick Builder and OpenRefine](https://www.youtube.com/watch?v=LKcXMvrxXzE)

## From Haystack Models

Translation from Project Haystack <4.0 models is available in [brickify](https://brickschema.readthedocs.io/en/latest/brickify/index.html#haystack-handler). Translation for Haystack 4.0 models is in development.

### Software
- [brickify](https://brickschema.readthedocs.io/en/latest/brickify/index.html): a robust tool for describing transformations to Brick from structured sources like tables (spreadsheets, CSV files)

## From IFC Models

Currently, only support for COBie spreadsheets has been developed; see the software below.

Support for translation from IFC models directly is in development.

### Software
- [rule-based-model-builder (COBie)](https://github.com/gtfierro/rule-based-model-builder/tree/main/examples/cobie): implementation of a COBie translator using the `rule-based-model-builder` package

## From BMS Point Labels

One of the most common forms of existing building metadata are BMS point labels. Unfortunately, these often lack consistent structure, or follow ad-hoc naming conventions which are rarely documented.
The primary challenge when dealing with these sources is figuring out what each entitity is.
Each point label explicitly corresponds to a Brick Point instance, but the label itself may also contain additional information describing related equipment, other assets or even some of the building topology.

The reconcilation API server linked below will attempt to infer the type of a point from common abbreviations in its label. This is a helpful tool for implementating translation software for different point naming schemes.

### Software
- [Brick Reconciliation API Server](https://github.com/BrickSchema/reconciliation-api): an implementation of the W3C reconciliation api for use with OpenRefine
- [OpenRefine](https://openrefine.org/): locally-hosted web-based tool for manipulating mess ydata

### Tutorials
- [Creating a Brick model with Brick Builder and OpenRefine](https://www.youtube.com/watch?v=LKcXMvrxXzE)
