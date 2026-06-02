# Melanoma Brain Metastasis — Single-Cell Atlas
### Reproduction Project | Special Topics in Bioinformatics
### Member 1(Nawal Babar) — Paper Understanding & README:Carefully read the full paper including all supplemental methods and figures, then wrote the complete repository README covering the purpose, background, methodology, results, and figures summary in plain language
### Member 2 (Manal Tufail)— Tumor Cell Reproduction Notebooks:Reproduced Figures 1, 2, and 3 focusing on the cancer cell side of the atlas
### Member 3(Ashna Abrar) — Immune Cell Reproduction Notebooks:Reproduced Figures 4, 5, and 6 covering the tumor microenvironment 
### Member 4 (Washma Sajjad) — Final Presentation & Validation Notebook:Reproduced Figure 5I–K on the B cell to plasma cell differentiation trajectory (and supplemental Figure S7D–F on immunoglobulin chain sharing), and built the complete 18-slide presentation integrating findings from all members 
> **Original Paper:** Biermann, Melms et al., *Cell* 185, 2591–2608 (July 7, 2022)  
> **DOI:** https://doi.org/10.1016/j.cell.2022.06.007  
> **Original Repo:** https://github.com/IzarLab/Melanoma_Brain_Metastasis

---

## Table of Contents

