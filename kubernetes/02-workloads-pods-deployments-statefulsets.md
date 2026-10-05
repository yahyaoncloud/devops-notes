# Kubernetes Workloads: Pods, Deployments & StatefulSets

Workload controllers manage Pod lifecycles to ensure declared state is continuously maintained.

---

## 1. Controller Comparison

| Workload Type | State Management | Pod Naming & Identity | Storage Binding | Primary Use Case |
| :--- | :--- | :--- | :--- | :--- |
| **Deployment** | Stateless | Random hash suffix (`web-7f89d-abc12`) | Shared or ephemeral storage | Web servers, REST APIs, microservices |
| **StatefulSet** | Stateful | Deterministic index (`db-0`, `db-1`, `db-2`) | Unique Dedicated PVC per Pod (`volumeClaimTemplates`) | Databases (PostgreSQL, MongoDB), Kafka, ZooKeeper |
| **DaemonSet** | Node-level daemon | Bound to each eligible node | Host storage / Host network | Log collectors (Fluent Bit), CNI agents, Node Exporters |
| **Job / CronJob** | Run-to-completion | Unique per run | Ephemeral | Database migrations, nightly batch reports, backups |

---

## 2. Production Pod Resource Governance

Always specify `requests` and `limits` to guarantee predictable scheduling and Quality of Service (QoS):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
  namespace: production
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 0
  selector:
    matchLabels:
      app: order-service
  template:
    metadata:
      labels:
        app: order-service
    spec:
      containers:
      - name: order-service
        image: ghcr.io/yahyaoncloud/order-service:v2.1.0
        resources:
          requests:
            cpu: "250m"
            memory: "256Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"
        # Health probes
        startupProbe:
          httpGet:
            path: /healthz
            port: 8080
          failureThreshold: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          periodSeconds: 5
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
          periodSeconds: 15
```

### Pod QoS Classes
* **Guaranteed**: Every container has `requests.cpu == limits.cpu` and `requests.memory == limits.memory`. Lowest probability of being OOM-killed under node pressure.
* **Burstable**: Containers have `requests < limits`.
* **BestEffort**: No requests or limits specified. First to be terminated when a node runs out of memory.
