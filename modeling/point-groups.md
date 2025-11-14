Point Groups
============

Point Groups are Brick collections that organize sets of related points (and optionally other point groups) so they can be managed as a coherent bundle. They are intentionally lightweight: the membership of a Point Group does **not** imply any functional or causal relationship between its members. Instead, Point Groups serve UI, configuration, and lifecycle tasks such as provisioning a controller, presenting related telemetry to operators, and exchanging configuration packages between software systems.

## Core Concepts

- **`brick:Point_Group`** is a subclass of `brick:Collection`. Point Groups can be created whenever you need a named bundle of points. Use `rdfs:label` to provide a human-friendly title such as `"VAV1 Frost Detection Points"`.
- **Membership** is described with `rec:includes`. A Point Group may include:
  - individual `brick:Point` instances
  - other `brick:Point_Group` instances (nesting lets you build hierarchies such as *Zone Groups → Unit Groups → Individual Points*)
- **Controllers host groups, not raw points.** Controllers relate to Point Groups via `brick:hosts`. This reflects how most BAS controllers exchange and deploy configuration bundles. The inverse relationship is `brick:isHostedBy`.
- **Controllers and equipment.** Controllers interact with the equipment they supervise via `brick:controls` / `brick:isControlledBy`. Controllers can also indicate the zones they are configured to affect through `brick:concerns` / `brick:isConcernedBy`.

```{important}
Avoid inferring semantics from group membership alone. For example, adding a setpoint and its min/max limits to the same Point Group does *not* mean those limits constrain that setpoint. Model explicit relationships if you need to express functional ties.
```

This is *not* a replacement for `brick:hasPoint`! You should still use `brick:hasPoint`/`brick:isPointOf` to relate points to their equipment. This modeling construct focuses on the networking/instrumentation of the system.

## Modeling Guidelines

- **Scope and intent:** Use Point Groups for display, configuration exchange, and packaging telemetry. They are not a substitute for equipment, loop, or system modeling.
- **Granularity:** Prefer smaller, task-specific groups (e.g., startup points, alarm points) rather than one monolithic bundle. Nest groups when you need coarse and fine-grained organization.
- **Custom subclasses:** If your organization needs formal semantics, define project-specific subclasses of `brick:Point_Group` (`my:Startup_Point_Group`) and document their intended meaning. Brick itself does not ship a standardized taxonomy.
- **Labeling and metadata:** Provide `rdfs:label` and, when useful, add entity properties (for example a revision number or deployment status) to describe the package.

## Example: Controller Hosting a Point Group

```turtle
@prefix bldg: <http://example.com/controller#> .
@prefix brick: <https://brickschema.org/schema/Brick#> .
@prefix rec: <https://w3id.org/rec#> .
@prefix unit: <http://qudt.org/vocab/unit/> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

bldg:Controller_1 a brick:Controller ;
    rdfs:label "Main Building Controller" ;
    brick:hosts bldg:VAV1PointGroup ;
    brick:controls bldg:VAV1 ;
    brick:concerns bldg:Zone1 .

bldg:VAV1PointGroup a brick:Point_Group ;
    rdfs:label "VAV1 Point Group" ;
    rec:includes bldg:VAV1_Temperature_Sensor, bldg:VAV1_Occupancy_Sensor .

bldg:VAV1_Temperature_Sensor a brick:Supply_Air_Temperature_Sensor ;
    brick:hasUnit unit:DEG_C .

bldg:VAV1_Occupancy_Sensor a brick:Occupancy_Sensor .

bldg:VAV1 a brick:Variable_Air_Volume_Box ;
    brick:feeds bldg:Zone1 .

bldg:Zone1 a rec:HVACZone .
```

This pattern scales to larger controllers by nesting Point Groups—for example, `bldg:StartupPoints` and `bldg:DiagnosticsPoints` contained inside `bldg:AHU1PointGroup`, which is hosted by the controller.

## Query Patterns

Retrieve all points hosted by a controller (following nested groups):

```sparql
SELECT ?point WHERE {
    ?controller a/rdfs:subClassOf* brick:Controller ;
                brick:hosts ?group .
    ?group rec:includes+ ?point .
    ?point a/rdfs:subClassOf* brick:Point .
}
```

Identify the Point Groups associated with a zone of interest:

```sparql
SELECT DISTINCT ?group WHERE {
    ?controller brick:concerns :Zone1 ;
                brick:hosts ?group .
    ?group a brick:Point_Group .
}
```

## Migration Notes

- When exporting configuration to external tools, deliver the Point Group identifiers and their member points. Consuming systems that do not yet understand Point Groups can treat them as named collections of points without additional semantics.
- Future Brick releases may add convenience shapes or inference rules (for example, deriving `brick:hosts` relationships to points). Until then, use property paths (`rec:includes+`) in queries and tooling that need to traverse Point Group hierarchies.
