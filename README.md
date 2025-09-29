# Spatial and single-cell Transcriptomics Integration Tool for CHaracterization (STITCH)

**This repo is retired. Refer to the latest repo [here](https://github.com/dimi-lab/STITCH) from dimi-lab.** 

## Introduction
This Nextflow pipeline provides a comprehensive framework for tertiary analysis of Visium spatial transcriptomics data and single-cell RNA-sequencing (scRNA-seq). The pipeline includes the following modules:

1. **Quality Control (QC):** Evaluate the quality of the input data, and identify thresholds for QC metrics.
2. **Normalization:** Apply data normalization to ensure comparability between samples.
3. **Data Integration and Merging:** Combine multiple samples using integration-based analysis (high batch-effect) or merge-based analysis (minimal batch-effect or different sample-to-sample compositions).
4. **Clustering:** Identify clusters of cells or regions based on gene expression and/or spatial context.
5. **Differential Expression Analysis:** Detect differentially expressed genes across conditions or clusters.
6. **Reporting:** Generate summary reports for QC and a final analysis report.

This pipeline is designed to provide reproducible and efficient analysis workflows, generating both intermediate and final outputs. The pipeline is largely built on Seurat framework.

---

## Setup Instructions

### 1. Clone the Repository
Clone the pipeline repository to your local machine using the following command:

```bash
git clone https://github.com/Liuy12/STITCH.git
cd STITCH
## optional, specify .cache directory for renv
mkdir .cache/
export RENV_PATHS_CACHE="$PWD/.cache/"
```

### 2. Install R Dependencies
Ensure you have R version **4.4.1** installed and loaded. 

```bash
## optional
## e.g. load required R version via module
module load r/4.4.1
## load pandoc
module load pandoc
```

Open a new R session, then use `renv` to restore the required R packages:

```R
renv::restore()
```

This will install all the necessary R packages specified in the repository, and might take a while.

### 3. Prepare the Sample Information Sheet
Create a sample information sheet in tab-delimited format with at least the following first three columns. Ensure that the first three column names are "sampleid", "condition", "secondary_output". Make sure all fields are tab-seperated. Otherwise, the pipeline will fail. 

- **sampleid:** Unique identifier for each sample. Ensure the sample ids do not contain space or special characters.
- **condition:** Experimental condition for the sample, e.g. Control or Case.
- **secondary_output:** Path to the secondary output directory from Cell Ranger (scRNA-seq) or Space Ranger (Visium). **DO NOT set this to the 'filtered_feature_bc_matrix' folder. Set it to the 'outs' folder that include all output from Cell Ranger/Space Ranger.**

An example `samplesheet.tsv`. Noted secondary_output is set to the 'outs' folder:

```tsv
sampleid    condition   secondary_output
sample1 control /path/to/sample1/outs
sample2 treatment   /path/to/sample2/outs
```

### 4. Modify the Configuration File
Adjust the provided configuration file (e.g., `nextflow.config.scRNAseq.human`) to suit your analysis. Some key parameters to examine/modify include:

- **feature_list:** Path to genes of interest, one gene per line; Final report will generate visualizations of expression levels for those genes. Set to 'NA' to disable. 
- **output_dir:** Path to pipeline output directory.
- **geneinfo:** Path to gene level annotation file. This is used to add feature level meta data. **Make sure to check the version of reference used**
- **qc_only:** Whether to stop the pipeline after QC. This could be helpful to identify cutoffs for various QC metrics. **Recommendation is to set qc_only to true -> evaluate QC report -> update qc cutoffs (if needed) -> set qc_only to false to resume pipeline**
- **adaptive_cutoff_flag:** Whether to apply adaptive cutoff identification (based on IQR). Rather than selecting the same cutoffs across all samples, this will identify cutoffs based on distribution of QC metrics within each sample to create sample-specific cutoffs. Could be helpful if you expect different metric distributions across samples. **Recommendation is to always turn on 'adaptive_cutoff_flag' during qc step -> evaluate qc report -> enable or disable adaptive_cutoff_flag**
- **norm_dimreduc:** Normalization method for dimension reduction/differential testing, either SCT or LogNormalize. **Recommend to use SCT for scRNA-seq and LogNormalize for Visium.** 
- **norm_diff:** Normalization method for differential testing, either SCT or LogNormalize. **Recommend to use LogNormalize.** 
- **cellcycle_correction_flag:** Whether to estimate and correct for cell-cycle effect for clustering.
- **merge_analysis/integration_analysis:** whether to perform merge-based/integration-based analysis.
- **merge_only/integration_only:** Whether to stop after merge-based/integration-based analysis. **Recommend to enable it if you want to adjust resolution parameter before proceeding to DE analysis.**
- **integration_method:** Integration strategy.
- **sketch_flag:** Whether to perform sketch-based workflow. **Recommend to enable if you have large number of cells (e.g.>100K)**
- **resolution:** Resolution parameter used to identify number of clusters.
- **spatial_cluster:** Method for spatial clustering.
- **control_var/case_var:** control/case group for differential expression analysis.
- **covariate_list:** Covariates to adjust, when performing differential analysis between conditions.
- **test:** Statistical test.

Example config files are provided for human and mouse:

- **nextflow.config.scRNAseq.human:** human scRNAseq.
- **nextflow.config.scRNAseq.mouse:** mouse scRNAseq.
- **nextflow.config.Visium.human:** human Visium.
- **nextflow.config.Visium.mouse:** human Visium.

### 5. Run the Pipeline
Execute the pipeline with the following command:

```bash
nextflow run main.nf --samplesheet samplesheet.tsv -c nextflow.config.scRNAseq.human -work-dir ./work 
```

- **`--samplesheet`:** Path to the prepared sample sheet.
- **`-c`:** Specifies the configuration file.
- **`-work-dir`:** Specifies processing directory.

The pipeline currently supports local(default, `-profile local`) or slurm (`-profile slurm`). You can modify the config file based on your own needs. 
You can add `-resume` option to the command if you want to resume a pipeline.

**Recommend to add `-profile slurm` when launching the pipeline. Also add `-resume` option to resume a pipeline, e.g. after QC or updating a parameter**

---

## Notes
- Ensure all required dependencies (Nextflow, R, and other tools) are installed and configured.
- Customize the pipeline to suit your specific data and experimental design.
- For further assistance, consult the documentation or open an issue in the repository.
