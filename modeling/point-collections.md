Automation and Point Collections
================================

Automation and Point Collections are named groups for organizing building automation entities. They make it possible to describe application-level bundles such as frost detection, ventilation control, room control applications, startup sequences, alarm packages, and operator-facing point groups.

These collections are organizational. They help tools find related entities, but they do not replace the Brick relationships that describe physical composition, point ownership, topology, or control behavior.

```{important}
Do not infer functional semantics from collection membership alone. If a sensor, command, alarm, and piece of equipment are in the same collection, that only says they belong to the same modeled bundle. Use explicit Brick relationships to describe what is measured, commanded, controlled, hosted, or physically part of something else.
```

## At-a-Glance

- Create a `brick:Point_Collection` for point-only groups.
- Create a `brick:Automation_Collection` for mixed automation bundles.
- Put member entities into the collection with `rec:includes`.
- Attach every point to what it describes with `brick:hasPoint` or `brick:isPointOf`.
- Attach every hosted point to its controller or ICT equipment with `brick:hostsPoint` or `brick:isHostedBy`.
- Use `brick:hasPart` only for equipment composition, not collection membership.
- Add `rdfs:label` so tools can present the collection clearly.

## The Two Collection Types

Use `brick:Automation_Collection` for a logical automation bundle. An Automation Collection can include equipment, points, point collections, and other automation collections. It is the right choice when a grouping represents a control application, sequence, alarm package, or other automation function that spans more than just points.

Use `brick:Point_Collection` for a point-only bundle. A Point Collection can include points and nested Point Collections. It is the right choice for operator displays, trend packages, export packages, and other groupings whose members are only points.

Both types use `rec:includes` for membership:

```turtle
@prefix bldg: <urn:example/> .
@prefix brick: <https://brickschema.org/schema/Brick#> .
@prefix rec: <https://w3id.org/rec#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

bldg:VentilationControl a brick:Automation_Collection ;
    rdfs:label "Ventilation control" ;
    rec:includes bldg:AirHandler,
        bldg:VentilationPoints .

bldg:VentilationPoints a brick:Point_Collection ;
    rdfs:label "Ventilation points" ;
    rec:includes bldg:DamperPosition,
        bldg:CO2Setpoint .
```

## Relationship Roles

Use collection membership for organization, and use the normal Brick relationships for the model semantics:

- Use `rec:includes` to put an entity into an Automation Collection or Point Collection.
- Use `brick:hasPart` for physical or structural composition, such as an AHU having a fan or coil.
- Use `brick:hasPoint` / `brick:isPointOf` to attach a point to the equipment, location, or zone it describes.
- Use `brick:hostsPoint` / `brick:isHostedBy` to describe the ICT equipment, controller, gateway, or device that hosts or exposes a point.
- Use `brick:controls` / `brick:isControlledBy` for control relationships between controllers and the equipment they supervise.

Equipment, spaces, and zones can include automation collections with `rec:includes`. This is useful when an asset or zone has a logical application bundle, but the bundle is not a physical part of that asset or zone.

## Choosing the Right Pattern

Use a Point Collection when all of these are true:

- the grouping only contains points or nested point groups
- the grouping is mainly for display, export, trending, configuration, or navigation
- membership should not imply control logic, equipment structure, or a sequence of operation

Use an Automation Collection when any of these are true:

- the grouping includes equipment or other non-point entities
- the grouping represents an automation application, control package, alarm package, or sequence
- the grouping needs nested Point Collections for subsets of points
- the grouping should be associated with an equipment instance, space, or zone as a named application

Define a custom subclass when the grouping has stable meaning in a project or organization. For example, `mpo:FrostDetectionCollection` can be declared as a subclass of `brick:Automation_Collection`.

## Example: Frost Detection

This example models an AHU with a frost detection package. The AHU uses `brick:hasPart` for physical components and `rec:includes` for the logical automation bundle. The points are still attached to the entities they describe with `brick:isPointOf` / `brick:hasPoint`.

