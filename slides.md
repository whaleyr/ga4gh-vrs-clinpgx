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

Our main focus:  
__Annotation of genomic variant & drug relationships in literature, guidelines, drug labels, and other data sets__

---
transition: slide-left
---

# Technical Background

- PostgreSQL 15
- Custom Java Model & API (using Jersey & JAX-RS)
- Hibernate ORM
- Website: React SPA / webpack

## Our products

- Websites: ClinPGx, PharmDOG, CPIC
- API: ClinPGx, CPIC
- File Artifacts: ClinPGx TSV/JSON files, CPIC DB exports
- Software: PharmCAT

---
transition: slide-left
---

# How we organize ClinPGx data

<img src="/AccessionObject.png" width="300"  alt="Accession object structure" style="float: right "/>

Data structure is defined as Java classes organized in packages, interfaces, etc. 

__Accession Objects__ = major entities, for example:
- Gene / Drug / Disease
- Drug / Variant

__Annotations__ = links between entities, authored by curators, and linked to a publication/document
- Variant Annotations (VAs)
- Summary Annotations (SAs)
- Guideline Annotations

__Supporting Objects__ = cross-references, ontology terms, literature, etc...

---
transition: slide-left
---

### Current ClinPGx Data Model

```mermaid {scale: 0.45}
classDiagram
    direction LR
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
    Haplotype .. VariantLocation: haplotypes
    Variant .. VariantLocation: variant
```

---
transition: slide-left
---

# Why does this need to change?

1. No single AccessionObject for each allele/variation
2. Difficult to aggregate all annotations on the same allele
3. Multiple types of variation crammed into the same model
4. Benefit from better inheritance rules for balanced shared/specific properties
5. Easier documentation and dev experience

---
transition: slide-left
---

### New ClinPGx Data Model (inspired by [VRS 1.3](https://vrs.ga4gh.org/en/1.3/schema.html))

```mermaid {scale: 0.38}
classDiagram
    direction LR
    AccessionObject <|-- Variation
    AccessionObject: String accessionId
    AccessionObject: String name
    class AccessionObject:::accId
    class Variation:::accId
    class UnlocatedAllele:::accId
    class CopyNumberAllele:::accId
    class Genotype:::accId
    class Haplotype:::accId
    class Allele:::accId
    classDef accId fill:#f9f;
    Variation <|-- Allele
    Variation: AmpTier tierStatus
    Variation: AccessionObject[] relatedGenes
    Variation: LinkOut[] linkOuts
    Variation: boolean important
    Variation: GeneFunction[] geneFunctions
    Allele: AlleleDefinition[] definitions
    Variation <|-- Haplotype
    Haplotype: Allele[] members
    Haplotype: String[] hgvs
    Haplotype: LinkOut definingSequence
    Haplotype: String structuralVariation
    Variation <|-- UnlocatedAllele
    Variation <|-- CopyNumberAllele
    CopyNumberAllele: Variation[] members
    CopyNumberAllele: String comparator [">=", "<=", "x"]
    CopyNumberAllele: String count [1..9, N]
    CopyNumberAllele: boolean isNonIdentical() [derived from members]
    note for CopyNumberAllele "can handle CNVs and\nnon-identical duplications"
    AlleleDefinition: String hgvs
    AlleleDefinition: String assembly
    AlleleDefinition: CrossReference sequence
    AlleleDefinition: String chromosome
    AlleleDefinition: int start
    AlleleDefinition: int end
    AlleleDefinition: String referenceAllele
    AlleleDefinition: String allele
    UnlocatedAllele: String description
    Allele "1" .. "1..*" AlleleDefinition : definitions
    Haplotype "1..*" .. "1..*" Allele : members
    Variation "0..*" .. "0..*" AlleleFunction: geneFunctions
    AlleleFunction: OntologyTerm functionTerm
    AlleleFunction: String activityScore
    AlleleFunction: AccessionObject gene
    Variation <|-- Genotype
    Genotype: Variation allele1
    Genotype: Variation allele2
    Genotype: String toString()
```

---
transition: slide-left
layout: two-cols-header
---

# [Comparison]{style="text-decoration: underline"}

::left::

## Model Map

__ClinPGx >>> VRS 1.3__

- Variation >>> Variation
- Allele >>> Allele
- Genotype (or Diplotype) >>> [None]{style="color: red; font-style: italic"}
- CopyNumberAllele >>> CopyNumber
- Haplotype >>> Haplotype
- UnlocatedAllele >>> Text

::right::

## Friction

- Naming: Haplotypes - Cis-Phased Blocks
- Delineation of ref & alt alleles
- Multiple location classes
- Update to VRS 2.0 is not as straight-forward
