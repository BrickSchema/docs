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


Using Brick and RealEstateCore Together
===============

```{note}
Brick-REC integration was introduced in Brick v1.4 and expanded in v1.5.
```

This document summarizes the changes to Brick that allow it to be used in conjunction with RealEstateCore  (REC).


## Locations

All Brick `Location` classes have been deprecated as of 1.4 and replaced with equivalents in RealEstateCore.
These deprecations appear like this in the Turtle file:

```turtle
brick:Building
  owl:deprecated "true"^^xsd:boolean ;
    brick:deprecatedInVersion "1.4.0" ;
    brick:deprecationMitigationMessage "Brick location classes are being phased out in favor of RealEstateCore classes. For a replacement, consider rec:Building" ;
    brick:isReplacedBy rec:Building ;
.
```

The `brick:isReplacedBy` property indicates the REC class that should be used in place of the deprecated Brick class.
Using a Brick class in a Brick model will raise warnings, not errors, during SHACL validation.
SHACL rules included in Brick will also add the new REC class to the model, so that the model is still valid.

## Relationships

Brick relationships have been mapped to REC relationships where possible.

See this table:

| Brick Relationship | REC Relationship |
|--------------------|------------------|
| `brick:hasLocation` | `rec:locatedIn` |
| `brick:isLocationOf` | `rec:isLocationOf` |
| `brick:feeds` | `rec:feeds` |
| `brick:isFedBy` | `rec:isFedBy` |
| `brick:hasPoint` | `rec:hasPoint` |
| `brick:isPointOf` | `rec:isPointOf` |
| `brick:hasPart` | `rec:hasPart` |
| `brick:isPartOf` | `rec:isPartOf` |
| `brick:hasPart` (collection membership) | `rec:includes` |

## Collections

```{note}
This section describes changes introduced in Brick v1.5.
```

As of Brick v1.5.0, `brick:Collection` has been deprecated in favor of `rec:Collection` to remove redundancy between the two ontologies. Custom collections should now be typed as `rec:Collection` and use `rec:includes` to express membership:

```turtle
@prefix rec: <https://w3id.org/rec#> .
@prefix : <urn:bldg#> .

:my_collection a rec:Collection ;
    rdfs:label "All of the dampers" ;
    rec:includes :dmp1, :dmp2 .
```

All built-in Brick collection classes (e.g. `brick:HVAC_System`, `brick:Air_Loop`, `brick:PV_Array`) are now subclasses of `rec:Collection` rather than the deprecated `brick:Collection`.
