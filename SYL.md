# DevOps, Kubernetes & Terraform Study Mindmap & Syllabus (SYL)

A hierarchical keyword-driven syllabus and curriculum map for mastering modern **DevOps Methodologies**, **Linux & Systems Engineering**, **Kubernetes Cluster Operations**, and **Terraform Infrastructure as Code**.

---

## 1. DevOps & Cloud-Native Engineering Mindmap

```mermaid
mindmap
  root((DevOps Engineering))
    DevOps and Systems
      Linux Internals
        Kernel / Syscalls / Procfs
        Systemd Units / Journalctl
        Process States / Signals / Limits
        POSIX ACLs / File Permissions
        LVM Partitioning / Volume Groups
      Linux Networking
        TCP IP Stack / Three-Way Handshake
        nmcli / NetworkManager
        Iptables / Firewalld / NFTables
        DNS Resolution / Resolv.conf
      Containerization
        Namespaces / Cgroups
        Layer Caching / Multi-Stage Builds
        Distroless / Non-Root Users
        Docker Compose / Bridge Networks
      Configuration and Automation
        Ansible Playbooks / Roles
        Jinja2 Templates / Vault
        Idempotency / Handlers
      CI CD and Delivery
        GitHub Actions / Runners / OIDC
        Artifact Registries (GHCR / ECR)
        Semantic Versioning / Release Automation
        GitOps Reconciliation (ArgoCD / Flux)
      Observability and SRE
        Four Golden Signals
        Prometheus Exporters / PromQL
        Grafana Dashboards / Alertmanager
        Distributed Tracing (OTel / Tempo)
        Log Aggregation (Loki / Fluent Bit)
        SLOs / SLIs / SLAs / Error Budgets
    Kubernetes
      Control Plane Architecture
        Kube-apiserver / etcd Raft
        Kube-scheduler (Filters / Scores)
        Kube-controller-manager
        Admission Webhooks (Mutating / Validating)
      Worker Node Components
        Kubelet / CRI (containerd)
        Kube-proxy (iptables / IPVS)
        CNI Plugins (Calico / Cilium eBPF)
      Workloads and Controllers
        Pods / InitContainers / Ephemeral
        Deployments / RollingUpdate / Replicas
        StatefulSets / Headless Services
        DaemonSets / Jobs / CronJobs
        QoS Classes (Guaranteed / Burstable)
      Networking and Policies
        ClusterIP / NodePort / LoadBalancer
        Ingress Controllers / Gateway API
        NetworkPolicies (Zero-Trust Pod Isolation)
        CoreDNS / Upstream Resolution
      Storage Architecture
        StorageClasses / Dynamic Provisioning
        PersistentVolumes / Claims (PV / PVC)
        CSI Drivers / Access Modes (RWO / RWX)
      Package Management and Operations
        Helm Charts / Values / Templates
        ArgoCD ApplicationSets / Sync Hooks
        Cluster Triage (CrashLoopBackOff / OOMKilled)
    Terraform IaC
      HCL2 Core
        Providers / Resources / Data Sources
        Variables / Locals / Outputs
        Dynamic Blocks / For Each / Splats
      State Management
        Remote State (S3 / GCS Backends)
        State Locking (DynamoDB)
        State Isolation / Workspaces
        Terraform State MV / Import / RM
      Modular Design
        Root vs Child Modules
        Semantic Module Versioning
        Composition Patterns / DRY
      Lifecycle and CI CD
        Terraform Plan / Apply / Destroy
        Targeted Operations / Refresh-Only
        Drift Detection Pipelines
      Policy as Code and Security
        Checkov / Trivy IaC Scanners
        Open Policy Agent (OPA) / Sentinel
        Secret Management Zero-Hardcoding
```

---

## 2. Text Keyword Hierarchy

