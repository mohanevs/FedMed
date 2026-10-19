<div align="center">

# 🧠 FedMed
### Privacy-Preserving Decentralized 3D Brain Tumor MRI Federated AI Platform

[![Python 3.10+](https://img.shields.io/badge/python-3.10%2B-blue.svg)](https://www.python.org/)
[![PyTorch](https://img.shields.io/badge/PyTorch-2.0%2B-EE4C2C.svg)](https://pytorch.org/)
[![Flower FL](https://img.shields.io/badge/Flower-1.7%2B-gold.svg)](https://flower.ai/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.110%2B-009688.svg)](https://fastapi.tiangolo.com/)
[![React 19](https://img.shields.io/badge/React-19-61DAFB.svg)](https://react.dev/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

*A production-grade federated learning platform training volumetric 3D U-Net segmentation models across decentralized hospital networks without transferring a single byte of raw patient data.*

[Key Highlights](#-key-highlights) • [Architecture](#-architecture) • [Quick Start](#-quick-start) • [Verification & Testing](#-verification--testing) • [Project Structure](#-project-structure) • [Security & Compliance](#-security--compliance)

</div>

---

## ⚡ Key Highlights

* **Zero Raw Patient Data Egress:** Training executes strictly in-situ on local hospital hardware. Only encrypted mathematical weight updates ($\Delta W$) leave the hospital boundary.
* **Clinical-Grade 3D U-Net:** MONAI-native volumetric neural network benchmarked on BraTS 2021 multi-modal MRI scans (FLAIR, T1, T1ce, T2) segmenting Whole Tumor (WT), Tumor Core (TC), and Enhancing Tumor (ET) with **88.42% Mean Dice**.
* **Quality-Weighted Aggregation (`FedMedStrategy`):** Dynamic federated aggregation that weights updates by validation Dice performance in addition to sample volume, penalizing noisy or miscalibrated clinical nodes.
* **Cryptographic Defense-in-Depth:**
  * **In-Transit:** Diffie-Hellman ephemeral parameter channels over TLS 1.3.
  * **At-Rest:** Authenticated AES-256-GCM symmetric encryption with HMAC-SHA256 integrity verification on global checkpoints.
  * **Differential Privacy:** Calibrated Gaussian noise addition bounding parameter reconstruction risk.
* **Full-Stack Operational Telemetry:** FastAPI backend with WebSocket streaming, SQLite/SQLAlchemy relational audit trail, and an interactive React 19 UI featuring live analytics and an adaptive dark theme.

---

## 🏗️ Architecture

```
                 ┌──────────────────────────────────────────────┐
                 │          FedMed Coordinator Server           │
                 │   (FastAPI REST/WS  •  Flower Server)        │
                 │   • FedMedStrategy Quality-Weighted FedAvg   │
                 │   • AES-256-GCM Checkpoint Encryption       │
                 │   • Live Telemetry Feeds (/ws/telemetry)    │
                 └──────────────────────┬───────────────────────┘
                                        │
           ┌────────────────────────────┼────────────────────────────┐
           │ Ciphertext Weight Deltas   │ Ciphertext Weight Deltas   │ Ciphertext Weight Deltas
           ▼                            ▼                            ▼
┌───────────────────────┐    ┌───────────────────────┐    ┌───────────────────────┐
│   Hospital Node A     │    │   Hospital Node B     │    │   Hospital Node C     │
│  (Flower NumPyClient) │    │  (Flower NumPyClient) │    │  (Flower NumPyClient) │
│  • In-situ 3D U-Net   │    │  • In-situ 3D U-Net   │    │  • In-situ 3D U-Net   │
│  • Local PACS MRI     │    │  • Local PACS MRI     │    │  • Local PACS MRI     │
│  • AES Encrypted      │    │  • AES Encrypted      │    │  • AES Encrypted      │
└───────────────────────┘    └───────────────────────┘    └───────────────────────┘
```

---

## 🚀 Quick Start

### Prerequisites
* Python 3.10+
* Node.js 18+ & npm

### 1. Installation
```bash
# Clone the repository
git clone https://github.com/mohanevs/FedMed.git
cd FedMed

# Install Python backend & federated learning dependencies
pip install -r requirements.txt

# Install frontend dependencies
cd frontend && npm install && cd ..
```

### 2. Launch the Platform
Run the backend and frontend in separate terminal windows:

```bash
# Terminal 1: Start the Production Backend (Port 8000)
python start_backend.py

# Terminal 2: Start the React 19 UI (Port 5173)
npm --prefix frontend run dev
```

* **Interactive Web App:** `http://localhost:5173`
* **API Documentation (Swagger):** `http://localhost:8000/docs`
* **WebSocket Live Stream:** `ws://localhost:8000/ws/telemetry`

---

## 🧪 Verification & Testing

FedMed includes a comprehensive test suite of 17 test suites (104 tests) covering models, transforms, clients, server aggregation, cryptographic encryption, relational DB schemas, and FastAPI routes:

```bash
# Run the entire test suite
python run_all_tests.py
```

### Standalone Runners (Testing & Diagnostics)
```bash
# Launch a standalone coordinator server for gRPC testing
python backend/scripts/start_server.py --port 8080 --rounds 5 --min-clients 2

# Connect a hospital worker node
python backend/scripts/start_client.py --hospital-id hospital_a --server 127.0.0.1:8080 --epochs 1
```

---

## 📁 Project Structure

```
FedMed/
├── backend/                        # Production FastAPI Backend
│   ├── app.py                      # Application factory & ASGI entry point
│   ├── database.py                 # SQLAlchemy schemas & seed accounts
│   ├── dependencies.py             # Singleton services & state
│   ├── routes/                     # Modular API routers (auth, control, federation, etc.)
│   └── scripts/                    # Testing runners (start_server, start_client, clean_db)
├── federation/                     # Core Federated Learning Engine
│   ├── client/                     # Flower NumPyClient, checkpointing & fallback
│   ├── server/                     # FedMedStrategy, model manager & round sync
│   ├── models/                     # MONAI 3D U-Net, DiceCELoss & subregion metrics
│   ├── datasets/                   # Volumetric MRI transforms & non-IID partitioner
│   └── security/                   # AES-256-GCM encryption & differential privacy
├── frontend/                       # React 19 + Vite Web Application
│   ├── src/pages/                  # Landing, Coordinator, Hospital, Simulation, About
│   ├── src/components/             # Responsive UI components & SVG icons
│   ├── src/context/                # AuthContext (JWT) & ThemeContext (Dark/Light)
│   └── src/styles/index.css        # Clean medical design system & dark mode tokens
├── tests/                          # Consolidated Automated Test Suite
│   ├── backend/                    # FastAPI routes, DB & RBAC tests
│   └── federation/                 # 3D U-Net, client, server, and crypto tests
├── start_backend.py                # Main backend entry point
├── run_all_tests.py                # Master test runner (104 tests)
└── requirements.txt                # Production dependency specification
```

---

## 🔒 Security & Compliance

| Layer | Standard | Technical Guarantee |
|---|---|---|
| **Data Residency** | HIPAA / GDPR | Zero raw DICOM/NIfTI scans leave local hospital storage |
| **Model Weights at Rest** | AES-256-GCM | Checkpoints encrypted with 256-bit symmetric keys & HMAC-SHA256 |
| **Transport Security** | TLS 1.3 / gRPC | Authenticated gradient transmission without plaintext exposure |
| **Inference Privacy** | Differential Privacy | Calibrated Gaussian noise bounds gradient reconstruction risk |
| **Access Control** | JWT + PBKDF2 | Role-Based Access Control (`ADMIN`, `HOSPITAL_STAFF`, `AUDITOR`) |

---

## 📄 License
This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
