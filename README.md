

# 🧠 Federated Learning — Distributed MLP on MNIST

![Python](https://img.shields.io/badge/Python-3.10+-blue?logo=python)
![PyTorch](https://img.shields.io/badge/PyTorch-2.x-EE4C2C?logo=pytorch)
![gRPC](https://img.shields.io/badge/gRPC-Protocol-brightgreen?logo=google)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?logo=docker)
![Streamlit](https://img.shields.io/badge/Dashboard-Streamlit-FF4B4B?logo=streamlit)
![Redis](https://img.shields.io/badge/Registry-Redis-DC382D?logo=redis)
![Status](https://img.shields.io/badge/Status-Academic%20Project-lightgrey)

> **Authors:** Rihab Zitouni · Taha Hakim · Houda Moustaine · Aya Boughalem
> **Supervisor:** Dr. Nadiri Abdeljalil — EMSI Marrakech, 2025–2026

A full federated learning system implementing the **FedAvg** algorithm over **gRPC**, with distributed Docker workers, Byzantine fault detection, chaos testing, and a real-time **Streamlit** monitoring dashboard.

---

## ✨ Features

- 🔗 **gRPC communication** between aggregator server and workers
- 🧮 **FedAvg aggregation** — weighted average of local model updates
- 🛡️ **Fault tolerance** — round timeout, minimum participation threshold (50%), SHA-256 checksum verification
- ☠️ **Byzantine attack simulation** — detection of malicious gradient updates
- 🌪️ **Chaos testing** — random worker crash injection via `fault_injector.py`
- 📊 **Real-time dashboard** — accuracy, loss, participation rate, and node status per round
- ⚙️ **IID / Non-IID data partitioning** — Dirichlet distribution support
- 🐳 **Fully containerized** — one command to launch everything

---

## 🏗️ Architecture

```
Aggregator Server (gRPC :50051)
    ├── Worker-0  (private MNIST partition)
    ├── Worker-1  (private MNIST partition)
    ├── Worker-2  (private MNIST partition)
    └── Worker-N
Redis         → node registry + metrics storage
Streamlit     → real-time monitoring dashboard (localhost:8501)
```

**Model:** MLP `784 → 128 → 64 → 10` (MNIST)
**Aggregation:** `w_global = Σ (n_k / N) × w_k`

---

## 📁 Project Structure

```
federated-learning/
├── server/
│   ├── aggregator.py          # gRPC server + FedAvg logic
│   └── proto/
│       └── federated.proto    # gRPC service contract
├── client/
│   ├── worker.py              # Local training + gRPC client
│   └── model.py               # PyTorch MLP definition
├── data/
│   └── partition.py           # IID / Non-IID data partitioning
├── simulation/
│   └── fault_injector.py      # Chaos engineering & crash injection
├── dashboard/
│   └── app.py                 # Streamlit monitoring app
├── tests/
│   └── test_fedavg.py         # Unit tests for FedAvg
├── docker-compose.yml
├── Dockerfile.server / .worker / .dashboard
├── config.yaml                # Centralized hyperparameters
└── launch.sh                  # One-command launcher
```

---

## 🚀 Quick Start

### Prerequisites
- Docker Desktop (Windows/macOS) or Docker Engine (Linux)
- Docker Compose v2+

### Launch with 5 workers
```bash
bash launch.sh 5
```

### Manual launch
```bash
docker compose build
docker compose up -d redis server dashboard

for i in 0 1 2 3 4; do
  WORKER_ID=$i docker compose run -d --no-deps -e WORKER_ID=$i worker
done
```

**Dashboard →** http://localhost:8501

---

## ⚙️ Configuration (`config.yaml`)

| Parameter | Default | Description |
|-----------|---------|-------------|
| `max_rounds` | 20 | Maximum number of FL rounds |
| `min_clients` | 2 | Minimum workers to start a round |
| `fraction_fit` | 0.6 | Fraction of workers selected per round |
| `round_timeout` | 60 | Per-round timeout in seconds |
| `training.epochs` | 3 | Local epochs per round |
| `dataset.iid` | true | IID or Non-IID (Dirichlet α=0.5) |

---

## 🧪 Demo Scenarios

### ✅ Scenario 1 — Normal convergence
```bash
bash launch.sh 5
# Watch accuracy climb to ≥95% on the dashboard
```

### 💥 Scenario 2 — Live worker crash
```bash
docker kill fl-worker-2
# Node turns red on dashboard — round continues with remaining workers
```

### 🦹 Scenario 3 — Byzantine attack
```bash
BYZANTINE_WORKERS=3 docker compose run -d -e WORKER_ID=3 -e BYZANTINE_WORKERS=3 worker
docker logs fl-server | grep -i byzantine
```

---

## 🧪 Unit Tests

```bash
# Inside server container
docker exec fl-server pytest tests/ -v

# Locally (with venv)
pip install -r requirements.txt
pytest tests/ -v
```

---

## 📊 Monitored Metrics

| Metric | Description |
|--------|-------------|
| Global accuracy | Per-round accuracy on validation set |
| Cross-entropy loss | Aggregated loss per round |
| Participation rate | % of workers that responded in time |
| Node status | `alive` / `crash` / `byzantine` per worker |
| Time to 90% accuracy | Convergence speed indicator |

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|-----------|
| ML Framework | PyTorch |
| Communication | gRPC / Protocol Buffers |
| Orchestration | Docker Compose |
| Registry & Metrics | Redis |
| Dashboard | Streamlit |
| Testing | pytest |
| Language | Python 3.10+ |

---

## 📚 References

- McMahan et al. (2017) — [Communication-Efficient Learning of Deep Networks from Decentralized Data](https://arxiv.org/abs/1602.05629)
- Blanchard et al. (2017) — Byzantine-resilient distributed learning

---

## 📝 Report

Full technical report available upon request.

---

## 📄 License

Academic project — EMSI Marrakech, Distributed Systems S8, 2025–2026.
