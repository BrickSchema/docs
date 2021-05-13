Collections, Systems and Loops
==============================

A *collection* is a group of related entities, typically organized around a fixed use or function. A collection is modeled as a Brick entity which contains other entities; the relationship between a collection and its contents is `brick:hasPart`. Collections allow a modeler to associate entities together to make them easier to find later. Systems and Loops are special kinds of Collections.

Brick defines multiple kinds of collections and allows modelers to create their own.

Examples of Brick collections include:
- **Portfolio**: a group of `brick:Site`
- **Loop**: a group of `brick:Equipment` and `brick:Point`
- **System**: a group of `brick:Equipment`, `brick:Point` and `brick:Loop`
- **Photovoltaic Array**: a group of `brick:PV_Panel`

Several kinds of systems are also defined, including `brick:Lighting_System`, `brick:HVAC_System`, `brick:Chilled_Water_System` and `brick:Energy_Generation_System`.
Typically, collections can only contain equipment and points. The main exception is a `Portfolio` which can only contain `brick:Site`.

There are few rules on what kinds of equipment and points can be included in each of the collection types, so the composition of a collection is quite flexible.

```{note}
At this time, collections must be created manually by the modeler, though it is not out of the question to consider a post-processing tool that assigns entities to different collections based on their types and other relationships. For example, this post-processor may add all chilled water coils to a chilled water system, and all hot water coils to a hot water system.
```

## Example: HVAC System

Below is simple example of an HVAC system with AHUs and VAVs, demonsrating use of `brick:HVAC_System` and `brick:Air_Loop` collections.
An HVAC System is a group of equipment, points and loops that implement or handle heating, ventilation and/or air-conditioning in the building.
An Air Loop is a group of equipment and points that are all connected and pass air between them; this does not model arbitrary air flow in a building but rather the system's intended transit of air through the system.

```turtle
@prefix brick: <https://brickschema.org/schema/Brick#> .
@prefix : <urn:bldg#> .

:hvac_system    a   brick:HVAC_System .
:air_loop_1 a   brick:Air_Loop .
:air_loop_2 a   brick:Air_Loop .

:ahu1   a   brick:AHU ;
    brick:feeds     :vav1, :vav2 ;
    brick:isPartOf :hvac_system, :air_loop_1, :air_loop_2 .
:vav1   a   brick:VAV ;
    brick:hasPart :dmp1 ;
    brick:hasPoint  :sats1 ;
    brick:feeds     :zone1 ;
    brick:isPartOf :hvac_system, :air_loop_1 .
:vav2   a   brick:VAV ;
    brick:hasPart :dmp2 ;
    brick:hasPoint  :sats2 ;
    brick:feeds     :zone2 ;
    brick:isPartOf :hvac_system, :air_loop_2 .
    
:zone1  a   brick:HVAC_Zone .
:zone2  a   brick:HVAC_Zone .
:dmp1 a brick:Damper ;
    brick:hasPoint  :pos1 .
:dmp2 a brick:Damper ;
    brick:hasPoint  :pos2 .
:pos1   a brick:Position_Command .
:pos2   a brick:Position_Command .
:sats1   a brick:Supply_Air_Temperature_Sensor .
:sats2   a brick:Supply_Air_Temperature_Sensor .
```

After applying [inference](lifecycle/inference), several new conclusions can be drawn:
- the members of `:air_loop_1` are `:vav1`, `:ahu1` (indicated explicitly) but also `:dmp1`, `:pos1` and `:sats1`
- likewise, the members of  `:air_loop_2` are `:vav2`, `:ahu2` (indicated explicitly) but also `:dmp2`, `:pos2` and `:sats2`
- the mebmers of `:hvac_system` are the members of the loops above as well as the loops themselves
- *at this time* the zones themselves are not part of the loop **this may change in the future depending on community feedback**

This permits useful discovery queries such as the following: *"What are the contents of the air loop feeding Zone 1?*"

```sparql
SELECT ?content WHERE {
    ?content    brick:isPartOf ?loop .
    ?equip    brick:isPartOf ?loop .
    ?loop   a   brick:Air_Loop .
    ?equip    brick:feeds :zone1 .
}
```

## Custom Collections

It is possible for a modeler to create their own ad-hoc collections, depending on how they want to organize entities in the model. The generic `brick:Collection` class is available for this purpose.
These can be created by instantiating a new entity that is of type `brick:Collection`; it is recommended to use `rdfs:label` to name the collection.

Using the example of the HVAC system above, one can create a collection containing just the dampers.

```turtle
@prefix brick: <https://brickschema.org/schema/Brick#> .
@prefix : <urn:bldg#> .

:my_collection  a   brick:Collection ;
    rdfs:label "All of the dampers" ;
    brick:hasPart   :dmp1, :dmp2 .
```
