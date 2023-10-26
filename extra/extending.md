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

Extending Brick
===============

This page documents best practices for extending Brick. Extending Brick may include adding new classes, new relationships, new entity properties, new documentation or ontology rules.

Whenever possible, it is recommended to submit extensions to Brick as a pull request on the [GitHub repository](https://github.com/BrickSchema/Brick) so that they can be made available to the broader community. The [CONTRIBUTING.md](https://github.com/BrickSchema/Brick/blob/master/CONTRIBUTING.md) document in the root of the repository contains detailed instructions and guidelines for how to achieve this.

However, in some cases, it may make sense to develop a proprietary or internal extension to Brick which defines classes/relationships that (a) refer to company-specific products or features, (b) refer to internal classes not exposed to customers or users, or (c) are being developed and will be released at a later date.

## Proprietary Extensions

Proprietary extensions should be distributed as an independent RDF file (using a `.ttl`, `.n3` or `.xml` extension) containing the *definitions* of the new classes/relationships in the extension, and their relationship to existing Brick definitions. Relating proprietary extensions to existing Brick definitions is crucial for the correct interpretation/discovery of those classes for any consumers of the extension. This allows, for example, a proprietary definition of a special kind of equipment to be understood as a more specific type of an existing Brick equipment class.

### Namespace

Proprietary extension classes and relationships should be placed in their own namespace. A namespace is a URI prefix which groups the names of the classes, relationships and definitions in a graph; it does not need to be resolvable, but it is generally best practice if an HTTP GET on the namespace URI returns the definition of the extension [^brick]. Once published, a namespace should change rarely, if ever. For a hypothetical "Example Corporation" developing its extensions to Brick, possible namespaces might be: `https://example.com/schema/BrickExtension#`, `https://example.com/brick/extension/` and so on. It is helpful, but not strictly necessary, for the namespace to end in a `#` or `/`.

### Adding Classes and Relationships

The names of the classes and relationships in the extension are suffixed to the namespace chosen above, resulting in a URI which uniquely identifies that class or relationship. For example, to add a ice-making machine class in the "Example Corporation" extension, the class name `IceMachine` may be chosen. Prefixing this with the namespace results in the URI `https://example.com/schema/BrickExtension#IceMachine`.

New classes should have several properties attached to them (see the example below):
- `a owl:Class`: classes should be annotated as instances of OWL classes
- `skos:definition "definition goes here"`: classes should have textual definitions of what they are and how they are intended to be used
- `rdfs:subClassOf <Brick class>`: classes should be related to *existing Brick classes* through the `rdfs:subClassOf` relationship. A extension's class can subclass multiple Brick classes, and those classes can be as generic or specific as required. Some classes, such as `IceMachine`, may have no obvious parent classes in Brick other than `brick:Equipment`. Other classes, such as a specific kind of VFD, may have `brick:VFD` as a parent class instead.

New relationships or properties have similar requirements:
- `a owl:ObjectProperty`: if the relationship has other entities/URIs as values, `ObjectProperty` should be used. If the relationship will have scalar values such as integers, floats or strings, then `owl:DatatypeProperty` is more appropriate
- `skos:definition "definition goes here"`: relationships should have textual definitions of what they are and how they are intended to be used
- `rdfs:domain <class>`: where appropriate, relationships should be annotated with the types of entities that will have the relationship
- `rdfs:range <class>`: where appropriate, relationships should be annotated with the types of entities that will be the object of the relationship (or the datatype of the relationship)
- `rdfs:subPropertyOf <Brick relationship>`: if the extension's relationship is a specialization of an existing Brick relationship, then the extension should indicate which relationship is being specialized.

### Example

Below develops a simple extension defining:
- an `IceMachine` equipment which produces ice,
- a `SnowConeMaker` equipment which *recieves* ice from an `IceMachine` and makes snow cones, and
- a `feedsIce` relationship to indicate which snow cones receive ice from which ice machines. This is a subproperty of the `brick:feeds` relationship

Notice that the `ext` prefix is used in the Turtle file below; the choice of prefix is arbitrary and can be anything convenient. It is recommended to choose a prefix that clearly identifies the extension.

```turtle
@prefix brick: <https://brickschema.org/schema/Brick#> .
@prefix ext: <https://example.com/schema/BrickExtension#> .

ext:IceMachine  a   owl:Class ;
    rdfs:subClassOf brick:Equipment ;
    skos:definition "A machine made by Example Corp. that produces ice" .

ext:SnowConeMaker   a   owl:Class ;
    rdfs:subClassOf brick:Equipment ;
    skos:definition "A machine made by Example Corp. that uses ice to make snow cones" .

ext:feedsIce    a   owl:ObjectProperty ;
    rdfs:subPropertyOf  brick:feeds ;
    skos:definition "The subject feeds ice to the object" ;
    rdfs:domain ext:IceMachine ;
    rdfs:range  ext:SnowConeMaker .
```


## Creating Extensions

There are two supported ways to create and maintain Brick extensions.

### Direct/Manual Maintenance

One way of extending Brick is to manually maintain an RDF graph file, e.g. serialized in a Turtle (`.ttl`) file.
This can be done by editing the file directly, or using an external ontology
tool like Protege. This is the most flexible option, though maintaining RDF
ontologies "by hand" can be tricky to manage.

The [occupancy extension](https://github.com/gtfierro/brick-occupancy-extension) is one example of how this can be done.

### Python-based Extensions

The other way to extend Brick is to create a Python extension file and compile it with Brick.
The [demo_extension](https://github.com/BrickSchema/Brick/tree/master/demo_extension) folder in the Brick repository contains an example of this.

To compile Brick with this extension, provide the Python import path to the extension's `.py` file when invoking `generate_brick.py`:

```bash
$ python generate_brick.py demo_extension.new_sensors
```

The advantage of this method is that you can use the same Python constructs to implement Brick features like [Entity Properties](../metadata/entity-properties).

<details>
<summary>Example demo extension file</summary>

```python
import rdflib
from datetime import datetime
from bricksrc.namespaces import BRICK, SKOS, SH, XSD, RDFS, DCTERMS, RDF, SDO, OWL

# define the namespace to hold all of our terms, classes, properties, etc
DEMO = rdflib.Namespace("urn:demo_extension#")

# this is the ontology metadata dictionary. It MUST be named 'ontology_definition'
ontology_definition = {
    # required 'namespace' key for ontology declaration
    "namespace": DEMO,
    # optional list of creators (individuals)
    DCTERMS.creator: [
        {
            RDF.type: SDO.Person,
            SDO.email: rdflib.Literal("gtfierro@mines.edu"),
            SDO.name: rdflib.Literal("Gabe Fierro"),
        },
    ],
    # first date of release of extension/ontology
    DCTERMS.issued: rdflib.Literal("2023-07-13"),
    # keep this to ensure the 'modified' date matches when this was last ran
    DCTERMS.modified: rdflib.Literal(datetime.now().strftime("%Y-%m-%d")),
    # a version number for the ontology
    OWL.versionInfo: rdflib.Literal("0.0.1"),
    # a human-readable label for the extension/ontology
    RDFS.label: rdflib.Literal("Demo Extension"),
    # metadata on the publisher of the extension/ontology
    DCTERMS.publisher: {
        # see schema.org for other types, e.g. Consortium or Person
        RDF.type: SDO.Organization,
        SDO.legalName: rdflib.Literal("Not a real org"),
        SDO.sameAs: rdflib.Literal("http://my fake organization website.org"),
    },
    # key-value pairs of prefix to URI of ontology being imported. This will
    # add owl:imports statements to the generated extension
    "imports": {
        "shacl": "http://www.w3.org/ns/shacl#",
    },
    # namespace declarations for any SHACL rules
    "decls": {
        "rdf": RDF,
        "rdfs": RDFS,
        "brick": BRICK,
        "owl": OWL,
        "sh": SH,
        "demo": DEMO,
    }
}

# optional
# the *first* level of this dictionary should have Brick (or otherwise existing)
# classes as keys, and class definition dictionaries as values. Anything further
# nested can follow the normal class dictionary construction.
# This dictionary MUST be named 'classes'
classes = {
    BRICK.Equipment: {
        DEMO["Sensor_Platform"]: {},
        DEMO["PurpleAir_Weather_Station"]: {
            "parents": [BRICK.Weather_Station],
        },
    },
}


# optional
# this dictionary MUST be named 'entity_properties'
entity_properties = {
    DEMO.manufacturer: {
        SKOS.definition: rdflib.Literal("the manufacturer"),
        SH.datatype: XSD.string,
        RDFS.label: rdflib.Literal("manufacturer"),
        "property_of": BRICK.Equipment,
    },
    DEMO.version: {
        SKOS.definition: rdflib.Literal("a MAJOR.MINOR.PATCH version number"),
        SH.node: DEMO.VersionShape,
        RDFS.label: rdflib.Literal("version"),
        "property_of": BRICK.Equipment,
    },
}

# optional
# this dictionary MUST be named 'property_value_shapes'
property_value_shapes = {
    DEMO.VersionShape: {
        "properties": {
            DEMO.versionMajor: {
                SKOS.definition: rdflib.Literal("Major version"),
                "datatype": XSD.integer,
            },
            DEMO.versionMinor: {
                SKOS.definition: rdflib.Literal("Minor version"),
                "datatype": XSD.integer,
            },
            DEMO.versionPatch: {
                SKOS.definition: rdflib.Literal("Patch version"),
                "datatype": XSD.integer,
            },
        },
    },
}
```
</details>

## Using Extensions


Extensions may be distributed in a downloadable file containing the contents of the extension. This extension can be loaded into a graph in the same manner that the Brick ontology definitions can.


Consider the following code snippet which loads the above extension (serialized into a `example_extension.ttl` file) into a graph using the [brickschema](https://brickschema.readthedocs.io/en/latest/index.html) Python package:

```python
import brickschema

g = brickschema.Graph()
g.load_file("Brick.ttl")
g.load_file("example_extension.ttl")
g.load_file("my_building_graph.ttl")
```


In the Brick model file, `my_building_graph.ttl`, the extension classes and relationships can be used with Brick classes and relationships.

```turtle
@prefix brick: <https://brickschema.org/schema/Brick#> .
@prefix ext: <https://example.com/schema/BrickExtension#> .
@prefix bldg: <urn:snow_cone_factory#> .

bldg:factory    a   brick:Building ;
    brick:buildingPrimaryFunction [ brick:value "Manufacturing/Industrial Plant" ] .

bldg:ice_machine_1  a   ext:IceMachine ;
    brick:hasLocation   bldg:factory ;
    ext:feedsIce    bldg:snow_cone_maker_1 .

bldg:snow_cone_maker_1  a   ext:SnowConeMaker ;
    brick:hasLocation   bldg:factory .
```


## Integrating Extensions into Brick

It may come to pass that a proprietary extension is developed and used internally, but then an equivalent classes/relationships are developed in Brick, or the extension is offered for inclusion in a future release of Brick. In these cases, it is likely that there will be "old" models using the extension's version of the class and "new" models using the Brick version of the same class. These two definitions can be reconciled in a few different ways.

1. `<brick class> owl:equivalentClass <proprietary class>`: this statement can be added to the proprietary extension file to mark the two classes as being *equivalent*. After applying a [reasoner](lifecycle/inference), both classes will be assigned to all instances of either class.
2. `<brick class> rdfs:subClassOf <proprietary class>`: this statement can be added to the proprietary extension file to mark the Brick class as being more specific. This means that instances of the Brick class are *implied* to be instances of the proprietary class, but not vice-versa
2. `<proprietary class> rdfs:subClassOf <brick class>`: this statement can be added to the proprietary extension file to mark the proprietary class as being more specific. This means that instances of the proprietary class are *implied* to be instances of the Brick class, but not vice-versa

[^brick]: Visiting the Brick namespace in the browser will download the latest release of Brick! Try it: [https://brickschema.org/schema/Brick](https://brickschema.org/schema/Brick)
