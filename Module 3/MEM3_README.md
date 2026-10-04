# Module 3 — Limitation Analysis and Concept Drift Simulation

## Member 

**Name:** Abhigyan Gaurav  
**Role:** Member 3 — Limitation Analysis & Concept Drift Simulation  
**GitHub Username:** ABHI2005-web

---

## Objective

The objective of Module 3 is to analyse an important limitation of the baseline DeepLog model.

DeepLog learns normal log-event sequences from the training data. If a legitimate normal log pattern is not present during training and later appears during testing, the model may incorrectly identify that normal behaviour as an anomaly.

This module performs a controlled experiment to study this false-positive behaviour.

---

## Experimental Approach

The experiment was performed using the processed HDFS log dataset and the baseline DeepLog methodology.

The main steps were:

1. Verified the processed HDFS dataset containing **575,061 log sequences**.
2. Analysed the frequency of the **34 log templates**.
3. Selected **Log Key 6** as the held-out normal template.
4. Verified the mapping between the Module 3 log-key representation and the baseline DeepLog event representation:
   - **Log Key 6 → Event ID E6**
5. Reproduced the baseline **80:20 train-test split**.
6. Identified normal sequences containing E6.
7. Removed all normal E6 sequences from the training data.
8. Verified that E6 was completely unseen during model training.
9. Trained an E6-held-out DeepLog LSTM model using the remaining normal training sequences.
10. Tested the trained model separately on:
    - Seen normal sequences
    - Unseen E6 normal sequences
11. Compared their false-positive rates using **Top-5 next-event prediction**.

---

## Dataset Split

The complete dataset contains:

- **Total sequences:** 575,061
- **Training sequences:** 460,048
- **Testing sequences:** 115,013

### Normal Training Data

- Original normal training sequences: **446,578**
- Normal training sequences containing E6: **1,443**
- Remaining normal training sequences after removing E6: **445,135**

### Normal Testing Data

- Total normal test sequences: **111,645**
- Seen normal test sequences: **111,310**
- Unseen E6 normal test sequences: **335**

The 1,443 normal E6 training sequences were removed so that E6 would remain unseen during training.

---

## Held-Out Template

The selected held-out template was **Log Key 6**.

Its template is:

`DATE NUM NUM INFO dfs.DataNode$DataXceiver: Received block BLK src: IP dest: IP of size <*>`

The mapping analysis verified:

**Log Key 6 → E6**

Therefore, E6 was used as the held-out event in the DeepLog experiment.

---

## DeepLog Configuration

The E6-held-out DeepLog model was trained using the following configuration:

- **Window size:** 20
- **Top-K:** 5
- **Embedding dimension:** 32
- **Hidden size:** 64
- **LSTM layers:** 2
- **Batch size:** 256
- **Optimizer:** Adam
- **Learning rate:** 0.001
- **Epochs:** 5

A total of **573,302 training windows** were generated.

Before training, verification confirmed:

- Training windows containing E6: **0**
- Target events equal to E6: **0**

Therefore, E6 was completely unseen during model training.

---

## Training Result

The training loss decreased across the five epochs:

| Epoch | Training Loss |
|---:|---:|
| 1 | 0.2154 |
| 2 | 0.1161 |
| 3 | 0.1104 |
| 4 | 0.1082 |
| 5 | 0.1068 |

The decreasing loss shows stable learning during training.

---

## Evaluation Method

The trained model was evaluated using **Top-5 next-event prediction**.

For each eligible sequence:

1. The previous **20 log events** were given to the LSTM.
2. The model predicted the Top-5 most likely next events.
3. The actual next event was compared with these predictions.
4. If the actual event was outside the Top-5 predictions, the window was considered anomalous.
5. If a normal block contained an anomalous window, the block was counted as a false positive.

The seen-normal and unseen-E6 normal groups were evaluated separately.

---

## Experimental Results

| Test Group | Eligible Blocks | False Positives | False-Positive Rate |
|---|---:|---:|---:|
| Seen Normal | 26,483 | 0 | 0.00% |
| Unseen E6 Normal | 302 | 70 | 23.18% |

The false-positive rate increased by **23.18 percentage points** for the unseen E6 normal group.

---

## Key Finding

The experiment produced a clear difference between seen and unseen normal behaviour.

### Seen Normal Behaviour

- Eligible blocks: **26,483**
- False positives: **0**
- False-positive rate: **0.00%**

### Unseen E6 Normal Behaviour

- Eligible blocks: **302**
- False positives: **70**
- False-positive rate: **23.18%**

This controlled experiment demonstrates that the E6-held-out DeepLog model can incorrectly flag legitimate normal behaviour when that behaviour contains a pattern that was unseen during training.

This result provides evidence for the limitation being studied in Module 3 and motivates the later adaptation/improvement stage of LogSentry.

---

## Module 3 Workflow

```text
Processed HDFS Data
        ↓
Dataset Verification
        ↓
Template Frequency Analysis
        ↓
Select Log Key 6
        ↓
Verify Log Key 6 → E6 Mapping
        ↓
Reproduce Baseline Train-Test Split
        ↓
Remove E6 from Normal Training Data
        ↓
Verify E6 is Unseen During Training
        ↓
Train E6-Held-Out DeepLog Model
        ↓
Evaluate Seen Normal Data
        ↓
Evaluate Unseen E6 Normal Data
        ↓
Compare False-Positive Rates
        ↓
Identify DeepLog Limitation