### I. Core DevOps Engineering & Systems
* **Linux Systems Internals**: Linux Kernel space vs User space • System calls (`fork`, `execve`, `clone`, `epoll`) • `/proc` and `/sys` filesystems • `systemd` target units, services, timers, and dependencies • `journalctl` log inspection • Process states (`R`, `S`, `D`, `Z`), signals (`SIGTERM`, `SIGKILL`), CPU scheduling priorities, and `ulimit` file descriptor boundaries • Memory architecture (Resident Set Size RSS, Virtual Memory VSZ, Page Faults, Swap space)
* **Storage & Systems Hardening**: Disks & Partitions (GPT, MBR) • Logical Volume Management (LVM: Physical Volumes, Volume Groups, Logical Volumes, dynamic filesystem growth) • File Permissions (rwx bitmasks, SUID, SGID, Sticky Bit) • POSIX Access Control Lists (`getfacl`, `setfacl`) • SELinux contexts, types, booleans, and audit logs
* **Enterprise Linux Networking**: TCP/IP protocol suite • TCP 3-way handshake & connection termination • Socket states (`LISTEN`, `ESTABLISHED`, `TIME_WAIT`) • Linux NetworkManager (`nmcli`) • Firewall management (`firewalld` zones, rich rules, `nftables`, raw `iptables`) • Network troubleshooting (`ip route`, `ss`, `tcpdump`, `traceroute`, `curl`, `dig`)
* **Container Internals & Docker**: Linux Namespaces (`pid`, `net`, `mnt`, `ipc`, `uts`, `user`) • Control Groups (`cgroups v2` CPU/Memory throttling) • OverlayFS storage drivers & copy-on-write (CoW) • Dockerfile optimization (layer caching, multi-stage builds, minimal base images like Alpine and Distroless, non-root user execution `USER nonroot`) • Docker networking (Bridge, Host, Overlay, Macvlan) • Docker Compose multi-service composition
* **Configuration Management & Automation**: Ansible architecture (agentless push over SSH) • Inventory files & dynamic inventories • Playbooks, tasks, and idempotency • Ansible Roles directory conventions • Jinja2 templating filters • Ansible Vault credential encryption
* **Continuous Integration & Delivery (CI/CD)**: GitHub Actions workflows, matrix builds, composite actions, and self-hosted runners • OIDC token exchange (Zero-secret cloud authentication) • BuildKit image caching • Semantic versioning & changelog generation • Artifact signing (Cosign) & Software Bill of Materials (SBOM)
* **Observability & SRE (Site Reliability Engineering)**:
  * Google SRE 4 Golden Signals (Latency, Traffic, Errors, Saturation)
  * Metrics: Prometheus time-series database, node_exporter, PromQL aggregations (`rate`, `irate`, `sum`, `histogram_quantile`), Alertmanager routing
  * Dashboards: Grafana data sources, variables, alert rules
  * Logs: LogQL with Grafana Loki, Fluent Bit log collection & parsing
  * Traces: Distributed tracing, OpenTelemetry (OTel) collectors, trace/span context propagation
  * SRE Core: Service Level Indicators (SLI), Service Level Objectives (SLO), Service Level Agreements (SLA), Error Budgets, and Incident Postmortems

---

### II. Kubernetes Engineering & Cluster Operations
* **Control Plane Architecture**:
  * `kube-apiserver`: Declarative API, validation, admission controllers (Mutating & Validating Webhooks)
  * `etcd`: Raft consensus, quorum calculation ($F=(N-1)/2$), snapshot backup and restore
  * `kube-scheduler`: Two-phase scheduling (Filtering/Predicates & Scoring/Priorities), node affinity, taints & tolerations, pod topology spread constraints
  * `kube-controller-manager`: Reconciliation loops, Node controller, ReplicaSet controller, EndpointSlice controller
