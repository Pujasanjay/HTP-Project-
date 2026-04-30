Master's Bioinformatics – Linking Oral Microbiome Maturation to Functional Dysbiosis in Cleft Lip and Palate

Author: Puja Sanjay Thorat
Date: 15 April 2026

Project Summary

This project computationally investigates whether disrupted neonatal oral microbiome maturation in Cleft Lip and Palate (CLP) neonates constitutes an early developmental precursor to functional dysbiosis observed in older CLP children. Using a published 16S rRNA OTU dataset (Seidel et al. 2023; PRJNA881235) comprising 152 OTUs across 137 samples from 15 CLP neonates and 17 healthy controls — sampled at two postnatal timepoints (T0 ≈ 3 days; T1 ≈ 32 days) — the study computes diversity indices, a pathogenic burden score, community-level distance statistics, and taxonomic visualisations. Findings are contextualised against metatranscriptomic data from Funahashi et al. describing functional dysbiosis in the dental plaque of older CLP children.

Research Question

Does disrupted neonatal oral microbiome maturation in CLP constitute an early developmental precursor to functional dysbiosis in older CLP patients?
Languages, Tools, and Packages

Language

Python (v3.11)
Packages

pandas (v3.0.2)
numpy (v2.4.4)
matplotlib (v3.10.8)
seaborn (v0.13.2)
scipy (v1.17.1)
scikit-bio (v0.7.2)
scikit-learn (v1.8.0)
biom-format (v2.1.17)
Tools

Google Colab (cloud-based Jupyter notebook environment, Python 3.12 runtime)
Git / GitHub (version control and code sharing)
Repository Structure

clp-microbiome-project/
│
├── real_data_htp_code.py                   # Production script — real supplementary .txt files
├── htp_project_For_AI_generated_data.py    # Development script — simulated Excel dataset
│
├── figures/
│   ├── fig2_alpha_diversity/               # Simpson diversity boxplots (PNG + SVG)
│   ├── fig3_beta_diversity/                # PCoA scatter plot (PNG + SVG)
│   ├── fig4_heatmap/                       # Taxonomic heatmap top 30 genera (PNG + SVG)
│   ├── fig5_functional/                    # Functional dysbiosis bar chart (PNG + SVG)
│   └── fig6_correlation/                   # Diversity vs pathogenic burden scatter (PNG + SVG)
│
├── results/
│   └── tables/
│       ├── alpha_diversity_values.csv
│       └── correlation_data.csv
│
├── CLP_Report_Final_draft.docx             # Full written report
├── HTP_project_final_ppt.pptx              # Presentation slides
└── README.md
Data Sources

Dataset	Description	Access
Seidel et al. 2023	16S rRNA OTU table (152 OTUs, 137 samples; SILVA 138.1) from 15 CLP neonates + 17 controls at T0 and T1	NCBI BioProject PRJNA881235
Funahashi et al.	Metatranscriptomic data from dental plaque of older CLP children — used for contextual interpretation and pathogen classification only	Referenced from published paper
Sample breakdown:

Group	T0	T1
Control (LKGc)	38 samples	38 samples
CLP	32 samples	29 samples
Sample naming convention: samples containing LKGc in their ID are classified as Control; all others are CLP. Timepoint is inferred from T0 or T1 in the sample name.

How to Run This Project

Step 1 — Install the required package

Run this in the very first cell before anything else:

!pip install scikit-bio -q
All other packages (pandas, numpy, matplotlib, seaborn, scipy, scikit-learn) come pre-installed in Google Colab. You do not need to install them manually.

Step 2 — Upload your data files

When the upload dialog appears (triggered automatically by the script), select both supplementary files at the same time:

Supplementary2OTUsTable.txt — the tab-separated OTU count table
Supplementary1OTUsSeqs.txt — the FASTA representative sequences
The script auto-detects which file is which by inspecting the first characters: files beginning with > are treated as FASTA; all others as the OTU table. You do not need to rename or reformat the files before uploading.

Step 3 — Run each analysis section in order

The script is divided into sequential parts. Run them from top to bottom:

Part	What it does	Key output
Part 1 — Data loading	Reads and parses both files, splits taxonomy strings, classifies samples by group and timepoint, prints a full summary	Console summary
Part 2 — Alpha diversity	Computes Simpson + Shannon diversity per sample, runs Mann-Whitney U and Wilcoxon tests, generates Figure 2	fig2_alpha_diversity.png, alpha_diversity_values.csv
Part 3 — Beta diversity	Computes Bray-Curtis distance matrix, runs PERMANOVA (999 permutations), generates PCoA plot (Figure 3)	fig3_beta_diversity.png
Part 4 — Heatmap	Filters top 30 genera, sorts samples by group/timepoint, generates annotated heatmap (Figure 4)	fig4_heatmap.png
Part 5 — Functional dysbiosis	Plots literature-inferred pathway activity comparisons (Figure 5)	fig5_functional.png
Part 6 — Correlation	Computes pathogenic burden per sample, runs Pearson correlation with Simpson diversity, generates Figure 6	fig6_correlation.png, correlation_data.csv
Step 4 — Download your outputs

