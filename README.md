# improved-data-evaluation-fatigue-tester

![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/janpfrang-hash/improved-data-evaluation-fatigue-tester/blob/main/Improved_Data_evaluation_fatigue_tester.ipynb)

# Fatigue Testing Data Evaluation & ISO 12107 S-N Curve Analysis

A modular, Google Colab-ready Python toolset for processing cyclic fatigue test data from voice coil test stands, performing DIN 50100 conformity checks, and generating S-N (Wöhler) curves with statistical verification in accordance with **ISO 12107:2012**.

---

## Table of Contents
1. [Overview](#overview)
2. [Key Features](#key-features)
   - [Part 1: Multi-File Logger Processing & DIN 50100 Verification](#part-1-multi-file-logger-processing--din-50100-verification)
   - [Part 2: ISO 12107 S-N Curve Generator GUI](#part-2-iso-12107-s-n-curve-generator-gui)
3. [Prerequisites & Dependencies](#prerequisites--dependencies)
4. [Step-by-Step User Guide (Google Colab)](#step-by-step-user-guide-google-colab)
5. [Input Data Specifications](#input-data-specifications)
6. [Statistical Methodology & Standards Compliance](#statistical-methodology--standards-compliance)
7. [Troubleshooting & FAQs](#troubleshooting--faqs)

---

## Overview

This repository provides an automated data pipeline for fatigue durability testing:
1. **Raw Log Processing:** Ingests raw multi-cycle test stand logs (`DTA;` / CSV), monitors force stability against **DIN 50100** criteria, tracks stiffness degradation, and compiles print-optimized PDF summary reports.
2. **S-N Curve Evaluation:** Takes the resulting summary report (or any logger overview PDF), extracts specimen metrics, fits a log-linear Wöhler model, estimates scatter with horizontal Gaussian distributions, computes lower tolerance bounds, and executes a live **ISO 12107 Data Validation & Usability Check**.

---

## Key Features

### Part 1: Multi-File Logger Processing & DIN 50100 Verification
* **Automated Log Parsing:** Parses configuration blocks (`CNF;`) and measurement rows (`DTA;`), correctly accounting for logging downsampling intervals (e.g. 10× cycle factors).
* **DIN 50100 Force Stability Criteria:** Evaluates test validity using the normative \\(\pm 3\%\\) allowable force deviation limit:
  1. **\\(3\sigma\\) Criterion:** Confirms that \\(3 \cdot s(F_{\max}) \le 0.03 \cdot F_{\text{target}}\\).
  2. **Cycle Deviation Criterion:** Confirms that peak force variations during cyclic intervals do not exceed the \\(3\%\\) tolerance.
* **Stiffness Degradation Tracking:** Calculates dynamic force stroke over travel stroke (\\(c = \Delta F / \Delta s\\)), tracks normalized stiffness, and records cycles to \\(10\%\\), \\(20\%\\), and \\(50\%\\) stiffness loss.
* **Drift Analysis:** Linear regression over time/cycles to detect transducer drift, settling, or machine compliance loss.
* **Print-Optimized PDF Export:** Produces vector-raster balanced PDF reports formatted for high-speed printing without PDF raster stalls.

### Part 2: ISO 12107 S-N Curve Generator GUI
* **Multi-Page PDF Parsing:** Extracts sample rows from `DATA OVERVIEW` tables across line wraps, deduplicating copy suffixes (e.g. `(1)`, `(2)`) down to unique specimens.
* **Log-Linear Least Squares Fit:** Computes \\(\log_{10}(N) = b_0 + b_1 \log_{10}(F)\\), extracting the positive slope exponent \\(k = -b_1\\) and fatigue constant \\(C = 10^{b_0}\\).
* **t-Distribution Confidence Intervals:** Computes exact two-sided \\(95\%\\) confidence bounds for the Wöhler slope exponent \\(k\\).
* **ISO 12107 One-Sided Tolerance Limit:** Calculates the lower design bound for a \\(10\%\\) failure probability at a \\(95\%\\) confidence level (\\(p=0.10, 1-\alpha=0.95\\)) using non-central \\(t\\)-distribution tolerance factors.
* **Horizontal Scatter Distributions:** Renders fitted normal density curves along the cycle axis at each load horizon.
* **\\(R_{\text{avg}}\\) Range Display:** Computes and displays the average stress ratio range (\\(R_{\text{avg}} = F_{\min} / F_{\max}\\)) across the plotted horizons.
* **Embedded Validation Panel:** Adds a visual pass/fail validation block at the bottom of the graph assessing sample size, load horizons, \\(R^2\\), normality, and outliers.
* **Direct High-Res PNG Download:** Includes a one-click button to download the 200 DPI publication-ready graphic.

---

## Prerequisites & Dependencies

The scripts run in Python 3.10+ (tested in Google Colab). Required packages:
* `numpy` (array mathematics and regression sums)
* `scipy` (Student’s \\(t\\), non-central \\(t\\), and Shapiro-Wilk statistical distributions)
* `matplotlib` (graphical generation and PDF backend rendering)
* `pypdf` (PDF text extraction and table token parsing)
* `ipywidgets` (interactive graphical user interface elements)
* `pandas` (tabular time-series and logfile operations)

---

## Step-by-Step User Guide (Google Colab)

### 1. Launch Environment
Open a new notebook in [Google Colab]([suspicious link removed]).

### 2. Install Packages
Run the installation cell:
```python
!pip install pypdf scipy matplotlib numpy ipywidgets -q
```

### 3. Generate the ISO 12107 S-N Curve
Paste and run the **Cell 2** script:
1. Click **Select PDF** to upload your logger summary document (e.g. `SN_data_QTR_4481_print.pdf` or `QTR4147_HB901_SN_data_new_samples.pdf`).
2. Choose your evaluation basis under **Evaluate on:**:
   * **Nominal Force Horizons** (e.g. \\(50\,\text{N}\\), \\(60\,\text{N}\\), \\(70\,\text{N}\\)) — *Recommended for standard S-N curves*.
   * **Actual Measured Peak Force (\\(F_{\max}\\))** — *Uses individually recorded specimen forces*.
3. Click **Create S-N curve**.
   * The console outputs the extracted specimen table and the ISO 12107 verification report.
   * The S-N curve displays inline with statistical parameters, \\(R_{\text{avg}}\\) range, and the validation panel.
4. Click **Download Curve (PNG)** to save the 200 DPI graphic directly to your local drive.

---

## Input Data Specifications

### Raw Log Files (`.csv` / `.txt`)
Expected formatting for raw test stand ingestion:
* **Header Block:** Contains configuration parameters starting with `CNF;` (including nominal force `kraft2_N`, target tolerance `toleranz_pct`, and shutdown thresholds).
* **Data Rows:** Delimited rows starting with `DTA;` followed by cycle counts, positions, forces, and error codes:
  ```text
  DTA;Cycles;Position_1;Force_1;Travel_1;Position_2;Force_2;Travel_2;Travel_Stroke;ErrorCode
  ```

### Summary PDF Files (`.pdf`)
Expected format for the S-N Curve Generator GUI:
* Must contain a standard summary table (e.g. `DATA OVERVIEW`) listing:
  * Specimen identifier (e.g. `..._70N (1).csv`)
  * Minimum force \\(F_{\min}\\) [N]
  * Maximum peak force \\(F_{\max}\\) [N]
  * Stress ratio \\(R\\)
  * Failure cycles \\(N\\)

---

## Statistical Methodology & Standards Compliance

The calculations implemented in the code adhere strictly to the following normative standards:

| Standard & Clause | Analysis Requirement | Implementation in Script |
| :--- | :--- | :--- |
| **ISO 12107 §8.2.1** | Log-linear least squares fit | \\(\log_{10}(N) = b_0 + b_1 \log_{10}(F)\\); \\(k = -b_1\\); \\(C = 10^{b_0}\\) |
| **ISO 12107 §8.2.1** | Coefficient of determination | Evaluates \\(R^2 \ge 0.90\\) for acceptable goodness of fit |
| **ISO 12107 §8.3.3 / §6.4** | Normality of residuals | Evaluates log-normal life scatter via Shapiro-Wilk test on regression residuals (\\(p \ge 0.05\\)) |
| **ISO 12107 §8.3.2** | Outlier detection | Standardized residuals \\(z = (Y - \hat{Y}) / s\\); flags points where \\(\lvert z \rvert > 3.0\sigma\\) |
| **ISO 12107 §8.4 & Table B.1** | Lower tolerance limit | Calculates non-central \\(t\\)-factor \\(k_{\text{tol}}\\) for \\(10\%\\) failure probability at \\(95\%\\) confidence (\\(p=0.10, 1-\alpha=0.95\\)) |
| **ISO 12107 §8.5** | Sample size guidelines | Verifies \\(n \ge 10\\) for preliminary/exploratory curves; notes requirement of \\(n \ge 30\\) for structural design curves |
| **DIN 50100:2022 §6** | Load ratio consistency | Tracks mean stress ratio across load horizons and reports the \\(R_{\text{avg}}\\) range |
| **DIN 50100:2022 §11.5** | Force stability during testing | Enforces a \\(\pm 3\%\\) allowable deviation envelope on peak load \\(F_{\max}\\) |

---

## Troubleshooting & FAQs

#### 1. Why does the console report `n = 33` instead of `n = 11`?
* This occurs when the PDF parser reads secondary summary tables (such as travel or \\(R\\)-value tables) in multi-page documents. 
* **Fix:** The parser filters out non-force tables (\\(F_{\max} < 20\,\text{N}\\)) and strips trailing copy indicators (`(1)`, `(2)`) down to the base specimen name.

#### 2. Why does `TypeError: unhashable type: 'list'` appear during file upload?
* In newer versions of `ipywidgets` (v8+), uploaded files are stored as tuples/lists of dictionaries rather than flat dictionaries. 
* **Fix:** The code uses direct iterator unpacking (`next(iter(uploaded_item))`), making it compatible with all widget versions.

#### 3. Does the S-N curve apply to \\(R = 0\\) (pulsating bending)?
* **No.** The test data reflects the actual test condition (\\(R \approx 0.15\text{–}0.16\\)). The script calculates the average stress ratio across horizons and displays it as \\(R_{\text{avg}}\\). If \\(R=0\\) or \\(R=-1\\) equivalent curves are needed, mean stress transformations (e.g. Goodman or FKM) must be applied prior to regression.
