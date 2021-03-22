Modeling Data Sources
=====================

Brick models describe data sources and their context --- what they are, where they are, what they mean, and how they relate to the building processes and structures that contain them. Because Brick models are only a means of *describing* data sources, we also need a way of representing the data itself. This document describes how to effectively describe timeseries data --- stored in an external database --- using the Brick ontology.

Specifically, this document covers:
- common design patterns for describing timeseries data with Brick
- using and verifying units of measure for timeseries data

This document assumes familiarity with the [Relationships documentation](/brick/relationships).

For information on how to link timeseries data in Brick to a database, see the [timeseries storage documentation](/metadata/timeseries-storage).

Here is the high-level description of the Brick data model for telemetry. Boxes represent entities; edges represent relationships. Bolded items are defined by the Brick ontology and do not need to be defined in a Brick model for a particular building.

```{image} ../img/brick-point-unit.png
:width: 600px
:align: center
```

## Categorizing Data Sources

Instances of the Brick `Point` class represent sources or sinks of telemetry. There are several major classes of telemetry in Brick:
- `Sensor`: represents the value of a device or instrument designed to detect and measure a variable
- `Setpoint`:  represents the value at which the desired property is set
- `Alarm`: represents signals that alert an operator to an off-normal condition which requires some form of corrective action
- `Command`: represents settings/actions that directly determines the behavior of equipment and/or affects relevant operational points.
- `Parameter`: represents configuration settings used to guide the operation of equipment and control systems; for example they may provide bounds on valid setpoint values
- `Status`: represents the current operating mode, state, position, or condition of an item. Statuses are observations and should be considered 'read-only'

Each of these classes is the root of a class hierarchy of more specific point types. See the [Brick documentation](https://brickschema.org/ontology/1.1/classes/Point) for details. To observe documentation for any Brick class, simply navigate to the Brick class URL in your browser. For example, the Brick class `brick:Air_Temperature_Sensor` is short for [`https://brickschema.org/schema/Brick#Air_Temperature_Sensor`](https://brickschema.org/schema/Brick#Air_Temperature_Sensor); navigating to that link will open a web page with the documentation describing that class.

## Point Instances

A single data source --- a particular sensor channel, setpoint, and so on --- is realized in a Brick model as an instance of a `Point` class. An instance is represented by a URI, typically in a namespace specific to the deployment site, which is related to a Brick `Point` class by means of the `rdf:type` relationship.

The snippet below defines a zone air temperature sensor named `mybldg:t1` in two equivalent ways.

```turtle
@prefix brick: <https://brickschema.org/schema/Brick#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix mybldg: <mybuilding#> .

mybldg:t1   rdf:type   brick:Air_Temperature_Sensor .
# 'a' is a common and universally recognized shorthand for 'rdf:type'
mybldg:t1   a   brick:Air_Temperature_Sensor .
```

An instance is commonly related to locations and/or equipment by means of the `brick:isPointOf` relationship, though these annotations are optional:

```turtle
@prefix brick: <https://brickschema.org/schema/Brick#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix mybldg: <mybuilding#> .

mybldg:t1   a   brick:Air_Temperature_Sensor .

mybldg:tstat1 a   brick:Thermostat ;
    brick:hasPoint mybldg:t1 .

# instead of the `brick:hasPoint` line above, we could have written
mybldg:t1 brick:isPointOf mybldg:tstat1 .
```

In addition, we can model the physical location of a data source (such as a sensor's placement in a room) using the `brick:hasLocation` relationship:


```turtle
@prefix brick: <https://brickschema.org/schema/Brick#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix mybldg: <mybuilding#> .

mybldg:t1   a   brick:Air_Temperature_Sensor .

mybldg:room1 a   brick:Room ;
    brick:isLocationOf mybldg:t1 .

# instead of the `brick:isLocationOf` line above, we could have written
mybldg:t1 brick:hasLocation mybldg:room1 .
```

An instance can have many other properties attached to it, including units of measure as we will see in the next section.

## Units of Measure

```{note}
A list of available units can be found [here](http://www.qudt.org/doc/DOC_VOCAB-UNITS.html)
```

An important piece of metadata to capture is the units of measure for a particular data stream. Brick builds on the [QUDT ontology](http://qudt.org/), which provides formal, semantic definitions of many common units. We have done some work to try and simplify the data model for the common cases, but there are some situations for which the necessary complexity comes through. We will point out those "sharp corners" in the documentation where they arise.

Units of measures are instances of the `qudt:Unit` class, and are represented by URIs such as [`unit:DEG_F`](http://qudt.org/vocab/unit/DEG_F) and [`unit:PPM`](http://qudt.org/vocab/unit/PPM). Units are associated with Brick `Point`s with the `brick:hasUnit` relationship:

```turtle
@prefix brick: <https://brickschema.org/schema/Brick#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix unit: <http://qudt.org/vocab/unit/> .
@prefix mybldg: <mybuilding#> .

mybldg:t1   a   brick:Air_Temperature_Sensor ;
    brick:hasUnit   unit:DEG_F .
```

*A list of available units can be found [here](http://www.qudt.org/doc/DOC_VOCAB-UNITS.html)*. Given a Brick `Point` instance, it is possible to query for valid potential units. The `?unit` values returned by this query can be associated with a `Point` instance using the `brick:hasUnit` relationship as seen above.

```sparql
PREFIX brick: <https://brickschema.org/schema/Brick#> .
PREFIX qudt: <http://qudt.org/schema/qudt/> .

SELECT ?unit  WHERE {
    mybldg:t1   brick:measures/qudt:applicableUnit ?unit .
}
```

---

Some QUDT units have symbols associated with them; these can be retrieved through altering the above query:

```sparql
SELECT ?unit ?symbol WHERE {
    mybldg:t1   brick:measures/qudt:applicableUnit ?unit .
    OPTIONAL {
        ?unit   qudt:symbol ?symbol
    }
}
```

```{note}
Most QUDT units have *labels*, which are human readable strings denoting the units. These may be a helpful alternative to `qudt:symbol` for rendering what units are associated with a point. These can be retrieved using the following query:

```sparql
SELECT ?unit ?label WHERE {
    mybldg:t1   brick:measures/qudt:applicableUnit ?unit .
    ?unit rdfs:label ?label
}
```


## Points, Quantities and Units

```{warning}
Section coming
```
