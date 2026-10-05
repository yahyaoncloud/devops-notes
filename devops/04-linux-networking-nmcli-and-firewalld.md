# Linux Enterprise Networking: `nmcli`, `iproute2` & `firewalld`

---

## 1. Network Configuration with NetworkManager CLI (`nmcli`)

```bash
# Display Device and Connection Status
nmcli device status
nmcli connection show

# Configure Static IP Address and Gateway
sudo nmcli connection add type ethernet con-name "eth0-static" ifname eth0 \
    ip4 10.10.30.50/24 gw4 10.10.30.1 ipv4.dns "1.1.1.1 8.8.8.8" ipv4.method manual

# Activate Connection
sudo nmcli connection up "eth0-static"

# Create 802.1Q VLAN Sub-Interface (VLAN 30)
sudo nmcli connection add type vlan con-name "eth0.30" ifname eth0.30 dev eth0 id 30 \
    ip4 10.10.30.50/24 gw4 10.10.30.1
```

---

## 2. Modern `iproute2` Diagnostic Commands

| Legacy Command | Modern `iproute2` Replacement | Action & Purpose |
| :--- | :--- | :--- |
| `ifconfig` | `ip -br addr show` | Display IP addresses and link states. |
| `route -n` | `ip route show` | Display kernel routing table. |
| `netstat -tulpn` | `ss -tulpn` | Display listening TCP/UDP sockets with process PIDs. |
| `arp -a` | `ip neighbor show` | Display Layer 2 ARP cache table. |

---

## 3. Host Perimeter Defense with `firewalld`

`firewalld` dynamically manages netfilter/nftables rules using zones (e.g. `public`, `internal`, `trusted`, `dmz`):

```bash
# Check Active Zones
sudo firewall-cmd --get-active-zones

# Allow Inbound Service in Public Zone (Permanent)
sudo firewall-cmd --zone=public --add-service=https --permanent
sudo firewall-cmd --zone=public --add-port=502/tcp --permanent

# Rich Rule: Restrict Port 502 strictly to Authorized Management / SCADA Subnet
sudo firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="10.10.10.0/24" port port="502" protocol="tcp" accept' --permanent

# Reload Rules into Kernel
sudo firewall-cmd --reload
sudo firewall-cmd --list-all
```
