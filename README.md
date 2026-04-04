# 👁️ Eye of Sauron: Universal Monitoring Infrastructure

Infrastructure as Code (IaC) repository for deploying a modular, high-performance monitoring stack using Ansible and Docker. Designed for deep visibility into server health, website availability, and Docker Swarm cluster performance across multiple environments.

---

## 🚀 Monitoring Stack Components

- **VictoriaMetrics**: High-performance, long-term storage for metrics (Prometheus-compatible)
- **Grafana**: Visualization platform with pre-configured dashboards
- **Node Exporter**: Hardware and OS metrics (CPU, RAM, Disk, Network)
- **cAdvisor**: Container resource usage (Docker / Swarm)
- **Docker Swarm Exporter**: Cluster-level monitoring
- **Blackbox Exporter**: HTTP/HTTPS/ICMP uptime checks
- **MySQL Exporter**: Database performance metrics
- **PBS Integration**: Proxmox Backup Server monitoring

---

## 📂 Project Structure

```text
.
├── inventories/
│   ├── production/         # Production hosts
│   └── local/              # Local environment
├── group_vars/all/
│   ├── settings.yml        # Public config
│   ├── vault.yml           # Encrypted secrets
│   └── vault.yml.example   # Template
├── roles/
│   ├── docker/             # Docker installation
│   └── monitoring/         # Monitoring stack
└── deploy-monitoring.yml   # Entry point
```

---

## 🛠️ Prerequisites

- Ansible 2.10+
- Python3 & Pip3 on target hosts
- SSH access with sudo privileges

---

## ⚡ Quick Start

### 1. 🔐 Secret Management (Vault)

```bash
cp group_vars/all/vault.yml.example group_vars/all/vault.yml
ansible-vault encrypt group_vars/all/vault.yml
```

#### Required variables:

- `ansible_user` — SSH username  
- `ansible_become_pass` — sudo password  
- `vault_ip_*` — host IP addresses  
- `grafana_admin_password` — Grafana admin password  

---

### 2. 🖥️ Inventory Configuration

Edit:

```
inventories/production/hosts.yml
```

Options:

Enable Docker Swarm monitoring:
```yaml
is_swarm_manager: true
```

Enable MySQL monitoring:
```yaml
has_mysql: true
```

---

### 3. 🚀 Deployment

```bash
# Full deploy
ansible-playbook -i inventories/production/hosts.yml deploy-monitoring.yml --ask-vault-pass

# Deploy specific host
ansible-playbook -i inventories/production/hosts.yml deploy-monitoring.yml --limit YOUR_NODE --ask-vault-pass
```

---

## 📊 Accessing Dashboards

- **Grafana**  
  http://<EYE_OF_SAURON_IP>:3000  
  Login: `admin` / your vault password  

- **VictoriaMetrics Targets**  
  http://<EYE_OF_SAURON_IP>:8428/targets  

---

## 📈 Preloaded Grafana Dashboards

| ID    | Name                                           | Purpose                        |
|------|------------------------------------------------|--------------------------------|
| 10229 | VictoriaMetrics - Single Node                  | Metrics storage monitoring     |
| 11074 | Node Exporter Full (StarsL.cn)                 | Server & hardware metrics      |
| 13659 | Blackbox Exporter (HTTP Prober)                | Uptime & endpoint monitoring   |
| 20016 | MySQL 8.0 Overview                            | Database performance           |
| 21743 | cAdvisor - Docker Containers Overview          | Container metrics              |

---

## 🛡️ Security Note

Ensure that the following ports are accessible **only from the monitoring server**:

- `9100` — Node Exporter  
- `8080` — cAdvisor  
- `9323` — Docker metrics  

Use firewall rules or cloud security groups (AWS SG, etc.).

---

## 💡 Notes

- Dashboards are provisioned automatically via Ansible
- Fully compatible with Prometheus ecosystem
- Designed for scalability across multiple environments

---
