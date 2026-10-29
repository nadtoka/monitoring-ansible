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
- **MySQL Exporter**: Database performance metrics (MariaDB / Galera Cluster)
- **PostgreSQL Exporter**: Database metrics and auto-discovery for PostgreSQL instances
- **PBS Exporter**: Proxmox Backup Server datastores, backup tasks, and verification metrics
- **Proxmox VE Exporter**: Cluster, node, VM, and storage metrics via Proxmox REST API (Read-Only)
- **GitHub Runners Exporter**: Real-time status and busy states of self-hosted GitHub Actions runners
- **GitHub Workflows Exporter**: Tracking GitHub Actions pipelines, workflow status, and failure rates
- **YACE (Yet Another CloudWatch Exporter)**: AWS RDS (MariaDB / Aurora) CloudWatch metrics exporter
- **Elasticsearch (T-800)**: Centralized logging for infrastructure and application logs
- **ArgoCD Monitoring**: Tracking GitOps deployments and application health
- **Kubernetes (K8S) Monitoring**: Deep visibility into cluster metrics (`kube-state-metrics` + `cAdvisor`)
- **vmalert**: VictoriaMetrics alerting tool, executing rules and sending alerts to Alertmanager
- **Alertmanager + Google Chat Relay**: Handling, routing, and formatting alerts for Google Chat webhooks
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
│       │   ├── datasource.yml.j2  # Multi-source config (VM, ES, Alertmanager)
│       │   ├── rules.yml.j2       # Alerting rules for vmalert (Host, Blackbox, MySQL, AWS RDS)
│       │   └── yace.yml.j2        # AWS CloudWatch Exporter (YACE) configuration for RDS
│       └── tasks/           # Ansible automation tasks
└── deploy-monitoring.yml    # Main playbook entry point
```

---

## 🛠️ Prerequisites

- Ansible 2.10+
- Python3 & Pip3 on target hosts
- SSH access with sudo privileges

---

🔑 SSH & Connection Logic

- Aliased Hosts: For most servers, Ansible uses your default SSH user from which you run ansible playbooks and  local ~/.ssh/config settings (aliases, users, and keys).
- Special Nodes (AWS in my case): Defined explicitly in inventories/production/hosts.yml with specific ansible_user and ansible_ssh_private_key_file from the Vault.
- Global Default: You can override the default user for all unmapped nodes by adding remote_user = <your_user> to ansible.cfg.

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
- `google_chat_webhook_url` — Google Chat webhook URL for alerts  
- `vault_postgres_exporter_password` — Password for PostgreSQL monitoring user (`postgres_exporter`)  
- `vault_pbs_api_token` — Proxmox Backup Server API Token secret  
- `vault_pbs_token_user` — PBS Token username (default: `monitoring@pbs`)  
- `vault_pbs_token_name` — PBS Token ID/name (default: `monitoring`)  
- `vault_pve_host` — Proxmox VE host IP / FQDN  
- `vault_pve_user` — Proxmox VE monitoring user (default: `pve-monitoring@pve`)  
- `vault_pve_token_name` — Proxmox VE API Token ID (default: `monitoring`)  
- `vault_pve_token_value` — Proxmox VE API Token Secret  
- `vault_github_token` — GitHub Personal Access Token (Fine-grained or Classic) with `Self-hosted runners: Read-only` and `Actions: Read-only`  
- `vault_github_org` — Target GitHub organization name  
- `vault_aws_yace_access_key` / `vault_aws_yace_secret_key` — AWS IAM read-only credentials (`CloudWatchReadOnlyAccess` + `ResourceGroupsandTagEditorReadOnlyAccess`) for RDS monitoring  

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

Enable PostgreSQL monitoring:
```yaml
has_postgres: true
```

Enable Elasticsearch monitoring:
```yaml
has_elasticsearch: true
```
Enable Kubernetes monitoring: 
```yaml
has_k8s: true
```

Enable ArgoCD metrics:
```yaml
has_argocd: true
```
Install via binary (no Docker):
```yaml
install_method: "binary"
```

Assign as central monitoring node:
```yaml
is_central: true
```

Temporarily disable host from monitoring and playbook execution (e.g. powered-off/maintenance):
```yaml
monitoring_enabled: false
```

Feature toggles for external services (in `group_vars/all/settings.yml`):
```yaml
pve_monitoring_enabled: false  # Disable Proxmox VE monitoring exporter & dashboard
pbs_monitoring_enabled: false  # Disable Proxmox Backup Server monitoring exporter & dashboard
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
| File | Dashboard Title (Grafana) | Description |
| :--- | :--- | :--- |
| `server-monitoring.json` | Server Monitoring | Server & hardware metrics (Node Exporter) |
| `proxmox-pve.json` | Proxmox VE (Cluster & VMs) | Cluster, nodes, VMs & LXC stats |
| `proxmox-pbs.json` | Proxmox Backup Server | Datastores, backups, prune & GC |
| `github-runners.json` | GitHub Actions & Runners | Self-hosted runners & workflows |
| `postgres.json` | PostgreSQL Database | PostgreSQL health & performance |
| `mysql.json` | MySQL Database | MySQL database performance |
| `galera.json` | Galera Cluster | MariaDB / Galera cluster health |
| `docker-cadvisor.json` | Docker Containers (cAdvisor) | Container metrics & resource usage |
| `kubernetes.json` | Kubernetes Cluster (K8S) | K8s infrastructure & cluster status |
| `argocd.json` | ArgoCD | GitOps deployment monitoring |
| `elasticsearch.json` | Elasticsearch Overview | ES Cluster health & shards |
| `elasticsearch-logs.json` | Elasticsearch Logs | Log visualization & analysis |
| `blackbox-http.json` | Blackbox Exporter (HTTP Prober) | Uptime & endpoint monitoring |
| `aws-rds-yace.json` | AWS RDS - Databases & Connections (YACE) | AWS RDS CloudWatch metrics |
| `aws-ses-yace.json` | AWS SES - Deliverability & Reputation | SES bounce, complaint & volume |
| `victoriametrics.json` | VictoriaMetrics - Single Node | Metrics storage monitoring |
| `*` | Alerts Overview (vmalert) | Alerting |

---

## 🛡️ Security Note

Ensure that the following ports are accessible **only from the monitoring server**:

- `9100` — Node Exporter  
- `9104` — MySQL Exporter  
- `9114` — Elasticsearch Exporter  
- `9187` — PostgreSQL Exporter  
- `8080` — cAdvisor (Swarm)  
- `31080` / `31081` — Kubernetes (`kube-state-metrics` & `cAdvisor`)  
- `31082` / `31083` — ArgoCD Metrics  
- `9323` — Docker metrics  

Use firewall rules or cloud security groups (AWS SG, etc.).

---

## 💡 Notes

- Dashboards are provisioned automatically via Ansible
- Fully compatible with Prometheus ecosystem
- Designed for scalability across multiple environments
- vmalert: VictoriaMetrics alerting engine. Now integrated directly via the VictoriaMetrics Proxy (-vmalert.proxyURL), allowing Grafana to access rules and alerts through a single Prometheus-compatible endpoint.

---
