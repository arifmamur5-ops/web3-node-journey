# Web3 Node Journey 

Starting my professional journey as a Web3 Infrastructure Engineer. This repository documents my hands-on experience in setting up and maintaining blockchain nodes.

## Project 1: Geth Holesky Testnet Node
Successfully deployed an Ethereum execution client (Geth) on the Holesky testnet using Docker.

### Hardware Specifications
I believe in optimizing resources, so I'm running this on:
* **Machine:** Lenovo V110
* **OS:** Arch Linux (Sway WM)
* **CPU:** AMD A9-9420 Radeon R5 (2 CPU Cores + 3 GPU Cores)
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

## Project 2: Lighthouse Consensus Client (Holesky Testnet)

**Current Status:** Active & Syncing (Genesis Mode)

### Tech Stack
- **Consensus Client:** Lighthouse
- **Network:** Holesky
- **OS:** Arch Linux (Sway WM)
- **Hardware:** AMD A9 Dual-Core, 8GB RAM

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
