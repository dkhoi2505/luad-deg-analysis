# Differential Expression Analysis of Lung Adenocarcinoma (GSE10072)

A reproducible, from-scratch differential expression (DE) analysis comparing lung adenocarcinoma (LUAD) tumor tissue with normal lung tissue, using the public microarray dataset **GSE10072**. The pipeline independently reproduces the headline cell-cycle gene signature reported in the original study, and recovers a coherent biological picture of tumor dedifferentiation and proliferation.

## Question

Which genes are differentially expressed between lung adenocarcinoma tumor tissue and adjacent normal lung tissue?

## Data

- **Source:** NCBI GEO, accession [GSE10072](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE10072)
- **Platform:** Affymetrix HG-U133A (GPL96), 22,283 probes
- **Samples:** 107 (58 tumor, 49 normal)
- Expression values are log2-transformed and normalized, as provided by GEO.

Raw data files are **not** committed to this repository (they are large and publicly available). See [`data/README.md`](data/README.md) for download instructions.

## Methods

1. **Load & inspect** the GEO series matrix (22,283 probes x 107 samples).
2. **Assign groups** by parsing sample titles ("Lung Tumor" vs "Normal Lung") -> 58 tumor, 49 normal.
3. **Per-probe statistics:** independent two-sample t-test (tumor vs normal) for every probe. Log2 fold change is computed as the difference of group means (valid because the data is already on a log2 scale).
4. **Multiple-testing correction:** Benjamini-Hochberg FDR (`p_adj`).
5. **Probe -> gene mapping** using the GPL96 annotation table.
6. **Collapse to gene level:** for genes measured by multiple probes, keep the probe with the highest mean expression - a criterion independent of the test result, to avoid selection bias.
7. **Filter significant genes:** `FDR < 0.001` and `|log2FC| > 0.585` (fold change > 1.5), matching the thresholds used in the original paper.
8. **Visualize** with a labelled volcano plot.

## Results

**1,464 differentially expressed genes** were identified (625 up-regulated, 839 down-regulated in tumor).

### Down-regulated genes: loss of lung identity and metabolic capacity

**Alveolar differentiation markers.** The most strongly down-regulated genes include markers of the two principal alveolar epithelial cell types: *SFTPC* (alveolar type II cells) [1] and *AGER* (alveolar type I cells) [2]. Their concurrent loss is consistent with tumor tissue losing its differentiated alveolar character. Beyond acting as a differentiation marker, *SFTPC* has itself been reported to suppress epithelial-to-mesenchymal transition (EMT) via the SOX7-WNT/beta-catenin axis in NSCLC [1], suggesting its loss may be functionally relevant to tumor progression.

**Metabolic genes.** Several down-regulated genes are metabolic in nature - *FABP4* (lipid metabolism) and *ADH1B* (alcohol metabolism). Their reduction may point to diminished tissue metabolic capacity, though the present analysis provides no basis for stronger conclusions.

### Up-regulated genes: proliferation, invasion, and the tumor microenvironment

**Proliferation.** *TOP2A* is up-regulated, alongside the mitotic cell-cycle genes discussed below (*NEK2*, *TTK*, *PRC1*), together pointing to an active proliferative program.

**Invasion and matrix remodeling.** *MMP1*, the most highly expressed interstitial collagenase (degrading fibrillar collagen types I, II, III), is up-regulated; its overexpression in tumor tissue is associated with invasion and metastasis [3]. Fibrillar collagens (*COL11A1*, *COL10A1*, *COL1A1*) are also up-regulated, consistent with a desmoplastic stromal reaction.

**Tumor microenvironment (more nuanced).** *SPP1* (osteopontin) shows the largest increase (~20x). It is linked to invasion and to the tumor immune microenvironment, and is a marker of pro-tumor tumor-associated macrophages (TAMs) in LUAD [4]. *MMP12* (macrophage metalloelastase) is also up-regulated; its role in cancer is context-dependent, with both pro- and anti-tumor effects reported depending on cellular source, and it is predominantly macrophage-derived [5].

*Notably*, both *SPP1* and *COL11A1* are among the most strongly up-regulated genes. One study has reported that SPP1 promotes migration and invasion by targeting COL11A1 in LUAD [6]. Their co-upregulation here is **logically consistent** with that reported relationship, although a differential-expression analysis alone **cannot demonstrate** a direct regulatory interaction between them.

> **Observation (personal interpretation, not yet independently validated):** Several of the up-regulated genes - *SPP1*, *MMP12*, and in part *MMP1* - are known to be macrophage-derived. This raises the possibility that part of the increased bulk signal reflects macrophage infiltration into the tumor rather than changes within tumor cells alone. This is a hypothesis based on the known cellular sources of these genes, not a conclusion drawn directly from this analysis, and it would require single-cell or deconvolution methods to test.

