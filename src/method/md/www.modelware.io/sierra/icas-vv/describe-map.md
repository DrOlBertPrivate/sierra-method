---
template:
  id: https://www.modelware.io/sierra/icas-vv/describe-map
  name: "ICAS V&V Describe and Map"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
    - id: target
      type: iri
      required: true
---
# ICAS V&V Capability Planning: Describe → Map

Describe a V&V platform and map each capability it provides into an existing V&V capability space. Use this step for any V&V platform.

Model context: [[${ontology}]]. Use a description ontology that imports the [ICAS V&V vocabulary](../../../../oml/www.modelware.io/sierra/icas-vv.oml), or a context that includes that description. The [platform demonstrator](../../../../../model/oml/fireforce6.github.io/icas-vv/capabilities.oml) provides examples of the instance structure.

Use the two editors below to create and update model instances in [[${target}]]. Identify the platform first, then create its capabilities in the Map editor before creating the platform and selecting those capabilities. The platform creation dialog requires at least one capability. For an existing platform, create additional capabilities and then update its Capabilities field.

Use the built-in Label field in the creation dialog for the readable name; the identity column displays that label. Editor actions update the OML document. Save the affected OML document in VS Code before checking the file on disk or running CLI validation.

## Before you begin

Select the existing `VVCapabilitySpace` to which the capabilities will belong and record its full instance IRI. The demonstrator uses `https://fireforce6.github.io/icas-vv/capabilities#ICASCapabilitySpace`; this is a reference, not a fixed target for this method.

Use the prefix `icas-vv` for `https://www.modelware.io/sierra/icas-vv#`, `base` for `https://www.modelware.io/sierra/base#`, and `rdfs` for `http://www.w3.org/2000/01/rdf-schema#`. Follow the existing description files: import vocabularies with `uses`; if the selected space is declared in a separate description, import that description with `extends` and reference its existing instance. Do not duplicate the space.

## Describe

```table-editor
---
target: ${target}
columns: { this: { label: "Platform" } }
---
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix base: <https://www.modelware.io/sierra/base#> .
@prefix icas-vv: <https://www.modelware.io/sierra/icas-vv#> .

icas-vv:VVPlatformShape
    a sh:NodeShape ;
    sh:targetClass icas-vv:VVPlatform ;
    sh:property [
        sh:path base:description ;
        sh:name "Description" ;
        dash:editor dash:TextAreaEditor ;
        sh:maxCount 1 ;
    ] ;
    sh:property [
        sh:path icas-vv:providesCapability ;
        sh:name "Capabilities" ;
        sh:class icas-vv:VVCapability ;
        sh:minCount 1 ;
    ] ;
    .
```

### 1. Identify and name the platform

Identify the technical implementation or test environment. Reuse its existing `icas-vv:VVPlatform` instance, or create a named instance with a stable identifier. Record its readable name with `@rdfs:label` and describe its technical scope with `base:description`.

Keep the platform separate from what it can investigate. System level, system representation, and V&V focus belong to capabilities, not to the platform.

### 2. Describe the capabilities provided

Ask: what can this platform investigate, and under which conditions? List the distinct investigation abilities and describe each with a short name, purpose, and relevant examples. Create or reuse an `icas-vv:VVCapability` instance for each ability and record its purpose with `base:description`.

Link the platform to each capability using `icas-vv:providesCapability`. A platform must provide at least one capability and may provide several.

### 3. Consider each capability separately

For each candidate capability, perform steps 4–6 independently. Separate capabilities of the same platform when their system level, system representation, or set of V&V focuses differs. Give each separate capability its own identifier and description.

Several focuses may characterize one coherent investigation ability at the same level and representation. Record that focus set together. Do not merge distinct abilities merely by taking the union of their focuses, and do not create one capability per focus automatically.

If an ability spans multiple levels or representations, identify the supported combinations and describe separate capabilities for them. Do not assume that every possible combination is supported. Document uncertain assignments in `base:description` and resolve them before treating the mapping as confirmed.

## Map

Create one row per capability. Select the existing capability space for each row. Level and representation are single values; Focus accepts multiple values (use Ctrl+click for individual choices or Shift for a range). Return to the Describe editor to create the platform with these capabilities, or link them to an existing platform.

