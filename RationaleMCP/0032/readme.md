# Modelica Change Proposal MCP-0032: selective model extension

**Authors:** Christoff Bürger (Christoff.Buerger@3ds.com)

The minutes of _Modelica Design Meetings_ discussing this MCP are summarized in [`minutes.md`](minutes.md).

## Status: in development

- [x] **Concept:** approved

- [x] **Design:** completed, waiting for approval

- [ ] **Specification-incorporation:** not started / to do next

## Summary
This MCP defines language extensions for structural non-monotonic model variation. The proposed language features enable selective model extension: the well-defined refinement of models by deselecting components and connections not of interest or inappropriate for a new design. Deselection is modeled in terms of non-exhaustive inheritance; to that end `extends`-clause modifiers now support exclusion of base-class elements from inheritance. Deselected elements are treated in extending models as if they never have been defined in their base-class. In case of deselected components, also all their connections are deselected.

The main contribution of selective model extension is to enable unforeseen structural variability without requiring deliberately prepared base-models. Such non-monotonic variations are for example required to incorporate clocking - i.e., use Modelica Synchronous - in existing continuous models, where the problem is that synchronous adaptations cannot be anticipated when continuous models are designed but require crosscutting structural changes that are often contradicting between different synchronous designs.

## Prerequisites

