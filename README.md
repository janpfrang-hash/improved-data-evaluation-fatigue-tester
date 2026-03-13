# improved-data-evaluation-fatigue-tester


# Fatigue Tester Log Analyzer: User Manual

This document provides a comprehensive guide on how to use the **Wöhler-Kurven Datenauswertung** Google Colab program. This tool is specifically designed to parse, analyze, and visualize log files from fatigue testing machines (Voice Coil Test Stands). 

*(Insert image of a typical S-N Wöhler fatigue testing curve here)*

It features advanced statistical evaluations, machine parameter decoding, DIN 50100 compliance checks, and a highly optimized PDF export tailored for fast printing.

---

## 1. Getting Started: The Basic Workflow

The user interface is divided into five sequential steps. Follow these steps to process your fatigue test data.

### Step 1: Load Files (`1️⃣ Dateien laden`)
Click the green **📁 Dateien auswählen** button to upload your log files. 
* You can select one or multiple log files simultaneously.
* The tool will automatically parse the file, count the data points, and attempt to decode the machine's configuration header (`CNF`).
* Progress and parsing success will be displayed in the output box below the button.

### Step 2: Select Files (`2️⃣ Dateien auswählen`)
Once uploaded, your files will appear as a list of checkboxes. 
* Check the boxes next to the files you want to include in the current analysis.
* If you select multiple files, the program will automatically generate comparison plots and tables, color-coding the different datasets.

### Step 3: Select Parameters (`3️⃣ Graphen auswählen`)
Choose which data parameters you want to analyze and visualize. The parameters are grouped into logical categories:
* **Messdaten (Raw Data):** Position, Force (Kraft 1/F_min, Kraft 2/F_max), and Displacement (Weg).
* **Wöhler-Parameter:** R-Wert (Stress ratio), Kraftamplitude (Force amplitude F_a), and Mittelkraft (Mean force F_m).
* **Steifigkeit (Stiffness):** Tracks the percentage of stiffness loss over the cycles. You can also toggle visual markers for 10%, 20%, and 50% stiffness loss criteria.

### Step 4: Data Processing Options
* **Daten glätten (Smooth Data):** Check this box to apply a rolling average (window size = 50) to the data. This helps filter out high-frequency noise and makes underlying trends much easier to see.

### Step 5: Generate Output (`4️⃣ Plots erstellen` & `5️⃣ Visualisierung`)
Use the action buttons at the bottom of the UI to generate your desired analysis. 

* **📊 Plots erstellen:** Generates standard time-series plots over the test cycles for all selected parameters.
* **📉 Q-Q Plots:** Generates Quantile-Quantile plots and histograms to check data distribution.
* **📈 Drift-Analyse:** Calculates and graphs the linear trend of selected parameters over time.
* **🔧 Maschinenparameter:** Decodes and displays the machine setup and DIN 50100 evaluation.
* **🖨️ PDF (Druck-optimiert):** Compiles all selected analyses into a single, high-performance PDF.

---

## 2. Deep Dive: Key Analytical Features

### Machine Parameters & DIN 50100 Check
By clicking the **🔧 Maschinenparameter** button, the tool reads the `CNF` header from your log files.

* **Configuration Display:** It reveals target forces, tolerance bands, emergency cutoffs, and device IDs. 
* **Comparison Mode:** If multiple files are selected, it generates a side-by-side comparison table, highlighting any discrepancies in the machine setup in red.
* **DIN 50100 Compliance:** The program automatically checks if the maximum force (F_max / Kraft 2) remains within the ±3% tolerance required by the DIN 50100 standard. It evaluates two criteria:
  1. **3-Sigma Check:** Is the 3-standard-deviation spread of F_max within the 3% limit?
  2. **10-Cycle Rolling Average:** Is the maximum deviation of a 10-cycle moving average within the 3% limit?

### Drift Analysis
The **📈 Drift-Analyse** module identifies systematic changes (drifting) in your tester's output over millions of cycles.

* It performs a linear regression on parameters like Force, R-Value, and Force Amplitude.
* **Metrics Provided:** You receive the absolute drift, relative drift percentage, and a standardized "Drift per 1,000 Cycles" rate.
* **Statistical Significance:** The tool evaluates the R² value and p-value to tell you if the trend is "Strong", "Medium", "Weak", or "Not Significant".

### Statistical Evaluation & Q-Q Plots
Clicking **📉 Q-Q Plots** provides a deep statistical view of your test data.

*(Insert image of a Q-Q plot showing normal distribution here)*

* **Normality Testing:** It generates a histogram alongside a Q-Q (Quantile-Quantile) plot. If the blue dots follow the red diagonal line, your data is normally distributed.
* **Automated Tests:** The program runs Shapiro-Wilk (for <5000 points) or Anderson-Darling tests to objectively confirm normal distribution.
* **Summary Tables:** Every standard plot includes a data table underneath showing Min, Max, Mean, Standard Deviation, Variance, ±3σ limits, and sensor resolution.

### Print-Optimized PDF Export
A major feature of this script is solving the "slow PDF printing" issue common with high-density scatter plots in Adobe Acrobat. 

* Click **🖨️ PDF (Druck-optimiert)** to generate a report.
* **How it works:** Instead of saving millions of individual vector points (which crashes PDF viewers), the program rasterizes the plots at 150 DPI and removes transparencies.
* **Result:** The document looks crisp on paper but renders 10x to 20x faster when sent to a Windows PDF printer. The PDF will include the Machine Parameters, Statistics Text, Standard Plots, Q-Q Plots, Drift Analysis, and a Final Summary Table based strictly on what checkboxes you activated in Step 3.

---

## 3. Performance Notes & Troubleshooting

* **Large Datasets:** The script utilizes an intelligent downsampling algorithm (`MAX_PLOT_POINTS = 5000`). It automatically reduces the visual data points to keep the Colab interface responsive without losing the shape of the Wöhler curve. The statistical calculations, however, are *always* performed on the full, raw dataset.
* **Missing CNF Warnings:** If you see a `⚠️ Kein CNF-Header gefunden` warning during upload, your specific log file does not contain the machine configuration block. The DIN 50100 check and Machine Parameter tables will be skipped for that file, but all raw data plots will still function normally.
