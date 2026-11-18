# IoT-23 Network Intrusion Detection Analysis

An end-to-end analysis of real IoT network traffic, asking whether connection-flow
characteristics can separate malicious traffic from benign traffic — and what the
answer depends on.

**[Read the full report](https://aniinloop.github.io/IoT-23-Network-Intrusion-Detection-Analysis/iot23-analysis.html)**

## Data

The analysis uses the **IoT-23** dataset from the Stratosphere Laboratory at the
Czech Technical University in Prague, scenario `CTU-IoT-Malware-Capture-34-1`:
network traffic captured from a Raspberry Pi infected with Mirai malware.

The data are read **directly from the public repository at run time**, so no data
file is stored here and the analysis reproduces from source:

```
https://mcfp.felk.cvut.cz/publicDatasets/IoT-23-Dataset-v2/CTU-IoT-Malware-Capture-34-1/
```

| | |
|---|---|
| Flows | 23,145 |
| Variables | 25 (Zeek connection log fields) |
| Target | `attack_status` — Benign or Malicious |
| Class balance | 91.7% malicious, 8.3% benign |

## What the analysis does

1. **Inspection** — variable types, missing values, skewness, class balance.
2. **Exploratory analysis** — protocol and connection-state distributions, flow
   duration, packet activity and traffic volume by class, and the relationship
   between packet count and byte volume.
3. **Preprocessing** — median imputation for numeric fields, an explicit `Unknown`
   category for missing categorical fields, and IQR outliers retained rather than
   removed, since unusually large or long flows can be genuine attack behaviour.
4. **Feature engineering** — seven new features covering total bytes, total packets,
   bytes per packet, log-transformed duration and volume, and time of day.
5. **Feature selection** — one-hot encoding expands 20 predictors to 29; a
   near-zero-variance filter reduces these to 16, and a correlation filter at
   r ≥ 0.90 reduces them to 12.
6. **Leakage prevention** — unique IDs, source and destination IP addresses, the
   original label columns and the raw timestamp are excluded from the predictor set.
7. **PCA** — six components explain 85.6% of predictor variance. The models are
   nonetheless trained on the original selected features, because a principal
   component mixes several network measurements and cannot be explained to a
   security analyst.
8. **Modelling** — Logistic Regression as a baseline against a tuned Random Forest
   (200 trees), under a stratified 80/20 split with 5-fold cross-validation.
9. **Evaluation** — accuracy, precision, recall, F1 and ROC-AUC, judged against
   success criteria fixed before modelling began.

## Results, and an open question

Both models score **1.000 on every metric** on the held-out test set.

That is not presented here as a success. A perfect score from a linear model on real
network traffic almost always means a feature is standing in for the label rather
than predicting it. Two specific reasons to be suspicious of this result:

- The surviving features include `conn_state.S0` — a connection attempt that
  received no reply. In a Mirai capture that is close to a direct description of
  scanning behaviour, which is what the malicious class consists of.
- The majority class is 91.7% of the data, so a model that answered "malicious"
  every time would score 91.7% accuracy before learning anything.

**Open work:** re-run with `conn_state` removed from the predictor set, then with the
port columns removed as well, and record how ROC-AUC moves at each step. The size of
that drop is the real finding, and this README will be updated with it.

## Reproducing the analysis

Requires R. Open `iot23-analysis.Rmd` in RStudio and knit it; the setup chunk installs
any missing packages (tidyverse, janitor, naniar, skimr, e1071, ggcorrplot, caret,
knitr, kableExtra, factoextra, randomForest, pROC). The random seed is fixed at 123,
so results are reproducible. An internet connection is needed, since the data are
fetched from the CTU repository at run time.

## Files

| File | Contents |
|---|---|
| `iot23-analysis.Rmd` | Full analysis source |
| `iot23-analysis.html` | Knitted report with all figures and tables |

## Limitations

The analysis covers a single malware scenario, so the results describe Mirai behaviour
on one device rather than intrusion detection in general. The class balance reflects a
capture environment, not an operational network. The models use connection-flow
features rather than packet payloads, which limits what can be detected but also keeps
the analysis free of packet contents.

## Authors

Atkiya Anisa Latifi, Fahad Ibne Firoz, Sahed Ahmed

