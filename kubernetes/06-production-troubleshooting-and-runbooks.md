# Kubernetes Production Troubleshooting & Incident Runbooks

Field diagnostics for resolving common Kubernetes production failures.

---

## 1. Top Pod Failure States & Remediation

| Status | Root Cause | Diagnosis Command | Resolution |
| :--- | :--- | :--- | :--- |
| **CrashLoopBackOff** | App crashes immediately after starting (unhandled exception, missing config, failed DB connect). | `kubectl logs <pod> --previous`<br>`kubectl describe pod <pod>` | Inspect stderr logs, verify environment variables and secret bindings. |
| **ImagePullBackOff** | Non-existent image tag, typo in registry URL, or missing pull secrets (`imagePullSecrets`). | `kubectl describe pod <pod>` | Verify image exists on registry, check Docker credentials secret. |
| **OOMKilled (Exit Code 137)** | Process exceeded container `limits.memory`. Linux kernel invoked OOM killer. | `kubectl describe pod <pod> \| grep -i "oom"` | Increase memory limit or profile application for heap memory leaks. |
| **Pending** | Insufficient CPU/Memory on worker nodes, or no node matches Taints / NodeSelectors. | `kubectl describe pod <pod>` (Events section) | Check cluster capacity (`kubectl top nodes`), adjust resource requests or add nodes. |
| **Evicted** | Node disk pressure (`nodefs.available < 10%`) or memory pressure. | `kubectl get events --sort-by='.metadata.creationTimestamp'` | Clean unused Docker images (`crictl rmi --prune`), expand root disk volume. |

---

## 2. Cluster Networking & DNS Triage

If Pods cannot resolve service names or external URLs:

```bash
# 1. Run an interactive ephemeral debug pod
kubectl run dns-test --rm -it --image=busybox:1.36 -- restart=Never -- sh

# 2. Check internal service DNS
nslookup kubernetes.default.svc.cluster.local

# 3. Check CoreDNS upstream health
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns --tail=50
```

---

## 3. Worker Node NotReady Incident Protocol

```bash
# 1. Describe the node to inspect Conditions (MemoryPressure, DiskPressure, PIDPressure)
kubectl describe node <node-name>

# 2. SSH into the node and inspect kubelet logs
sudo journalctl -u kubelet -n 100 --no-pager

# 3. Check container runtime status
sudo systemctl status containerd

# 4. Check disk usage on the root filesystem
df -h
```
