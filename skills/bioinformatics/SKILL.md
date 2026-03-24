---
name: bioinformatics
description: Bioinformatics guidelines for variant analysis, pathogenicity scoring, genomic standards, and pipeline tools. Focused on reliable sources for clinical and research bioinformatics workflows.
type: skill
---

# Bioinformatics Guidelines

> Skill for bioinformaticians covering variant scoring, genomic standards, annotation tools, and pipeline best practices.
> Invoke with `/bioinformatics` when working on genomic analysis, variant interpretation, or pipeline design.

---

## Section 1: Variant Pathogenicity Scoring

Scoring variant pathogenicity requires combining multiple in-silico predictors. No single tool is definitive — use ensemble approaches and always cross-reference with clinical databases.

### Core Scoring Tools

| Tool | Purpose | Source |
|------|---------|--------|
| SIFT | Predicts whether amino acid substitution affects protein function (score 0–1; <0.05 = damaging) | https://sift.bii.a-star.edu.sg/sift4g/ |
| REVEL | Ensemble score for missense variant pathogenicity (0–1; >0.5 = likely pathogenic) | https://sites.google.com/site/revelgenomics/ |
| MutationTaster | Predicts disease-causing potential of sequence alterations | https://www.mutationtaster.org/ |
| ANNOVAR | Functional annotation of genetic variants from sequencing data | https://annovar.openbioinformatics.org/en/latest/ |
| GATK Phred Scores | Quality scoring for variant calling confidence (Q30 = 99.9% accuracy) | https://gatk.broadinstitute.org/hc/en-us/articles/360035531872-Phred-scaled-quality-scores |

### Scoring Interpretation Guidelines

**SIFT**
- Score < 0.05 → **Damaging** (affects protein function)
- Score ≥ 0.05 → **Tolerated**
- Always check the median sequence conservation score alongside SIFT score

**REVEL**
- Score > 0.75 → Strong evidence of pathogenicity
- Score 0.5–0.75 → Moderate evidence
- Score < 0.15 → Likely benign
- Recommended threshold for clinical use: **> 0.5** (adjustable per study design)

**GATK Phred Quality Scores**
- GQ (Genotype Quality) ≥ 20 → Acceptable
- GQ ≥ 30 → High confidence
- DP (Depth) ≥ 10× → Minimum; ≥ 30× preferred for clinical
- Filter: `PASS` in FILTER column is mandatory before scoring

### Ensemble Scoring Workflow

```
1. Run GATK variant calling → apply VQSR or hard filters
2. Annotate with ANNOVAR (refGene, gnomAD, ClinVar, dbSNP)
3. Apply in-silico predictors: SIFT, REVEL, MutationTaster
4. Cross-reference with DECIPHER and ClinVar
5. Apply ACMG/AMP classification criteria (PS, PM, PP, BA, BS, BP categories)
6. Final classification: Pathogenic / Likely Pathogenic / VUS / Likely Benign / Benign
```

### ACMG/AMP Classification (Quick Reference)

Follow ACMG standards for variant classification:
- **PS** = Strong pathogenic evidence
- **PM** = Moderate pathogenic evidence
- **PP** = Supporting pathogenic evidence
- **BA** = Stand-alone benign evidence
- **BS** = Strong benign evidence
- **BP** = Supporting benign evidence

Full standards: https://www.acmg.net/ACMG/Medical-Genetics-Practice-Resources/Genetics_Lab_Standards/

### Key Databases for Cross-Referencing

- **ClinVar** — Clinical significance of variants: https://www.ncbi.nlm.nih.gov/clinvar/
- **DECIPHER** — Genomic variants in rare disease: https://www.deciphergenomics.org/
- **gnomAD** — Population allele frequencies (essential for filtering common variants)
- **NCBI GEO** — Gene expression datasets for functional context: https://www.ncbi.nlm.nih.gov/geo/

### Common Pitfalls

- Do NOT rely on a single predictor — SIFT and PolyPhen-2 agree only ~70% of the time
- Always filter by population frequency (gnomAD MAF < 0.01 for rare disease)
- VUS (Variant of Uncertain Significance) should NOT be reported as pathogenic without additional evidence
- Check strand orientation when interpreting ANNOVAR output for indels

---

> **More sections coming:** Genomics Data Sources (GEO/DECIPHER), Container & Workflow Management (Seqera), Clinical Reporting Standards (ACMG).

---

## Section 2: Pipeline Standards & Workflow Management

