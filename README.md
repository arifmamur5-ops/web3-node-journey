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

## Components
- **Execution Client:** Geth
- **Consensus Client:** Lighthouse
- **Monitoring:** Prometheus & Grafana
- **Hardware Metrics:** System-level performance tracking

## Monitoring Features
The Grafana dashboard is configured to visualize:
- **Node Health:** Service uptime for Geth and Lighthouse.
- **Sync Progress:** Lighthouse `peer_count` and `head_slot_height`.
- **Hardware Performance:** CPU load, 12GB RAM utilization, and Samsung 870 EVO SSD I/O.

## Getting Started
1. Clone this repository.
2. Launch the services using Docker Compose:
   ```bash
   docker-compose up -d


3. Access the interfaces:
   * **Grafana:** http://localhost:3000 (Default: admin/admin)
   * **Prometheus:** http://localhost:9090

## Note
If the metrics show "No Data," please check the peer_count in the Lighthouse logs. It may take a few minutes for the node to discover peers on the Holesky network before data starts flowing into Prometheus.

## Challenges Encountered

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

 
# Week 5: RPC Node Security - Implementation of Nginx Reverse Proxy & Rate Limiting

## 📌 Project Overview
This fifth-week project focuses on securing blockchain RPC Node infrastructure against brute-force attacks, request spamming, and potential DDoS threats using **Nginx** as a *Reverse Proxy* and *Rate Limiter*. 

To isolate and accurately test the security layer's efficiency without external network dependencies or local hardware choking, **Anvil (Foundry)** was utilized as a local mock RPC backend.

---

## 🏗️ System Architecture & Configuration

The Nginx web server is configured with strict traffic-shaping rules: a baseline rate of `1r/s` (one request per second) and a tolerance threshold of `burst=5` utilizing the *Leaky Bucket* algorithm with the `nodelay` flag.

Configuration file (`/etc/nginx/nginx.conf`):
```nginx
http {
    limit_req_zone $binary_remote_addr zone=rpc_limit:10m rate=1r/s;

    server {
        listen       2096;
        server_name  localhost;

        location / {
            limit_req zone=rpc_limit burst=5 nodelay;
            limit_req_status 429;

            proxy_pass http://localhost:8545;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }

        error_page 429 /429.html;
        location = /429.html {
            return 429 "Woi! Jangan spam, sabar dikit napa! (Too Many Requests)\n";
        }
    }
}

```
## 🌋 The 48-Hour Infrastructure Drama (Real-World Troubleshooting)
This project was a true test of resilience, executed on local low-spec hardware (Lenovo V110, AMD A9, 12GB RAM, running Arch Linux + Sway WM). Instead of a smooth deployment, it turned into a chaotic battle against multiple real-world bottlenecks:
 1. **Database Corruption (Geth):** The initial setup using the Ethereum Geth client (Holesky Testnet) suffered a fatal missing trie node error due to an unexpected system forced-shutdown. A complete removedb procedure was required to reset the state.
 2. **State Healing Bottleneck:** Upon restarting the synchronization from scratch, the process became stuck at **99.4% for nearly 48 hours**. The heavy cryptographic verification of the Merkle tree maxed out disk I/O, combined with severe ISP network throttling, causing Nginx to continuously throw 502 Bad Gateway errors.
 3. **Hardware Rebellion:** In the middle of intense blockchain debugging, the laptop's physical hardware began to fail—the touchpad died completely, and the built-in keyboard suffered from heavy "ghost touch" (random phantom typing). Configuration and navigation had to be executed 100% via external keyboard shortcuts inside the Sway window manager environment.
**The Tactical Pivot:** To prevent endless synchronization delays and ensure deliverability, I made the executive decision to pause the Geth node and pivot to **Anvil** as a local mock RPC backend on port 8545. This effectively isolated the testing environment, proving that Nginx rate-limiting architecture can be validated independently of external chain states.
## 🧪 Automated Load Testing & Verification
The defense mechanism was load-tested by firing 40 consecutive concurrent POST requests (eth_blockNumber) via an automated Bash loop script:
```bash
for i in {1..40}; do 
    echo -n "Request #$i: "
    curl -s -o /dev/null -w "%{http_code}\n" -X POST \
    -H "Content-Type: application/json" \
    --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' \
    http://localhost:2096
    sleep 0.1
done

```
### 📊 Terminal Execution Output
*(Insert your glorious dual-terminal screenshot showing HTTP 200 codes shifting to 429 here)*
**Results Analysis:**
 * **Requests 1-5 (HTTP 200):** Successfully forwarded to the Anvil backend as they fell within the allowed burst capacity.
 * **Requests 6-40 (HTTP 429):** Instantly dropped by Nginx, returning the specified custom Too Many Requests error string.
 * **System Resilience:** Once the loop pace allowed the token bucket to regenerate, access briefly opened back up for a single 200 response before instantly locking down subsequent spam.
## 💡 Conclusion
Week 5 provided invaluable experience in adaptability and infrastructure troubleshooting. In a production environment, resource constraints and hardware failures are inevitable. The ability to diagnose failure points, decouple dependencies, and execute a tactical pivot without compromising the core security architecture is what defines a competent engineer.
```
