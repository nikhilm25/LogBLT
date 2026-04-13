# LogBLT: A Byte Latent Transformer Framework for Log Anomaly Detection

Detecting anomalies in large-scale system logs poses significant challenges due to immense volume, unstructured formatting, and the unpredictable evolution of log templates over time. Most existing log parsing methodologies—heavily reliant on predefined heuristics, tokenization, or fixed regular expression (regex) pre-processing—struggle to adapt to out-of-vocabulary tokens and structural heterogeneity inherent in contemporary architectures.

LogBLT introduces an innovative, raw-byte processing framework for log anomaly detection, heavily inspired by the Byte Latent Transformer (BLT) architecture. By employing a dynamic Shannon entropy-based patching mechanism, LogBLT circumvents handcrafted regex parsing entirely. It processes unmasked UTF-8 streams to efficiently identify structural anomalies and achieves state-of-the-art performance across standard benchmark datasets within a compact 85-million parameter footprint.

---

## Core Methodologies

### Raw Byte-Level Ingestion
Prior architectures rely on a fragile preprocessing step that utilizes handcrafted regular expressions to mask dynamic parameters (e.g., IP addresses, block identifiers) with constant placeholder tokens. While this suppresses variability, it discards critical discriminative information. LogBLT ingests complete raw, unmasked byte streams natively. Each log message is directly converted to its UTF-8 byte sequence, removing the dependency on external domain-specific parsing functions.

### Dynamic Entropy-Based Patching
Rather than implementing fixed-size patching mechanisms, LogBLT utilizes a learned, lightweight entropy model (a 100M-parameter transformer) to estimate next-byte distributions dynamically. The sequence boundaries are triggered through two principal criteria:
1. **Global Entropy Threshold:** A patch begins when the next-byte entropy exceeds a global threshold ($\theta_g = 3.5$).
2. **Relative Monotonicity Criterion:** A boundary triggers upon a significant relative entropy increase against the preceding position ($\theta_r = 1.8$).

Consequently, boilerplate sections exhibiting low entropy generate computational-efficient large patches (64–128 bytes), while novel or anomaly-indicative regions with high entropy create fine-grained representations (8–16 bytes).

### Five-Stage Hierarchical Pipeline
1. Raw byte-level ingestion leveraging complete UTF-8 mapping.
2. Dynamic patch segmentation directed by Shannon entropy bounds.
3. **Local Encoder:** A 2-layer transformer block ($l_E = 2$, $h_E = 4$, $d_e = 128$, $d_p = 256$) summarizing individual bytes into discrete latent representations.
4. **Global Transformer:** A 4-layer transformer configuration ($l_G = 4$, $h_G = 8$, $d_{ffn} = 1024$) conducting self-attention and entropy-conditioned long-range reasoning over derived patches.
5. **Surprisal-Based Detection Head:** A customized MLP architecture ($256 \rightarrow 512 \rightarrow 256$) calculating the final anomaly threshold at the 95th percentile of validation surprisal scores.

---

## Framework Architecture & Process Flow

This repository houses the visual architectural layouts, intermediate processing states, and robust empirical analytics validating the framework’s high capability in identifying anomalies across benchmark datasets.

### Architectural Schematics

**LogBLT Architecture Part I: Global Pipeline**  
*The top-level visualization outlining the Byte Latent Transformer mechanism for anomalous log tracking. This illustrates the fundamental mechanism where raw UTF-8 streams are processed natively without intermediate formatting.*
<p align="center">
  <img src="assets/arch.jpg" width="100%" />
</p>

**LogBLT Architecture Part II: Local Encoding and Global Attention**  
*Internal breakdown illustrating sequence encoding mapping to 256 structural bytes. Continuous sequence chunks are isolated via the entropy patcher, evaluated across the sliding-window local encoder, and ultimately attended globally to highlight extreme entropy manifestations denoting outliers.*
<p align="center">
  <img src="assets/bltarch.jpg" width="100%" />
</p>

---

## Implementation and Experimental Details

The entire LogBLT framework is implemented directly in PyTorch, adopting an AdamW optimization strategy alongside a cosine learning rate schedule over an NVIDIA A40 computational environment. Standard parameters accommodate maximum byte sequence lengths of 8,192 strings, translating to window sizes observing 20 to 50 concurrent log messages.

### State-of-the-Art Baseline Comparisons
Evaluations span across multiple benchmark algorithms, including NeuralLog, FastLogAD, RAPID, and LogLLM (an 8.1-Billion parameter language model). 
*   **Average Performance**: LogBLT outputs a mean F1-score of 0.981 across the established HDFS, BGL, Liberty, and Thunderbird configurations.
*   **Dataset Accuracy Breakdowns**: Achieves F1-scores of 0.999 on HDFS, 0.963 on BGL, 0.978 on Liberty, and 0.985 on Thunderbird.
*   **Parameter Economy**: Notably achieves superior predictive performance while requiring significantly fewer computational parameters (85M parameters) in comparison to primary large-language architectures.

### Robustness to Log Noise and Template Evolution
A significant advantage provided by native byte-level processing is resilience against log noise (character variations) and continuous deployment template evolution. Subjected to synthetic perturbations—including sequence shuffles, randomly generated typos, and synthetic templates—LogBLT exhibits minimal degradation.
Under typographical operational constraints (altering 10% of strings), LogBLT's F1-score decays a mere 2.7%, sharply contrasted against LogLLM (9.1% penalty) and NeuralLog (17.0% penalty). 

### Feature Analysis and Ablation Results

Ablation validations confirm the critical impact of dynamic sub-system configurations:
*   Transitioning from entropy-based patching to static fixed patching incurs an average F1 penalty of 1.9%.
*   Substituting the custom surprisal anomaly detection mechanism with conventional binary classification yields a 2.3% performance degradation.
*   Retaining legacy regex-based pre-processing (masking parameters) forcibly reduces performance, establishing that conventional parsing strips critical discriminative entropy characteristics.

**Feature Mapping Analysis: Correlation Heatmap**  
*Visual representation indicating relational coefficients across entropy bounds, validation subsets, and localized patch structural limits.*
<p align="center">
  <img src="assets/Correlation_Heatmap.png" width="100%" />
</p>

**Occurrence Matrix Summary: Accuracy & Distribution**  
*Displays an aggregated volumetric outline measuring total normal benchmark patterns against identified anomaly configurations, detailing validation metrics across the entire classification horizon.*
<p align="center">
  <img src="assets/Bar_plot.png" width="80%" />
</p>

**Predictive Confusion Matrix Details**  
*Confirms high true-positive detection capacity alongside exceptionally minimal false-positive mapping characteristics, fundamentally driving precision evaluation thresholds above modern architectural baselines.*
<p align="center">
  <img src="assets/Confusion_Matrix.png" width="80%" />
</p>
