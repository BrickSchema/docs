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

Terminal Units
==============

```{note}
This page uses Brick and RealEstateCore concepts together, which is a new feature in Brick v1.4
```

This page provides a brief annotated reference model for how to approach modeling terminal units in Brick, along with their points and their relationship to zones and spaces.

```ttl
@prefix brick: <https://brickschema.org/schema/Brick#> .
@prefix rec: <https://w3id.org/rec#> .
@prefix bldg: <urn:my_building/> .
@prefix unit: <http://qudt.org/vocab/unit/> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix ref: <https://brickschema.org/schema/brick/ref#> .
@prefix bacnet: <http://data.ashrae.org/bacnet/2020#> .

bldg:VAV1 a brick:Variable_Air_Volume_Box_With_Reheat ;
    rdfs:label "VAV 1" ;
    brick:hasPoint bldg:sat1, bldg:saf1, bldg:sp1 ;
    brick:feeds bldg:zone1 .

bldg:sat1 a brick:Supply_Air_Temperature_Sensor ;
    brick:hasUnit unit:DEG_F ;
    ref:hasExternalReference [
        bacnet:object-identifier "analog-value,5" ;
        bacnet:object-name "BLDG-Z410-SAT" ;
        bacnet:objectOf bldg:sample-device ;
    ] .

bldg:sp1 a brick:Supply_Air_Temeprature_Setpoint ;
    brick:hasUnit unit:DEG_F ;
    ref:hasExternalReference [
        bacnet:object-identifier "analog-value,7" ;
        bacnet:object-name "BLDG-Z410-SAF" ;
        bacnet:objectOf bldg:sample-device ;
    ] .

bldg:saf1 a brick:Supply_Air_Flow_Sensor ;
    brick:hasUnit unit:FT3-PER-MIN ;
    ref:hasExternalReference [
        bacnet:object-identifier "analog-value,6" ;
        bacnet:object-name "BLDG-Z410-SAF" ;
        bacnet:objectOf bldg:sample-device ;
    ] .

bldg:zone1 a rec:HVACZone ;
    rec:hasPart bldg:room1 .

bldg:room1 a rec:Office ;
    rdfs:label "Personal Office" ;
    rec:isLocationOf bldg:sensor_box_1 .

bldg:sensor_box_1 a brick:Sensor_Equipment ;
    brick:hasPoint bldg:rmat1 .

bldg:rmat1 a brick:Room_Air_Temperature_Sensor ;
    brick:hasUnit unit:DEG_F ;
    ref:hasExternalReference [
        bacnet:object-identifier "analog-value,8" ;
        bacnet:object-name "BLDG-Z410-ROOM" ;
        bacnet:objectOf bldg:sample-device ;
    ] .

# BACnet network stuff
bldg:sample-device a bacnet:BACnetDevice ;
    bacnet:device-instance 123 ;
    bacnet:hasPort [ a bacnet:Port ] .
```

The `bldg:VAV1` entity models the RVAV unit itself, along with its associated data streams (`bldg:sat1`, `bldg:saf1`, and `bldg:sp1`) and which zone it is connected to (`bldg:zone1`).
Each of the data streams is a representation of the BMS point; the `ref:hasExternalReference` relationship connects each Point to the corresponding BACnet object.

The `bldg:zone1` entity is a representation of the HVAC zone; it contains a single room which is a personal office.
This office contains a piece of `Sensor_Equipment`, which is a physical sensing apparatus.
It also has a Sensor entity associated with it, which represents the data source containing temperature data from the sensing apparatus.
