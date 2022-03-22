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

External Representation
=======================

```{note}
This feature is new in Brick v1.3
```

Instances of `Point` in Brick are representations of I/O points and data sources and sinks.
Brick supports relating `Point`s to their representations in external systems, e.g. BACnet networks and timeseries databases.
This allows software to use a Brick model to configure its access to those other systems.

The generic relationship `brick:externalRepresentation` can be used to find these other representations.
A Brick `Point` can have any number of external representations.

Brick currently supports the following external representations:
- `brick:timeseries`: links a Brick `Point` to its data in a timeseries database
- `brick:BACnetRepresentation`: links a Brick `Point` to its corresponding BACnet object

Examples follow below.


## BACnet

The BACnet representation object is linked to a Brick `Point` with the `brick:BACnetRepresentation` relationship.
There are two possible forms of the BACnet object:

Option 1 supports the following fields:
- `bacnet:object-identifier`: the BACnet object ID: `object-type,object-instance-number`
- `bacnet:object-name`: the `name` field: the `description` field for the BACnet object
- `bacnet:object-type`: the BACnet type of the object, e.g. `analog-input`
- `bacnet:description`: the `description` field for the BACnet object
- `bacnet:read-property`: which property to read for the value; defaults to `present-value`

Option 2 supports the BACnet URI scheme:
- `brick:BACnetURI`: defined in clause Q.8 of the BACnet spec: `bacnet:// <device> / <object> [ / <property> [ / <index> ]]`


```turtle
@prefix bldg: <urn:example#> .
@prefix brick: <https://brickschema.org/schema/Brick#> .
@prefix bacnet: <http://data.ashrae.org/bacnet/2020#> .
@prefix unit: <http://qudt.org/vocab/unit/> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .

bldg:sample-device a bacnet:BACnetDevice ;
    bacnet:device-instance 123 ;
    bacnet:hasPort [ a bacnet:Port ] .

# Option 1: explicit fields
bldg:ts1 a brick:Zone_Air_Temperature_Sensor ;
    brick:hasUnit unit:DEG_C ;
    brick:BACnetRepresentation [
        bacnet:object-identifier "analog-value,5" ;
        bacnet:object-name "BLDG-Z410-ZATS" ;
        bacnet:objectOf bldg:sample-device ;
    ] .

# Option 2: BACnet URI
bldg:ts2 a brick:Zone_Air_Temperature_Sensor ;
    brick:hasUnit unit:DEG_C ;
    brick:BACnetRepresentation [
		brick:BACnetURI "bacnet://123/analog-input,3/present-value" ;
        bacnet:objectOf bldg:sample-device ;
    ] .
```

## Timeseries

See the [metadata/timeseries-storage](/metadata/timeseries-storage) section.

## Discovering External Representations

The `brick:externalRepresentation` relationships can be used to find external representations.
`brick:externalRepresentation` is a superproperty of `brick:BACnetRepresentation` and `brick:timeseries`, and is inferred automatically through the application of a reasoner.

Consider the following example:

```turtle
# example.ttl
bldg:ts1 a brick:Zone_Air_Temperature_Sensor ;
    brick:hasUnit unit:DEG_C ;
    brick:BACnetRepresentation [
        bacnet:object-identifier "analog-value,5" ;
        bacnet:object-name "BLDG-Z410-ZATS" ;
        bacnet:objectOf bldg:sample-device ;
    ] ;
    brick:timeseries [
        brick:hasTimeseriesId "756e1623-914f-4415-9000-b1b10ce8f5c9" ;
        brick:storedAt "postgres://1.2.3.4:5432/mydata" ;
    ] .
```

In Python, using the `brickschema` package, external representations can be found and typed as follows:

```python
import brickschema

g = brickschema.Graph()
g.load_file("Brick.ttl")
g.load_file("example.ttl")

g.expand("shacl")

query = """SELECT ?point ?rep ?type WHERE {
    ?point brick:externalRepresentation ?rep .
    ?rep   a ?type .
}
```
