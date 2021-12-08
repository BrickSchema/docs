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

# Software Interfaces for Brick

This section answers the question of how Brick can be used to create data-driven applications.
Rather than relying on a standard API to define interactions with the building instance, Brick provides a set of *references* that describe the interface between Brick and some other API, software library or other digital representation.
This affords a great deal of flexibility in how software can choose to interact with the building and its data.


Recall that the intent of a Brick model is to describe the data sources in a building and their context.
The data sources themselves are represented by instances of the Brick `Point` class, and the context of these data sources is provided by instances of locations, equipment, systems and other building entities.
These instances are associated with data sources through [Brick relationships](/brick/relationships).

This means that a Brick model can be used to find data sources and other entities.
This is usually done through SPARQL queries against the Brick model graph.
The information returned by these queries can be used to configure a piece of software.
For example, a SPARQL query can return:
- the configuration information required to access live or historical data through a time series database
- the parameters for accessing an I/O point on a BMS network
- the parameters for retrieving the geometry of an entity from an IFC model

See the [External Representations](/metadata/external-representations) section for details and examples of how these references are represented in Brick.

## Tutorials

- [Discovering and Retrieving Timeseries Data with Brick and TimescaleDB](https://www.youtube.com/watch?v=kZYNXoiM8gk): tutorial video showing how the Brick timeseries reference can be used to retrieve data from a TimescaleDB database --- the techniques are not TimescaleDB-specific.
- [Brick Data Retrieval Demo](https://github.com/gtfierro/brick-data-retrieval-demo): supporting repository for the above video, providing a Jupyter notebook implementing the data retrieval method in Python. The technique is not TimescaleDB-specific and only one function in the codebase is TimescaleDB-specific. The rest should be easily reusable.

## Brick Platforms

There are two main open-source platforms for interacting with Brick that *do* define APIs:
- [Mortar](https://github.com/gtfierro/mortar): a self-hostable platform for retrieving bulk timeseries data using Brick queries for context. The [Mortar platform website](https://mortardata.org/intro.html) has publicly available data and a [library](https://github.com/SoftwareDefinedBuildings/mortar-analytics) of public implementations of common data analytics
- [Brick Example Server](https://github.com/BrickSchema/brick-example-server): a self-hostable platform demonstrating how a Brick model can be abstracted by an HTTP API
