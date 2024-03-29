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

Meters
======

```{note}
This feature is new in Brick v1.3
```

Brick provides a basic but flexible model for describing meters, submeter hierarchies, and their relationships to the building.
Meters are equipment (they are subclasses of the Brick `Equipment` class) that measure the consumption or production of energy, steam, gas or other "substances" in the building.
The data produced by meters can be found in instances of the Brick `Point` class that are associated with the Meter.
These include power, energy, water and gas consumption sensors and the like.

## Meter Data

Meters can host several points which correspond to the data produced by the meter. These point instances are related to the meter via the `brick:isPointOf` relationship. See [[metadata/entity-properties]] for some examples of how those points can be described.
(The meter can also be related to the point via the `brick:hasPoint` relationship).

```ttl
bldg:building_energy_sensor a brick:Energy_Sensor ;
    brick:hasUnit unit:KiloW-HR ;
    brick:isPointOf bldg:main-meter ;
    brick:timeseries [ brick:hasTimeseriesId "a7523b08-7bc7-4a9d-8e88-8c0cd8084be0" ] .

bldg:building_peak_demand a brick:Peak_Power_Demand_Sensor ;
    brick:aggregate [ brick:aggregationFunction "max" ;
            brick:aggregationInterval "RP1D" ] ;
    brick:hasUnit unit:KiloW ;
    brick:isPointOf bldg:main-meter ;
    brick:timeseries [ brick:hasTimeseriesId "bcf9a85d-696c-446a-a2ac-97207ecfbc56" ] .

bldg:building_power_sensor a brick:Electric_Power_Sensor ;
    brick:hasUnit unit:KiloW ;
    brick:isPointOf bldg:main-meter ;
    brick:timeseries [ brick:hasTimeseriesId "fd64fbc8-0742-4e1e-8f88-e2cd8a3d78af" ] .

bldg:mybldg a brick:Building ;
    brick:isMeteredBy bldg:main-meter .
bldg:main-meter a brick:Building_Electrical_Meter .
```

## Meters and Submeters

Meters are instances of the `brick:Meter` class or any of its subclasses.
Submeter hierarchies are defined through the `brick:hasSubMeter` and `brick:isSubMeterOf` relationships.
In fact, meters can *only* be related to each other via these relationships.

```ttl
:building_meter a brick:Building_Meter ;
    brick:hasSubMeter :floor-meter-1, :floor-meter-2 .
:floor-meter-1 a brick:Meter .
:floor-meter-2 a brick:Meter .
```

```{warning}
Brick does not describe the nature of the submeter relationship -- only that it exists. If this is a problem, [let us know!](https://github.com/BrickSchema/Brick/issues)
```

To ask for the immediate submeters of a particular meter, one can query the model as follows:

```sparql
SELECT * WHERE {
    :building_meter brick:hasSubMeter ?submeter .
}
```

or more generally, for all immediate parent/child submeter relationships:

```sparql
SELECT ?meter ?submeter WHERE {
    ?meter rdf:type/rdfs:subClassOf* brick:Meter .
    ?meter brick:hasSubmeter ?submeter .
}
```

One can also ask for all meters that *aren't* submeters of any other meter. Assuming the Brick model is correct, this will avoid double counting:

```sparql
SELECT DISTINCT * WHERE {
    ?meter rdf:type/rdfs:subClassOf* brick:Meter .
    FILTER NOT EXISTS { ?meter brick:isSubMeterOf ?parent }
}
```

### Virtual Meters

It is useful to be able to model "virtual" meters which do not have a physical presence in the building.
Brick supports modeling "virtual" meters using the `brick:isVirtualMeter` Entity Property.
This property can only be "true" for instances of `brick:Meter` or subclasses thereof.

```ttl
:my_virtual_meter a brick:Electrical_Meter ;
    brick:isVirtualMeter [ brick:value true ] .
```

By default, virtual meters look exactly like physical meters so no queries need to be changed if you don't care about the distinction between them.
It is also possible to query *only* virtual meters:

```sparql
SELECT ?meter WHERE {
   ?meter rdf:type/rdfs:subClassOf* brick:Meter ;
          brick:isVirtualMeter/brick:value true .
}
```

or to query only meter which are *not* virtual meters

```sparql
SELECT ?meter WHERE {
   ?meter rdf:type/rdfs:subClassOf* brick:Meter .
   FILTER NOT EXISTS { ?meter brick:isVirtualMeter/brick:value true ] }
}
```


## Associating Meters

Meters can be associated with the entities that they are metering through the `brick:meters` / `brick:isMeteredBy` relationships. These entities can be instances of `brick:Equipment`, `brick:Location` *or* `brick:Collection`.

Metering `brick:Equipment`: useful when the Meter is measuring the consumption/production of a single Equipment instance.

```ttl
:chiller-1 a brick:Chiller ;
    brick:isMeteredBy :chiller-meter .
:chiller-meter rdf:type/rdfs:subClassOf* brick:Electric_Meter .
```

Metering `brick:Location`s: useful when the Meter is measuring the consumption/production of many entities within a single location, e.g. a building or floor or room.

```ttl
:floor-1 a brick:Floor ;
    brick:isLocationOf :camera-1, :camera-2 ;
    brick:isMeteredBy :floor-meter .
:floor-meter rdf:type/rdfs:subClassOf* brick:Electric_Meter .
:camera-1 a brick:Camera .
:camera-2 a brick:Camera .
```

Metering `brick:Collection`s: useful when the Meter is measuring the consumption/production of many entities that are not necessarily grouped by location. Recall that a `brick:Collection` is a named group of entities. This could represent the equipment in a particular air loop, in a particular chilled water system, all of the devices owned by a particular individual, or any other organization.

```ttl
# pv generation systems and pv_arrays are both collections
:pv_generation_system a brick:PV_Generation_System ;
    brick:hasPart :array1 ;
    brick:isMeteredBy :pv_meter .
:array1 a brick:PV_Array .
:panel11 a brick:PV_Panel ;
    brick:isPartOf :array1 ;
    brick:panelArea [ brick:hasUnit unit:M2 ;
            brick:value "5"^^xsd:double ] .
:panel12 a brick:PV_Panel ;
    brick:isPartOf :array1 .
:pv_meter   rdf:type/rdfs:subClassOf*   brick:Electrical_Meter .
```

### Querying these patterns

Brick makes it possible to identify all meters associated with any entity in the Brick model:

```sparql
SELECT ?meter WHERE {
    ?entity brick:isPartOf*/brick:isMeteredBy ?meter .
    ?meter rdf:type/rdfs:subClassOf* brick:Meter .
}
```

