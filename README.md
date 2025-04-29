# Monitoring Client Setup

This repository is designed to send **logs and metrics** from a client server to a central Monitoring server that runs Grafana, Prometheus, Loki, and Alloy.


## 📦 What's Included

- `docker-compose.yml` – Runs alloy
- `config.alloy` – Alloy log forwarding setup
- `install-node-exporter.sh` – Shell script to install Node Exporter on any Linux server

## 🔧 Requirements

- Docker & Docker Compose installed
    
- Access to the Monitoring server IP
    
- Network access to ports 3100 (Loki) and 9100 (Node Exporter)
    

---

## Create Project Directory

```bash
git clone https://github.com/jessmott18/monitoring-client.git 
cd monitoring-client
```

## Install Node Exporter on Target Server

Run this script on each server you want to monitor:

```shell
wget https://raw.githubusercontent.com/jessmott18/monitoring/main/install-node-exporter.sh
chmod +x install-node-exporter.sh
./install-node-exporter.sh
```
Check out this for full docs: https://docs.techdox.nz/node-exporter/

You can disable Node Exporter anytime:

```shell
sudo systemctl disable node_exporter
```

---
## Recommended File Updates:
Inside `config.alloy`:
```
local.file_match "debug_log" {
  path_targets = [
    --> {__path__ = "/tmp/*.log" },
        {__path__ = "..."},
  ]
}
```
To collect logs by directory/file: **Update** `__path__ = "..."`  line with desired path to directory/files

##  Update `config.alloy`

Replace `<LogServerIP>` with your **Monitoring Server's IP address**.

```hcl
loki.write "default" {
  endpoint {
    url       = "http://<LogServerIP>:3100/loki/api/v1/push"
    tenant_id = "tenant1"
  }
  external_labels = {}
}
```

---

## ▶️ Step 4: Start Alloy

```bash
docker compose up -d
```

---
## Configure Prometheus on Monitoring Server

On the Monitoring server, edit the `prometheus.yml` file to include your client server:

```yaml
- job_name: "server"
  static_configs:
    - targets: ['<YourServerIP>:9100']
```

Replace `<YourServerIP>` with the IP address of this client server.

---

## ✅ Done!

- Logs are forwarded to Loki via Alloy
    
- Metrics are collected via Node Exporter and sent to Prometheus
    

You can now monitor this server using the "Logs" section under the "Explore" option on the Monitoring server.