```turtle
@prefix bldg: <urn:example/collection#> .
@prefix brick: <https://brickschema.org/schema/Brick#> .
@prefix mpo: <http://my-private-ontology.com/> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix rec: <https://w3id.org/rec#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

mpo:FrostDetectionCollection a owl:Class ;
    rdfs:subClassOf brick:Automation_Collection ;
    rdfs:label "Frost Detection Collection" ;
    rdfs:comment "An automation collection that includes components for frost detection in an AHU." .

bldg:Ahu a brick:AHU ;
    brick:hasPart bldg:Hcl, bldg:Fan ;
    rec:includes bldg:FrostDetection .

bldg:Hcl a brick:Heating_Coil .

bldg:Fan a brick:Discharge_Fan .

bldg:FrostDetection a mpo:FrostDetectionCollection ;
    rdfs:label "AHU frost detection" ;
    rec:includes bldg:FrostDetectionSensor,
        bldg:FrostDetectionMonitoring,
        bldg:FrostPoints .

bldg:FrostDetectionSensor a brick:Sensor_Equipment ;
    brick:hasPoint bldg:FrostDetected .

bldg:FrostDetectionMonitoring a brick:Enable_Command ;
    brick:isPointOf bldg:Ahu .

bldg:FrostPoints a brick:Point_Collection ;
    rdfs:label "Frost detection points" ;
    rec:includes bldg:FrostDetected,
        bldg:MixedAirTemp .

bldg:FrostDetected a brick:Frost_Sensor ;
    brick:isPointOf bldg:FrostDetectionSensor .

bldg:MixedAirTemp a brick:Mixed_Air_Temperature_Sensor ;
    brick:isPointOf bldg:Ahu .
```

The Automation Collection groups the whole frost detection application. The nested Point Collection groups only the telemetry associated with that application.

## Example: Room Application

Automation Collections can represent applications within a zone or space. In this example, a zone includes an HVAC application. The HVAC application includes equipment and point-only collections for specific application areas.

```turtle
@prefix bldg: <urn:example/room#> .
@prefix brick: <https://brickschema.org/schema/Brick#> .
@prefix rec: <https://w3id.org/rec#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

bldg:Zone a rec:Zone ;
    rec:includes bldg:Hvac .

bldg:Room a rec:Room ;
    rec:hasPart bldg:Zone .

bldg:Hvac a brick:Automation_Collection ;
    rdfs:label "HVAC" ;
    rec:includes bldg:RadiantCeiling,
        bldg:RoomTemperatureSetpointDetermination,
        bldg:AirVolumeFlowTracking,
        bldg:VentilationControl .

bldg:RadiantCeiling a brick:Radiant_Panel ;
    rdfs:label "Radiant ceiling" ;
    brick:feeds bldg:Room .

bldg:RoomTemperatureSetpointDetermination a brick:Point_Collection ;
    rdfs:label "Room temperature setpoint determination" ;
    rec:includes bldg:RoomTemperatureSetpoint,
        bldg:RoomTemperatureSetpointShift .

bldg:RoomTemperatureSetpoint a brick:Target_Zone_Air_Temperature_Setpoint ;
    rdfs:label "Room temperature setpoint" ;
    brick:isPointOf bldg:Zone .

bldg:RoomTemperatureSetpointShift a brick:Temperature_Adjust_Sensor ;
    rdfs:label "Room temperature setpoint shift" ;
    brick:isPointOf bldg:Zone .

bldg:AirVolumeFlowTracking a brick:Point_Collection ;
    rdfs:label "Air volume flow tracking" ;
    rec:includes bldg:RoomSupplyAirVolumeFlow,
        bldg:RoomExtractAirVolumeFlowSetpoint .

bldg:RoomSupplyAirVolumeFlow a brick:Supply_Air_Flow_Sensor ;
    rdfs:label "Room supply air volume flow" ;
    brick:isPointOf bldg:Zone .

bldg:RoomExtractAirVolumeFlowSetpoint a brick:Exhaust_Air_Flow_Setpoint ;
    rdfs:label "Present setpoint for room extract air volume flow" ;
    brick:isPointOf bldg:Zone .

bldg:VentilationControl a brick:Point_Collection ;
    rdfs:label "Ventilation control" ;
    rec:includes bldg:PresentVentilationSetpoint,
        bldg:RoomAirQualitySetpointComfort,
        bldg:RoomAirQualitySetpointPreComfort .

bldg:PresentVentilationSetpoint a brick:Damper_Position_Setpoint ;
    rdfs:label "Present ventilation setpoint" ;
    brick:isPointOf bldg:Zone .

bldg:RoomAirQualitySetpointComfort a brick:CO2_Setpoint ;
    rdfs:label "Setpoint room air quality for comfort" ;
    brick:isPointOf bldg:Zone .

bldg:RoomAirQualitySetpointPreComfort a brick:CO2_Setpoint ;
    rdfs:label "Setpoint room air quality for pre-comfort" ;
    brick:isPointOf bldg:Zone .
```

