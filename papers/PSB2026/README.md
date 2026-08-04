# HyperNiche

**Learning Spatially Localized Higher-Order Cellular Neighborhoods**

HyperNiche is a supervised hypergraph neural network for jointly learning cellular representations and spatially localized higher-order cellular neighborhoods from spatial transcriptomics data.

Unlike approaches that construct fixed hyperedges using spatial proximity or molecular similarity, HyperNiche learns soft hyperedge memberships during training. Each cell anchors a candidate hyperedge, and an asymmetric compatibility function determines which spatial neighbors participate in the resulting cellular neighborhood.

> **Manuscript:** [HyperNiche: Learning Spatially Localized Higher-Order Cellular Neighborhoods](./HyperNicheV1.pdf)

## Overview

Spatial transcriptomics preserves both gene-expression measurements and the physical organization of cells within tissue. Many existing approaches represent this organization using pairwise graphs or hypergraphs whose incidence structures are fixed before training.

HyperNiche instead learns the hypergraph structure jointly with the cellular representations. The framework:

- restricts candidate hyperedge members to spatially local neighborhoods;
- learns asymmetric anchor-member compatibility rather than requiring molecular similarity;
- assigns soft membership strengths to candidate members;
- learns adaptive hyperedge weights;
- propagates information through the inferred hypergraph; and
- uses structural regularization to promote selective and reproducible neighborhoods.

The learned hyperedges are interpreted as spatially localized cellular neighborhoods. Their structural and compositional properties provide evidence of biological plausibility, but not independent experimental validation of functional cellular niches.

## Model

For each tissue core, HyperNiche:

1. encodes cell-level gene-expression profiles;
2. constructs a spatial candidate neighborhood around every anchor cell;
3. calculates asymmetric compatibility between the anchor and each candidate member;
4. converts compatibility scores into soft hyperedge memberships;
5. estimates the importance of each hyperedge;
6. performs hypergraph message passing; and
7. jointly optimizes cell-type prediction and structural regularization objectives.

Each cell anchors one candidate hyperedge. Candidate membership is spatially constrained, but the anchor and its members are not required to have similar molecular representations or identical cell-type annotations.

## Study Cohorts

HyperNiche was evaluated using two 10x Genomics Xenium spatial transcriptomics cohorts from [GEO accession GSE308148](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE308148).

| Cohort | Patients | Tissue cores | Cells before QC | Cells after QC | Genes |
|---|---:|---:|---:|---:|---:|
| Non-small cell lung cancer | 5 | 24 | 56,107 | 53,473 | 536 |
| Breast cancer | 4 | 18 | 30,917 | 30,704 | 536 |
| **Total** | **9** | **42** | **87,024** | **84,177** | **536** |

Reference cell-type annotations were established independently of HyperNiche before model fitting. HyperNiche used these annotations in its supervised objective.

## Evaluation Design

Evaluation used leave-one-patient-out outer folds:

- one patient was held out for testing;
- one remaining patient was designated for validation;
- all other patients were used for model fitting;
- all tissue cores from the same patient remained in the same partition; and
- fold assignments were fixed across five initialization seeds: `0`, `7`, `42`, `123`, and `999`.

The validation patient was excluded from model fitting and used for checkpoint and prespecified hyperparameter selection. The outer test patient was used only for final evaluation.

This procedure is an outer leave-one-patient-out evaluation with a deterministic patient-level validation split, rather than exhaustive nested cross-validation.

## Comparison Methods

HyperNiche was compared with:

