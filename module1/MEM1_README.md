# Module 1 — Log Ingestion & Parsing
### LogSentry Capstone — B.Tech Computer Science Engineering (Final Year)

**Author:** Akshat Pandey  
**University:** VIT Bhopal University  
**Team:** 28  
**Role:** Log Ingestion & Parsing / Data Pipeline Lead  
**GitHub:** `mem1-team28`  
**Supervisor:** Dr. Dheresh Soni  
**Project:** LogSentry — Incremental Vocabulary Adaptation for DeepLog  
**Base paper:** *DeepLog: Anomaly Detection and Diagnosis from System Logs through Deep Learning* (Du et al., CCS 2017)

---

## 1. What This Module Does

This module reproduces the **log-parsing stage** of DeepLog on the HDFS_v1 dataset and produces clean inputs for the LSTM training module.

### Pipeline

1. **Extract** the raw HDFS log file (11,175,629 lines) from the shared dataset.
2. **Parse** every line with Drain3 into **34 unique log templates**.
   Block IDs, IP addresses, numbers, and dates are masked with regex before parsing.
3. **Group** parsed events by HDFS block to build **575,061 sequences** of log keys.
4. **Export** sequences, labels, and metadata for the LSTM module.

The final output has a mean sequence length of **19.43**, compared with the **19.4** reported in the DeepLog paper, and uses a vocabulary size of **34** compared with the paper's **29**.

---

## 2. Repository Layout

This module lives inside the team repository `LogSentry-Capstone`. The repo-level `README.md` is maintained by the team leader; this file documents **only Module 1**.

```text
LogSentry-Capstone/                        (repo root)
├── README.md                              ← team-level README
└── module1/                               ← Module 1 (this module)
    ├── README.md                          ← this file
    ├── .gitignore
    │
    ├── code/
    │   ├── parsing_pipeline.ipynb         ← clean, reproducible pipeline (deliverable)
    │   └── parsing_pipeline_dev.ipynb     ← development notebook (evidence of iterations)
    │
    └── data/
        ├── raw/
        │   ├── HDFS_v1.zip.download.txt   ← pointer to the raw dataset (file itself NOT in repo)
        │   └── anomaly_label.csv          ← block-level ground truth (17.8 MB)
        │
        └── processed/
            ├── deeplog_input.txt          ← LSTM input sequences (23 MB)
            ├── deeplog_labels.txt         ← ground truth labels (17 MB)
            ├── templates.csv              ← log_key → template mapping
            ├── vocab_info.txt             ← vocabulary metadata
            └── parsed_logs.csv.download.txt ← pointer to full line-level parse (file itself NOT in repo)
```

### Why some files are only pointers

GitHub rejects any file larger than **100 MB**, so two large files cannot be stored in this repository:

| Large file | Size | In GitHub repo | Where the real file is |
|---|---:|---|---|
| `data/raw/HDFS_v1.zip` | 178 MB | Pointer only: `HDFS_v1.zip.download.txt` | Google Drive link inside the pointer file (view-only access) |
| `data/processed/parsed_logs.csv` | 368 MB | Pointer only: `parsed_logs.csv.download.txt` | Google Drive link inside the pointer file (view-only access) |

Each `.download.txt` file is a small text file that tells you where to download the real file and where to place it. Everything else in `module1/` is stored directly in the repo.

> **Note:** For LSTM training you do **not** need either large file. `deeplog_input.txt` and `deeplog_labels.txt` are in the repo and are all that Module 2 normally needs.

---

## 3. Data Files — Full Description

### `deeplog_input.txt` — LSTM INPUT

- Size: ~23 MB
- Format: one line per HDFS block, with space-separated integer log keys
- Line count: **575,061**
- Each line represents one ordered sequence of events for a block.
- **In repo:** yes

Example:

```text
4 1 3 3 4 1 5 5 5 3 4 3 4 3 4 3 4 3 4 3 4
1 2 1 1 3 4 5 3 4 3 4 5 3 4 3 4 3 4 5 3 4
1 1 1 2 3 4 3 4 5 5 5 3 4 3 4 3 4 3 4 3
```

### `deeplog_labels.txt` — GROUND TRUTH

