### Introduction

This implementation guide presents a set of microbiology results illustrating composition patterns in FHIR. The examples demonstrate various approaches to associating organism identifications with their susceptibility results in DiagnosticReport resources.

### Background

We believe our destination EHR is using pattern 7. We propose to adopt this pattern, but also to use the `.focus` element as in pattern 3 to make the semantics unambiguous for CDS and LLM consumption.

### Open Questions

**Question 1:** Is there a problem with our intent to add `.focus` to this pattern?

**Question 2:** For imputed classes:
- We can use the LOINC for the drug for all drugs found
- We can use the LOINC for bacteria identified by culture for all bacteriology results
- We need a more generic value for the top grouper; we propose SCT Bacterial Culture (104178000)

### Pattern Summary

| Pattern | File | Key Mechanism | Semantic? |
|---------|------|---------------|-----------|
| [1. Flat](patterns.html#example-1-flat) | microExample_1_flat.json | All observations in result | No grouping |
| [2. Members](patterns.html#example-2-members) | microExample_2_members.json | hasMember only | No |
| [3. Focus](patterns.html#example-3-focus) | microExample_3_focus.json | hasMember + focus | Yes |
| [4. Isolate](patterns.html#example-4-isolate) | microExample_4_isolate.json | Specimen + hasMember | Yes |
| [5. Groupers](patterns.html#example-5-groupers) | microExample_5_groupers.json | Panel grouper observations | No |
| [6. One Grouper](patterns.html#example-6-one-grouper) | microExample_6_onegrouper.json | Susceptibility panel only | No |
| [7. 4-Layer Groupers](patterns.html#example-7-4-layer-groupers) | microExample_7_4layergroupers.json | Organism + susceptibility groupers | No |
| [8. Isolate (no hasMember)](patterns.html#example-8-isolate-without-hasmember) | microExample_8_isolate_no_hasmember.json | Specimen only, flat result | Yes |

See the [Composition Patterns](patterns.html) page for detailed diagrams and explanations.
