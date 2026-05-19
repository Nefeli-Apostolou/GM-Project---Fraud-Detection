# GM Project — Fraud Detection in the Ethereum Transaction Network

This repository contains the code and analysis for the Graph Mining course project focused on detecting and characterising fraudulent behaviour in the Ethereum transaction network.

In this project we build a directed transaction graph from a multi-day sample of Ethereum transactions and study whether suspicious accounts differ from ordinary accounts in terms of structural position, transaction flow, community membership, temporal motifs, and Graph Neural Network embeddings.

R code, full datasets, intermediate outputs, and large generated files are available in the shared Google Drive folder:

[Google Drive — Graph Mining Project Files](https://drive.google.com/drive/folders/1TGysVam3qqoxEQeXC_PNHLCyV_dDrk17?usp=sharing)

---

## Project Objective

The main objective is to analyse whether known or suspected fraudulent Ethereum addresses can be distinguished from typical addresses using graph-based methods.

The project focuses on three related questions:

1. Do fraudulent addresses have different structural properties from normal addresses?
2. Do Time Motifs aid in the identification of said fraudulent addresses?
3. Can community structure, basic node features and temporal motifs used in a  GNN-based representations help identify suspicious accounts or suspicious communities?

---

## Data

The main dataset consists of Ethereum transactions collected over a short multi-day time window.

Each transaction contains:

| Column | Description |
|---|---|
| `block_number` | Ethereum block number |
| `hash` | Transaction hash |
| `from_address` | Sender address |
| `to_address` | Receiver address |
| `value` | Amount transferred, in wei |
| `block_timestamp` | Unix timestamp of the transaction |

Additional files include detected fraudulent addresses, community assignments, temporal motif counts, Snap ML motif features, and model outputs.

Large files are not stored directly in this repository and can instead be accessed from the linked Google Drive folder.

---

## Repository Structure

```text
GM-Project---Fraud-Detection/
│
├── Ethereum_Data_Collection.ipynb
├── EDA.ipynb
├── Graph_Mining_Project_cleaned.ipynb
├── ethereum_motif_analysis.ipynb
├── snapml_motifs.ipynb
├── time_motifs_all_nodes.ipynb
├── time_motifs_communities.ipynb
│
├── phishing_addresses_detected.csv
├── mixer_addresses_detected.csv
├── wash_trading_addresses_detected.csv
│
└── README.md
```

### Main notebooks

| Notebook | Purpose |
|---|---|
| `Ethereum_Data_Collection.ipynb` | Code used to collect Ethereum transaction data |
| `EDA.ipynb` | Exploratory analysis of the Ethereum transaction graph |
| `Graph_Mining_Project_cleaned.ipynb` | Final cleaned version of the main project pipeline |
| `ethereum_motif_analysis.ipynb` | Motif-based analysis of suspicious transaction structures |
| `snapml_motifs.ipynb` | Snap ML motif feature extraction |
| `time_motifs_all_nodes.ipynb` | Temporal motif extraction for all nodes |
| `time_motifs_communities.ipynb` | Temporal motif extraction at community level |

---

## Methodology and Past DDMCS Project explanation

### 1. Graph Construction

Ethereum transactions were represented as a directed graph:

```text
from_address  →  to_address
```

Each node corresponds to an Ethereum address, and each directed edge corresponds to a transaction from one address to another.

The graph was used both as:

- a static aggregated directed network for structural analysis;
- a temporal transaction network for cascade and motif analysis;
- an input graph for Graph Neural Network experiments.

---

### 2. Fraud Address Detection

We considered three main categories of suspicious behaviour:

| Fraud type | Description |
|---|---|
| Phishing | Addresses suspected of receiving funds from victims and redistributing them |
| Mixers | Addresses involved in obfuscating transaction flows |
| Wash trading | Addresses involved in repeated or artificial circular transaction behaviour |

The detected addresses were stored in separate CSV files:

```text
phishing_addresses_detected.csv
mixer_addresses_detected.csv
wash_trading_addresses_detected.csv
```

These files were later combined with known scam-address data to create a broader fraud label set.

---

### 3. Structural Feature Analysis

For each node, we computed graph-level structural features such as:

| Feature | Meaning |
|---|---|
| `in_degree` | Number of incoming transaction links |
| `out_degree` | Number of outgoing transaction links |
| `pagerank` | Importance of the node based on incoming flow |
| `betweenness` | Extent to which the node acts as a bridge |

These features were used as the basic node-level input features for the GNN models.

---

### 4. Community Detection (DDMCS past project)

Community detection was used to understand whether fraudulent accounts were isolated or concentrated in specific regions of the graph.

The project explored both:

- Leiden community detection;
- Infomap community detection.

Infomap was later used more consistently because it is well suited to flow-based structures, which are particularly relevant for transaction networks.

Community analysis was used to study:

- whether fraud nodes cluster together;
- whether suspicious communities have distinctive internal structure;
- whether transaction cascades remain inside one community or cross community boundaries.

---

### 5. Temporal Cascade Analysis (DDMCS past Project)

We analysed how transactions propagate from suspicious seed addresses over time.

For each suspicious seed, temporal cascades were constructed by following outgoing transactions within a fixed time horizon.

The cascade analysis measured:

| Metric | Meaning |
|---|---|
| Cascade size | Number of nodes reached from a seed |
| Cascade depth | Maximum propagation distance |
| Branching structure | Whether the flow spreads broadly or narrowly |
| Time-to-reach | How quickly nodes are reached |
| Community containment | Whether the cascade remains inside one community |

This helped distinguish simple local activity from broader transaction diffusion patterns.

---

### 6. Temporal Motif Analysis

Temporal motifs were used to capture repeated short-term transaction patterns around each node.

Examples of temporal motifs include:

| Motif type | Interpretation |
|---|---|
| Temporal out-star | One address sends to multiple addresses in a short period |
| Temporal in-star | One address receives from multiple addresses in a short period |
| Temporal chain | Funds move sequentially across addresses |
| Temporal reciprocal | Back-and-forth transaction behaviour |
| Temporal cycle | Circular flow involving multiple addresses |

Temporal motif counts were computed both:

- for all nodes;
- within communities.

These motif features were later tested as additional inputs for machine learning models.

---

## Graph Neural Network Experiments

The project implemented several GraphSAGE-based models for fraud classification.

The general semi-supervised setup was:

1. Build the full Ethereum transaction graph.
2. Label a limited subset of nodes as fraudulent or normal.
3. Train the GNN on the full graph structure.
4. Compute the loss only on labelled nodes.
5. Use the trained model to predict fraud probabilities for other nodes.

---

## Model Variants

Several feature configurations were tested.

| Model | Input Features | Main Purpose |
|---|---|---|
| Basic GraphSAGE | Structural node features | Baseline model |
| Motif-only MLP | Temporal/Snap ML motif features only | Test whether motifs alone are informative |
| Basic GraphSAGE + MLP | GNN embeddings from basic features | Learn node representations from graph structure |
| GraphSAGE + Motifs + MLP | GNN embeddings combined with motif features | Test whether motifs improve the baseline |
| Snap ML motif-only model | Snap ML motif-derived features only | Evaluate motif-derived features without structural features |

---

## Main Findings

The most important result is that motif features alone were not sufficient for reliable fraud classification, as also shown in various papers.

The basic structural features were necessary for the models to learn meaningful fraud-related patterns. Motif-derived features were more useful when interpreted as complementary information rather than as a replacement for basic graph features.

In general:

- structural graph features were the strongest baseline;
- motifs alone performed poorly;
- combining GraphSAGE embeddings with motif features was conceptually useful, but did not automatically guarantee better performance;
- validation-based threshold selection was important because the dataset was highly imbalanced;
- the full graph structure was useful even when only a subset of nodes was labelled.

---

## Important Files in Google Drive

The Google Drive folder contains the large files required to reproduce the full analysis.

Examples include:

```text
eth_tx_last4days.csv
infomap_communities.csv
all_detected_fraud_accounts.csv
all_communities_temporal_motif_features_master_1h.csv
nodes_communities_with_all_motifs.csv
snap_ml_nodes_output.csv
```

These files are not all stored directly on GitHub because some of them are too large.

Drive folder:

[https://drive.google.com/drive/folders/1TGysVam3qqoxEQeXC_PNHLCyV_dDrk17?usp=sharing](https://drive.google.com/drive/folders/1TGysVam3qqoxEQeXC_PNHLCyV_dDrk17?usp=sharing)

---

## Reproducibility Notes

Some notebooks were designed to run in Google Colab and may require mounting Google Drive.
---

## Limitations

This project should be interpreted as an exploratory graph mining study rather than a production fraud detection system.

Main limitations include:

- fraud labels are partially heuristic and therefore noisy;
- many normal nodes are unlabelled rather than certainly legitimate;
- our labelled normal nodes may actually be fraudulent ones so false positive count might be actually lower in reality;
- Ethereum transaction behaviour is highly imbalanced;
- motif features depend strongly on the chosen time window;
- model performance depends on the negative sampling strategy;
- the dataset covers only a short observation period.

---

## Authors

- Yara Osama Abbas Farid Youssef
- Nefeli Apostolou
- Augusto de Luzenberger Milnernsheim
- Angelina Kolopova