- Size: ~17 MB
- Format: CSV with header: `block_id,label`
- Label values: `Normal` or `Anomaly`
- **In repo:** yes
- **Critical:** line N of this file (after the header) corresponds to line N of `deeplog_input.txt`. The two files are aligned and should be loaded together.

Example:

```text
block_id,label
blk_-1608999687919862906,Normal
blk_7503483334202473044,Normal
blk_-3544583377289625738,Anomaly
```

### `templates.csv` — HUMAN-READABLE DICTIONARY

- Size: ~4 KB
- Format: CSV with `log_key,template`
- Use this when debugging or interpreting a sequence.
- It maps each numeric `log_key` back to the log template it represents.
- **In repo:** yes

Example:

```text
log_key,template
1,"DATE NUM NUM INFO dfs.DataNode$DataXceiver: Receiving block BLK src: IP dest: IP"
3,"DATE NUM NUM INFO dfs.DataNode$PacketResponder: PacketResponder NUM for block BLK <*>"
4,"DATE NUM NUM INFO dfs.DataNode$PacketResponder: Received block BLK of size NUM from IP"
```

### `vocab_info.txt` — METADATA

- **In repo:** yes

```text
vocab_size=34
num_templates=34
num_sequences=575061
num_blocks_total=575061
mean_sequence_length=19.43
max_sequence_length=298
```

### `anomaly_label.csv` — RAW BLOCK-LEVEL LABELS

- Size: ~17.8 MB
- Block-level ground truth from the HDFS_v1 dataset, used to build `deeplog_labels.txt`.
- **In repo:** yes

### `HDFS_v1.zip` — RAW DATASET (pointer only in repo)

- Size: ~178 MB
- Contains the raw HDFS log file (11,175,629 lines).
- **In repo:** only `HDFS_v1.zip.download.txt`, which points to the real file.
- Needed only if you want to re-run the parsing pipeline from scratch.

### `parsed_logs.csv` — RAW PARSE REFERENCE (pointer only in repo)

- Size: ~368 MB
- Format: `line_number,log_key,block_id`
- Contains one row per original log line.
- Mainly useful for auditing and debugging.
- **In repo:** only `parsed_logs.csv.download.txt`, which points to the real file.
- The LSTM module normally does **not** need this file.

---

## 4. Code Files

### `parsing_pipeline.ipynb` — THE DELIVERABLE

- Clean, reproducible end-to-end pipeline (25 cells)
- Runs top-to-bottom in Google Colab
- Uses Drain3 with:
  - `sim_th=0.4`
  - `depth=4`
  - `max_children=100`
- Uses regex masking for dates, block IDs, IP addresses, and numbers
- Reproduces the processed outputs from the raw data
- This is the notebook to use for reproduction, submission, and reference

### `parsing_pipeline_dev.ipynb` — DEVELOPMENT EVIDENCE

This notebook (46 cells) contains the iterative development process:

- **Attempt 1:** raw parse → 157 templates
- **Attempt 2:** regex masking added → 96 templates
- **Attempt 3:** fixed `cluster_id` vs `template_string` issue → **34 templates**

Use this notebook as evidence for the development process and for the "Difficulties Faced" section of the project report.

It is **not intended to be the main reproduction notebook**.

---

## 5. Getting Started

Pick the path that matches what you need.

### Path A — I only need the processed data (Module 2 / LSTM training)

No large downloads required.

```bash
git clone <LogSentry-Capstone repo URL>
cd LogSentry-Capstone/module1/data/processed
```

Use `deeplog_input.txt` and `deeplog_labels.txt` directly (see Section 6 for a loader), and `templates.csv` / `vocab_info.txt` for reference.

### Path B — I need one of the large files (`HDFS_v1.zip` or `parsed_logs.csv`)

1. Open the matching pointer file:
   - `module1/data/raw/HDFS_v1.zip.download.txt`
   - `module1/data/processed/parsed_logs.csv.download.txt`
2. Follow the instructions inside to download the file.
3. Place it in the **same folder** as its pointer file, keeping the exact file name (`HDFS_v1.zip` or `parsed_logs.csv`).

> `.gitignore` is set up so these large files are not committed. Do not try to `git add` them. GitHub will reject anything over 100 MB.

### Path C — I want to re-run the full parsing pipeline

The pipeline is designed to run in Google Colab and reads the raw dataset (`HDFS_v1.zip`) from Google Drive.