### Independent reproduction of the original finding

The mitotic cell-cycle genes *NEK2*, *TTK*, and *PRC1* - the headline signature of the original study - were all recovered as significantly up-regulated with very small adjusted p-values. This serves as a positive control confirming the pipeline works.

![Volcano plot](figures/volcano_gene_level.png)

## How to reproduce

```bash
# 1. Create the environment
conda env create -f environment.yml
conda activate pybio

# 2. Download the data (see data/README.md) into data/

# 3. Run the notebook
jupyter lab notebooks/luad_analysis.ipynb
```

Then run **Kernel -> Restart Kernel and Run All Cells** to reproduce every result from scratch.

## Limitations

This project is a methods demonstration and reproduction, **not** a claim of novel biomarker discovery. Several simplifications are worth stating explicitly:

- **No confounder adjustment.** A simple t-test was used, whereas the original study used ANOVA adjusting for confounders (age, sex, smoking status). Because tumor/normal status is entangled with these variables, the unadjusted test likely inflates the number of significant genes.
- **Paired design not exploited.** Tumor and normal samples were taken from the same patients, but an independent (unpaired) t-test was used, which is less statistically powerful than a paired approach.
- **Cell-composition confounding.** Strong down-regulation of lung-specific genes (e.g. *SFTPC*, *AGER*) may partly reflect differences in cell-type composition between tumor and normal tissue (fewer alveolar cells in tumor), rather than per-cell transcriptional changes. The same caveat applies to the macrophage-derived up-regulated genes noted above. Bulk expression data cannot distinguish these possibilities.
- **One probe per gene.** For multi-probe genes, only the highest-expressed probe was retained; multi-gene probes were dropped.
- **Statistical vs biological significance.** With n = 107, many very small differences reach statistical significance; effect-size filtering (fold change > 1.5), not p-value alone, does the real work of selecting meaningful genes.

## Future work

- **Confounder-adjusted model (R):** fit a multiple linear regression per gene (`expression ~ tumor + smoking + age + sex`) to isolate the tumor effect from clinical confounders - directly addressing the first limitation above.
- **Diagnostic modelling (R):** logistic regression with ROC/AUC to assess how well candidate genes discriminate tumor from normal.
- **Survival analysis (R):** Kaplan-Meier and Cox proportional-hazards models, if clinical outcome data can be linked.

These extensions will be added to this repository as a second analysis stage.

## Repository structure

```
├── notebooks/luad_analysis.ipynb   # full Python analysis
├── figures/                        # volcano plot (PNG + PDF)
├── results/                        # DEG tables (CSV)
├── data/README.md                  # data download instructions
├── environment.yml                 # conda environment
└── .gitignore
```

## References

1. Zhang Q, An N, Liu Y, et al. Alveolar type 2 cells marker gene SFTPC inhibits epithelial-to-mesenchymal transition by upregulating SOX7 and suppressing WNT/beta-catenin pathway in non-small cell lung cancer. *Front Oncol*. 2024;14:1448379.
2. AGER (RAGE) is an established marker of alveolar type I cells; its expression is restricted to type I pneumocytes by immunohistochemistry (see e.g. Shirasawa M, et al. Receptor for advanced glycation end-products is a marker of type I lung alveolar cells. *Genes Cells*. 2004;9(2):165-174).
3. Sauter W, Rosenberger A, Beckmann L, et al. Matrix metalloproteinase 1 (MMP1) is associated with early-onset lung cancer. *Cancer Epidemiol Biomarkers Prev*. 2008;17(5):1127-1135.
4. Matsubara E, Komohara Y, Esumi S, et al. SPP1 derived from macrophages is associated with a worse clinical course and chemo-resistance in lung adenocarcinoma. *Cancers (Basel)*. 2022;14(18):4374. doi:10.3390/cancers14184374.
5. Lv FZ, Wang JL, Wu Y, Chen HF, Shen XY. Knockdown of MMP12 inhibits the growth and invasion of lung adenocarcinoma cells. *Int J Immunopathol Pharmacol*. 2015;28(1):77-84. doi:10.1177/0394632015572557. (See also reviews on the context-dependent, macrophage-derived roles of MMP12 in cancer.)
6. Yi X, Luo L, Zhu Y, et al. SPP1 facilitates cell migration and invasion by targeting COL11A1 in lung adenocarcinoma. *Cancer Cell Int*. 2022;22:324. doi:10.1186/s12935-022-02749-x.

## Acknowledgements

Data from Landi MT, Dracheva T, Rotunno M, et al. *Gene expression signature of cigarette smoking and its role in lung adenocarcinoma development and survival.* PLoS ONE. 2008;3(2):e1651. doi:10.1371/journal.pone.0001651 (GEO: GSE10072).

This repository is an independent reanalysis carried out for learning purposes.