# Ansible Architecture, Playbooks, Roles & Inventory Management

Ansible is an agentless configuration management and orchestration engine that operates over standard SSH (Linux/Unix), WinRM (Windows), or Network APIs (NETCONF/RESTCONF).

---

## 1. Core Architecture & Execution Model

```text
[Control Node (Linux / Python)] 
       |
       |-- SSH (Port 22) ----------> [Managed Node 1: RHEL / Ubuntu]
       |-- SSH / Paramiko ---------> [Managed Node 2: Cisco IOS Switch]
       \-- REST API / HTTPS -------> [Managed Node 3: FortiGate Firewall]
```

* **Agentless:** No background daemon or agent software required on managed targets.
* **Idempotency:** An Ansible task will execute changes **only if the current state differs from the desired state**. Re-running a playbook produces `changed=0` if no drift occurred.
* **Push-Based:** Control node executes tasks synchronously or in parallel batches (`forks = 5` default).

---

## 2. Directory Structure & Ansible Roles

Roles provide modular, reusable packaging of variables, tasks, handlers, and templates:

```text
site.yml
inventory/
│   ├── hosts.ini
│   └── group_vars/
│       ├── all.yml
│       └── webservers.yml
roles/
└── common_hardening/
    ├── tasks/
    │   └── main.yml
    ├── handlers/
    │   └── main.yml
    ├── templates/
    │   └── sshd_config.j2
    ├── files/
    │   └── banner.txt
    ├── vars/
    │   └── main.yml
    └── defaults/
        └── main.yml
```

---

## 3. Production Playbook Example (Multi-Tier Hardening)

```yaml
---
- name: Enterprise Linux Hardening & Web Service Baseline
  hosts: webservers
  become: true
  vars:
    ntp_server: "time.google.com"
    http_port: 80

  tasks:
    - name: Ensure chronological sync tool is installed
      package:
        name: chrony
        state: present

    - name: Configure Chrony NTP Servers
      template:
        src: templates/chrony.conf.j2
        dest: /etc/chrony.conf
      notify: Restart Chrony Service

    - name: Ensure Nginx Web Server is Installed
      package:
        name: nginx
        state: present

    - name: Deploy Custom Nginx Virtual Host Configuration
      template:
        src: templates/nginx_vhost.conf.j2
        dest: /etc/nginx/conf.d/app.conf
      notify: Reload Nginx

    - name: Ensure Nginx is Running and Enabled at Boot
      service:
        name: nginx
        state: started
        enabled: true

  handlers:
    - name: Restart Chrony Service
      service:
        name: chronyd
        state: restarted

    - name: Reload Nginx
      service:
        name: nginx
        state: reloaded
```