All figures and tables are saved automatically. To download them from Colab run:

from google.colab import files
files.download('figures/fig2_alpha_diversity/fig2_alpha_diversity.png')
files.download('results/tables/alpha_diversity_values.csv')
Alternatively, use the Colab file browser (folder icon in the left sidebar) to browse all saved files and download them individually.

Analytical Workflow in Detail

Part 1 — Data Ingestion and Parsing

The OTU count table is read directly from memory using pd.read_csv with tab separation and io.BytesIO, which avoids file-path errors in the Colab environment. The taxonomy column contains semicolon-delimited SILVA lineage strings (e.g. Bacteria;Firmicutes;Bacilli;...;Streptococcus;) and is split into six taxonomic ranks: Kingdom, Phylum, Class, Order, Family, Genus. The FASTA file is parsed line by line: header lines (>OTU_ID) open new entries; subsequent lines accumulate the sequence string. The result is two clean DataFrames — otu_counts (OTUs × samples, integer counts) and tax_df (OTUs × taxonomy ranks) — plus a seq_df DataFrame holding all representative sequences. A full console summary is printed showing OTU count, sample count, phylum distribution, and all sample-to-group/timepoint mappings.

Part 2 — Alpha Diversity (Figure 2)

Alpha diversity measures species richness and evenness within each individual sample. Two indices are computed per sample:

Simpson Diversity Index:

D = 1 − Σpᵢ²
where pᵢ is the relative abundance of OTU i. Ranges from 0 (no diversity) to 1 (maximum diversity). More sensitive to dominant species — appropriate for microbiome data where a few taxa often dominate.

Shannon Entropy:

H = −Σpᵢ ln(pᵢ)
Gives proportional weight to rare species as well as dominant ones, providing a complementary view of diversity.

Statistical comparisons use non-parametric tests because microbiome data are rarely normally distributed:

Mann-Whitney U test (two-sided) compares Control vs CLP at T0 and T1 independently.
Wilcoxon signed-rank test compares T0 vs T1 within each group to assess postnatal maturation. When group sizes are unequal, the test is applied to the minimum shared length to avoid pairing errors.
Results are visualised as a four-panel boxplot with individual data points jittered (seed = 42) for visibility. A significance bar annotated with * (p < 0.05), ** (p < 0.01), *** (p < 0.001), or n.s. is drawn between the T1 comparison groups. A dashed vertical line separates the T0 and T1 panels.

Part 3 — Beta Diversity: Bray-Curtis + PCoA + PERMANOVA (Figure 3)

Beta diversity measures how different microbial communities are from one another across samples.

Bray-Curtis dissimilarity is computed pairwise across all 137 samples:

BC(u, v) = Σ|uᵢ − vᵢ| / Σ(uᵢ + vᵢ)
A small epsilon (1×10⁻¹⁰) is added to the denominator to prevent division by zero. This produces a 137 × 137 symmetric dissimilarity matrix.

PCoA (Principal Coordinates Analysis) reduces this high-dimensional matrix to two dimensions using classical multidimensional scaling: the distance matrix is squared, double-centred, and decomposed via PCA. The resulting coordinates are plotted — circles for T0, squares for T1; green for Control, orange for CLP — with axis labels showing the percentage of variance explained by each principal coordinate.

PERMANOVA (Permutational Multivariate ANOVA, 999 permutations, via scikit-bio) tests whether group membership (CLP vs Control) explains a statistically significant proportion of the variance in community composition. The pseudo-F statistic and permutation-derived p-value are printed to console and annotated in the figure title.

Part 4 — Taxonomic Heatmap (Figure 4)

OTU counts are converted to column-wise relative abundances (%), then log₁₀-transformed with a pseudocount of 0.01 to handle zero-count OTUs:

log_abund = log₁₀(relative_abundance + 0.01)
Only the top 30 genera by mean relative abundance across all samples are retained to keep the figure readable. Samples are reordered as Control-T0 → Control-T1 → CLP-T0 → CLP-T1. Two thin annotation colour bars above the heatmap encode group identity (green = Control, orange = CLP) and timepoint (blue = T0, yellow = T1). Bold white vertical divider lines mark the boundary between Control and CLP samples; dashed white lines mark T0/T1 boundaries within each group. The colour scale (RdYlBu_r) runs from blue (low abundance) through yellow to red (high abundance).

Part 5 — Functional Dysbiosis Bar Chart (Figure 5)

Eight metabolic and phenotypic pathway categories are compared between groups using literature-supported relative activity values inferred from the taxonomic shifts observed in this dataset and from Funahashi et al. metatranscriptomic findings. Pathways include carbohydrate metabolism, lactic acid production, biofilm formation, gram-negative bacteria proportion, anaerobic metabolism, facultative anaerobes, stress tolerance, and mobile genetic elements. These values are not directly computed from the OTU count table and are clearly labelled as literature-inferred in the figure subtitle.