Reproducible, portable pipelines are the backbone of reliable bioinformatics. Follow community standards to ensure your workflows are shareable, scalable, and audit-ready.

### Core Frameworks & Standards Bodies

| Resource | Purpose | Source |
|----------|---------|--------|
| nf-core | Community-curated Nextflow pipelines for bioinformatics | https://nf-co.re/ |
| GA4GH | Global standards for genomic data sharing and interoperability | https://www.ga4gh.org/ |
| GATK Best Practices | Variant discovery pipeline guidelines from Broad Institute | https://gatk.broadinstitute.org/hc/en-us/categories/360002369672 |
| Seqera Containers | Reproducible, versioned containers for bioinformatics tools | https://seqera.io/containers/ |

### nf-core Pipeline Standards

**Before writing a pipeline, check nf-core first** — there may already be a maintained pipeline for your use case.

Key nf-core pipelines:
- `nf-core/sarek` — Germline/somatic variant calling (WGS/WES)
- `nf-core/rnaseq` — RNA-seq quantification and QC
- `nf-core/methylseq` — Bisulfite sequencing analysis
- `nf-core/chipseq` — ChIP-seq peak calling
- `nf-core/ampliseq` — 16S/ITS amplicon sequencing

**nf-core compliance checklist:**
- [ ] Use `nf-core/tools` to scaffold new pipelines (`nf-core create`)
- [ ] Pin all tool versions (no `latest` tags in production)
- [ ] Use `nf-core/configs` for institutional HPC profiles
- [ ] Include `--outdir`, `--input`, and `--genome` as standard params
- [ ] Pass `nf-core lint` before sharing

### GA4GH Standards for Data Interoperability

Apply GA4GH standards when sharing or receiving genomic data across institutions:

- **htslib/VCF spec** — Standard variant format; always validate with `bcftools stats`
- **SAM/BAM/CRAM** — Alignment formats; prefer CRAM for storage efficiency
- **GA4GH Passports** — Identity/access tokens for federated data access
- **DRS (Data Repository Service)** — Standard API for accessing genomic files across clouds
- **WES (Workflow Execution Service)** — Submit and monitor workflow runs via standard API

### GATK Best Practices Workflow

```
Germline Short Variant Discovery (WGS/WES):
1. FastQC / MultiQC → assess raw read quality
2. BWA-MEM2 → align to reference genome (GRCh38 preferred)
3. GATK MarkDuplicates → remove PCR duplicates
4. GATK BaseQualityScoreRecalibration (BQSR) → recalibrate base scores
5. GATK HaplotypeCaller → call variants per-sample (gVCF mode)
6. GATK GenomicsDBImport + GenotypeGVCFs → joint genotyping
7. VQSR (≥30 samples) or Hard Filters (<30 samples) → filter variants
8. Annotate → ANNOVAR / VEP / SnpEff
```

### Container & Reproducibility Standards

**Use Seqera Containers for tool versioning:**
- Every pipeline tool should have a pinned container image
- Prefer `docker://` or `singularity://` URIs in Nextflow configs
- Use `seqera.io/containers` to find pre-built, tested images

**Nextflow config best practices:**
```groovy
// Always pin process containers
process {
    withName: 'GATK_HAPLOTYPECALLER' {
        container = 'community.wave.seqera.io/library/gatk4:4.5.0.0'
    }
}

// Use profiles for portability
profiles {
    docker { docker.enabled = true }
    singularity { singularity.enabled = true }
    slurm { process.executor = 'slurm' }
}
```

### Common Pitfalls

- Never use `GRCh37` for new projects — use `GRCh38` (hg38) as the default reference
- Always run `FastQC` before AND after trimming — trimming can introduce bias
- Do not mix samples from different sequencing runs without batch correction
- VQSR requires ≥30 samples for reliable tranche calibration; use hard filters below that
- CRAM files require the reference genome for decoding — always store the reference alongside

### Quality Control Metrics (Minimum Thresholds)

| Metric | Minimum | Preferred |
|--------|---------|-----------|
| Mean coverage (WGS) | 30× | 50× |
| Mean coverage (WES) | 100× | 200× |
| % bases ≥ Q30 | 75% | 85% |
| Duplication rate | < 20% | < 10% |
| % on-target reads (WES) | > 70% | > 85% |
| Ti/Tv ratio (WGS) | 2.0–2.1 | 2.0–2.1 |
| Ti/Tv ratio (WES) | 2.6–3.5 | 2.8–3.3 |

---

## Section 3: Genomic Database Reference

A curated reference of essential databases for variant interpretation, population genetics, disease association, and functional genomics.

