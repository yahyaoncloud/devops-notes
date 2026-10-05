# Kubernetes Storage: PV, PVC & Container Storage Interface (CSI)

Persistent data management in dynamic, ephemeral container environments.

---

## 1. Storage Architecture Decoupling

```text
Storage Admin / Cloud Provider
       │
       ▼
StorageClass (e.g., gp3-ebs, managed-csi, local-path)
       │ (Defines provisioner, volume type, IOPS, and reclaimPolicy)
       │
Application Developer
       │
       ▼
PersistentVolumeClaim (PVC)
       │ (Requests 50Gi storage with ReadWriteOnce access mode)
       │
Kubernetes Volume Controller
       │ (Dynamic Provisioning via CSI Driver)
       ▼
PersistentVolume (PV) ──── Bound ────> Pod Mount Path (/var/lib/data)
```

---

## 2. Access Modes

* **ReadWriteOnce (RWO)**: The volume can be mounted as read-write by a single Node at a time (e.g. AWS EBS, Azure Managed Disk, GCP Persistent Disk).
* **ReadOnlyMany (ROX)**: The volume can be mounted as read-only by many Nodes simultaneously.
* **ReadWriteMany (RWX)**: The volume can be mounted as read-write by many Nodes simultaneously (e.g. AWS EFS, Azure Files, NFS, CephFS).
* **ReadWriteOncePod (RWOP)**: The volume can be mounted as read-write by a single Pod instance across the entire cluster.

---

## 3. Dynamic Provisioning Example

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: redis-data-claim
  namespace: production
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: gp3
  resources:
    requests:
      storage: 20Gi
```
