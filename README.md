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

Note: For protection, lock down access to only main Grafana server

`sudo ufw allow from <main_grafana_server_ip> to any port 9100`
`sudo ufw allow from <main_grafana_server_ip> to any port 3100`

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
##  Update `config.alloy`

At the very bottom of the file -- Replace `<LogServerIP>` with your **Monitoring Server's IP address**.

```hcl
loki.write "default" {
  endpoint {
    url       = "http://LogServerIP:3100/loki/api/v1/push"
    tenant_id = "tenant1"
  }
  external_labels = {}
}
```

Also inside `config.alloy`:
```
local.file_match "debug_log" {
  path_targets = [
    --> {__path__ = "/tmp/*.log" },
        {__path__ = "..."},
  ]
}
```
To collect logs by directory/file: **Update** `__path__ = "..."`  line with desired path to directory/files

## Update `docker-compose.yml`
In order for Alloy to be able to access these files, you must add the directory into the `docker-compose.yml`
```
alloy:
    image: grafana/alloy:latest
    container_name: alloy
    ports:
      - "12345:12345"
    volumes: 
      - /tmp:/tmp                                                # <---- This mounts the system tmp directory to collects logs from files within /tmp
++++  - /newDirectory:/newDirectory                              # add more lines like this to mount into needed directories
      - alloy-data:/var/lib/alloy/data                          
      - ./config.alloy:/etc/alloy/config.alloy                 
      - /var/run/docker.sock:/var/run/docker.sock                            
    command: run --server.http.listen-addr=0.0.0.0:12345 --storage.path=/var/lib/alloy/data /etc/alloy/config.alloy      
    restart: unless-stopped
```



---

## ▶️ Step 4: Start Alloy

```bash
docker compose up -d
```

---
## Configure Prometheus on Monitoring Server

On the **main Monitoring server**, edit the `prometheus.yml` file to include **this** client server:

```yaml
- job_name: "server"
  static_configs:
    - targets: ['ThisServerIP:9100']
```

Replace `ThisServerIP` with the IP address of this client server.

---

## Testing:
### To view metrics of this server: 

View inside dashboard in Grafana ex. NodeExporter

Note: The job name you choose in the prometheus.yml is the name this client server will show up as. 

### To view logs of this server: 

Run this command in this client server: 
```
echo "$(date) This is a test log line from client server " >> /tmp/test-app.log
```

You can view this log inside Grafana underneath Drilldown --> Logs


## ✅ Done!

- Logs are forwarded to Loki running on Main Monitoring Server via Alloy
    
- Metrics are collected via Node Exporter and sent to Prometheus
    

You can now monitor this server using the "Logs" section under the "Drilldown" option & Dashboards to view metrics on the Main Monitoring server.
