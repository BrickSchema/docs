Using Tags
==========

A **tag** is an atomic term that is used to annotate an entity in a Brick model.
The set of tags associated with a Brick entity can be queried by users and applications; Brick also supports the automatic association of tags with entities based on which Brick class they are an instance of.

In contrast to other metadata efforts such as Project Haystack, Brick does not use tags to define what an entity is.
Tags constitute a ["folksonomic"](https://en.wikipedia.org/wiki/Folksonomy) approach to capturing knowledge.
Because tags are usually just words (e.g. `hot`, `water`, `sensor`, `evaporative`), there is no explicit mechanism to state how a certain word is *intended*.
As a result, while tags are great for to informally annotate properties whose meaning is already known to the user, they are not an effective mechanism for communicating semantic information in a consistent and interpretable manner {cite}`fierro2019beyond`.

```{note}
You may want to review the section on [Inference](../lifecycle/inference) to understand how an external software reasoner supports use of a Brick model.
```

## Querying Tags

Tags are instances of the `brick:Tag` class, and are defined as part of the Brick distribution in the `https://brickschema.org/schema/1.1/BrickTag#` namespace (commonly abbreviated as `tag:`).
The full set of Brick tags can be retrieved with the following SPARQL query:

```sparql
PREFIX brick: <https://brickschema.org/schema/1.1/Brick#> .
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
PREFIX tag: <https://brickschema.org/schema/1.1/BrickTag#> .

SELECT ?tag ?label WHERE {
    ?tag    a          brick:Tag .
    ?tag    rdfs:label ?label
}
```

### Tags and Brick Classes

Most, if not all, Brick classes have a set of associated tags, which will be "inherited" by any Brick entity which is an instance of that class.
Tags are related to a Brick class via the `brick:hasAssociatedTag` relationship.
For example, to fetch the tags associated with the `brick:Zone_Air_Temperature_Sensor` class, execute the following query:

```sparql
PREFIX brick: <https://brickschema.org/schema/1.1/Brick#> .
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
PREFIX tag: <https://brickschema.org/schema/1.1/BrickTag#> .

SELECT ?tag WHERE {
    brick:Zone_Air_Temperature_Sensor brick:hasAssociatedTag ?tag
}
```

### Tags and Brick Entities

After **inference** is applied to a Brick model, Brick entities will inherit tags from their Brick classes.
These inferred tags are related to the Brick entity via the `brick:hasTag` relationship.
The set of tags associated with a Brick entity will be the __union__ of the tags associated with each of the classes it instantiates.

For example, consider an instance of `brick:Zone_Air_Temperature_Sensor` in a Brick model (before inference is applied):

```turtle
@prefix brick: <https://brickschema.org/schema/1.1/Brick#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix mybldg: <mybuilding#> .

mybldg:t1   a   brick:Zone_Air_Temperature_Sensor .
```

After inference, the model will consist of the following:

```turtle
@prefix brick: <https://brickschema.org/schema/1.1/Brick#> .
@prefix tag: <https://brickschema.org/schema/1.1/BrickTag#> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix mybldg: <mybuilding#> .

# all parent classes of brick:Zone_Air_Temperature_Sensor are added to the model
mybldg:t1   a   brick:Zone_Air_Temperature_Sensor,
                brick:Air_Temperature_Sensor,
                brick:Temperature_Sensor,
                brick:Sensor,
                brick:Point ;
# here is the union of all tags associated with the above Brick classes
            brick:hasTag    tag:Zone, tag:Air, tag:Temperature,
                            tag:Sensor, tag:Point .
```

To retrieve the set of tags associated with a specific entity, use the following query (remembering to apply inference first):

```sparql
PREFIX brick: <https://brickschema.org/schema/1.1/Brick#> .
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
PREFIX tag: <https://brickschema.org/schema/1.1/BrickTag#> .

SELECT ?tag WHERE {
    mybldg:t1   brick:hasTag ?tag
}
```

### Finding Stuff with Tags

The primary use case of tags is as a form of "keyword search".
Memorizing the whole Brick schema is neither tractable nor expected; tags can be used by user interfaces, automated tools or users themselves to help guide or direct a search for relevant classes or entities in a Brick model.

To find all classes with a given tag (helpful for figuring out which class to use for a new entity), use a variation of the following query (this example finds all classes with the `air` and `temperature` tags):

```sparql
PREFIX brick: <https://brickschema.org/schema/1.1/Brick#> .
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
PREFIX tag: <https://brickschema.org/schema/1.1/BrickTag#> .

SELECT ?class WHERE {
    ?class  brick:hasAssociatedTag tag:Air, tag:Temperature .
}
```

To find all *instances* with a given tag, remember to apply inference, and then execute a query like:

```sparql
PREFIX brick: <https://brickschema.org/schema/1.1/Brick#> .
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
PREFIX tag: <https://brickschema.org/schema/1.1/BrickTag#> .

SELECT ?entity WHERE {
    ?entity  brick:hasTag tag:Air, tag:Temperature .
}
```

## Tag Inference Implementation

Coming soon...


## References

```{bibliography} ../references.bib
```
