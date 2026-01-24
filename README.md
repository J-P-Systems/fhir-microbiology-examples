A set of microbiology results illustrating composition patterns in FHIR.

* microExample_1.json: Flat. A DiagnosticReport with six Observations: one gram stain, one Organism Identification, and four drug susceptibilities.
* microExample_2.json: Members. A DiagnosticReport with eleven Observations: one gram stain, two Organism Identifications, and four Susceptibilities per Organism. The Organism Identification observation uses hasMember relationships to group the susceptibility tests. Note that the hasMember relationship does not conduct context: it is purely syntactic.
* microExample_3.json: Focus. Same as 2, but adding in the .focus property to make the association between Organism Identification and Susceptibility semantic.
* microExample_4.json: Isolate. Same as 2, but adding in child Specimen instances to make the association between Organism Identification and Susceptibility semantic.
* microExample_5.json: Groupers. Same as 2, but adding in imputed 'panel' classes to reiterate the non-semantic association already present in hasMember.

