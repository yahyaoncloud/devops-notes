# Docker Networking, Persistent Storage & Docker Compose

---

## 1. Docker Networking Drivers

| Network Driver | Architecture & Isolation | Use Case |
| :--- | :--- | :--- |
| **Bridge (Default)** | Creates private internal software bridge (`docker0` or user-defined bridge). Containers communicate via internal DNS and use NAT for outbound access. | Single-host container communication. |
| **Host** | Bypasses network isolation; container shares the host network stack directly (`--net=host`). | Maximum performance (e.g. high-throughput monitoring/Suricata). |
| **None** | Completely disables container networking (isolated loopback only). | Offline cryptographic secrets generation, air-gapped tasks. |
| **Overlay** | Distributed multi-host VXLAN overlay network. | Docker Swarm / multi-node clusters. |
| **Macvlan** | Assigns dedicated physical MAC address to container, making it appear as a physical host on the LAN. | Direct integration with legacy network monitoring tools. |

---

## 2. Persistent Storage: Named Volumes vs Bind Mounts

* **Named Volumes (`/var/lib/docker/volumes/...`):** Managed completely by Docker. Safest for production database persistence (isolated from host OS user permissions).
* **Bind Mounts (`/opt/app/logs:/var/log/app`):** Direct mapping of host filesystem path into the container. Excellent for configuration files and development source code mounting.

---

## 3. Multi-Container Orchestration with Docker Compose

```yaml
version: '3.8'

services:
  suricata-ids:
    image: jasonish/suricata:8.0.7
    container_name: suricata-nids
    restart: unless-stopped
    network_mode: host
    cap_add:
      - NET_ADMIN
      - NET_RAW
      - SYS_NICE
    volumes:
      - /etc/suricata:/etc/suricata:ro
      - suricata_logs:/var/log/suricata

  evebox-siem:
    image: jasonish/evebox:latest
    container_name: evebox-dashboard
    restart: unless-stopped
    ports:
      - "5636:5636"
    volumes:
      - suricata_logs:/var/log/suricata:ro
    environment:
      - EVEBOX_INPUT=/var/log/suricata/eve.json
    depends_on:
      - suricata-ids

volumes:
  suricata_logs:
```
