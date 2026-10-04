# MFSDA-SUR
This package proposes a model-free procedure for FDR-controlled variable selection in high-dimensional, right-censored survival data within the sufficient dimension reduction (SDR) framework. 
# MFSDA-SUR

**R package:** `MFSDASUR`  
**Author and maintainer:** Jian Xiao <xiaoj771@sina.com>  
**Version:** 0.1.0

An installable R implementation of the supplied MFSDA-SUR survival variable
selection program, with documentation, examples, numerical regression tests,
and diagnostics. R package names cannot contain hyphens, so the project is
called **MFSDA-SUR** and is installed and loaded as **MFSDASUR**.

## Repository materials

- [Chinese user guide](README.zh-CN.md)
- [Validation report](docs/MFSDA-SUR_验证报告.txt)
- [Installable source archive](dist/MFSDASUR_0.1.0.tar.gz)
- [Original source bundle](dist/MFSDA-SUR_source_0.1.0.zip)
- [Runnable example](inst/examples/quick_start.R)

Download the source archive from `dist/` before following the installation
commands below. The R package source is in `MFSDA-SUR/` within
`xiaojian513686/xiaojian_zuel`. To build from a repository checkout, run
`R CMD build MFSDA-SUR` from its root directory.

## Installation

After these files are published to the target repository, install directly
from its package subdirectory:

```r
install.packages("remotes", repos = "https://cloud.r-project.org")
remotes::install_github("xiaojian513686/xiaojian_zuel",
                        subdir = "MFSDA-SUR")
library(MFSDASUR)
```

Alternatively, download the source archive from `dist/` and install locally.

R >= 4.1.0 is required. Install the external dependency and then the source
archive from its saved location:

```r
install.packages("glmnet", repos = "https://cloud.r-project.org")
install.packages("MFSDASUR_0.1.0.tar.gz", repos = NULL, type = "source")
library(MFSDASUR)
```

`MFSDASUR` contains no compiled code. `glmnet` has its own dependencies;
install an available binary for your R release, or use the appropriate
build tools if installing it from source. `splines` and `stats` ship with R.
The package has not been published to CRAN, so `install.packages("MFSDASUR")`
alone is not the installation method for this delivery.

## Quick example

```r
set.seed(2026)
n <- 300
p <- 8
X <- matrix(rnorm(n * p), n, p)
colnames(X) <- paste0("x", seq_len(p))
log_time <- 1.5 * X[, 1] - X[, 2] + 0.8 * X[, 3] + rnorm(n)
log_censor <- rnorm(n, mean = 3, sd = 2)
Y <- pmin(log_time, log_censor)
delt <- as.integer(log_time <= log_censor)

fit <- MFSDA_SUR(X, Y, delt, h = 3, DS = 5, B = 100,
                 fdrlevel = c(0.1, 0.2), seed = 2026)
fit$result
fit$selection_rate
fit$diagnostics
rownames(fit$result)[fit$result[, "0.1"] == 1]
```

`X` is an observation-by-predictor numeric matrix. `Y` contains one finite
observed outcome per row, including censored outcomes. `delt = 1` means
observed event; `delt = 0` means right censoring. The function sorts internally
and does **not** automatically log `Y`. No missing-value imputation is performed.

```r
combined <- MFSDA_SUR_combine(X, Y, delt, h_value = 3,
                              DS = 5, B = 100, seed = 2026)
combined$h_opt
combined$selection_counts
combined$result
```

The combination function chooses from `h = 1, ..., h_value` by the largest
total number of selected matrix entries, with the first tie retained.
It is the supplied count-based heuristic, **not** predictive cross-validation.
With multiple FDR levels, the score is summed over all levels.

## Compatibility and diagnostics

All nine original functions are exported, and `$result` remains the main
0/1 selection matrix. `B`, `subsample_fraction`, `nfolds`, `vote_threshold`,
and optional local `seed` are new controls. Defaults retain 100 subsamples,
the 0.632 subsample fraction, 10 requested folds, and a 0.6 vote threshold.

The implementation retains full-sample weights and spline bases, transformed
least-squares intercepts, the original SE scaling, the uncorrected negative/
positive tail ratio, and zero selections when at most one candidate is
screened. In particular, accepting a one-column matrix does not change that
last statistical rule: it still gives no final selections.

Selected-direction missing coefficients are replaced by zero; missing or
zero SEs are replaced by one, as in the supplied program. New diagnostics
make these fallbacks visible. Input validation, small-event safeguards,
constant-direction handling, zero padding for single-column glmnet inputs,
and stable matrix dimensions are documented in `NEWS.md` and the help pages.

The original row-wise tie convention is preserved: event/censor ordering
within equal observed times can affect KM weights. Weight truncation uses
all weights including zeros. A low quantile with heavy censoring can remove
all positive weights; that case now gives a clear error.

Software regression tests verify behavior on their fixtures. They are not
an empirical FDR study and do not establish a general FDR guarantee for the
voting or adaptive direction-count procedure.

## Help, examples, and development

```r
help(package = "MFSDASUR")
?MFSDA_SUR
?MFSDA_SUR_combine
source(system.file("examples", "quick_start.R", package = "MFSDASUR"))
```

From a shell in the source package's parent directory:

```sh
R CMD build MFSDA-SUR
R CMD check --no-manual MFSDASUR_0.1.0.tar.gz
```

The tests use base R and the package dependencies. They include an independent
direct reference implementation of the supplied computation, dimension and
input checks, seeded reproducibility, and selection-count tie handling.
See `README.zh-CN.md` for the Chinese user guide.

## License

Copyright (c) 2026 Jian Xiao. No open-source license was specified for this
development delivery. Rights are reserved as stated in `LICENSE`; contact
the author to designate a license or obtain permissions before redistribution.