1. [What This Paper Is About (In Plain Terms)](#1-what-this-paper-is-about-in-plain-terms)
2. [Why This Problem Matters](#2-why-this-problem-matters)
3. [What Was Already Known — Background](#3-what-was-already-known--background)
4. [What we Did — Study Design Overview](#4-what-they-did--study-design-overview)
5. [How we Did It — Methodology in Detail](#5-how-they-did-it--methodology-in-detail)
6. [What we Found — Key Results](#6-what-they-found--key-results)
7. [Figures Summary](#7-figures-summary)
8. [Reproduction Project Structure](#8-reproduction-project-structure)
9. [How to Reproduce (Quick Start)](#9-how-to-reproduce-quick-start)
10. [Data Access](#10-data-access)
11. [Dependencies](#11-dependencies)
12. [Team & Contributions](#12-team--contributions)

---

## 1. What This Paper Is About 

Imagine melanoma (skin cancer) that spreads to the brain. Once it gets there, it behaves very differently from melanoma that spreads to other parts of the body like the liver or lymph nodes — it's harder to treat, patients respond less well to therapies, and doctors don't fully understand why.

This paper tries to answer a simple but very hard question: **what is actually going on inside melanoma tumors that have spread to the brain, at the level of individual cells?**

The authors took tumor samples from 22 patients whose melanoma had spread to the brain (**MBM = Melanoma Brain Metastasis**) and 10 patients whose melanoma had spread elsewhere (**ECM = Extracranial Melanoma Metastasis**). They used cutting-edge single-cell sequencing to read the gene activity of over 114,000 individual cells, essentially taking a "snapshot" of every cell's identity and state inside these tumors.

The punchline of what they found:
- Brain metastasis cancer cells have more **chromosomal chaos** (their DNA is more scrambled)
- These cancer cells start to look and behave like **brain/nerve cells** — a kind of disguise that might help them survive in the brain
- The immune environment in brain tumors is more **immunosuppressive** — the immune cells meant to fight cancer are being blocked or exhausted
- There are more **pro-tumor macrophages** (a type of immune cell that ends up helping the tumor instead of killing it) in the brain

This work is essentially a comprehensive **biological atlas** — a detailed map of the cellular landscape of melanoma brain metastasis that didn't exist before.

---

## 2. Why This Problem Matters

Melanoma brain metastasis is the **third most common type of brain metastasis** after lung and breast cancer. It causes serious illness and death. Even when modern immunotherapy (drugs that activate the immune system to fight cancer) works well on melanoma elsewhere in the body, brain metastasis often continues to grow or develops separately.

Patients with brain metastasis have very limited treatment options, and the biological reasons why their tumors are different from extracranial tumors were largely a mystery before this study. Building this cellular map is the first step toward:
- Designing treatments specifically for brain metastasis
- Understanding why existing treatments fail
- Identifying new therapeutic targets unique to brain metastasis

---

## 3. What Was Already Known — Background

Before this paper, here's what the field understood:

**On Melanoma Biology:**
- Melanoma cells can exist in different "states" — a melanocyte lineage state (MITF-high, behaves more like a normal pigment cell) versus an invasive, drug-resistant state (AXL-high, behaves more like a mesenchymal cell). These states are plastic — cells can switch between them.
- Chromosomal instability (CIN), where chromosomes break apart and rearrange incorrectly, was already linked to cancer spread (metastasis) in general.

**On Brain Metastasis:**
- Genomic studies of other cancers (lung, breast) had found different mutations in brain versus non-brain metastases, but this hadn't been clearly shown in melanoma.
- Treatment with combination immune checkpoint blockade (nivolumab + ipilimumab) can work for some MBM patients, but many don't respond and it's unclear why.
- A few prior studies of brain metastasis existed, but they used fresh tissue (hard to collect, introduces processing artifacts), often included patients who had already received treatment (making it hard to see the underlying biology), and didn't do spatial transcriptomics.

**On Single-Cell Sequencing:**
- scRNA-seq had already been used to map metastatic melanoma in general (Tirosh et al., 2016 — a foundational paper in the field), identifying cell states linked to drug resistance.
- The technology had advanced to allow sequencing of nuclei from frozen tissue (snRNA-seq), which is far more practical for working with archived patient samples.

**The Gap:**
No study had done a comprehensive, treatment-naive, multi-modal single-cell analysis of **brain-specific** melanoma metastasis at scale, with spatial context and functional validation.

---

## 4. What They Did — Study Design Overview

Here is the overall architecture of the study (corresponding to **Figure 1A** in the paper):

```
Patient Samples (Treatment-Naive)
         |
         ├── 22 Melanoma Brain Metastases (MBM) — from 21 patients
         └── 10 Extracranial Melanoma Metastases (ECM)
                     |
         ┌───────────┼───────────────┐
         ▼           ▼               ▼
    5 samples    17 samples      11+5 tissue
    (fresh)      (frozen OCT)    sections
    scRNA-seq    snRNA-seq       SlideSeq-V2
    +TCR-seq                 (Spatial Transcriptomics)
         |           |               |
         └───────────┴───────────────┘
                     |
             114,455 cell transcriptomes
             7,575 T cell clonotypes
             16 spatial sections
                     |
         ┌───────────┼──────────────────┐
         ▼           ▼                  ▼
   Validation   Xenograft Models    Multiplexed IF
   (bulk RNA-seq, (5B1/4L + MBM03/    (42 additional
    proteomics,   MBM05 → mice)        patient samples)
    ATAC-seq)
```
![Figure 1F – Cell-type composition per patient](reproduction%20scripts/01_global_umap/output%20figures/Fig1F_composition-1.png)
**Key design choice:** All specimens were collected BEFORE any treatment. This is called "treatment-naive" and it's important because it means the results reflect the actual biology of the disease, not changes caused by prior therapy.

**Another key innovation:** They showed that frozen archival tissue (some samples >15 years old!) could be profiled with snRNA-seq and give results comparable to fresh tissue. This massively expands what historical tissue banks can be used for.

---

## 5. How They Did It — Methodology in Detail

### 5.1 Getting the Data Out of Tumors

**Fresh Tissue (scRNA-seq):**
Tumor was cut into ~0.5–1 mm³ pieces, digested with enzymes to break tissue apart into single cells, then sorted by FACS (fluorescence-activated cell sorting) into two buckets: immune cells (CD45+) and non-immune cells (CD45−). Both fractions were then loaded onto a 10x Genomics Chromium Controller for single-cell RNA sequencing.

**Frozen Tissue (snRNA-seq):**
Archival samples were cut from OCT-embedded frozen blocks, and nuclei were extracted (not whole cells) using a salt-Tween buffer with RNase inhibitor. This is gentler and avoids the "stress artifact" that happens when you try to dissociate fresh brain tissue — neurons and microglia are extremely sensitive to the dissociation process and turn on stress genes that contaminate fresh-tissue data.

**Spatial Transcriptomics (SlideSeqV2):**
This technology places tissue sections on special glass beads that have unique "barcodes." When you sequence the RNA, you get gene expression AND a map of where in the tissue each transcript came from (at ~10µm resolution). This was done on 16 tissue sections from 11 matched tumors.

### 5.2 Processing the Raw Sequencing Data

The raw data pipeline (detailed in STAR Methods, summarized in **Figure S1E**):

```
FASTQ files
    │
    ▼
Cell Ranger (alignment to GRCh38 human genome)
    │
    ▼
CellBender (remove ambient RNA / background noise)
    │
    ▼
Seurat v4 (quality control filtering)
    │   - Keep cells with 500–10,000 genes
    │   - Keep cells with 1,000–60,000 UMIs
    │   - <10% mitochondrial reads
    │   - Doublet removal (Scrublet + DoubletFinder)
    ▼
Integration across patients / methods
    │   - Canonical Correlation Analysis (CCA/Seurat)
    │     ← This was benchmarked against Harmony, Conos, STACAS
    │   - CCA won based on iLISI + cLISI scores
    ▼
UMAP dimensionality reduction (top 50 PCs)
    │
    ▼
Clustering + Cell Type Annotation
```

**Why benchmarking integration?** When you combine samples from different patients or different sequencing methods, batch effects can confuse the analysis. The team tested four different methods for correcting this and chose the best one based on quantitative metrics (LISI scores).

### 5.3 Identifying Cancer Cells vs. Normal Cells

This is a tricky problem. In a tumor, you have a mix of cancer cells and normal cells (immune cells, blood vessel cells, brain cells, etc.). How do you tell them apart just from RNA?

The answer is **copy number inference (inferCNV)**. Cancer cells have large-scale chromosomal deletions and amplifications (e.g., chromosome 7 is often amplified in melanoma). Normal cells don't. By looking at patterns of gene expression across chromosomes, you can infer which cells have an abnormal genome — those are the cancer cells.

Immune cells (identified by SingleR) were used as the diploid reference baseline.

### 5.4 Characterizing Cancer Cell Programs (NMF)

Once cancer cells were identified, the team used **KINOMO** — a semi-supervised non-negative matrix factorization (NMF) approach — to find programs of gene expression that are shared across patients.

Think of it like this: each tumor has a unique mix of cancer cell "modes." NMF finds those modes mathematically. KINOMO improves on standard NMF by adding biological prior knowledge as a regularization constraint, making it more robust to noise.

The process:
1. Run NMF on each patient's cancer cells individually → get "factors" (gene programs)
2. Cluster those factors by Spearman correlation across patients → get "metaprograms" (MPs)
3. Describe what each MP represents biologically

They found **7 metaprograms**, of which **MP7 was exclusively found in MBM** and contained genes related to neuronal development, differentiation, and synapse formation.

### 5.5 Characterizing the Immune Microenvironment

**Myeloid cells (macrophages, microglia, DCs, monocytes):**
- Diffusion component (DC) analysis was used to find continuous variation among macrophage populations
- RNA velocity (scVelo) was used to infer the temporal order of differentiation — which cell state comes first?
- Two main macrophage populations emerged: MDM-c1 (more pro-tumorigenic) and FTL+ MDM (higher antigen presentation, more anti-tumor features)

**T cells:**
- ProjecTILs (a reference-based projection method) was used to annotate T cell states consistently across scRNA-seq and snRNA-seq (which capture slightly different genes)
- TCR sequencing provided clonality information — which T cells are actually expanded (activated and proliferating)?

**B cells and plasma cells:**
- Immunoglobulin heavy and light chain co-expression was reconstructed from single-cell data to track B cell differentiation into plasma cells

### 5.6 Spatial Transcriptomics Analysis

SlideSeqV2 data was processed with:
- **RCTD (Robust Cell Type Decomposition):** Deconvolves each spatial "pixel" (bead) into a mixture of cell types using the snRNA-seq data as reference
- **Moran's I:** A spatial autocorrelation statistic to find genes whose expression is spatially clustered (not random)
- **CSIDE:** A cell-type-specific method to find spatially variable expression within specific cell types

The spatial data provided geographic context — where in the tumor are the plasma cell aggregates? Where is the oxidative phosphorylation program active? Are the antigen-presenting cancer cells adjacent to the T cells or far from them?

### 5.7 Validation Strategy

The authors went to great lengths to validate their computational findings experimentally:

| Finding | Validation Method |
|---|---|
| CIN in MBM | Micronuclei quantification by microscopy in isogenic cell lines |
| MBM-specific gene signature | Scoring on independent bulk RNA-seq cohort (Fischer et al., 2019, n=138) |
| MBM organotropism | Intracardiac injection of MBM/ECM cell lines into NSG mice |
| Neuronal-like state (NCAM1) | Multiplexed immunofluorescence on 42 independent patient samples |
| Pro-tumorigenic macrophages (CD163) | Protein quantification by IF in independent cohort |
| ATAC-seq | Chromatin accessibility profiling of 8 cell lines (4 MBM, 4 ECM) |
| Proteomics | Short-term culture proteomics (Kleffman et al., 2022) |

---

## 6. What They Found — Key Results

### 6.1 Frozen Archival Tissue Works Just as Well as Fresh Tissue

The very first thing they show (**Figure 1B–E**) is a technical validation: they split a single MBM tumor in half, processed one half fresh (scRNA-seq) and snap-froze the other for snRNA-seq 12 months later. The two methods gave comparable gene detection, similar cell type recovery, and the same copy number alterations — but the frozen/nuclei method had *lower* stress artifact expression. This is a big deal for the field because it means decades of archived tissue can now be studied.

### 6.2 Brain Metastases Have More Chromosomal Chaos (Figure 2)

Cancer cells in MBM showed significantly more genome alteration than ECM — more chromosomal amplifications and deletions. This was confirmed three ways:
- InferCNV from single-cell data
- Whole-exome sequencing from an external cohort (Fischer et al.)
- Micronuclei counting (a physical measure of CIN) in matched MBM/ECM cell lines
![Figure 2C – MBM vs MPM CNA burden](reproduction%20scripts/02_cna_analysis/output%20figures/Fig2B_CNA_violin-1.png)
![Figure 2D – Per-patient CNA burden](reproduction%20scripts/02_cna_analysis/output%20figures/Fig2D_patient_CNA-1.png)

The MBM cell line (5B1) also had higher migratory capacity in vitro and, when injected into mice, specifically spread to the brain — while the matched ECM cell line (4L) went to the liver instead. This shows that CIN isn't just a byproduct of being in the brain — it's associated with the very capacity to metastasize there.

### 6.3 Brain Metastasis Cancer Cells Look Like Neurons (Figure 3)

MBM cancer cells scored higher for the MITF-high program (more melanocyte-like) and lower for the AXL-high program, compared to ECM which had more of the invasive, mesenchymal character.

The differential gene expression analysis revealed that MBM-specific genes included things like **NCAM1, NELL1, NRG3, CADPS2, LRRC1** — these are all genes associated with neurons, not melanocytes. This was called the "neuronal-like" cell state.

The NMF analysis formalized this: **Metaprogram 7 (MP7)** was exclusive to MBM and contained neural differentiation genes like NGFR, NLGN3, NRXN, SNCA, SYT11, and GPHN.

This neuronal-like signature was:
- Validated in independent bulk RNA-seq (n=138 patients)
- Confirmed in xenograft RNA-seq
- Verified at the protein level: NCAM1 protein was significantly enriched in cancer cells in MBM vs. ECM by multiplexed immunofluorescence on 42 patient samples

Why does this matter? Melanoma comes from neural crest cells embryologically, so some neural gene expression is expected. But this goes beyond baseline — MBM cells seem to be actively taking on more neuron-like characteristics, possibly to better fit into and survive in the brain microenvironment.
![Figure 3C – Neuronal program key finding](reproduction%20scripts/03_cancer_cell_states/output%20figures/Fig3C_neuronal-1.png)
![Figure 3D – Neuronal genes MBM vs MPM](reproduction%20scripts/03_cancer_cell_states/output%20figures/Fig3D_neuronal_dot-1.png)

### 6.4 Macrophages in Brain Tumors Are Pro-Tumor (Figure 4)

MBM had a higher proportion of monocyte-derived macrophages (MDMs) compared to ECM, and these macrophages showed:
- **Higher CD163, MERTK, AXL, TLR2** — markers linked to a pro-tumorigenic, immunosuppressive state
- **Lower HLA genes (HLA-A/B/C/E, B2M, CD74)** — reduced antigen presentation, meaning they're less able to recruit killer T cells to the tumor

The microglia (brain-resident macrophages) also showed two states: an activated SPP1+ subpopulation (MG1) associated with pro-tumor behavior, and a quiescent MG2 state.

### 6.5 T Cells in Brain Tumors Are Dysfunctional But Different (Figure 5)

MBM had more **TOX+CD8+ T cells** (dysfunctional/exhausted T cells) than ECM. However, interestingly, these exhausted T cells expressed **lower levels of immune checkpoints** like PD-1 (PDCD1), TIM-3 (HAVCR2), LAG3, and TIGIT in MBM compared to ECM.

This seems counterintuitive — if the T cells are exhausted, shouldn't they have MORE checkpoint expression? One possible interpretation: the brain's immunosuppressive environment suppresses T cell function through mechanisms OTHER than classical checkpoint pathways, which has implications for how checkpoint blockade therapies might work differently in brain vs. non-brain tumors.

Clonotype analysis (TCR-seq) showed that the trajectory from progenitor-like (TCF7+) to terminally differentiated (TOX+) T cells is conserved in both MBM and ECM, but the brakes (checkpoints) applied along that journey differ.

### 6.6 B Cells Differentiate Into Plasma Cells INSIDE the Tumor (Figure 5 continued)

The full trajectory from naive B cells → activated B cells → plasma cells was reconstructed from single-cell data. Within individual specimens, cells at different stages of this trajectory shared the same immunoglobulin variable chain combinations — meaning they're clonally related and the differentiation is happening **inside the tumor**, not in lymph nodes elsewhere.

Spatial analysis and multiplexed IF confirmed that plasma cell aggregates were significantly more abundant and more clustered in MBM than ECM. These likely represent tertiary lymphoid structures (TLS), which in other cancers have been linked to better responses to immunotherapy — a potentially important finding for treatment design.
![Figure 7A – B/Plasma cell UMAP](reproduction%20scripts/07_bcell_plasma_differentiation/output%20figures/fig7A_umap.png)
![Figure 7B – Per-sample composition](reproduction%20scripts/07_bcell_plasma_differentiation/output%20figures/fig7B_composition.png)
![Figure 7D – Pseudotime trajectory](reproduction%20scripts/07_bcell_plasma_differentiation/output%20figures/fig7D_pseudotime.png)

### 6.7 Spatial Patterns Reveal Tumor Geography (Figure 6)

The spatial data painted a more nuanced picture:
- **Plasma cell clusters** are spatially concentrated in tight, discrete zones
- **Antigen presentation** is broadly expressed across the tumor, but **type-I interferon response** (which would indicate actual immune attack) is restricted to small areas
- **TIMP1** (a protein that may help tumors evade the immune system) is spatially anti-correlated with MHC-I (antigen presentation) genes — where tumors express TIMP1, they express less antigen-presentation machinery
- **Metabolic heterogeneity:** Some regions of the tumor use oxidative phosphorylation (OxPhos, using oxygen efficiently), while other regions use glycolysis — this metabolic geography is spatially structured

---

## 7. Figures Summary

> **Note on figure numbering:** This paper has **6 main figures** (Figures 1–6) and **7 supplemental figures** (S1–S7). Supplemental figures (prefix "S") are additional panels that support the main results but didn't fit in the main paper — they are referenced throughout the text as "Figure S1", "Figure S2", etc. There is **no Figure 7** in this paper; the B→plasma cell differentiation content is Figure 5I–K.

> **Note on reproduction:** Each team notebook reproduces the key panels of each figure. Because main figures each contain many sub-panels (e.g., Figure 3 has panels A–P), not every panel is reproduced — see Section 8 for the notebook-to-figure mapping.

| Figure | What It Shows | Key Message | Reproduced In |
|--------|---------------|-------------|---------------|
| **Figure 1** | Study design + fresh vs. frozen comparison | snRNA-seq from frozen tissue is as good as fresh; less stress artifact | `Fig1_global_UMAP_atlas.ipynb` |
| **Figure 2** | Global UMAP of all 114K cells + CNA analysis + in vivo metastasis | MBM has higher chromosomal instability; MBM cell lines home to brain in mice | `Fig2_CNA_instability.ipynb` |
| **Figure 3** | Cancer cell programs, NMF metaprograms, NCAM1 validation | Brain metastasis cancer cells adopt a neuronal-like state (MP7); confirmed at protein level | `Fig3_NMF_neuronal_program.ipynb` |
| **Figure 4** | Myeloid landscape — macrophages, microglia | MBM macrophages are more pro-tumorigenic; microglia have activated SPP1+ subpopulation | `Fig4_myeloid_macrophages.ipynb` |
| **Figure 5** | T/NK cells + B/plasma cells + TCR clonality | More exhausted T cells in MBM but with lower checkpoints; intra-tumoral B→plasma differentiation (panels I–K) | `Fig5_T_cell_exhaustion.ipynb` + `Fig5_Bcell_plasma.ipynb` |
| **Figure 6** | SlideSeqV2 spatial transcriptomics | Plasma cell clusters, spatially variable immune/metabolic programs, TIMP1 anti-correlation with MHC-I | `Fig6_spatial_transcriptomics.ipynb` |
| **Figure S1** | Workflow + QC + integration benchmarking | Technical supplement to Fig 1; CCA/Seurat integration method was best by iLISI/cLISI metrics | (Described in Fig1 notebook) |
| **Figure S2** | Full cellular landscape + organotropism details | Technical supplement to Fig 2; 26 distinct cell types; 5B1 vs 4L organotropism experiments | (Described in Fig2 notebook) |
| **Figure S3** | Cancer cell variability drivers + SCENIC/VIPER TFs | Technical supplement to Fig 3; MITF + RELB enriched in MBM; SOX4 + BACH1 in ECM | (Described in Fig3 notebook) |
| **Figure S4** | ATAC-seq, TOBIAS footprinting, MP7 pathway enrichment, NCAM1 gating | Technical supplement to Fig 3; chromatin accessibility confirms MBM vs ECM differences | (Described in Fig3 notebook) |
| **Figure S5** | Myeloid subtype detail | Technical supplement to Fig 4; MDM subtypes, microglia markers, RNA velocity of macrophage differentiation | (Described in Fig4 notebook) |
| **Figure S6** | T cell atlas (ProjecTILs) | Technical supplement to Fig 5; reference atlas validation; tech-independent annotation of T cell states | (Described in Fig5 T cell notebook) |
| **Figure S7** | TCR clonality + B/plasma IG chain details + SlideSeq cell type fractions | Technical supplement to Figs 5–6; intra-tumoral B-to-plasma differentiation with shared variable chains | (Described in Fig5 B cell notebook) |

---

## 8. Reproduction Project Structure

```
Human-Melanoma-Brain-Metastasis/
│
├── README.md
│
└── reproduction scripts/
    ├── 01_global_umap/
    │   ├── output figures/
    │   └── 01_Global_UMAP_Atlas.ipynb
    │
    ├── 02_cna_analysis/
    │   ├── output figures/
    │   └── 02_CNA_Instability.ipynb
    │
    ├── 03_cancer_cell_states/
    │   ├── output figures/
    │   └── 03_NMF_Cancer_States.ipynb
    │
    ├── 04_spatial_analysis/
    │   ├── output figures/
    │   └── Figure4_Spatial_Transcriptomics.ipynb
    │
    ├── 05__tcell_exhaustion/
    │   ├── output figures/
    │   └── Figure5_T_Cell_Exhaustion.ipynb
    │
    ├── 06__myeloid_analysis/
    │   ├── output figures/
    │   └── Figure6_Myeloid_Macrophages.ipynb
    │
    └── 07_bcell_plasma_differentiation/
        ├── output figures/
        └── 07_bcell_plasma_differentiation.ipynb
```

## 9. How to Reproduce 

### Step 1: Clone the Repository

```bash
git clone https://github.com/YOUR_USERNAME/Melanoma_Brain_Metastasis.git
cd Melanoma_Brain_Metastasis
```

### Step 2: Get the Data

The processed data is publicly available at:

- **GEO Accession:** GSE185386
  - sc/snRNA-seq count matrices
  - TCR-seq
  - SlideSeqV2 spatial data
  - RNA-seq from in vitro cell lines
  - ATAC-seq from cell lines

- **Raw data:** dbGaP accession phs002944.v1.p1 (requires application, contains human patient data)

Download processed data from GEO:
```bash
# Install SRA toolkit or use GEO website directly
# Processed .h5 files can be downloaded via:
# https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE185386
```

For reproduction notebooks, we use the processed count matrices (not raw FASTQ), so no alignment is needed.

### Step 3: Set Up Your Environment

**Option A — Google Colab (Recommended for Reproduction)**

Each notebook in `/notebooks/` is designed to run in Google Colab with GPU runtime. Open directly:
- Go to [colab.research.google.com](https://colab.research.google.com)
- File → Open Notebook → GitHub → paste this repo URL
- Select the notebook you want

**Option B — Local Environment**

```bash
# Python environment (for scanpy-based notebooks)
pip install scanpy anndata matplotlib seaborn pandas numpy

# R environment (for Seurat-based analysis)
# Install R 4.1.1+
install.packages("Seurat")
install.packages("ggplot2")
install.packages("dplyr")
BiocManager::install("inferCNV")
```

### Step 4: Run the Notebooks

Each notebook is self-contained and includes:
- Data loading instructions
- Step-by-step commentary explaining what each code block does and why
- Expected output descriptions
- Reproduced figures saved to `/figures/reproduced/`

---

## 10. Data Access

| Data Type | Location | Access |
|-----------|----------|--------|
| Processed scRNA-seq / snRNA-seq | GEO: GSE185386 | Public |
| TCR-seq | GEO: GSE185386 | Public |
| SlideSeqV2 spatial data | GEO: GSE185386 | Public |
| RNA-seq (cell lines) | GEO: GSE185386 | Public |
| ATAC-seq (cell lines) | GEO: GSE185386 | Public |
| Raw human data (FASTQ) | dbGaP: phs002944.v1.p1 | Controlled (apply) |
| Proteomics | MassIVE: MSV000088814 | Public |
| External bulk RNA-seq validation | Fischer et al., 2019 | Public (see DOI in paper) |
| Original analysis code | github.com/IzarLab/Melanoma_Brain_Metastasis | Public |

---

## 11. Dependencies

### Python Packages (for Scanpy/AnnData-based notebooks)

| Package | Version Used in Paper | Purpose |
|---------|----------------------|---------|
| scanpy | 1.8.2 | Single-cell analysis |
| anndata | — | Data structure |
| scvelo | 0.2.4 | RNA velocity |
| pyscenic | 0.11.2 | TF regulatory network |
| squidpy | 1.1.2 | Spatial analysis |
| matplotlib | — | Plotting |
| seaborn | — | Plotting |
| pandas | — | Data manipulation |

### R Packages (for Seurat-based notebooks)

| Package | Version | Purpose |
|---------|---------|---------|
| Seurat | 4.1.0 | Core single-cell analysis |
| inferCNV | 1.10.1 | Copy number inference |
| harmony | 0.1.0 | Integration (benchmarked) |
| destiny | 3.9.0 | Diffusion maps |
| DESeq2 | 1.34.0 | Differential expression |
| hypeR | 1.10.0 | Gene set enrichment |
| ProjecTILs | 1.0 | T cell annotation |
| spacexr (RCTD) | 1.2.0 | Spatial deconvolution |

---

## 12. Team & Contributions

| Member | Role | Deliverables |
|--------|------|--------------|
| **Member 1 (Nawal Babar)** | Paper Understanding & README | This README; Introduction & Background presentation slides |
| Member 2 (Manal Tufail) | Tumor Cell Notebooks | Figure 1 (UMAP atlas + QC), Figure 2 (CNA + organotropism), Figure 3 (NMF + neuronal program) |
| Member 3 (Ashna Abrar) | Immune Cell Notebooks | Figure 4 (Myeloid/macrophages), Figure 5 (T cells + TCR), Figure 6 (Spatial transcriptomics) |
| Member 4 (Washma Sajjad) | Final Presentation & Notebook | Figure 5I–K (B→Plasma differentiation, supplemented by S7D–F), full slide deck (18 slides) |

---

## Concepts Glossary 

| Term | Plain-English Meaning |
|------|-----------------------|
| **scRNA-seq** | Single-cell RNA sequencing — reading which genes are active in each individual cell |
| **snRNA-seq** | Single-nucleus RNA sequencing — same idea but from the nucleus of frozen cells |
| **UMAP** | A way to compress thousands of gene measurements per cell into a 2D dot plot, where similar cells cluster together |
| **CNA** | Copy Number Alteration — chunks of DNA being duplicated or deleted |
| **CIN** | Chromosomal Instability — the ongoing, dynamic process of chromosomes breaking and rearranging |
| **MBM** | Melanoma Brain Metastasis |
| **ECM** | Extracranial Melanoma Metastasis |
| **NMF / Metaprogram** | A math method that finds recurring gene-activity patterns shared across patients |
| **MDM** | Monocyte-Derived Macrophage — immune cells that came from blood monocytes and entered the tumor |
| **TOX+CD8+ T cell** | An exhausted killer T cell that has stopped working effectively |
| **TCF7+CD8+ T cell** | A progenitor-like T cell that still has potential to become active |
| **TLS** | Tertiary Lymphoid Structure — immune cell aggregates that form inside tumors, associated with better immunotherapy response |
| **SlideSeqV2** | A spatial transcriptomics technology that tells you BOTH what genes are active AND where in the tissue |
| **RCTD** | Robust Cell Type Decomposition — software that figures out which cell types are present at each spatial location |
| **RNA Velocity** | A method to predict which direction cells are differentiating, based on ratios of mature to immature RNA |
| **VIPER / SCENIC** | Methods that infer which master regulator proteins (transcription factors) are driving gene expression changes |
| **Moran's I** | A spatial statistics measure — tells you if a gene's expression is spatially clustered or random |

---



*Last updated: May 2026*

