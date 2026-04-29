# Master's Bioinformatics – CLP Oral Microbiome Computational Analysis

**Author:** Puja Sanjay Thorat  
**Date:** 15 April 2026  

---

## Languages, Tools, and Packages

**Language**

- Python (v3.11)

**Packages**

- pandas (v3.0.2)
- numpy (v2.4.4)
- matplotlib (v3.10.8)
- seaborn (v0.13.2)
- scipy (v1.17.1)
- scikit-bio (v0.7.2)
- scikit-learn (v1.8.0)
- biom-format (v2.1.17)

**Tools**

- Google Colab (cloud-based Jupyter environment)
- Git / GitHub (version control)

---

## Program Overview

The program addresses the question of whether disrupted neonatal oral microbiome maturation in Cleft Lip and Palate (CLP) constitutes an early developmental precursor to functional dysbiosis in older CLP patients. It accomplishes this by reanalysing a published 16S rRNA OTU dataset (Seidel et al., PRJNA881235; 152 OTUs, 137 samples) alongside contextual metatranscriptomic data from Funahashi et al., computing diversity metrics, a pathogenic burden index, and community-level statistics to test for group differences across two timepoints (T0 ≈ 3 days; T1 ≈ 32 days postnatal).

The workflow consists of five sequential analytical stages. First, raw data ingestion: the OTU count table (tab-separated, SILVA 138.1 taxonomy) and representative FASTA sequences are uploaded, auto-detected by file format, and parsed into structured DataFrames with taxonomy split into Kingdom–Genus levels. Second, alpha diversity: Simpson Diversity Index (D = 1 − Σpᵢ²) and Shannon entropy are computed per sample; Mann-Whitney U tests compare CLP vs Control at each timepoint, and Wilcoxon signed-rank tests assess T0→T1 change within each group. Third, beta diversity: Bray-Curtis dissimilarity matrices are computed pairwise across all samples, visualised via Principal Coordinates Analysis (PCoA through classical MDS), and tested with PERMANOVA (999 permutations, scikit-bio). Fourth, taxonomic visualisation: a heatmap of log₁₀-transformed relative abundances for the top 30 most abundant genera is produced with annotated group and timepoint colour bars. Fifth, pathogenic burden and correlation: 46 of 152 OTUs are classified as pathogenic based on Funahashi et al. differential abundance findings; burden is the summed relative abundance of those OTUs per sample; Pearson correlation tests the relationship between Simpson diversity and pathogenic burden at T1.

Additional considerations include the following. Two separate scripts are provided: `htp_project_For_AI_generated_data.py` was developed and validated against a simulated Excel-based dataset with known ground truth; `real_data_htp_code.py` is the production version adapted for the actual supplementary `.txt` files (TSV OTU table + FASTA). Sample group classification relies on the naming convention `LKGc` = Control, all other prefixes = CLP; timepoint is inferred from `T0`/`T1` suffixes. Wilcoxon tests are applied to the minimum shared sample count when group sizes are unequal to avoid pairing errors. The functional dysbiosis figure (Figure 5) uses literature-derived relative activity values inferred from taxonomic shifts and is clearly labelled as such rather than directly computed from the OTU data.

---

## Inputs

**Files**

- `Supplementary2OTUsTable.txt`: Tab-separated OTU count table (152 OTUs × 137 samples) with SILVA 138.1 semicolon-delimited taxonomy in the final column. Source: Seidel et al. 2023 (PRJNA881235). Serves as the primary count matrix for all diversity and burden analyses.
- `Supplementary1OTUsSeqs.txt`: FASTA file containing representative 16S rRNA sequences for each of the 152 OTUs. Source: Seidel et al. 2023 supplementary material. Used for sequence-level reference and loaded into a `seq_df` DataFrame for downstream traceability.
- `Simulated_OTU_Table` (Excel sheet, AI-generated data script only): Simulated OTU count matrix used for development and validation of the analysis pipeline before applying it to real data.

**Parameters**

- `permutations`: Number of permutations for PERMANOVA / default value: 999
- `top_n_genera`: Number of most-abundant genera retained for the heatmap / default value: 30
- `pathogen_offset`: Pseudocount added before log₁₀ transformation to avoid log(0) / default value: 0.01
- `random_seed`: Seed for reproducible jitter in scatter/boxplots / default value: 42
- `alpha_threshold`: Significance threshold for all statistical tests / default value: 0.05

---

## Outputs

- `alpha_diversity_values.csv`: Per-sample Simpson and Shannon diversity indices with group and timepoint labels. Saved to: `results/tables/alpha_diversity_values.csv`
- `correlation_data.csv`: Per-sample Simpson diversity, pathogenic burden (%), and group label used for Figure 6 regression analysis. Saved to: `results/tables/correlation_data.csv`
- `fig2_alpha_diversity.png / .svg`: Boxplot with jitter showing Simpson Diversity Index across Control and CLP groups at T0 and T1, with Mann-Whitney significance bars. Saved to: `figures/fig2_alpha_diversity/`
- `fig3_beta_diversity.png / .svg`: PCoA scatter plot of Bray-Curtis dissimilarity with PERMANOVA p-value annotated in the title; markers distinguish timepoint (circle = T0, square = T1) and colour distinguishes group. Saved to: `figures/fig3_beta_diversity/`
- `fig4_heatmap.png / .svg`: Annotated heatmap of log₁₀ relative abundance for the top 30 genera, with group and timepoint colour bars above the matrix and white divider lines separating Control from CLP columns. Saved to: `figures/fig4_heatmap/`
- `fig5_functional.png / .svg`: Grouped bar chart comparing literature-inferred relative functional activity (%) across eight metabolic/phenotypic pathways between Control and CLP. Saved to: `figures/fig5_functional/`
- `fig6_correlation.png / .svg`: Scatter plot of Simpson Diversity Index vs pathogenic burden (%) at T1, colour-coded by group, with a linear regression line and Pearson r and p-value in the title. Saved to: `figures/fig6_correlation/`
- Figures saved to: `figures/` (subdirectory per figure)
- Tables saved to: `results/tables/`

---

## Generative AI Disclosure

Generative AI tools were used for code debugging, error resolution, and documentation assistance. All analysis logic, results, and interpretations were reviewed and verified by the author.
