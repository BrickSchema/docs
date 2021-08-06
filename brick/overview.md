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

Design Principles
======================

Brick is an ontology-based metadata schema that captures the entities and relationships necessary for effective representations of buildings and their subsystems.
Brick describes buildings in a machine readable format to enable programmatic exploration of different operational, structural and functional facets of a building.

## Design Principles

Brick adheres to the following design principles:

* **Completeness**: A schema should represent all the information (such as a sensor’s location, type, etc.) required by building applications.
* **Expressivity**: A schema should capture the diverse family of entities and relationships between them that are present in a building's BMS and expressed in canonical energy-, operations- and management-oriented applications and scenarios.
* **Usability**: A schema should be not too complex for users to easily understand and use.
* **Consistency**: A schema should be able to enforce consistency in modeling processes across different users.
* **Extensibility**: A schema should be easily extensible to cover new concepts in a consistent way.