The top-level HVAC collection identifies the room automation application. Its point-only subcollections make it easy for tools to retrieve the points for each part of the application.

## Example: Controller-Hosted Points

Collections do not replace point hosting. A controller should still use `brick:hostsPoint` for the points it exposes, while Point Collections can organize those points for display or configuration.

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
    brick:hostsPoint bldg:VAV_1_SAF_Sensor .

bldg:VAV_1PointCollection a brick:Point_Collection ;
    rdfs:label "VAV 1 Point Collection" ;
    rec:includes bldg:VAV_1_SAF_Sensor .

bldg:VAV_1_SAF_Sensor a brick:Supply_Air_Flow_Sensor ;
    brick:hasUnit unit:FT3-PER-MIN ;
    brick:isPointOf bldg:VAV_1 .
```

One can nest Point Collections to model more complex setups: `brick:Controller` hosts the individual points and the collection organizes them.


## Query Patterns

The following queries assume standard RDFS subclass reasoning is either available in the graph or handled by the `a/rdfs:subClassOf*` property paths shown below.

Retrieve all members of an Automation Collection, including nested automation and point collections:

```sparql
PREFIX brick: <https://brickschema.org/schema/Brick#>
PREFIX rec: <https://w3id.org/rec#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?member WHERE {
    ?collection a/rdfs:subClassOf* brick:Automation_Collection ;
                rec:includes+ ?member .
}
```

Retrieve all points in a Point Collection, including points in nested Point Collections:

```sparql
PREFIX brick: <https://brickschema.org/schema/Brick#>
PREFIX rec: <https://w3id.org/rec#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?point WHERE {
    ?collection a/rdfs:subClassOf* brick:Point_Collection ;
                rec:includes+ ?point .
    ?point a/rdfs:subClassOf* brick:Point .
}
```

Find all automation bundles associated with an AHU and the entities they include:

```sparql
PREFIX brick: <https://brickschema.org/schema/Brick#>
PREFIX rec: <https://w3id.org/rec#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?collection ?member WHERE {
    ?ahu a/rdfs:subClassOf* brick:AHU ;
         rec:includes ?collection .
    ?collection a/rdfs:subClassOf* brick:Automation_Collection ;
                rec:includes+ ?member .
}
```

Find automation bundles associated with a zone and the points included in those bundles:

```sparql
PREFIX brick: <https://brickschema.org/schema/Brick#>
PREFIX rec: <https://w3id.org/rec#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?collection ?point WHERE {
    ?zone a rec:Zone ;
          rec:includes ?collection .
    ?collection a/rdfs:subClassOf* brick:Automation_Collection ;
                rec:includes+ ?point .
    ?point a/rdfs:subClassOf* brick:Point .
}
```

Retrieve both physical parts and logical automation bundles from an AHU:

```sparql
PREFIX brick: <https://brickschema.org/schema/Brick#>
PREFIX rec: <https://w3id.org/rec#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?related WHERE {
    ?ahu a/rdfs:subClassOf* brick:AHU .
    {
        ?ahu brick:hasPart+ ?related .
    } UNION {
        ?ahu rec:includes+ ?related .
    }
}
```

Find the equipment or zone that each point in a collection describes:

```sparql
PREFIX brick: <https://brickschema.org/schema/Brick#>
PREFIX rec: <https://w3id.org/rec#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?point ?entity WHERE {
    ?collection rec:includes+ ?point .
    ?point a/rdfs:subClassOf* brick:Point ;
           brick:isPointOf ?entity .
}
```
