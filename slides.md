---
theme: default
background: /pascal-van-de-vendel-RqjNWnQbWGU-unsplash.jpg
title: ClinPGx Variation Data Model
# apply UnoCSS classes to the current slide
class: text-center
# https://sli.dev/features/drawing
drawings:
  persist: false
transition: slide-left
mdc: true
---

# ClinPGx Variation Data Model


<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---
transition: slide-left
---

# ClinPGx Background

<img src="/clinpgx.svg" width="400"  alt="ClinPGx logo" style="float: right "/>

- Clinical Pharmacogenomics
- https://www.clinpgx.org (formerly pharmgkb.org)
- Notable projects:
    - [CPIC](https://cpicpgx.org)
    - [PharmCAT](https://pharmcat.clinpgx.org)
    - [PharmDOG](https://pharmdog.clinpgx.org)

---
transition: slide-left
---

# Technical Background

- PostgreSQL 15
- Java API (using Jersey & JAX-RS)
- Hibernate ORM
- Website: React SPA / webpack

---
transition: slide-left
---

# How we organize our data

<img src="/AccessionObject.png" width="300"  alt="Accession object structure" style="float: right "/>

Data structure is defined as Java classes organized in packages, interfaces, etc. 

__Accession Objects__ are entities, for example:
- Gene
- Drug
- Variant
- Haplotype
- Disease

__Annotations__ are links between entities, authored by curators, and linked to a publication/document
- Variant Annotations (VAs)
- Summary Annotations (SAs)

__Supporting Objects__ are things like cross-references, ontology terms, and some genomic location info

---
transition: slide-left
---

### Current Variation Data Model

```mermaid {scale: 0.45}
classDiagram
    AccessionObject <|-- Variant
    AccessionObject <|-- Haplotype
    AccessionObject: String accessionId
    AccessionObject: String name
    style AccessionObject fill:#f9f, stroke: #000
    Variant: SequenceLocation[] locations
    style Variant fill:#f9f, stroke: #000
    SequenceLocation: CrossReference sequence
    SequenceLocation: HgvsSequenceType type
    SequenceLocation: int begin
    SequenceLocation: int end
    SequenceLocation: String referenceAllele
    SequenceLocation: String[] variantAlleles
    SequenceLocation: Assembly assembly
    Variant "1" --> "0..*" SequenceLocation: locations
    Haplotype: HaplotypeAllele[] alleles
    Haplotype: AccessionObject gene
    Haplotype: String copyNumber
    Haplotype: Haplotype original
    Haplotype: String structuralVariation
    Haplotype: String hgvs
    Haplotype: String functionTerm
    Haplotype: String activityValue
    Haplotype: boolean reference
    style Haplotype fill:#f9f, stroke: #000
    Haplotype "1" --> "1..*" HaplotypeAllele: alleles
    HaplotypeAllele: VariantLocation location
    HaplotypeAllele: Variant variant
    HaplotypeAllele: String allele
    VariantLocation: VariantLocationType type
    VariantLocation: AccessionObject[] genes
    VariantLocation: boolean isTagGene
    VariantLocation: String copyNumber
    VariantLocation: AccessionObject[] haplotypes
    VariantLocation: Diplotype[] diplotypes
    VariantLocation: String rsid
    VariantLocation: String chromosomeName
    VariantLocation: String chromosomeId
    VariantLocation: int gpPosition
    VariantLocation: String gpBuildVersion
    VariantLocation: CrossReference refSeq
    VariantLocation: int refSeqPosition
    VariantLocation: OntologyTerm genePhenotype
    VariantLocation: Variant variant
    note for VariantLocation "this is what we attach to VA's and SA's"
```

---
transition: slide-left
---

### New Variation Data Model

```mermaid {scale: 0.38}
classDiagram
    AccessionObject <|-- Variation
    AccessionObject: String accessionId
    AccessionObject: String name
    class AccessionObject:::accId
    class Variation:::accId
    class UnlocatedAllele:::accId
    class DuplicationAllele:::accId
    class Diplotype:::accId
    class Haplotype:::accId
    class Allele:::accId
    classDef accId fill:#f9f;
    Variation <|-- Allele
    Variation: AmpTier tierStatus
    Variation: AccessionObject[] relatedGenes
    Variation: LinkOut[] linkOuts
    Variation: boolean important
    Allele: AlleleDefinition[] definitions
    Variation <|-- Haplotype
    Haplotype: Allele[] members
    Haplotype: String[] hgvs
    Haplotype: LinkOut definingSequence
    Haplotype: String structuralVariation
    Variation <|-- UnlocatedAllele
    Variation <|-- DuplicationAllele
    DuplicationAllele: Variation[] members
    DuplicationAllele: String comparator [">=", "<=", "x"]
    DuplicationAllele: String count [1..9, N]
    DuplicationAllele: boolean isNonIdentical() [derived from members]
    note for DuplicationAllele "can handle CNVs and\nnon-identical duplications"
    AlleleDefinition: String hgvs
    AlleleDefinition: String assembly
    AlleleDefinition: CrossReference sequence
    AlleleDefinition: String chromosome
    AlleleDefinition: int start
    AlleleDefinition: int end
    AlleleDefinition: String referenceAllele
    AlleleDefinition: String allele
    UnlocatedAllele: String description
    Allele "1" --> "1..*" AlleleDefinition : definitions
    Haplotype --> Allele : members
    Variation "0..*" --> "0..*" AlleleFunction: geneFunctions
    AlleleFunction: DataSource source
    AlleleFunction: OntologyTerm functionTerm
    AlleleFunction: String activityScore
    Variation <|-- Diplotype
    Diplotype: Haplotype allele1
    Diplotype: Haplotype allele2
    Diplotype: String toString()
```