- [SpaGCN](https://github.com/jianhuupenn/SpaGCN)
- [GraphST](https://github.com/JinmiaoChenLab/GraphST)
- HyperGCN
- HyperSTAR

A static spatial \(k\)-nearest-neighbor hypergraph and a symmetric-similarity constructor were also evaluated as ablation controls.

HyperNiche used reference cell-type labels during representation learning, whereas the comparison methods used their native unsupervised objectives. Consequently, the common downstream clustering analysis evaluates annotation alignment but is not a supervision-matched comparison of unsupervised clustering ability.

## Main Results

Under a common \(K\)-means evaluation protocol applied to the learned embeddings, HyperNiche achieved the strongest agreement with the reference annotations among the evaluated methods.

| Cohort | ARI | NMI |
|---|---:|---:|
| Non-small cell lung cancer | **0.428** | **0.474** |
| Breast cancer | **0.382** | **0.449** |

These results indicate that HyperNiche's supervised embeddings were strongly aligned with the reference cell-type annotations. ARI and NMI do not independently establish that an inferred hyperedge represents a biologically functional niche.

## Learned Neighborhoods

Consensus neighborhoods were characterized using cross-seed reproducibility, membership size, cell-type-composition entropy, spatial radius, enrichment, compactness, and nearby-neighborhood overlap.

| Cohort | Membership Jaccard | Mean niche size | Normalized entropy | Mean spatial radius |
|---|---:|---:|---:|---:|
| Non-small cell lung cancer | 0.859 ± 0.127 | 9.79 ± 2.17 | 0.087 ± 0.050 | 18.50 ± 2.30 μm |
| Breast cancer | 0.792 ± 0.090 | 11.39 ± 1.65 | 0.119 ± 0.062 | 20.88 ± 4.08 μm |

The learned neighborhoods were reproducible across random initializations, spatially localized, and compositionally selective. Their low normalized entropy indicates that many neighborhoods were dominated by one annotated cell type rather than being uniformly heterogeneous.

Enrichment beyond matched spatial neighborhoods varied between cohorts. These findings therefore provide descriptive evidence of structural consistency and biological plausibility rather than independent functional validation.

## Ablation Findings

The ablation analysis distinguishes cell-type prediction from meaningful neighborhood formation.

Key findings include:

- Replacing learned memberships with static spatial \(k\)-nearest-neighbor hyperedges produced the largest predictive degradation.
- Symmetric similarity reduced annotation-recovery performance and produced less compositionally selective neighborhoods.
- Removing adaptive hyperedge weighting caused a smaller but consistent predictive decline.
- Removing the degeneracy penalty preserved classification performance but prevented reproducible consensus-neighborhood recovery.
- Removing hypergraph message passing preserved classification performance but produced complete, nonselective candidate neighborhoods.
- The explicit spatial-logit term contributed little beyond the spatial candidate-neighborhood constraint in the evaluated breast-cancer cohort.

These results suggest that predictive accuracy alone is insufficient for evaluating a neighborhood-learning method.

## Software Environment

The experiments reported in the paper used:

- Python 3.10
- PyTorch 2.6
- SciPy 1.15.3
- scikit-learn 1.2.2
- AnnData
- NVIDIA V100 Tensor Core GPUs

Complete dependency specifications and reproducibility instructions will be added with the public code release.

## Repository Status

This repository currently provides the full HyperNiche manuscript. The reproducibility release is being prepared and is expected to include:

- data-preparation specifications;
- patient and tissue-core mappings;
- fixed patient-level fold assignments;
- model configuration files;
- training and evaluation scripts;
- baseline and ablation configurations;
- initialization seeds;
- consensus-neighborhood analysis;
- scripts for reproducing the reported tables and figures; and
- software and dependency versions.

Data or annotations that cannot be redistributed will be accompanied by accession information or instructions for obtaining authorized access.

## Limitations

The current study has several important limitations:

- the two cohorts contain only nine patients in total;
- HyperNiche depends on reference cell-type supervision;
- the targeted Xenium panel captures only a subset of the transcriptome;
- independent functional niche labels were unavailable;
- enrichment against the stricter spatial null was limited in lung cancer and was not retained in breast cancer;
- one anchored candidate hyperedge per cell may yield overlapping or partially redundant neighborhoods; and
- the findings have not yet been externally validated against clinical outcomes, perturbation experiments, or independently defined functional microenvironments.

## Citation

If you use HyperNiche or build upon this work, please cite:

```bibtex
@misc{mahmud2026hyperniche,
  title        = {HyperNiche: Learning Spatially Localized Higher-Order Cellular Neighborhoods},
  author       = {Mahmud, Md Ishtyaq and
                  Venkata, Anish Turumala and
                  Chitrala, Kumaraswamy Naidu and
                  Banerjee, Tania},
  year         = {2026},
  note         = {Manuscript}
}
```

The citation will be updated when the final publication information becomes available.

## Authors

- Md Ishtyaq Mahmud — University of Houston
- Anish Turumala Venkata — University of Florida
- Kumaraswamy Naidu Chitrala — University of Houston
- Tania Banerjee — University of Houston

For questions about the project, please contact:

- Md Ishtyaq Mahmud: `mmahmud4@cougarnet.uh.edu`
- Tania Banerjee: `tbanerjee@uh.edu`

## Acknowledgments

This project was supported by the National Center for Advancing Translational Sciences, National Institutes of Health, through Grant Award Number **UM1TR004539**. The content is solely the responsibility of the authors and does not necessarily represent the official views of the NIH.

The authors thank Dr. Kunal Rai and his team at MD Anderson Cancer Center for discussions that helped shape this research, and Dr. Hyeongseon Jeon and his team at the University of Houston for feedback on the dataset.

## License

The manuscript is distributed under the [Creative Commons Attribution-NonCommercial 4.0 International License](https://creativecommons.org/licenses/by-nc/4.0/).

Licensing information for the software implementation will be provided with the code release.