* **Worker Node Architecture**:
  * `kubelet`: Node registration, static pods, CRI interface, cgroup management, PLEG (Pod Lifecycle Event Generator)
  * Container Runtime Interface (CRI): containerd, CRI-O, OCI runtime specifications
  * `kube-proxy`: ClusterIP routing, iptables vs IPVS mode
  * Container Network Interface (CNI): Calico (IPAM, BGP routing, VXLAN), Cilium (eBPF packet filtering and routing bypassing iptables)
* **Workloads & Controllers**:
  * Pod lifecycle (`Pending`, `Running`, `Succeeded`, `Failed`), initContainers, ephemeral debug containers
  * Probes: `startupProbe`, `readinessProbe`, `livenessProbe`
  * Quality of Service (QoS): `Guaranteed`, `Burstable`, `BestEffort`
  * Workload Types: `Deployment` (rolling update, recreate), `StatefulSet` (stable network ID, `volumeClaimTemplates`), `DaemonSet` (one per node), `Job` & `CronJob`
* **Cluster Networking & Ingress**:
  * Pod-to-Pod and Pod-to-Service communication
  * Service Types: `ClusterIP`, `NodePort`, `LoadBalancer`, `ExternalName`
  * Ingress Controllers: NGINX Ingress, Traefik, Envoy, Kubernetes Gateway API
  * NetworkPolicies: Ingress/Egress isolation, podSelectors, namespaceSelectors, CIDR blocks
  * CoreDNS: Service discovery (`<service>.<namespace>.svc.cluster.local`), resolv.conf ndots tuning
* **Storage & Persistence**:
  * PersistentVolumes (PV) vs PersistentVolumeClaims (PVC)
  * StorageClasses & Dynamic Provisioning
  * Container Storage Interface (CSI) drivers
  * Access Modes: `ReadWriteOnce` (RWO), `ReadOnlyMany` (ROX), `ReadWriteMany` (RWX), `ReadWriteOncePod` (RWOP)
  * Volume Reclaim Policies: `Delete` vs `Retain`
* **GitOps & Production Troubleshooting**:
  * Helm: Chart structure, values inheritance, template helpers (`_helpers.tpl`), dependencies, release lifecycle
  * ArgoCD: Declarative GitOps, Application CRD, ApplicationSets, automated sync, self-healing, sync waves & hooks
  * Diagnostic Runbooks: Resolving `CrashLoopBackOff`, `OOMKilled` (Exit Code 137), `ImagePullBackOff`, `NodeNotReady`, etcd disk latency warnings, and ephemeral port exhaustion

---

### III. Terraform Infrastructure as Code (IaC)
* **HCL2 Syntax & Foundations**: Provider declarations, required providers, version constraints • Resources vs Data Sources • Input Variables (type constraints, validation blocks) • Local values • Output values (sensitive attribute masking) • Dynamic blocks, `for_each`, `count`, ternary conditionals, and splat operators (`[*]`)
* **State Management**: Terraform state file schema • Remote backends (AWS S3, Google Cloud Storage, Terraform Cloud) • Distributed state locking (AWS DynamoDB) • State security & sensitive data mitigation • Workspace isolation (dev/stage/prod) • State surgery commands: `terraform state list`, `terraform state show`, `terraform state mv`, `terraform state rm`, `terraform import`
* **Modular Infrastructure Design**: Root modules vs reusable child modules • Module composition patterns • Input contracts & output schemas • Semantic versioning of Terraform modules • DRY (Don't Repeat Yourself) infrastructure patterns
* **Execution Lifecycle & CI/CD**: Dependency graph computation (`terraform graph`) • Core workflow: `terraform init`, `terraform validate`, `terraform plan`, `terraform apply`, `terraform destroy` • Partial deployments with `-target` • Drift detection with `terraform plan -refresh-only`
* **Policy as Code & IaC Security**: Static code analysis & misconfiguration scanning (Checkov, tfsec, Trivy) • Automated linting (`tflint`) • Policy as Code (HashiCorp Sentinel, Open Policy Agent Rego policies) • Zero-hardcoded-secret enforcement with dynamic cloud credentials