```table-editor
---
target: ${target}
columns: { this: { label: "Capability" } }
---
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix base: <https://www.modelware.io/sierra/base#> .
@prefix icas-vv: <https://www.modelware.io/sierra/icas-vv#> .

icas-vv:VVCapabilityShape
    a sh:NodeShape ;
    sh:targetClass icas-vv:VVCapability ;
    sh:property [
        sh:path base:description ;
        sh:name "Description" ;
        dash:editor dash:TextAreaEditor ;
        sh:maxCount 1 ;
    ] ;
    sh:property [
        sh:path icas-vv:belongsToCapabilitySpace ;
        sh:name "Capability Space" ;
        sh:class icas-vv:VVCapabilitySpace ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
    ] ;
    sh:property [
        sh:path icas-vv:hasSystemLevel ;
        sh:name "System Level" ;
        sh:datatype xsd:string ;
        sh:in ( "Component" "Subsystem" "System" ) ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
    ] ;
    sh:property [
        sh:path icas-vv:hasSystemRepresentation ;
        sh:name "System Representation" ;
        sh:datatype xsd:string ;
        sh:in ( "Physical" "Hybrid" "Virtual" ) ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
    ] ;
    sh:property [
        sh:path icas-vv:hasVVFocus ;
        sh:name "V&V Focus" ;
        sh:datatype xsd:string ;
        sh:in ( "FunctionalBehaviour" "PhysicalEffects" "SystemIntegration" "FaultScenarios" ) ;
        sh:minCount 1 ;
    ] ;
    .
```

### 4. Assign exactly one System Level

Set `icas-vv:hasSystemLevel` on the capability to exactly one of:

- `Component`
- `Subsystem`
- `System`

Choose the level of the system being investigated, rather than the size or complexity of the platform itself. Use separate capability instances if different levels apply.

### 5. Assign exactly one System Representation

Set `icas-vv:hasSystemRepresentation` on the capability to exactly one of:

- `Physical`
- `Hybrid`
- `Virtual`

Choose the representation used for this investigation ability. If multiple representations apply to distinct configurations, describe separate capabilities. Qualifiers such as "predominantly physical" or "partly hybrid" remain explanatory text; they are not additional enumeration values or numerical weights.

### 6. Assign one or more V&V Focus values

Set `icas-vv:hasVVFocus` on the capability to one or more of:

- `FunctionalBehaviour`
- `PhysicalEffects`
- `SystemIntegration`
- `FaultScenarios`

In OML, repeat the `icas-vv:hasVVFocus` assertion for each selected value, as in the demonstrator. Do not store a comma-separated list as one string. Revisit step 3 if the selected focuses describe separate abilities with different scopes.

### 7. Record the structured capability mapping

Assign each capability to the selected existing space using `icas-vv:belongsToCapabilitySpace`. Each capability must belong to exactly one space.

Produce one row per capability using the following result format. Use instance IRIs for identity and labels for readability. Repeat the platform reference across rows when it provides several capabilities.

| Platform IRI / name | Capability IRI / name | Investigation description | Capability Space IRI | System Level | System Representation | V&V Focus set | Assumptions / open points |
|---|---|---|---|---|---|---|---|
| Platform reference | Separate capability reference | Purpose and examples | Existing space reference | One allowed value | One allowed value | One or more allowed values | Explanation, or none |

The editors record the mapping in the OML description: platform-to-capability links on `VVPlatform`; space, level, representation, and focus assertions on each `VVCapability`. Keep assumptions in the Description field (`base:description`); the ontology has no dedicated assumption or confirmation-status property. The result table is an optional reporting format, not a new ontology element.

## Reference examples

The [existing demonstrator](../../../../../model/oml/fireforce6.github.io/icas-vv/capabilities.oml) illustrates the structure:

| Platform | Capability | System Level | System Representation | V&V Focus set |
|---|---|---|---|---|
| CONCERTO | `CONCERTOComponentPhysical` | Component | Physical | FunctionalBehaviour, PhysicalEffects |
| CONCERTO | `CONCERTOSubsystemHybrid` | Subsystem | Hybrid | FunctionalBehaviour, FaultScenarios |
| WISDOM | `WISDOMSubsystemHybrid` | Subsystem | Hybrid | FunctionalBehaviour, PhysicalEffects, SystemIntegration |
| NewFlAir | `NewFlAirSubsystemHybrid` | Subsystem | Hybrid | FunctionalBehaviour, SystemIntegration |
| NewFlAir | `NewFlAirSystemVirtual` | System | Virtual | FunctionalBehaviour, SystemIntegration, FaultScenarios |

These are illustrative mappings. In particular, the CONCERTO and NewFlAir combinations include documented demonstrator assumptions. Use their structure for a new platform; determine its actual assignments from its own investigation abilities.

## Completion criteria

- The platform has a stable identity, readable name, and technical description.
- It provides at least one separately identified and described capability.
- Every capability belongs to exactly one existing capability space.
- Every capability has exactly one allowed system level, exactly one allowed system representation, and at least one allowed focus.
- Different level, representation, or focus-set assignments remain separate capabilities; no dimension is asserted directly on the platform.
- The structured result and the OML assertions agree, and uncertain assignments are explicitly documented.

Save the affected OML document, then run the existing `oml lint` and `oml validate` commands. Review the completion criteria as well: tool validation alone does not establish that the chosen investigation scopes or dimension assignments are factually correct.

This step ends with the described and mapped capabilities. Coverage assessment, required capability spaces, gap analysis, and planning are subsequent work outside this step.