---

### Population Frequency Databases

Use population databases to filter common variants and assess allele frequencies in diverse cohorts. Variants with MAF > 1% in population databases are generally not considered causative for rare Mendelian disease.

| Database | Description | URL |
|----------|-------------|-----|
| **gnomAD** | Genome Aggregation Database — allele frequencies across >140,000 exomes and >60,000 genomes from diverse populations. Primary reference for population filtering. | https://gnomad.broadinstitute.org/ |
| **ExAC** | Exome Aggregation Consortium — predecessor to gnomAD; ~60,000 exomes. Use gnomAD v3+ for new analyses, ExAC for legacy compatibility. | http://exac.broadinstitute.org/ |
| **1000 Genomes Project** | Whole-genome sequencing of >2,500 individuals across 26 populations. Foundational reference for population structure and common variant cataloguing. | https://www.internationalgenome.org/data |
| **dbSNP** | NCBI's database of short genetic variants (SNPs and indels). Primary registry for variant identifiers (rs numbers). | https://www.ncbi.nlm.nih.gov/snp |
| **NHLBI-ESP** | NHLBI Exome Sequencing Project — allele frequencies from ~6,500 exomes; focused on heart, lung, and blood disease populations. | https://esp.gs.washington.edu/ |

**Filtering recommendation:** Apply gnomAD MAF < 0.01 (rare disease) or < 0.001 (ultra-rare / de novo) as the primary frequency filter before in-silico scoring.

---

### Disease & Clinical Variant Databases

| Database | Description | URL |
|----------|-------------|-----|
| **ClinVar** | NCBI's archive of human variants with clinical significance interpretations (Pathogenic, Likely Pathogenic, VUS, Benign). Always check submission review status. | https://www.ncbi.nlm.nih.gov/clinvar |
| **OMIM** | Online Mendelian Inheritance in Man — authoritative catalogue of genes and genetic phenotypes. Essential for gene–disease association lookup. | https://www.omim.org |
| **HPO** | Human Phenotype Ontology — standardized vocabulary for human disease phenotypes. Use for phenotype-driven variant filtering and patient matching. | https://hpo.jax.org/ |
| **HGMD** | Human Gene Mutation Database — comprehensive collection of published germline mutations in human disease genes. Requires license for full access. | http://www.hgmd.org |
| **Orphanet** | Rare disease encyclopedia with gene–disease associations, prevalence data, and clinical summaries. Valuable for ultra-rare disease context. | https://www.orpha.net/ |
| **HGVS** | Human Genome Variation Society — curators of variant nomenclature standards and links to locus-specific databases. | http://www.hgvs.org/content/databases-tools |
| **Gene4Denovo** | Database of de novo mutations associated with developmental disorders; useful for trio-based WES/WGS analysis. | http://genemed.tech/gene4denovo/home |

**Usage note:** Cross-reference ClinVar and HGMD for any variant classified as VUS before reporting. Conflicting interpretations between submitters require manual review.

---

### Sequence & Reference Genome Databases

| Database | Description | URL |
|----------|-------------|-----|
| **NCBI Genome** | Central repository for reference genome sequences, assemblies, and annotations across all organisms. | https://www.ncbi.nlm.nih.gov/genome |
| **RefSeqGene** | NCBI curated reference sequences for human gene regions — the standard for variant coordinates in clinical reporting. Always use RefSeq transcript IDs (NM_/NR_) in reports. | https://www.ncbi.nlm.nih.gov/refseq/rsg |
| **Ensembl** | Genome browser and annotation database for vertebrates and other eukaryotes. Provides GENCODE transcripts, regulatory features, and cross-species comparisons. | https://www.ensembl.org/index.html |

---

### Protein Sequence & Function Databases

| Database | Description | URL |
|----------|-------------|-----|
| **UniProt** | Comprehensive resource for protein sequence and functional annotation. Use to assess domain impact of missense variants and review protein-level evidence for pathogenicity. | https://www.uniprot.org/ |

---

### Missense Variant Effect Predictors

No single predictor is sufficient. Use multiple tools and look for concordance. Conflicting predictions should increase caution about classification.

