# Wavelet features and stock price manipulation on the Indonesia Stock Exchange

Code and data for the article *"Do Wavelet Features Help Detect Stock Price Manipulation? Evidence from Enforcement Cases on the Indonesia Stock Exchange"* (Anggit Suryopratomo, Universitas Ma'soem), submitted to *Scientific Journal of Informatics (SJI)*.

## Description

Wavelet-based anomaly detectors are often proposed for detecting stock price manipulation. This repository tests them against real enforcement cases. It uses ten manipulation episodes in nine IDX stocks sanctioned by the Financial Services Authority (OJK), each matched with five control stocks.

The repository compares four **causal** wavelet anomaly scores with three simple surveillance rules. Causal means every feature on day *t* uses only data up to day *t*.

| Score | Definition |
| --- | --- |
| W-pump | min of short-scale (levels 1–2) wavelet energy z-scores of volume and returns |
| W-long | same as W-pump with levels 3–4 (exploratory) |
| W-mean | mean of all 12 wavelet energy z-scores |
| W-iforest | Isolation Forest on the wavelet z-scores, fitted on days before the evaluation window |
| Volume ratio | volume / 20-day average volume |
| 20-day volatility | 20-day standard deviation of log returns |
| 20-day return | 20-day cumulative log return |

All scores are standardized against the previous 250 trading days.

**Main finding.** No wavelet score beat the simple 20-day return or volatility rules. The confirmed IDX cases are gradual multi-month price ramps, which wavelet detail energy does not capture.

## Repository contents

| File / folder | Content |
| --- | --- |
| `Manipulation_detection_notebook.ipynb` | Google Colab notebook, saved with the outputs of the run reported in the article. It downloads the data, matches controls, builds the scores and runs every evaluation. |
| `cases.csv` | Enforcement episodes and the manipulation windows used as labels (Table 1 of the article) |
| `data_manip.zip` | Downloaded price/volume panel (`panel.pkl`) and Yahoo Finance sector cache (`sector_cache.csv`) used in the article |
| `results/` | Result files (CSV) produced by the notebook (see *Output files*) |
| `figures/` | Figures 1–2 of the article and the price/score plot for every episode |
| `requirements.txt` | Python dependencies |
| `LICENSE` | MIT License for the code |

## Dataset information

- **Source:** Yahoo Finance via the `yfinance` library (ticker suffix `.JK`); daily Open, High, Low, Close, Volume.
- **Stock universe for controls:** the IDX stock list (`daftar_saham.xlsx`, column `Kode`), downloaded from idx.co.id → Data Pasar → Data Saham → Daftar Saham in November 2026.
- **Labels:** manipulation periods stated in OJK sanction announcements (January–November 2026); see `cases.csv`. KIOS and IMPC were stated by month only, so whole months are used. AYLS and DSSA have several windows, and the label is their union. FILM has two separate episodes (FILM-21, FILM-23).
- **Controls:** same Yahoo Finance sector, closest mean log traded value over the 250 trading days before the first window, at least 90% data coverage, and not itself in an OJK trading sanction.
- **Evaluation window:** 365 calendar days before the first manipulation window to 180 days after the last one.
- **Zero-volume days** are kept (volume enters as log(1 + V)).
- **Not scorable:** BSML, which was listed in December 2021 and has too little history. W-iforest also could not be computed for AYLS, FILM-21 and IMPC.

**Reproducibility note:** Yahoo Finance data and sector labels can change after download. To reproduce the published numbers exactly, unzip `data_manip.zip` into the working folder before running. The notebook then loads `data_manip/panel.pkl` instead of downloading.

## Code information

| Section | Purpose | Article output |
| --- | --- | --- |
| 1. Setup and settings | Packages and the `CFG` parameters | – |
| 2. Enforcement cases | `CASES` list with OJK windows | Table 1 |
| 3. Control universe and matching | Reads the IDX list, sector lookup, control matching | – |
| 4. Price and volume data | Download (or load `panel.pkl`) | `controls.csv` |
| 5. Causal wavelet features and scores | Causal SWT energy, z-scores, the seven scores | Eqs. (1)–(2) |
| 6. Evaluation | Within-stock AUC, case-vs-control rank, alarms, paired tests, figures | Tables 2–4, Figures 1–2 |
| 7. Leakage check | Same evaluation with two-sided (non-causal) wavelet features | Look-ahead bias subsection |

**Main settings (`CFG`)**

| Setting | Value |
| --- | --- |
| Wavelet / levels | Haar / 4 (stationary wavelet transform, `pywt.swt`) |
| Causal window | 128 trading days |
| Energy | mean squared detail coefficients over the last 8 days |
| z-score history | previous 250 trading days (min. 150) |
| Controls per case | 5 |
| Alarm threshold | 99th percentile of control scores before the manipulation period |
| Tests | one-sided Wilcoxon signed-rank (AUC > 0.5, rank > 0.5); paired Wilcoxon (wavelet vs rule); Bonferroni level 0.05/7 |

## Usage instructions

**Google Colab (recommended)**

1. Open `Manipulation_detection_notebook.ipynb` in Google Colab.
2. For exact reproduction, upload `data_manip.zip` and run `!unzip -o data_manip.zip`. For a fresh download, upload the IDX stock list as `daftar_saham.xlsx` instead (about 10–20 minutes).
3. Select *Runtime → Run all*.

**Local Python**

```bash
git clone <repository-URL>
cd <repository-folder>
pip install -r requirements.txt
unzip data_manip.zip
jupyter notebook Manipulation_detection_notebook.ipynb
```

**Output files (`results/`)**

| File | Content |
| --- | --- |
| `table2_auc.csv` | Within-stock AUC per episode and score (Table 3 of the article) |
| `table3_rank.csv` | Case vs control mean score and percentile rank |
| `table4_detection.csv` | Alarm threshold, days to first alarm, alarm rates |
| `summary.csv` | Summary across episodes (Table 2) |
| `paired_comparison.csv` | Paired Wilcoxon tests, wavelet score vs simple rule (Table 4) |
| `summary_twosided.csv` | Summary with two-sided (non-causal) wavelet features |
| `controls.csv` | The five matched controls for each episode |

## Requirements

Python 3.10 or later; numpy, pandas, matplotlib, PyWavelets, scikit-learn, SciPy, yfinance, openpyxl. See `requirements.txt`. Runtime: a few minutes with `data_manip.zip`, or 10–20 minutes with a fresh download on Google Colab.

## Citation

> Suryopratomo A. Do wavelet features help detect stock price manipulation? Evidence from enforcement cases on the Indonesia Stock Exchange. *Scientific Journal of Informatics* (submitted).

## License

Code: MIT License (see `LICENSE`). Price data originate from Yahoo Finance and are provided for research reproducibility only; they remain subject to Yahoo's terms of use. Labels are taken from public OJK announcements.

## Contact

Anggit Suryopratomo – Department of Digital Business, Universitas Ma'soem, Sumedang, West Java, Indonesia – suryopratomo.anggit@gmail.com
