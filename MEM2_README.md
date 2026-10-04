# LogSentry — Member 2: DeepLog Baseline Reproduction

## Overview

This branch contains the **DeepLog baseline reproduction** for the LogSentry capstone project.

The purpose of this work is to establish a baseline anomaly detection system using an LSTM-based implementation of DeepLog. The baseline is used as a reference point for later experiments and improvements in the LogSentry project.

## Member 2 Responsibilities

The main responsibilities for Member 2 are:

- Set up the PyTorch environment.
- Prepare the parsed HDFS log event sequences.
- Reproduce the DeepLog LSTM baseline.
- Train the model on normal log sequences.
- Perform next-event prediction using Top-K predictions.
- Detect anomalous events when the actual next event is not among the Top-K predictions.
- Evaluate the baseline using classification metrics.
- Generate the training loss curve.
- Generate the confusion matrix.
- Save the trained baseline model and evaluation results.

## Dataset

The baseline uses the **HDFS log dataset**.

The parsed data used by this implementation is represented as event sequences such as:

```text
[E5, E22, E5, E22, ...]
