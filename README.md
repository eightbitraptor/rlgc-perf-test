# rlgc-perf-test

A reproducible performance analysis of Ruby's `ractor-local-gc` branch,
built from raw [ruby-bench](https://github.com/eightbitraptor/ruby-bench)
results. The report compares a candidate commit against its base commit,
calibrates every verdict against an A/A run (the same build on both sides,
which measures pipeline noise), and renders charts, tables, and prose
explanations of the statistics.

The rendered report is published automatically to GitHub Pages:
<https://www.eightbitraptor.com/rlgc-perf-test/>
## Layout

| Path | Purpose |
|---|---|
| `analysis.Rmd` | The parameterized RMarkdown report — all statistics live here |
| `data/output_054.json` | A/B run: base commit vs candidate commit |
| `data/output_053.json` | A/A run: base commit vs itself (noise calibration) |
| `.github/workflows/pages.yml` | Renders the report and deploys it as the Pages homepage |

## Getting started

Requirements: R (≥ 4.x) and pandoc.

```sh
# install the R packages once
Rscript -e 'install.packages(c("rmarkdown", "knitr", "ggplot2", "jsonlite",
  "dplyr", "tidyr", "purrr", "tibble", "stringr", "scales"))'

# render the report
Rscript -e 'rmarkdown::render("analysis.Rmd")'

open analysis.html
```

## Analyzing new results

Generate result files with ruby-bench, comparing two Ruby builds
(`--interleave` alternates executables per benchmark; `--warmup=50` keeps JIT
compilation out of the measurement window):

```sh
MIN_BENCH_TIME=30 ./run_benchmarks.rb --rss --interleave --warmup=50 \
  -e master::/path/to/base/ruby -e experiment::/path/to/candidate/ruby
```

Run it twice: once with the two builds under comparison (the A/B file), and
once with the identical base build on both sides (the A/A file). Drop both
JSON files into `data/` and render with parameters:

```sh
Rscript -e 'rmarkdown::render("analysis.Rmd",
  params = list(ab_file = "data/new_ab.json", aa_file = "data/new_aa.json"),
  output_file = "analysis-new.html")'
```

The A/A file is optional; without it the report falls back to a fixed 1%
significance threshold instead of the measured noise floor.

## Publishing

Every push to `main` renders `analysis.Rmd` and deploys it as `index.html`
on GitHub Pages. One-time setup: repository **Settings → Pages → Source →
GitHub Actions**.
