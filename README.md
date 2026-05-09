# Web3 Node Journey 

Starting my professional journey as a Web3 Infrastructure Engineer. This repository documents my hands-on experience in setting up and maintaining blockchain nodes.

## Project 1: Geth Holesky Testnet Node
Successfully deployed an Ethereum execution client (Geth) on the Holesky testnet using Docker.

### Hardware Specifications
I believe in optimizing resources, so I'm running this on:
* **Machine:** Lenovo V110
* **OS:** Arch Linux (Sway WM)
* **CPU:** AMD A9-9420 Radeon R5 dual-core
* **RAM:** 12GB
* **Storage:** 500GB Samsung 870 EVO SSD

### Tech Stack & Setup
* **Engine:** Docker & Docker Compose
* **Client:** Geth (Go-Ethereum)
* **Network:** Holesky Testnet
* **Sync Mode:** Snap Sync (Optimized for dual-core hardware)

### Lessons Learned
* Handling Docker permission issues with Git.
* Troubleshooting port mapping (8545 vs 8546).
* Optimizing sync modes (switching from light to snap sync due to Geth updates).

### Known Issues & Next Steps

### 1. Missing Beacon Client Warning
Currently, the Geth logs show: `Beacon client online, but no consensus updates.`.
* **Status:** Open
* **Context:** In the current Ethereum architecture (Post-Merge), Geth requires a Consensus Layer (CL) client to fully synchronize and verify blocks.
* **Plan:** My next step is to set up a lightweight Consensus Client (like Lighthouse or Prysm) using Docker Compose to complete the full node stack without overloading the Lenovo V110's resources.

### 2. Hardware Resource Management
* Syncing on a dual-core AMD A9 requires heavy I/O wait monitoring.
* **Optimization:** Considering limiting Docker CPU usage if the system becomes unresponsive. 

### Verification
Verified the node status using JSON-RPC:
```bash
curl -X POST -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}' \
  http://localhost:8545
```

## Project 2: Lighthouse Consensus Client (Holesky Testnet)

**Current Status:** Active & Syncing (Genesis Mode)

### Tech Stack
- **Consensus Client:** Lighthouse
- **Network:** Holesky
- **OS:** Arch Linux (Sway WM)
- **Hardware:** AMD A9-9420 Radeon R5, 2GB RAM

### Troubleshooting Log
- **System DNS Conflict:** Encountered persistent DNS resets on `/etc/resolv.conf` due to public WiFi NetworkManager settings.
    - *Fix:* Replaced the symlink with a static file pointing to Google DNS (8.8.8.8) and applied `sudo chattr +i` to prevent unauthorized overwrites.
- **Docker DNS Resolution:** Containers failed to resolve external domains despite host-level fixes.
    - *Fix:* Explicitly defined DNS servers (`8.8.8.8`, `1.1.1.1`) within the `docker-compose.yml` service configuration.
- **Checkpoint Sync Failure:** Network restrictions prevented connection to trusted checkpoint endpoints.
    - *Fix:* Pivoted to Genesis sync by implementing the `--allow-insecure-genesis-sync` flag to ensure node uptime.
- **Git Permission & Data Bloat:** Faced `Permission denied` errors during `git add` due to Docker-owned volumes.
    - *Fix:* Configured `.gitignore` to exclude heavy data directories (`geth-data/`, `lighthouse-data/`) and corrected ownership via `chown`.

### How to Run
```bash
docker-compose up -d
docker logs -f lighthouse-holesky
```

# Project 4: Ethereum Monitoring Stack

This repository contains the configuration for running and monitoring an Ethereum Node (Holesky Testnet) using Geth and Lighthouse, integrated with Prometheus and Grafana for real-time analytics.

## 🚀 Components
- **Execution Client:** Geth
- **Consensus Client:** Lighthouse
- **Monitoring:** Prometheus & Grafana
- **Hardware Metrics:** System-level performance tracking

## 📊 Monitoring Features
The Grafana dashboard is configured to visualize:
- **Node Health:** Service uptime for Geth and Lighthouse.
- **Sync Progress:** Lighthouse `peer_count` and `head_slot_height`.
- **Hardware Performance:** CPU load, 12GB RAM utilization, and Samsung 870 EVO SSD I/O.

## 🛠️ Getting Started
1. Clone this repository.
2. Launch the services using Docker Compose:
   ```bash
   docker-compose up -d
```
3. Access the interfaces:
   * **Grafana:** http://localhost:3000 (Default: admin/admin)
   * **Prometheus:** http://localhost:9090
## 📝 Note
If the metrics show "No Data," please check the peer_count in the Lighthouse logs. It may take a few minutes for the node to discover peers on the Holesky network before data starts flowing into Prometheus.

## ⚠️ Challenges Encountered

During the setup, several issues were identified and resolved:

1. **Docker Service Naming:**
   - **Problem:** Commands like `docker-compose restart` failed because of inconsistent service naming (e.g., trying to restart `lighthouse-holesky` while the service was named `lighthouse`).
   - **Solution:** Identified correct service names using `docker-compose ps` and updated the commands accordingly.

2. **Syncing & Peer Discovery:**
   - **Problem:** Initial Grafana metrics showed "No Data" with `peer_count: 0`.
   - **Solution:** Verified the Lighthouse logs, checked network connectivity, and waited for the node to handshake with peers on the Holesky network.

3. **Metrics Connectivity:**
   - **Problem:** Prometheus showed some targets as `DOWN` with 404 errors.
   - **Solution:** Configured `prometheus.yml` to point to the correct internal container ports (e.g., `lighthouse:5054`) and ensured metrics flags were enabled on the clients.

4. **Git Conflict & Merge Issues:**
   - **Problem:** Encountered "divergent branches" and "missing editor (vi)" errors during the push to GitHub.
   - **Solution:** Resolved by configuring `pull.rebase false` and executing a manual merge commit via terminal.
