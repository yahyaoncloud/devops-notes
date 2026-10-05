# Kubernetes Architecture & Control Plane Internals

Kubernetes is a declarative, distributed container orchestration platform. Understanding how the control plane and worker nodes interact is essential for production operations.

---

## 1. Cluster Component Architecture

```text
Control Plane (Master Node)
├── kube-apiserver          <── REST API Gateway, Authentication & Authorization, Admission Controllers
├── etcd                    <── Consistent, distributed key-value store (Raft consensus)
├── kube-scheduler          <── Assigns unscheduled Pods to optimal Nodes (filtering & scoring)
└── kube-controller-manager <── NodeLifecycle, ReplicaSet, Endpoints, ServiceAccount controllers
      │
      │ (mTLS API traffic)
      ▼
Worker Nodes
├── kubelet                 <── Node agent registering with API server, manages Pod lifecycles via CRI
├── CRI (containerd/CRI-O)  <── Container Runtime Interface executing OCI containers
├── kube-proxy              <── Manages network routing and iptables/IPVS packet forwarding
└── CNI (Calico/Cilium)     <── Container Network Interface assigning Pod IPs & routing
```

---

## 2. Pod Scheduling Lifecycle

1. **Submission**: User or CI runs `kubectl apply -f deployment.yaml`.
2. **Authentication & Admission**: `kube-apiserver` authenticates the token/certificate, checks RBAC permissions, and evaluates Mutating & Validating Webhooks.
3. **Storage**: Desired state is committed to `etcd`.
4. **Scheduling**: `kube-scheduler` watches for Pods with `nodeName: ""` (unscheduled):
   * **Filtering (Predicates)**: Filters out nodes lacking requested CPU/RAM or having matching Taints without Tolerations.
   * **Scoring (Priorities)**: Scores remaining nodes based on resource balance, topology spread, and affinity.
   * **Binding**: Writes the selected `nodeName` to the Pod object in `etcd`.
5. **Execution**: `kubelet` on the target node watches the API server, sees the Pod assigned to it, and instructs the CRI (`containerd`) to pull images and start containers.

---

## 3. High Availability (HA) Control Plane Best Practices

* **etcd Odd Quorum**: Run 3 or 5 etcd nodes. An odd number is required to avoid split-brain elections (Fault tolerance: $F = (N-1)/2$).
* **Dedicated SSDs for etcd**: etcd is extremely sensitive to disk write latency. Fsync latencies over 10ms trigger leader election timeouts.
* **Separation of Concerns**: Do not schedule user application workloads on control plane nodes (`node-role.kubernetes.io/control-plane:NoSchedule`).
