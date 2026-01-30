Point Collections
============

Point Collections are Brick collections that organize sets of related points (and optionally other point collections) so they can be managed as a coherent bundle. They are intentionally lightweight: the membership of a Point Collection does **not** imply any functional or causal relationship between its members. Instead, Point Collections serve UI, configuration, and lifecycle tasks such as presenting related telemetry to operators and exchanging configuration packages between software systems.

## Core Concepts

- **`brick:Point_Collection`** is a subclass of `brick:Collection`. Point Collections can be created whenever you need a named bundle of points. Use `rdfs:label` to provide a human-friendly title such as `"VAV1 Frost Detection Points"`.
- **Membership** is described with `brick:hasPart`. A Point Collection may include:
  - individual `brick:Point` instances
  - other `brick:Point_Collection` instances (nesting lets you build hierarchies such as *Zone Collections → Unit Collections → Individual Points*)
- **ICT Equipment hosts points directly.** Instances of `brick:ICT_Equipment` relate to points with `brick:hostsPoint` (inverse: `brick:isHostedBy`). Point Collections are optional logical groupings and are *not* what controllers host.
- **Controllers and equipment.** Controllers (a subclass of ICT equipment) interact with the equipment they supervise via `brick:controls` / `brick:isControlledBy`

```{important}
Avoid inferring semantics from group membership alone. For example, adding a setpoint and its min/max limits to the same Point Collection does *not* mean those limits constrain that setpoint. Model explicit relationships if you need to express functional ties.
```

This is *not* a replacement for `brick:hasPoint`! You should still use `brick:hasPoint`/`brick:isPointOf` to relate points to their equipment. This modeling construct focuses on the networking/instrumentation of the system.

### Examples

A VAV can have a `brick:Supply_Air_Flow_Sensor`, but that point is **hosted** by a controller.

```turtle
@prefix bldg: <http://example.com/vav#> .
@prefix brick: <https://brickschema.org/schema/Brick#> .

bldg:VAV_1 a brick:Variable_Air_Volume_Box ;
    brick:hasPoint bldg:VAV_1_SAF_Sensor .

bldg:Ctrl_1 a brick:Controller ;
    brick:controls bldg:VAV_1 ;
    brick:hostsPoint bldg:VAV_1_SAF_Sensor .

bldg:VAV_1_SAF_Sensor a brick:Supply_Air_Flow_Sensor .
```

A controller can host its own points (e.g., `brick:On_Off_Status`) even when those points describe controller state rather than any downstream equipment.
```turtle
@prefix bldg: <http://example.com/controller#> .
@prefix brick: <https://brickschema.org/schema/Brick#> .

bldg:Ctrl_2 a brick:Controller ;
    brick:hasPoint bldg:Ctrl_2_On_Off_Status ;
    brick:hostsPoint bldg:Ctrl_2_On_Off_Status .

bldg:Ctrl_2_On_Off_Status a brick:On_Off_Status .
```

## Modeling Guidelines

- **Scope and intent:** Use Point Collections for display, configuration exchange, and packaging telemetry. They are not a substitute for equipment, loop, or system modeling.
- **Granularity:** Prefer smaller, task-specific groups (e.g., startup points, alarm points) rather than one monolithic bundle. Nest groups when you need coarse and fine-grained organization.
- **Custom subclasses:** If your organization needs formal semantics, define project-specific subclasses of `brick:Point_Collection` (e.g. `my:Startup_Point_Collection`) and document their intended meaning. Brick itself does not ship a standardized taxonomy of Point Collection types.
- **Labeling and metadata:** Provide `rdfs:label` and, when useful, add entity properties (for example a revision number or deployment status) to describe the package.

## Example: Controller Hosting Points (with an optional Point Collection)

```turtle
@prefix bldg: <http://example.com/controller#> .
@prefix brick: <https://brickschema.org/schema/Brick#> .
@prefix rec: <https://w3id.org/rec#> .
@prefix unit: <http://qudt.org/vocab/unit/> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

bldg:Controller_1 a brick:Controller ;
    rdfs:label "Main Building Controller" ;
    brick:hostsPoint bldg:VAV1_Temperature_Sensor, bldg:VAV1_Occupancy_Sensor ;
    brick:controls bldg:VAV1 .

bldg:VAV1PointCollection a brick:Point_Collection ;
    rdfs:label "VAV1 Point Collection" ;
    brick:hasPart bldg:VAV1_Temperature_Sensor, bldg:VAV1_Occupancy_Sensor .

bldg:VAV1_Temperature_Sensor a brick:Supply_Air_Temperature_Sensor ;
    brick:hasUnit unit:DEG_C .

bldg:VAV1_Occupancy_Sensor a brick:Occupancy_Sensor .

bldg:VAV1 a brick:Variable_Air_Volume_Box ;
    brick:feeds bldg:Zone1 .

bldg:Zone1 a rec:HVACZone .
```

This pattern scales to larger controllers by nesting Point Collections—for example, `bldg:StartupPoints` and `bldg:DiagnosticsPoints` contained inside `bldg:AHU1PointCollection`. The controller still uses `brick:hostsPoint` for the individual points; the groups remain for logical organization.

## Query Patterns

Retrieve all points hosted by a piece of ICT Equipment (e.g., a Controller):

```sparql
SELECT ?point WHERE {
    ?controller a/rdfs:subClassOf* brick:ICT_Equipment ;
                brick:hostsPoint ?point .
    ?point a/rdfs:subClassOf* brick:Point .
}
```

Identify the Point Collections associated with a zone of interest (for UI organization):

```sparql
SELECT DISTINCT ?coll WHERE {
    ?controller brick:controls ?equip ;
                brick:hostsPoint ?point .
    ?equip brick:feeds+ ?zone .
    ?zone a brick:HVAC_Zone .
    ?coll brick:hasPart+ ?point ;
           a brick:Point_Collection .
}
```

## Migration Notes

- When exporting configuration to external tools, deliver the Point Collection identifiers and their member points. Consuming systems that do not yet understand Point Collections can treat them as named collections of points without additional semantics.
- Future Brick releases may add convenience shapes or inference rules (for example, deriving `brick:hostsPoint` relationships to points). Until then, use property paths (`brick:hasPart+`) in queries and tooling that need to traverse Point Collection hierarchies.
