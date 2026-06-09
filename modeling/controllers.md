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

Controllers
===========

```{note}
`brick:Controller`, `brick:controls`/`brick:isControlledBy`, and `brick:hosts`/`brick:isHostedBy` are new in Brick v1.5.
```

A **Controller** is a piece of ICT equipment that supervises and commands one or more physical equipment instances and exposes the points used to do so.
Brick provides two relationship pairs for modeling controllers:

- `brick:controls` / `brick:isControlledBy` — links a Controller to the Equipment it supervises.
- `brick:hosts` / `brick:isHostedBy` — links a Controller (or any ICT Equipment) to the Points it exposes on the network.

These relationships are distinct from `brick:hasPoint`, which expresses that a point *describes* a piece of equipment, regardless of which device exposes it.

## Relationships

`brick:controls`: the *subject* (a `brick:Controller`) commands the *object* (a `brick:Equipment` instance).

`brick:hosts`: the *subject* (a `brick:ICT_Equipment` or `brick:Controller`) exposes the *object* (a `brick:Point`) on the network. This is the physical/logical hosting relationship — the device through which the point is accessed.

A point may be hosted by a controller (`brick:hosts`) and also described as belonging to an equipment (`brick:hasPoint` / `brick:isPointOf`). These two relationships are independent.

## Example: VAV Controller

```turtle
@prefix bldg: <http://example.com/controller#> .
@prefix brick: <https://brickschema.org/schema/Brick#> .
@prefix rec: <https://w3id.org/rec#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

bldg:Controller_1 a brick:Controller ;
    rdfs:label "Main Building Controller" ;
    brick:controls bldg:VAV1 ;
    brick:hosts bldg:VAV1_Temperature_Sensor,
                bldg:VAV1_Occupancy_Sensor .

bldg:VAV1 a brick:Variable_Air_Volume_Box ;
    rdfs:label "VAV1" ;
    brick:hasPoint bldg:VAV1_Temperature_Sensor,
                   bldg:VAV1_Occupancy_Sensor ;
    brick:feeds bldg:Zone1 .

bldg:VAV1_Temperature_Sensor a brick:Temperature_Sensor .
bldg:VAV1_Occupancy_Sensor a brick:Occupancy_Sensor .

bldg:Zone1 a rec:HVACZone ;
    rdfs:label "Zone 1" .
```

Here `brick:controls` captures the supervisory relationship and `brick:hosts` captures which device exposes the points on the network.
`brick:hasPoint` remains on the VAV to express that those points describe the VAV — not the controller.

## Combining Controllers with Point Collections

`brick:hosts` and `brick:Point_Collection` serve different purposes and can be used together.
Use `brick:hosts` for the physical/network hosting relationship; use a `brick:Point_Collection` to group points for display, export, or application logic.

```turtle
@prefix bldg: <urn:example/vav#> .
@prefix brick: <https://brickschema.org/schema/Brick#> .
@prefix rec: <https://w3id.org/rec#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix unit: <http://qudt.org/vocab/unit/> .

bldg:VAV_1 a brick:Variable_Air_Volume_Box ;
    brick:hasPoint bldg:VAV_1_SAF_Sensor .

bldg:Ctrl_1 a brick:Controller ;
    brick:controls bldg:VAV_1 ;
    brick:hosts bldg:VAV_1_SAF_Sensor .

bldg:VAV_1PointCollection a brick:Point_Collection ;
    rdfs:label "VAV 1 Point Collection" ;
    rec:includes bldg:VAV_1_SAF_Sensor .

bldg:VAV_1_SAF_Sensor a brick:Supply_Air_Flow_Sensor ;
    brick:hasUnit unit:FT3-PER-MIN ;
    brick:isPointOf bldg:VAV_1 .
```

The controller hosts the sensor; the Point Collection organizes it for tools. See [Point Collections](point-collections) for more on organizing points into bundles.

## Query Patterns

Find all equipment controlled by a given controller:

```sparql
PREFIX brick: <https://brickschema.org/schema/Brick#>

SELECT ?equipment WHERE {
    ?controller a/rdfs:subClassOf* brick:Controller ;
                brick:controls ?equipment .
}
```

Find which controller hosts a given point:

```sparql
PREFIX brick: <https://brickschema.org/schema/Brick#>

SELECT ?controller WHERE {
    ?controller brick:hosts :my_point .
}
```

Find all points hosted by controllers in the model, and the equipment those points describe:

```sparql
PREFIX brick: <https://brickschema.org/schema/Brick#>

SELECT ?controller ?point ?equipment WHERE {
    ?controller a/rdfs:subClassOf* brick:Controller ;
                brick:hosts ?point .
    ?point brick:isPointOf ?equipment .
}
```
