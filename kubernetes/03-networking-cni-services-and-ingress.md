# Kubernetes Networking: CNI, Services & Ingress Architecture

Kubernetes enforces four fundamental network requirements:
1. All Pods communicate with all other Pods on all nodes without NAT.
2. All nodes communicate with all Pods without NAT.
3. The IP that a Pod sees itself as is the same IP that other Pods see it as.
4. Services provide a stable, load-balanced virtual IP across dynamic Pod replicas.

---

## 1. Container Network Interface (CNI) Comparison

| CNI Plugin | Mechanism | Routing Performance | NetworkPolicy Support | eBPF Acceleration |
| :--- | :--- | :--- | :--- | :--- |
| **Flannel** | VXLAN Overlay | Standard (packet encapsulation) | No (requires Canal/Calico) | No |
| **Calico** | BGP Native or VXLAN | High (native L3 routing) | Industry standard L3/L4 | Yes (optional eBPF data plane) |
| **Cilium** | eBPF Kernel Probes | Maximum (bypasses iptables entirely) | L3, L4, and L7 (HTTP/gRPC) | Native Core |
| **AWS VPC CNI** | AWS ENI Secondary IPs | Native AWS VPC Performance | Security Groups per Pod | No |

---

## 2. Kubernetes Services & Packet Traversal

```text
External Client
       │
       ▼
Ingress Controller (NGINX / Envoy / Traefik)
       │ (Evaluates host headers & paths)
       ▼
Kubernetes Service (ClusterIP: 10.96.0.100)
       │ (kube-proxy / iptables / IPVS rewrites virtual IP to target Pod IP)
       ▼
Target Pod (10.244.1.45:8080)
```

### Service Types
* **ClusterIP**: Internal-only virtual IP reachable from within the cluster.
* **NodePort**: Allocates a static port (30000-32767) across every worker node's external IP address.
* **LoadBalancer**: Provisions an external cloud load balancer (AWS NLB/ALB, Azure LB, GCP Cloud LB) pointing to NodePorts.
* **ExternalName**: CNAME record forwarding inside the cluster without proxying.

---

## 3. Least-Privilege NetworkPolicy Example

By default, all Pods in a Kubernetes cluster can talk to all other Pods. Use `NetworkPolicy` to enforce zero-trust isolation:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: isolate-database
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: postgres-db
  policyTypes:
  - Inbound
  ingress:
  # Allow incoming connections ONLY from backend API pods on port 5432
  - from:
    - podSelector:
        matchLabels:
          app: backend-api
    ports:
    - protocol: TCP
      port: 5432
```
