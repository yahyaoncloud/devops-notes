# DevOps, Kubernetes & Terraform Handbook 🛠️

A dedicated, comprehensive engineering guide covering Core **DevOps Systems**, Container Orchestration with **Kubernetes**, and Infrastructure as Code with **Terraform**.

---

## 📂 Repository Structure

```text
devops-notes/
├── devops/        # Linux internals, networking, Docker, Ansible, CI/CD, Observability & SRE
├── kubernetes/    # K8s control plane, Workloads, CNI, Ingress, Storage, Helm, ArgoCD & Runbooks
└── terraform/     # HCL2, Remote State Locking, Reusable Modules, Workspaces & Policy-as-Code
```

---

## 🚀 Sections & Modules

### 1. [Core DevOps Engineering](./devops/)
* **Systems Internals**: Linux kernel architecture, systemd service management, process scheduling, cgroups.
* **Storage & Network**: POSIX ACLs, LVM volume expansion, nmcli, firewalld, iptables.
* **Containers & Automation**: Docker layer caching, multi-stage builds, non-root users, Ansible playbooks & roles.
* **Delivery & Observability**: Production GitHub Actions CI/CD workflows, GitOps reconciliation, Prometheus metrics, PromQL alerting, and SRE incident triage.

### 2. [Kubernetes Engineering](./kubernetes/)
* **Control Plane Internals**: `kube-apiserver`, `etcd` Raft clustering, `kube-scheduler` predicates/priorities, `kubelet`.
* **Workload Governance**: Deployments, StatefulSets, DaemonSets, rolling update strategies, and QoS classes.
* **Cluster Networking**: CNI plugins (Calico, Cilium with eBPF), ClusterIP/NodePort/LoadBalancer packet flow, NetworkPolicies.
* **Storage**: PersistentVolumes, PersistentVolumeClaims, StorageClasses, Container Storage Interface (CSI).
* **GitOps & Production Troubleshooting**: Helm chart architecture, ArgoCD continuous sync, and diagnosis runbooks for CrashLoopBackOff, OOMKilled, and DNS failures.

### 3. [Terraform IaC Engineering](./terraform/)
* **HCL2 Mastery**: Advanced types, dynamic blocks, `for_each`, splat operators, and conditional expressions.
* **State Management**: S3/GCS remote backends, DynamoDB state locking, state isolation, and sensitive attribute handling.
* **Modular Architecture**: Root vs child modules, version pinning, composition patterns, and DRY patterns.
* **Operations & Compliance**: Drift detection (`terraform plan -refresh-only`), resource imports, `terraform state mv`, and policy-as-code with Checkov and Trivy.

---

## 🛠️ Maintained By
**Yahya** ([@yahyaoncloud](https://github.com/yahyaoncloud))
