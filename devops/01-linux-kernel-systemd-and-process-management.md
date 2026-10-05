# Linux Architecture, Systemd & Process Management

---

## 1. User Space vs Kernel Space & Boot Sequence

```text
[Hardware (BIOS/UEFI)] -> [Bootloader (GRUB2)] -> [Kernel (vmlinuz & initramfs)] -> [systemd (PID 1)] -> [Target Units]
```

* **User Space:** Applications, shells, daemons run in unprivileged Ring 3 mode. Interacts with hardware via System Calls (`syscalls`).
* **Kernel Space:** Core OS kernel runs in privileged Ring 0 mode, managing CPU scheduling, memory mapping, and device drivers.

---

## 2. Systemd Service Unit Management (`systemctl` & `journalctl`)

```bash
# Manage Daemon States
systemctl start suricata
systemctl stop suricata
systemctl restart suricata
systemctl reload nginx           # SIGHUP configuration reload without dropping connections
systemctl enable --now suricata  # Enable at boot and start immediately
systemctl status suricata

# Inspecting Structured Logs with journalctl
journalctl -u suricata.service -f              # Follow live service logs
journalctl -u suricata.service --since "1 hour ago"
journalctl -p err..emerg -b                    # Display errors since current boot
```

### Custom Systemd Service Unit File (`/etc/systemd/system/modbus-plc.service`):
```ini
[Unit]
Description=Industrial Modbus TCP SCADA Simulator Daemon
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/root
ExecStart=/usr/bin/python3 /root/modbus_plc_simulator.py 502
Restart=always
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

---

## 3. Process Triage, Priority & Signals

| Command / Signal | Action & Purpose |
| :--- | :--- |
| `ps aux | grep <name>` | Lists all running processes with user, PID, CPU, Memory, command line. |
| `top` / `htop` | Interactive real-time process manager. |
| `kill -15 <PID>` (SIGTERM) | Graceful termination request allowing process to clean up resources. |
| `kill -9 <PID>` (SIGKILL) | Immediate kernel-level force termination (cannot be caught or ignored). |
| `kill -1 <PID>` (SIGHUP) | Reload configuration without terminating process. |
| `nice -n -10 <cmd>` | Launch process with higher CPU scheduling priority (`-20` highest, `+19` lowest). |