The MCP is based on the paper [_Modelica language extensions for practical non-monotonic modelling: on the need for selective model extension_](https://modelica.org/events/modelica2019/proceedings/html/papers/Modelica2019paper3B1.pdf) published at the _13th International Modelica Conference 2019_ (cf. Section _References_). The motivation, concepts and examples presented in that paper are considered to be know.

## Rationale

### Syntax rules

Additional new syntax rules: introduce new top-level modification which is the only modification capable of performing deselections, i.e., inheritance modifications.

```
class-or-inheritance-modification :
      "(" [ argument-list ] ")"
    | "(" { inheritance-modification "," } inheritance-modification "," [ argument-list ] ")"

inheritance-modification :
      break connect-clause // Connection and…
    | break IDENT // …component deselection.
```

Changed existing syntax rules: use the new top-level modification at all places of the Modelica 3.4 grammar that are capable of non-nested modification applications.

```
long-class-specifier :
      IDENT string-comment composition end IDENT
    | extends IDENT [ class-or-inheritance-modification ] string-comment composition end IDENT

extends-clause :
    extends type-specifier [ class-or-inheritance-modification ] [annotation]
```

Inheritance modifications are not supported in `short-class-specifier` (which can be part of a `short-class-definition`) and `constraining-clause` because both can be within nested `class-modification`.

Open question:

```
annotation :
    annotation class-modification
```

Should it become:

```
annotation :
    annotation class-or-inheritance-modification
```

Seems to be a general issue: in how far is annotation syntax up-to-date w.r.t. other (recent) syntax fixes and changes that have been incorporated into the specification throughout the past?

### Semantic rules

**TODO**

## Backwards compatibility
The proposed language extensions do not introduce any new keywords. The new context-free derivations for `extends`-clause modifications to deselect connections and components - using the proposed `break`-based syntax - are syntax errors in current Modelica. As a consequence, selective model extension never changes the semantic of existing valid Modelica 3.4 models.

Models that are syntactically invalid could theoretically become valid however, but chances are extremely low. It is not likely that a syntax error happens to satisfy the proposed syntax. It is even less likely that the respective model will further satisfy the semantic constraints of selective model extension _and_ Modelica. Particularly, on the one hand deselections must actually deselect something whereas at the same time it is close to impossible that _just_ deselecting something results in a valid model (consider that deselected elements are likely used somehow and are therefore missing in the resulting model because of existing Modelica well-formedness constraints like _“referenced components must be declared”_ and _“the system of equations must be well-defined”_).

## Tool implementation

### _Dymola 2020_ prototype
A prototype of selective model extension has been developed by _Dassault Systèmes AB, Lund_. Selective extension is supported in _Dymola 2020_ in terms of tool vendor specific annotations for `extends`-clauses.

For example, to deselect the three connections `connect(voltageSensor.v, filter.u)`, `connect(speedSensor.w, setPointGain.u)` and `connect(voltageController.y, excitationVoltage.v)` of the `Modelica.Electrical.Machines.Examples.SynchronousInductionMachines.SMEE_Rectifier` SMEE rectifier example of the Modelica Standard Library one annotates the respective `extends`-clause as follows:

```
extends Modelica.Electrical.Machines.Examples.SynchronousInductionMachines.SMEE_Rectifier
    annotation(DiagramMap(__Dymola_deselect={
      "connect(voltageSensor.v, filter.u)",
      "connect(speedSensor.w, setPointGain.u)",
      "connect(voltageController.y, excitationVoltage.v)"}));
```

Likewise, components can be deselected via annotation. For example,

```
extends Modelica.Electrical.Machines.Examples.SynchronousInductionMachines.SMEE_Rectifier
    annotation(DiagramMap(__Dymola_deselect={
      "voltageController",
      "setPointGain"}));
```

deselects the `Modelica.Blocks.Continuous.LimPID voltageController` and `Modelica.Blocks.Math.Gain setPointGain` components and all connections they are part of. Of course, connector and component deselections can be mixed.

The _Dymola 2020_ prototype is not feature complete. Besides realizing deselections using annotations instead of the proposed syntax, it does not check all proposed semantic constraints, particularly that deselected elements indeed exist. The resulting diagrams are correctly rendered however, such that deselected elements are not shown. GUI support to deselect from within the diagram-layer is not supported yet.

Implementing the _Dymola 2020_ prototype took very little effort; less than 50 lines of code have been required.

### _Dymola 2020 FD01_ prototype

A next prototype implementation supporting proper syntax for deselections instead of tool-vendor specific annotations is planned. It is scheduled to be part of _Dymola 2020 FD01_.

### Examples library

A library with application examples is provided (`SelectiveExtension_13th_Modelica_Conference` library); different versions of it can be found in the [`examples`](examples) sub-directory. The contained examples are discussed in detail in the paper _Modelica language extensions for practical non-monotonic modelling: on the need for selective model extension_ (cf. Section _References_); the conclusions drawn in the paper hold for all library versions.

The available versions are:

1. **Version 0.1.0** for _Dymola 2020_, based on `Modelica` 3.2.2 and `Modelica_Synchronous` 0.92.1 and using tool-vendor specific annotations for deselection.
2. **Version 0.2.0** for _Dymola 2020_, based on `Modelica` 3.2.3 and `Modelica_Synchronous` 0.93.0 and using tool-vendor specific annotations for deselection (incorporates layout-adjustments for selective extensions of MSL base-classes whose layout changed from MSL 3.2.2 to 3.2.3, otherwise very same examples with the very same modeling as in version 0.1.0).

## Revisions

| Date        | Description                                                  |
| ----------- | ------------------------------------------------------------ |
| 2019-May-10 | Christoff Bürger. Initial version based on the discussion at the _98th Modelica Design Meeting_. |

## Contributor License Agreement

All authors of this MCP or their organizations have signed the "Modelica Contributor License Agreement".

## Required patents

To the best of our knowledge, there exist no conflicting or required patents to incorporate this MCP into the Modelica specification or implement it in Modelica tooling.

## References

The MCP is based on Modelica language extensions presented at the _13th International Modelica Conference 2019_:

> Christoff Bürger, [Modelica language extensions for practical non-monotonic modelling: on the need for selective model extension](https://modelica.org/events/modelica2019/proceedings/html/papers/Modelica2019paper3B1.pdf), 13th International Modelica Conference, March 4-6th 2019, Regensburg, Germany.

The provided `SelectiveExtension_13th_Modelica_Conference` library is documented in above publication.

The minutes of _Modelica Design Meetings_ discussing this MCP are summarized in [`minutes.md`](minutes.md).