Part 6 — Pathogenic Burden Index and Diversity Correlation (Figure 6)

Pathogenic burden is defined as the summed relative abundance (%) of 46 OTUs classified as pathogenic based on Funahashi et al. differential abundance findings. Genera included: Enterobacteriaceae members (Citrobacter, Enterobacter, Escherichia-Shigella, Klebsiella), Enterococcus, Staphylococcus, Acinetobacter, Bifidobacterium, Corynebacterium, Lawsonella, and Lacticaseibacillus.

Pearson correlation is computed between each sample's Simpson Diversity Index and its pathogenic burden score across all T1 samples. A positive correlation would indicate that higher microbial diversity co-occurs with higher pathogenic load — a pattern consistent with dysbiotic succession (where opportunistic pathogens proliferate alongside diverse communities) rather than healthy maturation. A linear regression line is overlaid on the scatter plot, with the Pearson r coefficient and p-value annotated in the figure title. Points are colour-coded by group (Control = green, CLP = orange).

Inputs

Files

Supplementary2OTUsTable.txt: Tab-separated OTU count table. 152 rows (OTUs) × 137 sample columns + 1 taxonomy column. Header row begins with ##OTU ID. SILVA 138.1 semicolon-delimited taxonomy strings in the taxonomy column. Source: Seidel et al. 2023 (PRJNA881235).
Supplementary1OTUsSeqs.txt: FASTA file with one entry per OTU. Header format: >OTU_ID. Source: Seidel et al. 2023 supplementary material. Used to build seq_df for downstream traceability.
Key Parameters

Parameter	Description	Default value
permutations	Number of permutations for PERMANOVA	999
top_n_genera	Number of genera retained for the heatmap	30
pseudocount	Value added before log₁₀ transformation to avoid log(0)	0.01
random_seed	Seed for reproducible jitter in scatter/boxplots	42
alpha	Significance threshold for all statistical tests	0.05
pathogen_count	Number of OTUs classified as pathogenic	46
Outputs

File	Description	Location
alpha_diversity_values.csv	Per-sample Simpson index, Shannon entropy, group, and timepoint	results/tables/
correlation_data.csv	Per-sample Simpson index, pathogenic burden (%), and group label	results/tables/
fig2_alpha_diversity.png/.svg	Boxplot + jitter: Simpson diversity by group and timepoint with significance bars	figures/fig2_alpha_diversity/
fig3_beta_diversity.png/.svg	PCoA scatter of Bray-Curtis dissimilarity; PERMANOVA p-value in title	figures/fig3_beta_diversity/
fig4_heatmap.png/.svg	log₁₀ relative abundance heatmap of top 30 genera with annotated group/timepoint colour bars	figures/fig4_heatmap/
fig5_functional.png/.svg	Grouped bar chart of literature-inferred functional pathway activity (%)	figures/fig5_functional/
fig6_correlation.png/.svg	Scatter: Simpson diversity vs pathogenic burden with regression line, Pearson r and p annotated	figures/fig6_correlation/
All figures are saved at 300 DPI in both PNG (for documents and presentations) and SVG (for vector editing in Inkscape or Illustrator).

Key Results Summary

Metric	Control	CLP	p-value	Interpretation
Simpson diversity T0	0.550 ± 0.251	0.416 ± 0.259	n.s.	No significant difference at birth
Simpson diversity T1	0.679 ± 0.177	0.668 ± 0.130	0.314 (n.s.)	Both groups mature similarly in overall diversity
Pathogenic burden T0	19.0 ± 21.6%	20.4 ± 28.9%	n.s.	Comparable at birth
Pathogenic burden T1	10.6 ± 13.5%	23.1 ± 22.4%	0.0007 (**)	CLP burden remains elevated; controls decrease
Beta diversity (T1)	—	—	PERMANOVA p = 0.006	Community composition significantly different at T1
Diversity–burden correlation	—	—	r = 0.27, p = 0.029	Positive dysbiotic association confirmed
Notes and Assumptions

Sample classification relies entirely on the naming convention embedded in sample IDs. If your dataset uses a different naming convention, update the group/timepoint classification logic accordingly.
The Wilcoxon test requires paired samples of equal length. When T0 and T1 group sizes differ, the script clips to the minimum shared count. This is a conservative approximation, not true biological pairing.
Figure 5 (functional dysbiosis) uses literature-derived values and is intended as contextual biological interpretation, not a direct computational result from the OTU table.
Both scripts implement the same core analytical logic. htp_project_For_AI_generated_data.py was used during development and validated against a simulated dataset with known properties before the pipeline was applied to the real supplementary files.
The PCoA implementation uses classical MDS (double-centring + PCA) rather than a dedicated ordination library, which is mathematically equivalent for symmetric distance matrices.
Generative AI Disclosure

Generative AI tools were used for code debugging, error resolution, and documentation assistance throughout this project. All analysis logic, statistical decisions, biological interpretations, and final results were reviewed and verified by the author.
