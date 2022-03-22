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

Timeseries Storage
==================

```{note}
This feature is new in Brick v1.2
```

It is possible to embed within a Brick model the metadata indicating where timeseries data is stored and how particular Points are identified --- this enables the automatic retrieval of timeseries data. This metadata is associated with Brick Point instances, which represent sources of data, through the `brick:timeseries` property.

The `brick:timeseries` property relates a Brick Point instance to a `brick:TimeseriesReference` object. This object has exactly one `brick:hasTimeseriesId` property, whose value is a string denoting the identifier or primary key of the instance's data in some database. The database itself is realized as an object and is related to the `brick:TimeseriesReference` through the `brick:storedAt` property.

At this point in time, Brick does not mandate what the properties are on the database instance. As best practices arise, this will be codified in a future release of Brick.

## Simple Example

```turtle
:sensor1    a   brick:Temperature_Sensor ;
    brick:hasUnit unit:DEG_F ;
    brick:timeseries [
        brick:hasTimeseriesId   "8f541ba4-c437-43ba-ba1d-5c946583fe54" ;
        brick:storedAt  :database ;
    ] ;
.

:sensor2    a   brick:Temperature_Sensor ;
    brick:hasUnit unit:DEG_F ;
    brick:timeseries [
        brick:hasTimeseriesId   "38b5fa0e-407e-4a23-8800-6ec4f6d60785" ;
        brick:storedAt  :database ;
    ] ;
.

# the properties on the database instance are non-normative
:database   a   brick:Database ;
    rdfs:label  "Postgres Timeseries Storage" ;
    :connstring "postgres://1.2.3.4/data" ;
.
```
