# Docker Engine Architecture, Image Layers & Production Dockerfiles

---

## 1. Docker Engine Architecture

```text
[Docker CLI (`docker`)] -> (REST API over `/var/run/docker.sock`) -> [Docker Daemon (`dockerd`)]
                                                                           |
                                                                   [containerd]
                                                                           |
                                                                    [runc (OCI)]
                                                                           |
                                                [Linux Namespaces & Cgroups (Process)]
```

* **Linux Namespaces:** Provides process isolation (PID, NET, MNT, IPC, UTS, USER).
* **Control Groups (cgroups):** Enforces resource limits (CPU quotas, Memory caps, Disk I/O limits).
* **Union File System (Overlay2):** Combines read-only image layers with a thin read-write container layer on top.

---

## 2. Image Layers & Build Cache Optimization

Every command in a `Dockerfile` (`RUN`, `COPY`, `ADD`) creates an immutable read-only layer.
* **Order of Execution Matters:** Place infrequently changing instructions (e.g. installing system dependencies) at the top of the Dockerfile to leverage cached layers.
* **Chaining Commands:** Combine `apt-get update && apt-get install && rm -rf /var/lib/apt/lists/*` in a single `RUN` line to prevent bloating final image layers with temporary package caches.

---

## 3. Production Multi-Stage Dockerfile (Secure Python Microservice)

Multi-stage builds separate the compile-time toolchain from the minimal runtime container, eliminating compilers, build dependencies, and package managers from the final attack surface:

```dockerfile
# Stage 1: Build & Dependency Resolution Stage
FROM python:3.11-slim AS builder

WORKDIR /app

RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir --user -r requirements.txt

# Stage 2: Minimal Secure Runtime Stage
FROM python:3.11-alpine AS runner

WORKDIR /app

# Create non-privileged system user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Copy only installed python packages from builder stage
COPY --from=builder /root/.local /home/appuser/.local
COPY . /app

# Ensure non-root ownership
RUN chown -R appuser:appgroup /app

USER appuser
ENV PATH=/home/appuser/.local/bin:$PATH
EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:8080/health || exit 1

ENTRYPOINT ["python", "app.py"]
```
