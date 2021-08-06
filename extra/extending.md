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
