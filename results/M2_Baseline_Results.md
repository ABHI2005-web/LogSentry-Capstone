# M2 — DeepLog Baseline Reproduction

## Dataset

- Dataset: HDFS
- Total parsed blocks: 575,061
- Unique event IDs: 29
- DeepLog window size: 20
- Total generated windows: 788677
- Training windows: 630941
- Testing windows: 157736

## Model Configuration

- Model: DeepLog LSTM
- Embedding size: 32
- Hidden size: 64
- LSTM layers: 2
- Batch size: 256
- Learning rate: 0.001
- Epochs: 5
- Optimizer: Adam
- Loss function: CrossEntropyLoss
- Top-K: 5
- Device: cpu

## Evaluation Results

| Metric | Value |
|---|---:|
| Accuracy | 0.9314 |
| Precision | 0.9713 |
| Recall | 0.0644 |
| F1 Score | 0.1208 |

## Confusion Matrix

|  | Predicted Success | Predicted Fail |
|---|---:|---:|
| Actual Success | 146165 | 22 |
| Actual Fail | 10805 | 744 |

## Training

The DeepLog LSTM was trained using only training windows associated
with normal (Success) sequences.

The training loss curve is saved as:

`training_loss_curve.png`

## Evaluation

DeepLog identifies a window as anomalous when the actual next event
does not appear among the Top-5 predicted events.

The confusion matrix is saved as:

`confusion_matrix.png`

## Important Note

These results represent the current baseline implementation in the
LogSentry project. The current notebook uses window-level evaluation.
Therefore, these numbers should not be described as an exact
paper-level reproduction unless the dataset split and evaluation
protocol are confirmed to match the original DeepLog experiment.

## Generated Artifacts

- `deeplog_baseline.pth`
- `deeplog_metrics.csv`
- `confusion_matrix.csv`
- `training_loss_curve.png`
- `confusion_matrix.png`
- `M2_Baseline_Results.md`
