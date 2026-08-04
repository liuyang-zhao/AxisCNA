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

> **Bioconda:** Submitting `spacna` to the public bioconda channel is **in progress** (PR).  
> After it is merged, you can also use:
>
> ```bash
> conda install -c bioconda -c conda-forge spacna
> ```
>
> Until then, please install with `-c liuyangzhao` as above.

---

## Install from this GitHub package (optional)

A pre-built Conda package is shipped under [`dist/`](dist/):

- `dist/spacna-0.1.0-0.tar.bz2`

```bash
conda create -y -n spacna
conda activate spacna
# install the local tarball (dependencies from conda-forge / bioconda)
conda install -y -c conda-forge -c bioconda ./dist/spacna-0.1.0-0.tar.bz2
spacna help
```

Prefer downloading the same file from **GitHub Releases** when available.

---

## What you get after install

- CLI: `spacna`
- Bundled 1 Mb references + analysis scripts under `$(spacna path)`
- R entry: `source("scripts/R/spacna.R")` after `setwd(system("spacna path", intern=TRUE))`

Full usage and environment variables: see the source repo  
https://github.com/liuyang-zhao/AXIS-DNA-seq

---

## Contributor

- [liuyang-zhao](https://github.com/liuyang-zhao)

## License

[MIT](LICENSE) © 2026 [liuyang-zhao](https://github.com/liuyang-zhao)

## Issues

https://github.com/liuyang-zhao/SpaCNA/issues
