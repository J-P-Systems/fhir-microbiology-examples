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
├─ gram-stain (mixed gram stain)
├─ organism-1-identification (S. aureus)
│   └─ hasMember:
│       ├─ organism-1-susceptibility-oxacillin (S)
│       ├─ organism-1-susceptibility-erythromycin (S)
│       ├─ organism-1-susceptibility-clindamycin (S)
│       └─ organism-1-susceptibility-trimethoprim... (S)
└─ organism-2-identification (S. pyogenes)
    └─ hasMember:
        ├─ organism-2-susceptibility-vancomycin (S)
        ├─ organism-2-susceptibility-ampicillin (S)
        ├─ organism-2-susceptibility-linezolid (S)
        └─ organism-2-susceptibility-daptomycin (S)
```

---

### Example 3: Focus (microExample_3_focus.json)
Same as 2, but adding in the .focus property to make the association between Organism Identification and Susceptibility semantic.

*Same as #2 + susceptibilities have focus back to organism*
```
DiagnosticReport
├─ gram-stain (mixed gram stain)
├─ organism-1-identification (S. aureus)            ←─┐
│   └─ hasMember:                                      │
│       ├─ organism-1-susceptibility-oxacillin         │ focus
│       ├─ organism-1-susceptibility-erythromycin      │
│       ├─ organism-1-susceptibility-clindamycin       │
│       └─ organism-1-susceptibility-trimethoprim...  ─┘
└─ organism-2-identification (S. pyogenes)          ←─┐
    └─ hasMember:                                      │
        ├─ organism-2-susceptibility-vancomycin        │ focus
        ├─ organism-2-susceptibility-ampicillin        │
        ├─ organism-2-susceptibility-linezolid         │
        └─ organism-2-susceptibility-daptomycin       ─┘
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
├─ gram-stain (mixed gram stain)
├─ organism-1-identification (S. aureus)     ──── specimen → culture-isolate-1
│   └─ hasMember:
│       ├─ organism-1-susceptibility-ampicillin          ─┐
│       ├─ organism-1-susceptibility-ciprofloxacin        │ specimen → culture-isolate-1
│       ├─ organism-1-susceptibility-ceftriaxone          │
│       └─ organism-1-susceptibility-trimethoprim...     ─┘
└─ organism-2-identification (S. pyogenes)   ──── specimen → culture-isolate-2
    └─ hasMember:
        ├─ organism-2-susceptibility-vancomycin          ─┐
        ├─ organism-2-susceptibility-ampicillin           │ specimen → culture-isolate-2
        ├─ organism-2-susceptibility-linezolid            │
        └─ organism-2-susceptibility-daptomycin          ─┘
```

---

### Example 5: Groupers (microExample_5_groupers.json)
Same as 2, but adding in imputed 'panel' classes to assist in navigation, per [this pattern](https://confluence.hl7.org/spaces/OO/pages/161056620/Diagnostic+Module+-+R6+Updates?preview=%2F161056620%2F308973361%2FParent-Child+Structures+-+Microbiology.pptx). Like hasMember, this tactic does not conduct context and is non-semantic.

*Culture panel + susceptibility panel groupers*
```
DiagnosticReport
├─ gram-stain (mixed gram stain)
└─ culture-panel (GROUPER - no value)
    ├─ organism-1-identification (S. aureus)
    │   └─ organism-1-susceptibility-panel (GROUPER - no value)
    │       ├─ organism-1-susceptibility-ampicillin (R)
    │       ├─ organism-1-susceptibility-ciprofloxacin (S)
    │       ├─ organism-1-susceptibility-ceftriaxone (S)
    │       └─ organism-1-susceptibility-trimethoprim... (S)
    └─ organism-2-identification (S. pyogenes)
        └─ organism-2-susceptibility-panel (GROUPER - no value)
            ├─ organism-2-susceptibility-vancomycin (S)
            ├─ organism-2-susceptibility-ampicillin (S)
            ├─ organism-2-susceptibility-linezolid (S)
            └─ organism-2-susceptibility-daptomycin (S)
```

---

### Example 6: One Grouper (microExample_6_onegrouper.json)
Same as 5, but only using the imputed panel at the susceptibility layer.

*Susceptibility panel grouper only; organisms direct in result*
```
DiagnosticReport
├─ gram-stain (mixed gram stain)
├─ organism-1-identification (S. aureus)
│   └─ organism-1-susceptibility-panel (GROUPER - no value)
│       ├─ organism-1-susceptibility-ampicillin (R)
│       ├─ organism-1-susceptibility-ciprofloxacin (S)
│       ├─ organism-1-susceptibility-ceftriaxone (S)
│       └─ organism-1-susceptibility-trimethoprim... (S)
└─ organism-2-identification (S. pyogenes)
    └─ organism-2-susceptibility-panel (GROUPER - no value)
        ├─ organism-2-susceptibility-vancomycin (S)
        ├─ organism-2-susceptibility-ampicillin (S)
        ├─ organism-2-susceptibility-linezolid (S)
        └─ organism-2-susceptibility-daptomycin (S)
