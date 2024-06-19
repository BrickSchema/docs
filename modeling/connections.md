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


Connections
===========

```{note}
This feature is new in Brick v1.4 and relies on the [ASHRAE 223P standard](https://open223.info)
```

Brick itself does not include connections (e.g., pipes, ducts, and wires) but as of Brick 1.4, it is possible to model connections using the related ASHRAE 223 ontology.
This mechanism allows more detailed descriptions of the topology of building equipment and subsystems, and permits the association of Brick `Point`s with the connections themselves, as well as where those connections attach to equipment and other entities.


## 223P Connections Overview

```{note}
Text borrowed from the [223p documentation](https://docs.open223.info/explanation/223_overview.html#topology)
```

ASHRAE 223 can be used to describe the topology of the equipment and spaces in a building, but not the geometric details.
Topology refers to the way entities are connected and how some media (e.g. water, air, or electricity) is conveyed between them.
There are several different classes used to describe which entities participate in connections and how they connect: [Connectables](https://explore.open223.info/s223/Connectable.html), which include the entities that are capable of connecting to each other; [ConnectionPoints](https://explore.open223.info/s223/ConnectionPoint.html), which model where Connectables can be connected; and [Connections](https://explore.open223.info/s223/Connection.html), which describe physical things through which the medium is conveyed, like pipes or ducts.
These [Mediums](https://explore.open223.info/s223/Substance-Medium.html) (e.g. gas, electricity, water) are defined as an [EnumerationKind](https://explore.open223.info/s223/EnumerationKind.html) in the standard.
There are also multiple relations used to describe the details of these connections, and how the multiple entities involved in a connection relate to each other.
The figure below summarizes these relations.

```{image} /img/connection-relationships.png
:align: center
:width: 700px
```

Though there are many relations to describe different perspectives of a connection, only [`s223:cnx`](https://explore.open223.info/s223/cnx.html) needs to be manually added to the model, and the rest can be automatically added to the model through the process of model inference.

Note that the `s223:connectedTo` relationship is equivalent to `brick:feeds` because it shows a directional "flow" relationship from one entity to another.

## Connections on Brick Entities

Brick `Equipment` are now subclasses of `s223:Equipment` in the Brick ontology.
This means that they can have `ConnectionPoint`s and `Connection`s associated with them.

### Common Connection Constructs

Here is a model of a Duct between a brick AHU and a brick VAV

```turtle
@prefix brick: <https://brickschema.org/schema/Brick#> .
@prefix s223: <http://data.ashrae.org/standard223#> .

# Define the AHU and VAV
:ahu  a  brick:AHU ;
    rdfs:label "AHU" .
:vav  a  brick:VAV ;
    rdfs:label "VAV" .

# air flow sensor
:air_flow_sensor  a  brick:Air_Flow_Sensor ;
    rdfs:label "Air Flow Sensor" .

# Define the duct
:duct  a  s223:Duct ;
    rdfs:label "duct" ;
    s223:hasMedium s223:Medium-Air ;
    brick:hasPoint  :air_flow_sensor ; # put a sensor in the duct!
    s223:connectsFrom  :ahu ;
    s223:connectsTo  :vav .
```

Here, we need to use `connectsFrom`/`connectsTo` to specify the direction of the connection.

We can also augment this model with connection points which are the physical locations where connections can be made.

```turtle
@prefix brick: <https://brickschema.org/schema/Brick#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix s223: <http://data.ashrae.org/standard223#> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix : <urn:duct_example/> .

<urn:duct_example> a owl:Ontology ;
    owl:imports <https://brickschema.org/schema/1.4/Brick>,
                <https://brickschema.org/extension/s223extension> .

:saf1  a  brick:Supply_Air_Flow_Sensor ;
    rdfs:label "Supply Air Flow Sensor" .

# define the AHU with a air supply connection point
:ahu  a  brick:AHU ;
    rdfs:label "AHU" ;
    s223:hasConnectionPoint  :ahu_air_supply .

:ahu_air_supply  a  s223:OutletConnectionPoint ;
    rdfs:label "AHU air supply" ;
    s223:hasRole s223:Role-Supply ;
    brick:hasPoint :saf1 ; # put the supply air flow sensor at the outlet of the AHU
    s223:hasMedium s223:Medium-Air .

# define the VAV with a air inlet connection point
:vav  a  brick:VAV ;
    rdfs:label "VAV" ;
    s223:hasConnectionPoint  :vav_air_inlet .
:vav_air_inlet  a  s223:InletConnectionPoint ;
    rdfs:label "VAV air inlet" ;
    s223:hasMedium s223:Medium-Air .

# now that we have connection points, we can define the duct using s223:cnx,
# which will infer the rest of the relationship and the correct direction
# of the connection
:duct  a  s223:Duct ;
    rdfs:label "duct" ;
    s223:hasMedium s223:Medium-Air ;
    s223:cnx  :ahu_air_supply, :vav_air_inlet .
```
