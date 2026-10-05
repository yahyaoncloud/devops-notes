# Container Security, Hardening & CIS Benchmarks

Securing the container runtime, image supply chain, and host boundaries against container breakout exploits.

---

## 1. Top Container Attack Vectors

* **Root User Execution:** Running containers as `root` (UID 0). If breakout occurs, attacker has full host root privileges.
* **Privileged Container Abuse (`--privileged`):** Disables all Linux namespace isolation and gives container direct hardware/kernel access.
* **Vulnerable Dependencies in Base Images:** Stale images containing outdated OpenSSL or library CVEs.
* **Docker Socket Mounting (`-v /var/run/docker.sock:/var/run/docker.sock`):** Gives the container total control over the host Docker daemon (equivalent to root access).

---

## 2. Hardening Best Practices Matrix

| Hardening Control | Docker CLI / Flag | Docker Compose / K8s Syntax | Security Benefit |
| :--- | :--- | :--- | :--- |
| **Non-Root User** | `--user 10001:10001` | `user: "10001:10001"` | Prevents root privilege escalation on host. |
| **Drop Linux Capabilities** | `--cap-drop=ALL --cap-add=NET_BIND_SERVICE` | `cap_drop: [ALL]` | Strips raw kernel capabilities (`CAP_SYS_ADMIN`, `CAP_RAWIO`). |
| **Read-Only Root Filesystem** | `--read-only` | `read_only: true` | Prevents malware or web shells from writing persistence files to disk. |
| **No New Privileges** | `--security-opt=no-new-privileges:true` | `security_opt: ["no-new-privileges:true"]` | Disables SUID binaries from escalating privileges. |
| **Resource Constraints** | `--memory="512m" --cpus="1.0"` | `mem_limit: 512m` | Protects host against CPU/Memory exhaustion DoS attacks. |

---

## 3. Vulnerability Scanning with Trivy

```bash
# Scan container image for OS and language package CVEs
trivy image --severity HIGH,CRITICAL my-custom-app:latest

# Scan Dockerfile for configuration flaws and CIS violations
trivy config ./Dockerfile
```