```

---

### Example 7: 4-Layer Groupers (microExample_7_4layergroupers.json)
Same as 5 but collapsing the groupers into four layers per [R4 example](https://hl7.org/fhir/R4/diagnosticreport-examples.html#10.3.7.1.1).

*Organism groupers + susceptibility panel groupers; identification as sibling*
```
DiagnosticReport
├─ gram-stain (mixed gram stain)
├─ organism-1-grouper (GROUPER - no value)
│   ├─ organism-1-identification (S. aureus)
│   └─ organism-1-susceptibility-panel (GROUPER - no value)
│       ├─ organism-1-susceptibility-ampicillin (R)
│       ├─ organism-1-susceptibility-ciprofloxacin (S)
│       ├─ organism-1-susceptibility-ceftriaxone (S)
│       └─ organism-1-susceptibility-trimethoprim... (S)
└─ organism-2-grouper (GROUPER - no value)
    ├─ organism-2-identification (S. pyogenes)
    └─ organism-2-susceptibility-panel (GROUPER - no value)
        ├─ organism-2-susceptibility-vancomycin (S)
        ├─ organism-2-susceptibility-ampicillin (S)
        ├─ organism-2-susceptibility-linezolid (S)
        └─ organism-2-susceptibility-daptomycin (S)
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
├─ gram-stain (mixed gram stain)                        ──── specimen → wound-1
├─ organism-1-identification (S. aureus)                ──── specimen → isolate-1
├─ organism-1-susceptibility-ampicillin                 ──── specimen → isolate-1
├─ organism-1-susceptibility-ciprofloxacin              ──── specimen → isolate-1
├─ organism-1-susceptibility-ceftriaxone                ──── specimen → isolate-1
├─ organism-1-susceptibility-trimethoprim...            ──── specimen → isolate-1
├─ organism-2-identification (S. pyogenes)              ──── specimen → isolate-2
├─ organism-2-susceptibility-vancomycin                 ──── specimen → isolate-2
├─ organism-2-susceptibility-ampicillin                 ──── specimen → isolate-2
├─ organism-2-susceptibility-linezolid                  ──── specimen → isolate-2
└─ organism-2-susceptibility-daptomycin                 ──── specimen → isolate-2
```

---

### Example 9: Flat with Optional Groupers (microExample_9_isolate_flat_plus.json)
Same as 8, but adding imputed grouper observations (culture-panel, organism groupers, susceptibility panels) that consumers can optionally use for navigation. All observations remain directly in DiagnosticReport.result (flat), with groupers providing hasMember links for hierarchical traversal.

*Flat result list + optional groupers with hasMember for hierarchy*
```
Specimen/wound-1
├─ Specimen/isolate-1 (parent)
└─ Specimen/isolate-2 (parent)

DiagnosticReport.result (all flat):
├─ gram-stain                                           ──── specimen → wound-1
├─ culture-panel (GROUPER)                              ──── hasMember → organism-1-grouper, organism-2-grouper
├─ organism-1-grouper (GROUPER)                         ──── hasMember → organism-1-identification, organism-1-susceptibility-panel
├─ organism-1-identification (S. aureus)                ──── specimen → isolate-1
├─ organism-1-susceptibility-panel (GROUPER)            ──── hasMember → susceptibility tests
├─ organism-1-susceptibility-ampicillin                 ──── specimen → isolate-1
├─ organism-1-susceptibility-ciprofloxacin              ──── specimen → isolate-1
├─ organism-1-susceptibility-ceftriaxone                ──── specimen → isolate-1
├─ organism-1-susceptibility-trimethoprim...            ──── specimen → isolate-1
├─ organism-2-grouper (GROUPER)                         ──── hasMember → organism-2-identification, organism-2-susceptibility-panel
├─ organism-2-identification (S. pyogenes)              ──── specimen → isolate-2
├─ organism-2-susceptibility-panel (GROUPER)            ──── hasMember → susceptibility tests
├─ organism-2-susceptibility-vancomycin                 ──── specimen → isolate-2
├─ organism-2-susceptibility-ampicillin                 ──── specimen → isolate-2
├─ organism-2-susceptibility-linezolid                  ──── specimen → isolate-2
└─ organism-2-susceptibility-daptomycin                 ──── specimen → isolate-2
```