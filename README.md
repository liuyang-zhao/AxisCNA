# SpaCNA

**Spatial DNA CNV toolkit** (Conda package distribution).

| | |
|--|--|
| **This repo** | Conda install package releases |
| **Source / scripts** | [AXIS-DNA-seq](https://github.com/liuyang-zhao/AXIS-DNA-seq) (kept separately) |
| **Conda package page** | https://anaconda.org/liuyangzhao/spacna |

---

## Installation (recommended)

Install from Anaconda.org channel **`liuyangzhao`**:

```bash
conda create -y -n spacna
conda activate spacna
conda install -y -c liuyangzhao -c conda-forge -c bioconda spacna
spacna help
```

Or:

```bash
conda install -y liuyangzhao::spacna -c conda-forge -c bioconda
```

> **Bioconda:** `spacna` is also on bioconda (`0.1.0+`). Prefer bioconda when you want the public channel; use `-c liuyangzhao` for the latest personal-channel build (e.g. `0.1.1`).
>
> ```bash
> conda install -c bioconda -c conda-forge spacna
> ```

---

## Install from this GitHub package (optional)

A pre-built Conda package is shipped under [`dist/`](dist/):

- `dist/spacna-0.1.1-0.tar.bz2` (current)
- `dist/spacna-0.1.0-0.tar.bz2`

```bash
conda create -y -n spacna
conda activate spacna
conda install -y -c conda-forge -c bioconda ./dist/spacna-0.1.1-0.tar.bz2
spacna help
```

Prefer downloading the same file from **GitHub Releases** when available.

---

## Main functions and usage

Load R APIs once:

```r
repo <- system("spacna path", intern = TRUE)
setwd(repo)
Sys.setenv(SPACNA_REPO_ROOT = repo)
source("scripts/R/spacna.R")
```

| Function | Role |
|----------|------|
| `spacna align-dna` / `spacna align-bulk` | CLI: spatial / bulk DNA → bin counts |
| `spacna_dna_cnv()` | Spatial DNA CNV clustering & plots |
| `spacna_compare_clones()` | Pairwise clone similarity (Pearson / Jaccard) |
| `spacna_export_medicc2()` | Export clone CN table for MEDICC2 |
| `spacna_spi()` | SPI lineage, fate, differential CNA, tree |
| `spacna_medicc2()` | Merged MEDICC2 helper pipeline |
| `spacna_rna()` | SpotClean / Seurat (+ optional inferCNV) |
| `spacna_cnv_expr()` | CNV–expression coupling |
| `spacna_gen_reference()` | Generate any bin-size reference |

### 1. Alignment (CLI)

```bash
export SPACNA_GENOME_HUMAN_INDEX=/path/to/GRCh38_bowtie2_index
spacna align-bulk <run_id> human 1
spacna align-dna /path/to/project <run_id> human 1
```

**Input layout (`project_dir`):**

```text
project_dir/
  rawdata/<run_id>_R1.fq.gz          # genomic reads
  rawdata/<run_id>_R2.fq.gz          # spatial barcode reads
  processed/<run_id>.spatial_barcodes_location.csv
```

Barcode CSV columns (typical): barcode / bead id, x, y (and related fields used by the pipeline).

**Output:**

```text
project_dir/results/<run_id>.sparse_counts_1Mb.txt
# (also intermediate BAM / processed files under processed/, tmp/)
```

Sparse matrix format (no header): `bin_index`, `bead_index`, `count` (1-based indices matching the bin table and barcode table order).

Bulk control output (from `align-bulk`): tab-delimited bin counts  
`chr`, `start`, `end`, `count` — used later as `bulk_control_path`.

---

### 2. Spatial DNA CNV — `spacna_dna_cnv()`

```r
spacna_dna_cnv(
  expt_id = "17_1",
  base_dir = "/path/to/project",   # contains data/ and results/
  species = "human",               # or "mouse"
  bin_size = "1Mb",
  bulk_control_path = "/path/to/normal_bulk_1Mb_total_reads.txt"
)
```

**Input (under `base_dir`):**

| File | Format |
|------|--------|
| `data/Spatial_DNA/<id>/<id>.sparse_counts_1Mb.txt` | 3 columns: bin, bead, count |
| `data/Spatial_DNA/<id>.spatial_barcodes_location.csv` | Bead coordinates |
| `bulk_control_path` | `chr start end count` (normal bulk) |
| Bundled refs (auto) | 1 Mb bins / GC / mappability |

**Output (`results/Spatial_DNA/<id>/`):**

| File | Content |
|------|---------|
| `<id>_spatial_res.rds` | Full result object (clusters, CN profiles, paths) |
| `<id>_Spatial_Clusters.pdf` | Spatial cluster map |
| `<id>_Silhouette_Scores.pdf` | Cluster number selection |
| `<id>.Genome_Tracks_Overlap.pdf` | Genome-wide CN tracks by clone |

---

### 3. Clone comparison — `spacna_compare_clones()`

```r
spacna_compare_clones("17_1", "19_1", base_dir = "/path/to/project")
```

**Input:** two samples already analyzed (`*_spatial_res.rds`).  
**Output:** `Comprehensive_Stats_<A>_vs_<B>.csv` (Spearman / Pearson / Jaccard per clone pair).

---

### 4. MEDICC2 export & SPI lineage

```r
# From DNA CNV results
spacna_export_medicc2(
  res_list = list(res_a, res_b),
  out_dir = "results/MEDICC2_Input",
  project_name = "MyProject"
)

spacna_spi(
  cn_table_path = "results/MEDICC2_Input/MyProject_medicc2_input.tsv",
  clone_groups = list(
    pre  = c("17_1_C1", "19_1_C1"),
    post = c("19_2_C1", "20_2_C1")
  ),
  out_dir = "results/SPI_lineage"
)
```

**CN table input** (`*_medicc2_input.tsv`):

| column | meaning |
|--------|---------|
| `sample_id` | Clone ID (e.g. `17_1_C1`) |
| `chrom` | Chromosome |
| `start` / `end` | Bin coordinates |
| `cn` | Integer copy number |

**SPI outputs (`out_dir/`):**

| File | Content |
|------|---------|
| `SPI_matrix.tsv` | Pre→post Subclone Persistence Index |
| `post_clone_fate.tsv` / `pre_clone_fate.tsv` | Persistent / emergent / eradicated |
| `differential_cna_1Mb_bins.tsv` | Surviving − eradicated ΔCN |
| `differential_regions_ranked.tsv` | Merged significant regions |
| `figures/SPI_heatmap.pdf` | SPI heatmap |
| `07_containment_lineage_tree_by_pair_events.png` | Containment lineage tree |

---

### 5. RNA / multi-omics (optional)

```bash
export SPACNA_RNA_DIR=/path/to/Spatial_RNA
spacna rna-cellranger <sample_id> human 1
```

```r
spacna_rna(
  sample_id = "1_3",
  raw_matrix_dir = "path/to/raw_feature_bc_matrix",
  out_dir = "results/Spatial_RNA/1_3"
)

spacna_cnv_expr(
  out_dir = "results/multiomics/cnv_exp",
  rna_matrix_tsv = ".../Decontaminated_filtered_matrix.tsv",
  cnv_integration_csv = ".../clone_cnv_for_expr_integration.csv",
  gene_bin_map_csv = ".../gene_bin_map.csv"
)
```

**RNA input:** 10x-style `raw_feature_bc_matrix/` (matrix.mtx + features + barcodes).  
**RNA output:** decontaminated matrix TSV, Seurat object / clustering under `out_dir`.  
**Coupling output:** cis-correlation tables, top bins/genes CSVs, run summary.

---

### 6. Custom bin-size reference — `spacna_gen_reference()`

Package ships **1 Mb** only. For other sizes:

```bash
spacna gen-reference --species human --bin-size 500kb --mode bins
# GC/map need FASTA + mappability bigWig:
spacna gen-reference --species human --bin-size 500kb --mode all \
  --fasta "$SPACNA_HUMAN_FASTA" --mappability-bw "$SPACNA_HUMAN_MAP_BW"
```

**Output:** `reference/genomic_bins/`, `gc_content/`, `mappability/` tables named by species and bin size.

---

## Changelog

### 0.1.1
- Fix `generate_sparse_counts.py` barcode indexing (header no longer shifts spot indices)
- Fix `spatial_dna_analysis.R` syntax error (extra `)` in `RUN_EXAMPLES` block)
- Spatial cluster maps: Y-axis 0 at top, resolution at bottom (`scale_y_reverse`)

### 0.1.0
- Initial conda package release

---

## License

[MIT](LICENSE) © 2026 [liuyang-zhao](https://github.com/liuyang-zhao)