**Prerequisites**

- Google Colab
- `HDFS_v1.zip` available in your Google Drive (download it from the pointer file, see Path B)
- Approximately 20 minutes
- Approximately 4 GB RAM
- CPU runtime is sufficient; GPU is not required

> **Where the notebook looks for the zip:** The pipeline expects the file at
> `MyDrive/LogSentry_Capstone/01_Dataset/HDFS_v1.zip`. If your Drive has a
> different structure, either move the zip to that path, or edit the
> `ZIP_PATH` variable in the Configuration cell (Cell 6 of
> `parsing_pipeline.ipynb`).

**Steps**

1. Open `module1/code/parsing_pipeline.ipynb` in Colab.
2. Select **Runtime → Change runtime type → CPU**.
3. Run all cells from top to bottom.
4. Approve the Google Drive mount request when prompted.
5. Wait for the pipeline to finish.
6. Processed outputs are written to `MyDrive/module1/data/processed/`.

> If you copy new outputs into the GitHub repo, remember that `HDFS_v1.zip` and `parsed_logs.csv` stay out of git.

**Approximate runtime**

| Step | Time |
|---|---:|
| Mount + install | ~30 sec |
| Extract HDFS.log | ~45 sec |
| Full parse (11M lines) | ~6 min |
| Group by block | ~30 sec |
| Write + copy outputs | ~15 sec |
| Verification | ~10 sec |
| **Total** | **~8 min first run** |

On later runs, extraction may be skipped when the local copy already exists.

---

## 6. For Module 2 — Loading Into PyTorch

The following loader reads the aligned sequence and label files. Paths assume you are running from the repo root (`LogSentry-Capstone/`).

```python
import torch
from torch.utils.data import Dataset, DataLoader
from torch.nn.utils.rnn import pad_sequence


class LogKeyDataset(Dataset):
    """Loads paired (sequence, label) data for LSTM training."""

    def __init__(self, seq_file, label_file):
        self.sequences = []

        with open(seq_file) as f:
            for line in f:
                self.sequences.append([int(x) for x in line.split()])

        self.block_ids = []
        self.labels = []

        with open(label_file) as f:
            f.readline()  # skip header

            for line in f:
                block_id, label = line.strip().split(',')
                self.block_ids.append(block_id)
                self.labels.append(1 if label == 'Anomaly' else 0)

        assert len(self.sequences) == len(self.labels), \
            f'Mismatch: {len(self.sequences)} seqs vs {len(self.labels)} labels'

    def __len__(self):
        return len(self.sequences)

    def __getitem__(self, idx):
        seq = torch.tensor(self.sequences[idx], dtype=torch.long)
        label = torch.tensor(self.labels[idx], dtype=torch.long)
        return seq, label


def collate_fn(batch):
    """Pads variable-length sequences within a batch."""
    seqs, labels = zip(*batch)

    seqs_padded = pad_sequence(
        seqs,
        batch_first=True,
        padding_value=0
    )

    return seqs_padded, torch.stack(labels)


dataset = LogKeyDataset(
    'module1/data/processed/deeplog_input.txt',
    'module1/data/processed/deeplog_labels.txt'
)

loader = DataLoader(
    dataset,
    batch_size=64,
    shuffle=True,
    collate_fn=collate_fn,
    num_workers=2,
)
```

---

## 7. Critical Things To Get Right

### Embedding layer size

Log keys are **1–34** and `0` is reserved for padding.

```python
# WRONG
self.embed = nn.Embedding(34, 128)

# RIGHT
self.embed = nn.Embedding(35, 128)
```

The extra row is required for the padding index `0`.

### Padding

The sequences have variable lengths. Padding is added when creating batches.

The LSTM should not learn from padded positions. Use either:

- `pack_padded_sequence`, or
- an appropriate mask so padded positions do not contribute to the loss.

### Class imbalance

The dataset contains:

- **558,223 Normal**
- **16,838 Anomaly**
- **2.93% anomalies**

A model that predicts `Normal` for everything can still achieve around 97% accuracy, which would not be useful for anomaly detection.

Possible approaches include:

- weighted cross-entropy
- focal loss
- anomaly oversampling

However, note that the original DeepLog approach is based on **top-k next-key prediction**, not simply binary classification. The exact training objective should follow the team's agreed Module 2 implementation.