| Tool | Description | URL |
|------|-------------|-----|
| **PolyPhen-2** | Predicts damaging effects of amino acid substitutions using sequence and structural features (score 0–1; >0.85 = probably damaging). | http://genetics.bwh.harvard.edu/pph2 |
| **SIFT** | Sequence-based prediction of amino acid substitution tolerance (score <0.05 = damaging). Based on evolutionary conservation. | http://sift.jcvi.org |
| **MutationTaster** | Predicts disease-causing potential using evolutionary conservation, splice-site changes, and protein features. | http://www.mutationtaster.org |
| **MutationAssessor** | Assesses functional impact of missense variants based on evolutionary conservation in protein families. | http://mutationassessor.org |
| **PROVEAN** | Predicts whether an amino acid substitution or indel affects protein function using sequence alignment scoring. | http://provean.jcvi.org/index.php |
| **InterVar** | Automated ACMG/AMP 2015 variant classification using multiple evidence sources. Outputs evidence codes (PS, PM, PP, BA, BS, BP) directly. | http://wintervar.wglab.org/ |
| **FATHMM** | Predicts functional consequences of missense variants using hidden Markov models; includes cancer-specific predictions. | http://fathmm.biocompute.org.uk/ |
| **CADD** | Combined Annotation Dependent Depletion — integrates >60 annotations into a single Phred-scaled score (C-score). CADD ≥ 20 = top 1% most deleterious variants. | https://cadd.gs.washington.edu/ |
| **dbNSFP** | Pre-computed functional predictions for all possible human SNVs across >30 tools. Use for batch annotation rather than individual queries. | https://sites.google.com/site/jpopgen/dbNSFP |

**Recommended ensemble approach:** Require ≥3 concordant damaging predictions before treating a missense variant as likely pathogenic.

---

### Splice Site Effect Predictors

Splice-disrupting variants are frequently pathogenic but often missed by standard missense tools. Always evaluate canonical splice sites (±1/2 bp) and cryptic splice sites.

| Tool | Description | URL |
|------|-------------|-----|
| **GeneSplicer** | Predicts splice sites using a combination of Markov models and information content; detects both donor and acceptor sites. | http://www.cbcb.umd.edu/software/GeneSplicer/ |
| **NetGene2** | Neural network-based splice site prediction for human, Arabidopsis, and C. elegans. | http://www.cbs.dtu.dk/services/NetGene2/ |
| **dbscSNV** | Database of splicing consensus SNVs with pre-computed scores for all SNVs within splicing consensus regions (±2 bp). | http://www.liulab.science/dbscsnv.html |
| **SpliceAI** | Deep learning tool for predicting splicing effects of variants up to 50 bp from splice junctions. Score ≥ 0.2 = likely splice-altering; ≥ 0.5 = high confidence. | https://spliceailookup.broadinstitute.org/ |
| **SpliceRegion** | VEP plugin that annotates variants in the broader splice region (±3–8 bp from exon boundary) beyond canonical ±1/2 positions. | Available as Ensembl VEP plugin |

**Clinical practice:** For variants within 10 bp of a splice site, always run SpliceAI and at least one additional predictor. A SpliceAI score ≥ 0.5 is considered strong functional evidence (PS3/PP3 level).

---

### Evolutionary Conservation Scores

Conservation scores reflect selective pressure — highly conserved positions tolerate fewer mutations. Use as supporting evidence, not standalone proof of pathogenicity.

| Tool | Description | URL |
|------|-------------|-----|
| **GERP** | Genomic Evolutionary Rate Profiling — estimates conservation based on maximum likelihood evolutionary rates. GERP RS > 2 indicates constrained positions. | http://mendel.stanford.edu/sidowlab/downloads/gerp/ |
| **PhyloP** | Measures conservation or acceleration at individual nucleotide positions using phylogenetic models. Positive scores = conserved; negative = accelerated evolution. | http://compgen.cshl.edu/phast/ |

---

### Repeat & Structural Variant Databases

| Tool | Description | URL |
|------|-------------|-----|
| **RepeatMasker** | Screens DNA sequences for interspersed repeats and low-complexity regions using Repbase repeat libraries. Essential for masking repetitive regions before alignment and annotation. | http://www.repeatmasker.org/ |

---

### Database Selection Guide

| Analysis Type | Primary Databases |
|--------------|-------------------|
| Rare disease variant filtering | gnomAD, ClinVar, OMIM, HPO |
| Missense pathogenicity | CADD, REVEL, SIFT, PolyPhen-2, InterVar |
| Splice variant assessment | SpliceAI, dbscSNV, NetGene2 |
| De novo variant analysis | gnomAD, Gene4Denovo, DECIPHER |
| Population genetics | gnomAD, 1000 Genomes, ExAC |
| Protein function | UniProt, MutationAssessor, PROVEAN |
| Reference coordinates | RefSeqGene, Ensembl, NCBI Genome |
