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
This feature is new in Brick v1.4 and relies on the integration with RealEstateCore
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
