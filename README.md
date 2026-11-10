# Secure Container Pipeline & AWS Registry Governance

## 🔬 Local Analysis & Verification Log
Before staging to the cloud, I conducted static security analysis (SAST) locally using Trivy to audit our base container layers against the global CVE vulnerability database.

### 🚨 Test 1: Standard Modern Image (`node:22`)
*   **Command:** `docker build -t app:v1 . && trivy image --severity HIGH,CRITICAL app:v1`
*   **Outcome:** **VULNERABILITY FOUND** (1 Critical CVE detected).
*   **DevOps Insight:** Even though Node 22 is modern, the standard variant uses a heavy Debian Linux base layer containing unpatched system utilities, failing enterprise security gates.

### ✅ Test 2: Security-Hardened Minimal Image (`node:22-alpine`)
*   **Command:** `docker build -t app:v2 . && trivy image --severity HIGH,CRITICAL app:v2`
*   **Outcome:** **CLEAN BILL OF HEALTH (0 Critical, 0 High vulnerabilities)**.
*   **DevOps Insight:** Upgrading to Alpine Linux strips out all non-essential operating system tools. This drops the threat score to absolute zero, allowing the container to cleanly pass the security gate.
EOF

