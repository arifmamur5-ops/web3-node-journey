# Web3 Node Journey 

Starting my professional journey as a Web3 Infrastructure Engineer. This repository documents my hands-on experience in setting up and maintaining blockchain nodes.

## Project 1: Geth Holesky Testnet Node
Successfully deployed an Ethereum execution client (Geth) on the Holesky testnet using Docker.

### Hardware Specifications
I believe in optimizing resources, so I'm running this on:
* **Machine:** Lenovo V110
* **OS:** Arch Linux (Sway WM)
* **CPU:** Intel Celeron N3350
* **RAM:** 12GB
* **Storage:** 500GB Samsung 870 EVO SSD

### Tech Stack & Setup
* **Engine:** Docker & Docker Compose
* **Client:** Geth (Go-Ethereum)
* **Network:** Holesky Testnet
* **Sync Mode:** Snap Sync (Optimized for low-resource hardware)

### Lessons Learned
* **Handling Docker permission issues with Git.
* **Troubleshooting port mapping (8545 vs 8546).
* **Optimizing sync modes (switching from light to snap sync due to Geth updates).

### Verification
Verified the node status using JSON-RPC:
```bash
curl -X POST -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}' \
  http://localhost:8545
