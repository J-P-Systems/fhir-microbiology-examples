A set of microbiology results illustrating composition patterns in FHIR.

## Examples

We believe our destination EHR is using pattern 7. We propose to adopt this pattern, but also to use the .focus element as in pattern 3 to make the semantics unambiguous for CDS and LLM consumption.

Question 1: is there a problem with our intent to add .focus to this pattern?

Question 2: for imputed classes:
    we can use the LOINC for the drug for all drugs found
    we can use the LOINC for bacteria identified by culture for all bacteriology results
    we need a more generic value for the top grouper; we propose SCT Bacterial Culture (104178000).

### Example 1: Flat (microExample_1_flat.json)
A DiagnosticReport with six Observations: one gram stain, one Organism Identification, and four drug susceptibilities. Although this pattern seems coherent, in order to support other cases with multiple organisms, one of the other patterns will be necessary.

*All observations directly in DiagnosticReport.result*
```
DiagnosticReport
├─ gram-stain-1 (Gram-negative rods)
├─ organism-1 (E. coli)
├─ susceptibility-ampicillin-1 (16 ug/mL, R)
├─ susceptibility-ciprofloxacin-1 (0.25 ug/mL, S)
├─ susceptibility-ceftriaxone-1 (0.5 ug/mL, S)
└─ susceptibility-gentamicin-1 (2 ug/mL, S)
```

---

### Example 2: Members (microExample_2_members.json)
A DiagnosticReport with eleven Observations: one gram stain, two Organism Identifications, and four Susceptibilities per Organism. The Organism Identification observation uses hasMember relationships to group the susceptibility tests. Note that the hasMember relationship does not conduct context: it is syntactic.

*Organisms in result; susceptibilities via hasMember only*
```
DiagnosticReport
├─ gram-stain-2 (mixed gram stain)
├─ organism-a (E. coli)
│   └─ hasMember:
│       ├─ organism-a-susceptibility-ampicillin (R)
│       ├─ organism-a-susceptibility-ciprofloxacin (S)
│       ├─ organism-a-susceptibility-ceftriaxone (S)
│       └─ organism-a-susceptibility-trimethoprim... (S)
└─ organism-b (S. pyogenes)
    └─ hasMember:
        ├─ organism-b-susceptibility-vancomycin (S)
        ├─ organism-b-susceptibility-ampicillin (S)
        ├─ organism-b-susceptibility-linezolid (S)
        └─ organism-b-susceptibility-daptomycin (S)
```

---

### Example 3: Focus (microExample_3_focus.json)
Same as 2, but adding in the .focus property to make the association between Organism Identification and Susceptibility semantic.

*Same as #2 + susceptibilities have focus back to organism*
```
DiagnosticReport
├─ gram-stain-2 (mixed gram stain)
├─ organism-2a (E. coli)                    ←─┐
│   └─ hasMember:                             │
│       ├─ org2a-susceptibility-ampicillin    │ focus
│       ├─ org2a-susceptibility-ciprofloxacin │
│       ├─ org2a-susceptibility-ceftriaxone   │
│       └─ org2a-susceptibility-trimethoprim..┘
└─ organism-2b (S. pyogenes)                ←─┐
    └─ hasMember:                             │
        ├─ org2b-susceptibility-vancomycin    │ focus
        ├─ org2b-susceptibility-ampicillin    │
        ├─ org2b-susceptibility-linezolid     │
        └─ org2b-susceptibility-daptomycin   ─┘
```

---

### Example 4: Isolate (microExample_4_isolate.json)
Same as 2, but adding in child Specimen instances to make the association between Organism Identification and Susceptibility semantic.

*Uses Specimen resources for isolates; observations link via specimen reference*
```
Specimen/wound-1
├─ Specimen/culture-isolate-1 (parent)
└─ Specimen/culture-isolate-2 (parent)

DiagnosticReport
├─ gram-stain-2 (mixed gram stain)
├─ organism-2a (E. coli)              ──── specimen → culture-isolate-1
│   └─ hasMember:
│       ├─ org2a-susceptibility-ampicillin     ─┐
│       ├─ org2a-susceptibility-ciprofloxacin   │ specimen → culture-isolate-1
│       ├─ org2a-susceptibility-ceftriaxone     │
│       └─ org2a-susceptibility-trimethoprim...─┘
└─ organism-2b (S. pyogenes)          ──── specimen → culture-isolate-2
    └─ hasMember:
        ├─ org2b-susceptibility-vancomycin     ─┐
        ├─ org2b-susceptibility-ampicillin      │ specimen → culture-isolate-2
        ├─ org2b-susceptibility-linezolid       │
        └─ org2b-susceptibility-daptomycin     ─┘
```

---

