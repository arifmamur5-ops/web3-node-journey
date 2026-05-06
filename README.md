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

**Status:** Running (Syncing from Genesis)

**Setup:**
- **Consensus Client:** Lighthouse
- **Network:** Holesky
- **Hardware:** Arch Linux (Sway) | AMD A9 | 8GB RAM

**Troubleshooting Log (Malam Jumat Berdarah):**
1. **DNS Issue:** WiFi publik nge-reset `/etc/resolv.conf`. 
   - *Fix:* Hapus symlink, buat file fisik, set DNS Google (8.8.8.8), dan kunci pake `sudo chattr +i /etc/resolv.conf`.
2. **Docker DNS:** Container tetep gak bisa resolve domain.
   - *Fix:* Tambahin config `dns: [8.8.8.8, 1.1.1.1]` langsung di `docker-compose.yml`.
3. **Checkpoint Sync Failure:** Gagal konek ke remote checkpoint karena masalah network.
   - *Fix:* Paksa jalan dari Genesis pake flag `--allow-insecure-genesis-sync` biar node tetep running meskipun DNS rewel.
4. **Permission Denied (Git):** Gagal `git add` karena folder data punya `root`.
   - *Fix:* Gunakan `.gitignore` buat nge-exclude `lighthouse-data/` dan `geth-data/`.
