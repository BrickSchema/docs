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

Writing Custom SHACL Shapes
===========================

This document describes how to write custom SHACL shapes for use with Brick models.

```{note}
Please file an issue if you would like additional examples or guidance on this topic
```

## Introduction

SHACL is a W3C standard for defining constraints on RDF data.
Brick uses both the base [SHACL](https://www.w3.org/TR/shacl/) vocabulary as well as the [SHACL-AF](https://www.w3.org/TR/shacl-af/) advanced features vocabulary.
These languages can also be used to define custom constraints on Brick models, for example to ensure that all instances of a certain class have a certain property, or to ensure that all instances of a certain class have a certain relationship to another class.

To learn SHACL, we recommend reading the definitions of [SHACL](https://www.w3.org/TR/shacl/) and [SHACL-AF](https://www.w3.org/TR/shacl-af/).
The [RDF Validation book](http://book.validatingrdf.com) is another good resource.

## Examples

### Example: Point list for a RVAV

A simple and common pattern is to define a custom SHACL shape which checks all instances of a class have the correct/required points (e.g., sensor points, setpoints).
Below is an example of a custom SHACL shape for a VAV with reheat which checks that all instances of the `brick:RVAV` class have a heating coil with a valve position command, and a damper with a position command.
It also checks that all instances of the `brick:RVAV` class have a supply air temperature sensor and setpoint.

```turtle
@prefix brick: <https://brickschema.org/schema/Brick#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix : <urn:vav_extension/> .

:RVAV_Point_List a sh:NodeShape ;
    # sh:targetClass is the class that this shape applies to
    sh:targetClass brick:RVAV ;
    # sh:property is a list of constraints that must be satisfied

    # RVAV must have a Heating coil which has a valve position command
    sh:property [
        sh:path brick:hasPart ;
        sh:qualifiedValueShape [
            sh:class brick:Heating_Coil ;
            sh:property [
                sh:path brick:hasPoint ;
                sh:qualifiedValueShape [
                    sh:class brick:Valve_Position_Command ;
                    sh:qualifiedMinCount 1 ;
                ] ;
            ] ;
        ] ;
    ] ;
    # RVAV must have a Damper which has a position command
    sh:property [
        sh:path brick:hasPart ;
        sh:qualifiedValueShape [
            sh:class brick:Damper ;
            sh:property [
                sh:path brick:hasPoint ;
                sh:qualifiedValueShape [
                    sh:class brick:Position_Command ;
                    sh:qualifiedMinCount 1 ;
                ] ;
            ] ;
        ] ;
    ] ;
    # RVAV must have a Supply Air Temperature Sensor
    sh:property [
        sh:path brick:hasPoint ;
        sh:qualifiedValueShape [
            sh:class brick:Supply_Air_Temperature_Sensor ;
            sh:qualifiedMinCount 1 ;
        ] ;
    ] ;
    # RVAV must have a Supply Air Temperature Setpoint
    sh:property [
        sh:path brick:hasPoint ;
        sh:qualifiedValueShape [
            sh:class brick:Supply_Air_Temperature_Setpoint ;
            sh:qualifiedMinCount 1 ;
        ] ;
    ] .
```


### Example: Custom Shape for a Heat Exchanger (with 223P)

In this example, we will define a custom SHACL shape for an energy recovery ventilation (ERV) heat exchanger.
This is an air-to-air heat exhanger with two inputs and two outputs.
We want to define a custom SHACL shape that checks that all instances of this new ERV type have exactly two air inlets and two air outlets.

Brick does not care about inlets and outlets, so extending Brick to support an ERV would only involve ensuring that the ERV has two `brick:feeds` and two `brick:isFedBy` relationships.

```turtle
@prefix brick: <https://brickschema.org/schema/Brick#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix : <urn:erv_extension/> .

:Energy_Recovery_Ventilation_Heat_Exchanger a owl:Class, sh:NodeShape ;
    rdfs:subClassOf brick:Heat_Exchanger ;
    rdfs:label "Energy Recovery Ventilation Heat Exchanger" ;
    sh:property [
        sh:path brick:isFedBy ;
        sh:message "ERV must have exactly two inputs" ;
        sh:minCount 2 ;
        sh:maxCount 2 ;
    ] ;
    sh:property [
        sh:path brick:feeds ;
        sh:message "ERV must have exactly two outputs" ;
        sh:minCount 2 ;
        sh:maxCount 2 ;
    ] .
```

Using [ASHRAE 223P](https://open223.info), we can add additional constraints on the Brick class which represent the actual connection points and get more specific as to what kind of fluid is being exchanged.

```turtle
@prefix brick: <https://brickschema.org/schema/Brick#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix owl: <http://www.w3.org/2002/07/owl#> .
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix s223: <http://data.ashrae.org/standard223#> .
@prefix : <urn:erv_extension/> .

:Energy_Recovery_Ventilation_Heat_Exchanger a owl:Class, sh:NodeShape ;
    rdfs:subClassOf brick:Heat_Exchanger ;
    rdfs:label "Energy Recovery Ventilation Heat Exchanger" ;
    sh:property [ rdfs:comment "An ERV has 2 air outlet connection points."^^xsd:string ;
            sh:minCount 2 ;
            sh:path s223:hasConnectionPoint ;
            sh:qualifiedMinCount 2 ;
            sh:qualifiedValueShape [ sh:class s223:OutletConnectionPoint ;
                    sh:node [ sh:property [ sh:class s223:Fluid-Air ;
                                    sh:path s223:hasMedium ] ] ] ],
        [ rdfs:comment "An ERV has 2 air inlet connection points"^^xsd:string ;
            sh:minCount 2 ;
            sh:path s223:hasConnectionPoint ;
            sh:qualifiedMinCount 2 ;
            sh:qualifiedValueShape [ sh:class s223:InletConnectionPoint ;
                    sh:node [ sh:property [ sh:class s223:Fluid-Air ;
                                    sh:path s223:hasMedium ] ] ] ] .
```
