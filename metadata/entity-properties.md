Entity Properties
=================

```{note}
This feature is new in Brick v1.2
```

Entity Properties are attributes or characteristics of Brick entities. The values of Entity Properties should change rarely, if ever; if a value is changing regularly, then it is likely better modeled as a Brick Point. Entity properties are useful for modeling static charcteristics such as:
- floor area, room volume
- phase count, phases, and flow of electric meters
- which operational stage a point is associated with
- how the values of a given Point are aggregated
- and many others!


An Entity Property has two parts:
- the *Entity Property* itself, which is a named relationship between a Brick entity and a property value
- the property value, which is its own entity with a set of properties capturing current value, associated units and others

Entity Properties, and their associated classes and relationships, are defined in the `brick` namespace.

The use of Entity Properties generally adheres to the following pattern:

```{image} ../img/entity-property.png
:width: 600px
:align: center
```

## Simple Example

To illustrate the use of entity properties, consider the following models

### Area of a Room

```turtle
:room1  a   brick:Room ;
    brick:hasArea  [
        brick:value  "100"^^xsd:decimal ;
        brick:hasUnit   unit:FT2 ;
    ] ;
.
```

```{note}
The `[]` square brackets above are a [blank node](https://www.w3.org/2007/02/turtle/primer/) notation. Everything in the `[]` is the predicate and object for an *implied* entity that is also the object of the `brick:hasArea` relationship. You can consider the above equivalent to the following:


```turtle
:room1  a   brick:Room ;
        brick:hasArea  :a .
        
:a brick:value  "100"^^xsd:decimal ;
    brick:hasUnit   unit:FT2 .
```

The model here is relatively straight forward. The area is a (usually anonymous) Brick entity with a numerical value (indicated by `brick:value`) and a unit (indicated by `brick:hasUnit`). The area entity is associated with a Brick room through the `brick:hasArea` property.

### Peak Hourly Power Meter

This is the model of a power sensor which tracks the peak hourly real power.

```turtle
:hourly_peak_real_power_meter   a   brick:Power_Sensor ;
    brick:hasUnit   unit:KiloW ;
    brick:aggregate [
        brick:aggregationFunction    "max" ;
        brick:aggregationInterval    "RPT1H" ;
    ] ;
    brick:hasComplexity [
        brick:value  "real" ;
    ]
.
```

This is a little more complex than the earlier example. There are two entity properties associated with the `Power_Sensor` entity. The first is `brick:hasComplexity` which tells us that the sensor measures real power. The second is `brick:aggregate`. The two properties of the `brick:aggregate` value tell us that the sensor's values are aggregated on a 1 hour window; this is indicated by the `brick:aggregationInterval` property. The `brick:aggregationFunction` indicates that the values within that 1 hour interval are aggregated to the maximum of the values in that window.

```{note}
`brick:aggregationInterval` uses the [ISO 8601 Duration specification](https://en.wikipedia.org/wiki/ISO_8601#Durations) to indicate the length of an interval. It also incorporates an `R` prefix to indicate a [repeating duration](https://en.wikipedia.org/wiki/ISO_8601#Repeating_intervals).

Some common other intervals are:
- every 1 hour: `RPT1H`
- every 2 hours: `RPT2H`
- every 15 minutes: `RPT15M`
- every month: `RP1M`
- every 30 days: `RP30D`
```