### Train/test split

DeepLog does not use a simple random sequence split. The intended evaluation separates earlier and later time periods.

A completely random split can allow future patterns to leak into training.

---

## 8. Verification Checklist

Run this before starting model training (from the repo root):

```python
import numpy as np

with open('module1/data/processed/deeplog_input.txt') as f:
    seqs = [line.split() for line in f]

lengths = [len(s) for s in seqs]

print(f'Total sequences: {len(seqs):,}')
print(f'Mean length: {np.mean(lengths):.2f}')
print(f'Max length: {max(lengths)}')
print(f'Min length: {min(lengths)}')

all_keys = set(int(k) for seq in seqs for k in seq)

print(f'Unique log keys: {len(all_keys)}')
print(f'Min key: {min(all_keys)}, Max key: {max(all_keys)}')
```

Expected values:

```text
Total sequences: 575,061
Mean length: ~19.43
Max length: 298
Min length: 2
Unique log keys: 34
Min key: 1, Max key: 34
```

If these values do not match the expected statistics, verify the files and pipeline before starting model training.

---

## 9. Key Statistics

| Metric | Value |
|---|---:|
| Raw log lines | 11,175,629 |
| Unique templates / log keys | 34 |
| HDFS blocks / sequences | 575,061 |
| Mean sequence length | 19.43 |
| Max sequence length | 298 |
| Min sequence length | 2 |
| Normal samples | 558,223 (97.07%) |
| Anomaly samples | 16,838 (2.93%) |
| Full parse time | ~6 minutes |
| End-to-end runtime | ~8 minutes |

---

## 10. Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| `FileNotFoundError` for `HDFS_v1.zip` or `parsed_logs.csv` after cloning | These two files are pointer-only in GitHub | Follow the matching `.download.txt` pointer file (Section 5, Path B) |
| `git push` rejected for a large file | File exceeds GitHub's 100 MB limit | Do not commit `HDFS_v1.zip` or `parsed_logs.csv`; keep only the pointer files in git |
| `AssertionError: Mismatch` | Sequence and label files are not aligned or were loaded incorrectly | Check both files and their line counts |
| `IndexError` on Embedding | Padding row was not included | Use `nn.Embedding(35, ...)` |
| Model predicts only Normal | Class imbalance is being ignored | Use an appropriate imbalance strategy |
| Padding affects training | Padded positions are contributing to the model/loss | Use sequence packing or masking |
| Sequence lengths look wrong | Wrong input file was loaded | Confirm `deeplog_input.txt` is being used |
| Colab Drive mount fails | Session was restarted | Re-run the Drive mount cell |

---

## 11. Development Notes

Three major iterations were needed to obtain the final 34-template output:

```text
Attempt 1 — Raw parse
→ 157 templates
→ Block IDs and IPs prevented effective clustering

Attempt 2 — Regex masking added
→ 96 templates
→ Drain template strings evolved over time and caused fragmentation

Attempt 3 — cluster_id used as log key
→ 34 templates
→ Final output
```

### Important implementation lesson

When using Drain3 for this pipeline, the stable identifier should be the parser's **cluster ID**, rather than using the evolving template string itself as the numeric log key.

### Repository note

The two large files (`HDFS_v1.zip` ~178 MB and `parsed_logs.csv` ~368 MB) exceed GitHub's 100 MB per-file limit. They are excluded from the repository via `.gitignore`, and small `.download.txt` pointer files are committed in their place. Each pointer file documents where to fetch the real file and where to place it. All other outputs are small enough to be stored in the repository directly.

---

## 12. Contact

### Module Owner

**Akshat Pandey**  
VIT Bhopal University  
**Team 28**  
**Role:** Log Ingestion & Parsing / Data Pipeline Lead  
**GitHub:** `mem1-team28`

### Project

**LogSentry — Incremental Vocabulary Adaptation for DeepLog**  
B.Tech Computer Science Engineering — Final Year  
**Supervisor:** Dr. Dheresh Soni

For questions, issues, or anything related to the Module 1 data pipeline,
parsing process, processed files, or this README, open an issue on the repo
or reach me on GitHub at **@mem1-team28**.

---

*Module 1 complete — LogSentry Capstone, B.Tech Computer Science Engineering (Final Year)*
