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

Inference and Reasoning
=======================

**Inference** is the process by which implied information is discovered and made explicit in a model. Informally, you can think of this process as automatically adding additional metadata to your Brick model.

The Brick ontology definition includes a set of formal axioms that outline what information can be implied by the statements in a particular Brick model. These axioms are interpreted by a piece of software called a **reasoner**, which derives all implied information and adds it to the Brick model. The process of performing inference is sometimes referred to as **reasoning**.

```{note}
Brick uses the OWL 2 RL profile, described [in this W3C document](https://www.w3.org/TR/owl2-profiles/). [This section](https://www.w3.org/TR/owl2-profiles/#Reasoning_in_OWL_2_RL_and_RDF_Graphs_using_Rules) of the W3C documentation provides first-order definitions of the OWL 2 RL axioms, which may be helpful in understanding the specific reasoning mechanisms at hand. Reading and understanding these documents are *not* necessary to effective use of Brick.
```

## Brick Inference Results

Applying inference to a Brick model can automatically add a great deal of information that would otherwise need to be manually added. This includes, but is not limited to:

- **Superclasses:** for Brick entities that are instances of Brick classes, reasoning will also attach the superclasses to the instance
- **Inverse relationships**: reasoning will automatically add the "inverse" Brick relationships where needed (e.g. `brick:feeds` is the reverse of `brick:isFedBy`)
- **Tags**: reasoning will add tags to Brick entities which are instances of Brick classes

While using a reasoner is not strictly necessary for effective use of Brick, it does fill in a substantial amount of information that makes Brick easier to use and a more consistent experience.

### Example: Superclasses

Before reasoning:

```turtle
@prefix brick: <https://brickschema.org/schema/Brick#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix mybldg: <mybuilding#> .

mybldg:t1   a   brick:Zone_Air_Temperature_Sensor .
```

After reasoning:

```turtle
@prefix brick: <https://brickschema.org/schema/Brick#> .
@prefix tag: <https://brickschema.org/schema/BrickTag#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix mybldg: <mybuilding#> .

# The reasoner adds all parent classes of brick:Zone_Air_Temperature_Sensor as types
# of the mybldg:t1 entity.
# For clarity, we are eliding the tags that would also be associated with mybldg:t1
mybldg:t1   a   brick:Zone_Air_Temperature_Sensor,
                brick:Air_Temperature_Sensor,
                brick:Temperature_Sensor,
                brick:Sensor,
                brick:Point .
```

### Example: Tags

Before reasoning:

```turtle
@prefix brick: <https://brickschema.org/schema/Brick#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix mybldg: <mybuilding#> .

mybldg:t1   a   brick:Zone_Air_Temperature_Sensor .
```

After reasoning:

```turtle
@prefix brick: <https://brickschema.org/schema/Brick#> .
@prefix tag: <https://brickschema.org/schema/BrickTag#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix mybldg: <mybuilding#> .

# The reasoner adds the tags associated with brick:Zone_Air_Temperature_Sensor and all
# of its superclasses to the mybldg:t1 entity.
# For clarity, we are eliding the superclasses that would also be added
mybldg:t1   a   brick:Zone_Air_Temperature_Sensor,
            brick:hasTag    tag:Zone, tag:Air, tag:Temperature,
                            tag:Sensor, tag:Point .
```

### Example: Inverse Relationships

Before reasoning:

```turtle
@prefix brick: <https://brickschema.org/schema/Brick#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix mybldg: <mybuilding#> .

mybldg:t1   a   brick:Zone_Air_Temperature_Sensor .
mybldg:vav1 a   brick:VAV ;
    brick:hasPoint mybldg:t1 .
```

After reasoning:

```turtle
@prefix brick: <https://brickschema.org/schema/Brick#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix mybldg: <mybuilding#> .

# the inverse relationship (brick:isPointOf) is added by the reasoner.
# Note that for clarity we are eliding the tags and superclasses that would
# be associated with mybldg:t1 and mybldg:vav1
mybldg:t1   a   brick:Zone_Air_Temperature_Sensor ;
    brick:isPointOf mybldg:vav1 .
mybldg:vav1 a   brick:VAV ;
    brick:hasPoint mybldg:t1 .
```

## Performing Inference

Coming soon!

Resources:
- [`brickschema` package support](https://brickschema.readthedocs.io/en/latest/quickstart.html)
