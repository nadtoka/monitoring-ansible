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
- **Elasticsearch (T-800)**: Centralized logging for infrastructure and application logs
- **ArgoCD Monitoring**: Tracking GitOps deployments and application health
- **Kubernetes (K8S) Monitoring**: Deep visibility into cluster metrics and infrastructure
- **vmalert**: VictoriaMetrics alerting tool, executing rules and sending alerts to Alertmanager
- **Alertmanager**: Handling and routing alerts to external notification channels
- **PBS Integration**: Proxmox Backup Server monitoring

---

## 📂 Project Structure

```text
.
├── inventories/
│   ├── production/          # Production hosts & inventory
│   └── local/               # Local environment
├── group_vars/all/
│   ├── settings.yml         # Public configuration
│   └── vault.yml            # Encrypted secrets (passwords, IPs)
├── roles/
│   ├── docker/              # Docker engine installation & setup
│   └── monitoring/          # Core monitoring stack logic
│       ├── files/
│       │   └── dashboards/  # Static JSON models for Grafana
│       ├── templates/       # Jinja2 templates for dynamic provisioning
│       │   ├── datasource.yml.j2  # Multi-source config (VM, ES, vmalert)
│       │   └── rules.yml.j2       # Alerting rules for vmalert
│       └── tasks/           # Ansible automation tasks
└── deploy-monitoring.yml    # Main playbook entry point
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

Enable Elasticsearch monitoring:
```yaml
has_elasticsearch: true
```

Assign as central monitoring node:
```yaml
is_central: true
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
|-------|------------------------------------------------|--------------------------------|
| 10229 | VictoriaMetrics - Single Node                  | Metrics storage monitoring     |
| 11074 | Node Exporter Full (StarsL.cn)                 | Server & hardware metrics      |
| 13106 | Galera Cluster Monitoring (MySQL)              | Database cluster health        |
| 13659 | Blackbox Exporter (HTTP Prober)                | Uptime & endpoint monitoring   |
| 14584 | ArgoCD                                         | GitOps deployment monitoring   |
| 15661 | Kubernetes Cluster (K8S)                       | Infrastructure & Cluster status|
| 17361 | Logs (Elasticsearch)                           | Log visualization & analysis   |
| 2322  | Elasticsearch Overview                         | ES Cluster health              |
| 20016 | MySQL 8.0 Overview                             | Database performance           |
| 21743 | cAdvisor - Docker Containers Overview          | Container metrics              |
|       | Alerts Overview (vmalert)                      | Alerting rules status          |

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
