# Observability, Prometheus, Grafana & SRE Principles

Site Reliability Engineering principles, telemetry architectures, and alerting strategies.

---

## 1. The Three Pillars of Observability

| Pillar | Characteristics | Storage & Query Tool | Primary Value |
| :--- | :--- | :--- | :--- |
| **Metrics** | Numeric aggregable time-series data (`counter`, `gauge`, `histogram`). Low overhead. | Prometheus, VictoriaMetrics, Thanos, Mimir | Real-time dashboards, alerting, capacity planning |
| **Logs** | Discrete event text/JSON records with timestamps. High volume. | Loki, Elasticsearch/OpenSearch, Fluent Bit | Detailed debug forensics, audit trails, error traces |
| **Traces** | Request journey through microservices via span IDs. | OpenTelemetry (OTel), Jaeger, Tempo | Bottleneck latency profiling, service dependency mapping |

---

## 2. Google SRE Four Golden Signals

1. **Latency**: The time it takes to service a request (distinguish between successful request latency and error latency).
2. **Traffic**: A measure of demand (HTTP requests/second, network I/O throughput).
3. **Errors**: The rate of requests that fail (explicit 5xx errors, implicit timeouts, malformed responses).
4. **Saturation**: How full your service is (memory utilization, CPU queue depth, database connection pool exhaustion).

---

## 3. Essential PromQL Alerting Rules

```yaml
groups:
- name: production-alerts
  rules:
  # High Error Rate (SLO breach warning)
  - alert: HighHttp5xxRate
    expr: |
      sum(rate(http_requests_total{status=~"5.."}[5m]))
      /
      sum(rate(http_requests_total[5m])) * 100 > 1
    for: 2m
    labels:
      severity: critical
    annotations:
      summary: "High HTTP 5xx error rate (> 1%) on {{ $labels.service }}"

  # Pod CrashLooping
  - alert: KubernetesPodCrashLooping
    expr: rate(kube_pod_container_status_restarts_total[15m]) * 60 > 2
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "Pod {{ $labels.pod }} is restarting frequently"
```
