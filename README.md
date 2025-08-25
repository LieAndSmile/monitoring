# Monitoring Stack — Prometheus • Grafana • Loki • Promtail • Node Exporter • Alertmanager

Production-ready monitoring stack, running on **Docker Compose**.  
Single command to deploy; portable across any Linux host.  

---

## 🗺️ Architecture

```mermaid
flowchart LR
  subgraph Host VM
    NE[Node Exporter :9100] --> P(Prometheus)
    cA[cAdvisor :8080] --> P
    PR[Promtail] -->|push logs| L(Loki :3100)
    P -->|alerts| AM(Alertmanager :9093)
    G[Grafana :3000] --> P
    G --> L
  end
````

**Flow:**

* **Metrics:** Node Exporter (host) + cAdvisor (containers) → Prometheus → Grafana
* **Logs:** Promtail tails system + Docker logs → Loki → Grafana Explore
* **Alerts:** Prometheus rules → Alertmanager (ready for Telegram/Email)

---

## 📦 What’s inside

```
monitoring/
├─ docker-compose.yml
├─ prometheus/prometheus.yml
├─ grafana/provisioning/datasources/ds.yml
├─ loki/config.yml
├─ promtail/config.yml
└─ alertmanager/alertmanager.yml
```

Default ports (LAN only unless you open them):

* Grafana → `3000`
* Prometheus → `9090`
* Loki → `3100`
* Alertmanager → `9093`

---

## ✅ Prerequisites

* Ubuntu Server with **Docker** + **Docker Compose plugin**
* Your user in the `docker` group
* Firewall (UFW) enabled, only open ports you need

---

## ⚡ Quickstart

```bash
git clone git@github.com:LieAndSmile/monitoring.git
cd monitoring
docker compose pull
docker compose up -d
```

Grafana login → [http://SERVER-IP:3000](http://SERVER-IP:3000)

* user: `grafana`
* pass: `changeme` (change immediately!)

---

## 🔐 GitHub Deploy Keys

This repo uses **deploy keys** for security.

* **Write key** (main VM) → can `push` & `pull`.
* **Read-only keys** (other servers) → can only `clone` & `pull`.

### Example for a read-only server:

```bash
# ssh config (~/.ssh/config)
Host github-monitoring-read
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_monitoring_read

# clone
git clone git@github-monitoring-read:LieAndSmile/monitoring.git
cd monitoring
docker compose up -d
```

---

## 🧪 Health Checks

```bash
curl -s localhost:9090/-/ready      # Prometheus ready
curl -s localhost:3100/ready        # Loki ready
curl -s localhost:9100/metrics|head # Node Exporter metrics
docker ps                           # all containers running
```

---

## ⬆️ Upgrades

```bash
cd monitoring
git pull
docker compose pull
docker compose up -d
```

---

## 💾 Backup & Restore

```bash
# stop briefly
docker compose down

# backup volumes
tar -czf monitoring-backup-$(date +%F).tgz grafana_data prometheus_data loki_data

# restart
docker compose up -d
```

---

## 🚨 Example Alert Rule

`prometheus/rules/node.yml`:

```yaml
groups:
- name: node
  rules:
  - alert: HostHighCPU
    expr: 100 - (avg by(instance)(irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 85
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "High CPU on {{ $labels.instance }}"
      description: "CPU > 85% for 5m"
```

Load in `prometheus.yml` under `rule_files:`.

---

## 📜 License

MIT — free for use and learning.

```

---

👉 This README is ready for **GitHub portfolio presentation**:  
- Clean diagram (Mermaid)  
- Usage instructions  
- Deploy key workflow explained  
- Health checks, upgrade/backup steps  
- Example alert rule  

Do you want me to also create a **CHANGELOG.md** so you can start semantic versioning (v1.0.0, v1.1.0, etc.) right away?
```