### Example 5: Groupers (microExample_5_groupers.json)
Same as 2, but adding in imputed 'panel' classes to assist in navigation, per [this pattern](https://confluence.hl7.org/spaces/OO/pages/161056620/Diagnostic+Module+-+R6+Updates?preview=%2F161056620%2F308973361%2FParent-Child+Structures+-+Microbiology.pptx). Like hasMember, this tactic does not conduct context and is non-semantic.

*Culture panel + susceptibility panel groupers*
```
DiagnosticReport
├─ gram-stain-2 (mixed gram stain)
└─ culture-panel (GROUPER - no value)
    ├─ organism-a (E. coli)
    │   └─ susceptibility-panel-a (GROUPER - no value)
    │       ├─ organism-a-susceptibility-ampicillin (R)
    │       ├─ organism-a-susceptibility-ciprofloxacin (S)
    │       ├─ organism-a-susceptibility-ceftriaxone (S)
    │       └─ organism-a-susceptibility-trimethoprim... (S)
    └─ organism-b (S. pyogenes)
        └─ susceptibility-panel-b (GROUPER - no value)
            ├─ organism-b-susceptibility-vancomycin (S)
            ├─ organism-b-susceptibility-ampicillin (S)
            ├─ organism-b-susceptibility-linezolid (S)
            └─ organism-b-susceptibility-daptomycin (S)
```

---

### Example 6: One Grouper (microExample_6_onegrouper.json)
Same as 5, but only using the imputed panel at the susceptibility layer.

*Susceptibility panel grouper only; organisms direct in result*
```
DiagnosticReport
├─ gram-stain-2 (mixed gram stain)
├─ organism-a (E. coli)
│   └─ susceptibility-panel-a (GROUPER - no value)
│       ├─ organism-a-susceptibility-ampicillin (R)
│       ├─ organism-a-susceptibility-ciprofloxacin (S)
│       ├─ organism-a-susceptibility-ceftriaxone (S)
│       └─ organism-a-susceptibility-trimethoprim... (S)
└─ organism-b (S. pyogenes)
    └─ susceptibility-panel-b (GROUPER - no value)
        ├─ organism-b-susceptibility-vancomycin (S)
        ├─ organism-b-susceptibility-ampicillin (S)
        ├─ organism-b-susceptibility-linezolid (S)
        └─ organism-b-susceptibility-daptomycin (S)
```

---

### Example 7: 4-Layer Groupers (microExample_7_4layergroupers.json)
Same as 5 but collapsing the groupers into four layers per [R4 example](https://hl7.org/fhir/R4/diagnosticreport-examples.html#10.3.7.1.1).

*Organism groupers + susceptibility panel groupers; identification as sibling*
```
DiagnosticReport
├─ gram-stain-2 (mixed gram stain)
├─ organism-a-grouper (GROUPER - no value)
│   ├─ organism-a-identification (E. coli)
│   └─ susceptibility-panel-a (GROUPER - no value)
│       ├─ organism-a-susceptibility-ampicillin (R)
│       ├─ organism-a-susceptibility-ciprofloxacin (S)
│       ├─ organism-a-susceptibility-ceftriaxone (S)
│       └─ organism-a-susceptibility-trimethoprim... (S)
└─ organism-b-grouper (GROUPER - no value)
    ├─ organism-b-identification (S. pyogenes)
    └─ susceptibility-panel-b (GROUPER - no value)
        ├─ organism-b-susceptibility-vancomycin (S)
        ├─ organism-b-susceptibility-ampicillin (S)
        ├─ organism-b-susceptibility-linezolid (S)
        └─ organism-b-susceptibility-daptomycin (S)
```

---

### Example 8: Isolate without hasMember (microExample_8_isolate_no_hasmember.json)
Same as 4, but without any hasMember relationships. The association between organism identification and susceptibilities is made purely through the shared isolate Specimen reference. DiagnosticReport.result contains a flat list of all observations; consumers group by specimen to reconstruct the hierarchy.

*Uses Specimen resources for isolates; no hasMember; flat result list*
```
Specimen/wound-1
├─ Specimen/isolate-1 (parent)
└─ Specimen/isolate-2 (parent)

DiagnosticReport
├─ gram-stain-8 (mixed gram stain)
├─ organism-8a (E. coli)                 ──── specimen → isolate-1
├─ org8a-susceptibility-ampicillin       ──── specimen → isolate-1
├─ org8a-susceptibility-ciprofloxacin    ──── specimen → isolate-1
├─ org8a-susceptibility-ceftriaxone      ──── specimen → isolate-1
├─ org8a-susceptibility-trimethoprim...  ──── specimen → isolate-1
├─ organism-8b (S. pyogenes)             ──── specimen → isolate-2
├─ org8b-susceptibility-vancomycin       ──── specimen → isolate-2
├─ org8b-susceptibility-ampicillin       ──── specimen → isolate-2
├─ org8b-susceptibility-linezolid        ──── specimen → isolate-2
└─ org8b-susceptibility-daptomycin       ──── specimen → isolate-2
```