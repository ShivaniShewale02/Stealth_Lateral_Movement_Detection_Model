<div align="center">

# 🛡️ Stealth Lateral Movement Detection

### ML-Powered Detection Framework for Enterprise Threats

![Python](https://img.shields.io/badge/Python-3.9+-3776AB?style=flat-square&logo=python)
![PyTorch](https://img.shields.io/badge/PyTorch-Deep%20Learning-EE4C2C?style=flat-square&logo=pytorch)
![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)
![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=flat-square)

**Detect stealthy lateral movement attacks using Bi-LSTM, Graph Neural Networks, and Anomaly Detection**

[Quick Start](#-quick-start) • [Documentation](#-documentation) • [Features](#-features) • [License](#-license)

</div>

---

## 🎯 What Is This?

This project provides a **machine learning-based detection framework** to identify stealthy lateral movement attacks in enterprise networks. It combines:

- 🧠 **Bi-LSTM with Self-Attention** for behavioral sequence analysis
- 🔗 **Graph Neural Networks (GCN)** for host-user relationship modeling  
- 📊 **Isolation Forest** for anomaly scoring
- 💡 **Explainable predictions** with confidence scores

**Why?** Traditional threshold-based alerts miss low-and-slow attacks. This framework detects them through behavioral patterns.

---

## ✨ Key Features

| Feature | Description |
|---------|-------------|
| 🤖 **Hybrid ML Models** | Bi-LSTM + GNN + Anomaly Detection ensemble |
| 📊 **Dual Dataset Support** | Separate train/test datasets for validation |
| 🔄 **Sequence Learning** | 20-step event sequences per host |
| 📈 **Graph Embeddings** | Host-user relationship context |
| 🎯 **Weighted Anomalies** | Anomaly scores boost suspicious sequences |
| 🔍 **Attention Weights** | See which events matter most |
| ⚡ **GPU Accelerated** | CUDA support for fast training |

---

## 🏗️ Architecture

```
EVENT STREAM
    ↓
NORMALIZATION & FEATURE EXTRACTION
    ├─ Temporal features (hour, day, weekday)
    ├─ Categorical encoding
    └─ Anomaly scoring (Isolation Forest)
    ↓
SEQUENCE CONSTRUCTION (20 events per host)
    ↓
┌─────────────────────────────────────┐
│   DUAL INPUT PROCESSING             │
├─────────────────────────────────────┤
│ GNN Encoder                         │
│ (Host-User Graph)                   │
│ ↓                                   │
│ Node Embeddings                     │
└─────────────────────────────────────┘
    ↓ (concatenate)
┌─────────────────────────────────────┐
│ Bi-LSTM with Self-Attention         │
├─────────────────────────────────────┤
│ Input: [Event Features] + [GNN Embed]
│ Output: Logits (Normal/Malicious)   │
│ Attention: Which events triggered?  │
└─────────────────────────────────────┘
    ↓
PREDICTIONS & EXPLAINABILITY
```

---

## 📦 Project Structure

```
📁 Stealth_Lateral_Movement_Detection/
├── 📓 data_generator.ipynb
│   └─ Generates synthetic datasets with labeled attacks
├── 📓 model_main.ipynb
│   └─ Model training, validation, and evaluation
├── 📊 slm_train_dataset.csv
│   └─ Training data (179,951 events, 1,343 stealth events)
├── 📊 slm_test_dataset.csv
│   └─ Test data (same structure)
├── 📄 slm_metadata.json
│   └─ Attack windows and ground truth labels
└── 💾 slm_enhanced_model.pth
    └─ Trained PyTorch model checkpoint
```

---

## ⚡ Quick Start

### 1️⃣ Install Dependencies

```bash
pip install torch torch-geometric scikit-learn pandas numpy matplotlib seaborn
```

### 2️⃣ Generate Synthetic Data

```bash
jupyter notebook data_generator.ipynb
```

**Output:**
- `slm_train_dataset.csv` — 179,951 training events
- `slm_test_dataset.csv` — 179,951 test events  
- `slm_metadata.json` — Attack ground truth

### 3️⃣ Train the Model

```bash
jupyter notebook model_main.ipynb
```

**Training Config:**
```python
EPOCHS = 10
BATCH_SIZE = 64
SEQ_LENGTH = 20
LEARNING_RATE = 1e-3
DEVICE = 'cuda'  # GPU acceleration
```

**Expected Results:**
```
Epoch 10/10 completed. Avg Loss: 0.0945
Train Accuracy: 98.2%
Test Accuracy: 97.8%
```

### 4️⃣ Evaluate Performance

The notebook includes evaluation metrics:
- Precision, Recall, F1
- Confusion Matrix
- Attention Weight Visualization
- Feature Importance Analysis

---

## 📊 Model Components

### Bi-LSTM Encoder
```python
- Input: (batch, seq_len=20, features=30+64)
- Hidden: 64 units, bidirectional
- Output: Sequence embeddings
```

### Graph Neural Network
```python
- GCN with 2 convolutional layers
- Input: Host-User bipartite graph
- Output: 64-dim node embeddings
```

### Self-Attention Mechanism
```python
- Projects LSTM hidden states
- Learns attention weights per timestep
- Returns weighted sequence representation
```

### Classification Head
```python
- Input: Weighted LSTM output (128-dim)
- Output: 2 classes (Normal / Stealth LM)
- Loss: Weighted Cross-Entropy (class imbalance handling)
```

---

## 🔍 Attack Stages Detected

The synthetic data includes attacks across these phases:

| Stage | Window | Events |
|-------|--------|--------|
| 🔎 **Reconnaissance** | Days 0-2 | Nbtstat, net, netstat commands |
| 🔑 **Credential Access** | Days 3-5 | Mimikatz, procdump, LSASS access |
| 🖥️ **Lateral Movement** | Days 6-9 | PsExec, WMI, RDP to new hosts |
| 🔒 **Persistence** | Days 10+ | Registry, schtasks, file staging |

---

## 📈 Key Metrics

```
Total Training Events:      179,951
Stealth Events (Positive):  1,343 (0.75%)
Normal Events (Negative):   178,608 (99.25%)

Train/Test Split:           Same dataset (no leakage)
Class Weight:               ~133:1 (imbalance handling)
Sequence Length:            20 events per host
Feature Dimension:          30 base + 64 GNN embeddings = 94 total
```

---

## 🛠️ Customization

### Adjust Detection Sensitivity

```python
# In model_main.ipynb, modify:
THRESHOLD = 0.05          # Lower = more alerts
ANOMALY_WEIGHT_ALPHA = 1.5  # Higher = weight anomalies more
SEQ_LENGTH = 20           # Longer sequences = more context
```

### Add Custom Features

Edit `basic_time_features()` to add:
- Domain-specific patterns
- Business logic (e.g., user roles)
- External threat intel scores

### Use Different Data

Replace `slm_train_dataset.csv` with your own data. Required columns:

```csv
timestamp,host_id,user_id,event_type,process_name,
src_ip,dst_ip,protocol,port,bytes_in,bytes_out,
auth_result,privilege_level,is_stealth_lateral_movement
```

---

## 📚 Notebooks Explained

### `data_generator.ipynb`
Generates realistic synthetic telemetry with labeled attacks:
- ✅ Multi-host environment (100 hosts)
- ✅ 15-day timeline with attack phases
- ✅ Realistic event distributions
- ✅ Ground truth annotations

### `model_main.ipynb`
Complete ML pipeline:
1. Data loading & preprocessing
2. Anomaly detection scoring
3. Sequence construction
4. Graph building
5. Model training with GPU
6. Evaluation & metrics
7. Explainability analysis

---

## 🚀 Advanced Usage

### Load Trained Model

```python
import torch
model = torch.load('slm_enhanced_model.pth')
model.eval()

# Inference on new data
logits, attn_weights, lstm_outs, gnn_out = model(
    graph_data, seq_batch, host_node_idxs, seq_anom_means
)
```

### Visualize Attention

```python
# See which events triggered the alert
import matplotlib.pyplot as plt
plt.imshow(attn_weights[0].detach().numpy(), cmap='hot')
plt.xlabel('Event Index')
plt.ylabel('Attention Weight')
plt.show()
```

### Export for Production

```python
# Convert to ONNX for inference servers
torch.onnx.export(model, (graph_data, seq_batch, ...), 
                  'model.onnx')
```

---

## 📝 License

MIT License — See [LICENSE](LICENSE) file

---

## 🤝 Contributing

Contributions welcome! Areas of interest:

- 🔌 Support for additional telemetry sources
- 🧠 Alternative architectures (Transformer, etc.)
- 📊 Real enterprise datasets
- 🎯 Detection rule optimization
- 📖 Documentation & examples

---

## 📞 Support

- 📧 **Issues**: [Open an issue](https://github.com/ShivaniShewale02/Stealth_Lateral_Movement_Detection/issues)
- 💬 **Discussions**: [Start a discussion](https://github.com/ShivaniShewale02/Stealth_Lateral_Movement_Detection/discussions)
- 👤 **Maintainer**: [ShivaniShewale02](https://github.com/ShivaniShewale02)

---

<div align="center">

### 🔬 Built for Security Research & Enterprise Threat Detection

**Last Updated:** March 14, 2026 | **Version:** 1.0 | **Language:** Python 3.9+

⭐ If this helps you, please star the repo!

</div>
