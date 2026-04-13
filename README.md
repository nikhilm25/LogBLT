# LogBLT: Byte Latent Transformer for Log Anomaly Detection

[![Python 3.12+](https://img.shields.io/badge/python-3.12+-blue.svg)](https://www.python.org/downloads/)
[![PyTorch](https://img.shields.io/badge/PyTorch-native-orange)](https://pytorch.org/)

Detecting anomalies in large-scale system logs is a significant challenge due to the immense volume, unstructured nature of log data, and the unpredictable variations in log formats over time. Most existing log parsing approaches fail to adapt effectively to out-of-vocabulary tokens or rely on computationally expensive pre-processing steps.

LogBLT solves this by introducing a raw-byte processing framework for log anomaly detection based on Meta AI's Byte Latent Transformer (BLT) architecture. It utilizes a Shannon entropy-based patching mechanism to dynamically divide each log line into predictable (low-entropy) and unpredictable (high-entropy) segments, followed by a hierarchical restoration and attention network to effectively classify system anomalies natively.

---

## Key Innovations

*   **Raw Byte Processing**: Bypasses traditional log parsers and tokenizers by operating directly on standard UTF-8 encoded bytes, providing high out-of-distribution resilience.
*   **Entropy-Based Patching**: Computes Shannon entropy dynamically to construct fixed 8-byte patches, categorizing sequential byte chunks as High Entropy (HE) or Low Entropy (LE).
*   **Hierarchical Attention Architecture**: Utilizes a combination of a Sliding-Window Local Encoder and a Global Self-Attention Transformer to contextualize local byte structures against global log line features.
*   **Entropy-Aware Aggregator**: Adopts an intelligent attention pooling layer that adaptively weighs and extracts high-entropy (unpredictable) feature representations for the final classification sequence.
*   **Severe Imbalance Handling**: Supports tailored dataset representations leveraging Focal Loss and class weighting to penalize confident misclassifications on inherently imbalanced log domains. 

---

## Framework Architecture & Results

Our repository contains the implementation details, visual architectural layouts, and data analytics demonstrating our framework's capability in identifying anomalies across the BGL supercomputing log dataset:

### Framework Architecture

**LogBLT Architecture Part I: Global Architecture Pipeline**  
*This visualizes the overall architecture behind our Byte Latent Transformer for anomalous log tracking.*
<p align="center">
  <img src="assets/arch.jpg" width="100%" />
</p>

**LogBLT Architecture Part II: Byte-Level processing and Attention Components**  
*This figure illustrates the deeper internal mechanisms. Low-level strings are encoded and padded into a fixed length of 256 bytes. They pass through the designated entropy patcher, generating continuous chunks evaluated by the sliding window local encoder, subsequently attended globally to isolate high entropy outliers.*
<p align="center">
  <img src="assets/bltarch.jpg" width="100%" />
</p>

### Experimental Analytics

**Feature Analysis: Correlation Heatmap**  
*The Correlation Heatmap highlights the relationships across entropy patching configurations, dataset labels, and localized byte structures prior to the global attention stage.*
<p align="center">
  <img src="assets/Correlation_Heatmap.png" width="100%" />
</p>

**Performance Evaluation: Accuracy and Distribution**  
*Visualizes a comprehensive summary of normal versus anomalous occurrences and associated predictions in the dataset validation subset.*
<p align="center">
  <img src="assets/Bar_plot.png" width="80%" />
</p>

**State-of-the-Art Discriminative Performance**  
*The Confusion Matrix details the final predicted classification capability of the model on the testing set. LogBLT consistently identifies the anomaly class with a high recall profile and minimal false-positive rates.*
<p align="center">
  <img src="assets/Confusion_Matrix.png" width="80%" />
</p>

---

## Getting Started

### Prerequisites

*   Python 3.12+
*   PyTorch (CUDA runtime environment highly recommended for fast execution)
*   Pandas / Scikit-learn / Matplotlib

### Usage & Setup

#### Training
The `code.ipynb` notebook contains the full definition of the PyTorch framework, dataset loaders, and hyperparameter assignments.
Run the cells sequentially from the notebook environment to ingest the formatted BGL dataset and begin training.

**Key Parameters (Configurable inline):**
*   `max_bytes`: 256
*   `patch_size`: 8 
*   `entropy_threshold`: 1.5
*   `dim_local` / `dim_global`: 256 / 512
*   `epochs`: 15
*   `batch_size`: 32

#### Evaluation
The trailing cells inside `code.ipynb` visualize metric calculations spanning:
*   `Accuracy`
*   `Precision`, `Recall`, and `F1 Score`
*   `ROC AUC`

A model checkpoint is periodically saved to disk capturing the greatest validation F1 performance. 

---

## License
This project is open-sourced and falls under its respective distribution properties detailed in the `LICENSE` file.
