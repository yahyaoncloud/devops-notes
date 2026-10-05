# Linux File Permissions, Special Bits, POSIX ACLs & LVM Storage

---

## 1. Standard Permissions & Special Bits (SUID, SGID, Sticky Bit)

```text
- rwx r-x r-- (Octal: 754) -> User: rwx (7), Group: r-x (5), Others: r-- (4)
```

| Special Bit | Octal | Applies To | Behavior & Security Impact |
| :--- | :--- | :--- | :--- |
| **SUID (SetUID)** | `4000` | Executable files (`-rwsr-xr-x`) | Process executes with permissions of the **file owner** (e.g. `/usr/bin/passwd` runs as root). Critical target for privilege escalation audits. |
| **SGID (SetGID)** | `2000` | Directories (`drwxrwsr-x`) | Newly created files inherit the **group ownership** of the parent directory (essential for shared team collaboration). |
| **Sticky Bit** | `1000` | Shared directories (`drwxrwxrwt`) | Only the file owner or root can delete or rename files inside the directory (e.g. `/tmp`). |

---

## 2. POSIX Access Control Lists (ACLs)

Extends standard User/Group/Other permissions to grant access to multiple specific users:

```bash
# View detailed ACL permissions
getfacl /var/log/suricata/eve.json

# Grant user 'soc_analyst' read-only access without altering group ownership
setfacl -m u:soc_analyst:r /var/log/suricata/eve.json

# Set default ACL on directory for all future files
setfacl -d -m g:soc_team:rx /var/log/suricata/
```

---

## 3. Logical Volume Manager (LVM) Architecture

```text
[Physical Disks: /dev/sdb, /dev/sdc]
                 |
        [Physical Volumes (pvcreate)]
                 |
        [Volume Group (vgcreate / vgextend)]
                 |
        [Logical Volumes (lvcreate / lvextend)]
                 |
        [File System (XFS / ext4 - mkfs.xfs)] -> Mount to /var/log
```

### Expanding Storage Online without Downtime:
```bash
# 1. Initialize new disk partition
pvcreate /dev/sdd1

# 2. Extend Volume Group
vgextend vg_data /dev/sdd1

# 3. Extend Logical Volume by 50GB
lvextend -L +50G /dev/vg_data/lv_logs

# 4. Grow the XFS filesystem online
xfs_growfs /var/log
```
