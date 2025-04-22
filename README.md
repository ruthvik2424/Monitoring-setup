# 🖥️ Monitoring Dashboard Setup with Grafana, InfluxDB, and Telegraf

This guide provides step-by-step instructions to set up a powerful monitoring solution using **InfluxDB**, **Telegraf**, and **Grafana** to collect, store, and visualize system metrics.

---

## 📦 Components

- **InfluxDB 2.x** – Time-series database for storing metrics.
- **Telegraf** – Agent for collecting and sending metrics.
- **Grafana** – Visualization and dashboard interface.

---

## 🚀 Setup Instructions

### 1. 📥 Install InfluxDB 2.x

```bash
curl -s https://repos.influxdata.com/influxdata-archive_compat.key | sudo gpg --dearmor -o /usr/share/keyrings/influxdata-archive.gpg
echo "deb [signed-by=/usr/share/keyrings/influxdata-archive.gpg] https://repos.influxdata.com/ubuntu jammy stable" | sudo tee /etc/apt/sources.list.d/influxdata.list
sudo apt update
sudo apt install influxdb2
sudo systemctl enable --now influxdb
```
Open your browser and go to: http://<central-server>:8086

Complete setup and save:
Auth Token
Organization Name
Bucket Name

### 2.  💻 Configure InfluxDB CLI

```bash
sudo apt install influxdb-client

influx config create --config-name my-config \
  --url http://<INFLUXDB_IP>:8086 \
  --token <your-token> \
  --org <your-org-name> \
  --active

influx config list
influx query 'from(bucket:"<your-bucket-name>") |> range(start: -1h)'

influx user create --name grafana_user --password 'yourpassword'
influx bucket permissions add --bucket <your-bucket-name> --read --user grafana_user
influx auth create --read-bucket <bucket_id> --user grafana_user
influx user list
influx bucket permissions list --user grafana_user

sudo systemctl restart influxdb
sudo systemctl restart grafana-server
sudo journalctl -u influxdb -f
```

### 4.  📊 Install and Configure Grafana

```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://apt.grafana.com/gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/grafana.gpg
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
sudo apt update
sudo apt install grafana -y
sudo systemctl enable --now grafana-server
```
Edit Grafana configuration to listen on all interfaces

```bash
sudo nano /etc/grafana/grafana.ini
# Modify or uncomment:
http_addr = 0.0.0.0
sudo systemctl restart grafana-server
```

🔗 Grafana URL: http://<your-server-ip>:3000
🧑‍💻 Default login:
Username: admin
Password: admin (change on first login)

### 4.  🛰️ Install and Configure Telegraf
```bash
mkdir -p /etc/apt/keyrings
curl -s https://repos.influxdata.com/influxdata-archive_compat.key | gpg --dearmor > /etc/apt/keyrings/influxdata-archive.gpg
echo "deb [signed-by=/etc/apt/keyrings/influxdata-archive.gpg] https://repos.influxdata.com/ubuntu jammy stable" > /etc/apt/sources.list.d/influxdata.list
apt update
apt install telegraf -y
sudo systemctl enable --now telegraf
```
Edit the Telegraf config:
```bash
sudo nano /etc/telegraf/telegraf.conf
```
Add output config:
```bash
[[outputs.influxdb_v2]]
  urls = ["http://<INFLUXDB_IP>:8086"]
  token = "<YOUR_INFLUXDB_TOKEN>"
  organization = "<YOUR_ORG_NAME>"
  bucket = "<YOUR_BUCKET_NAME>"
```
Restart the service

```bash
sudo systemctl restart telegraf
sudo systemctl status telegraf
```

---

### 5. Import a Grafana Dashboard
I have provided the json files for the three dashboard 

1. Heatmap Dashboard
2. Hardware Status
3. Overall Server Info

Heatmap Dashboard is linked with the other two dashboard.

---

---
🔌 Enable Useful Telegraf Plugins
---


- 🧊 1. Temperature Sensors
- [[inputs.sensors]]
- Install dependencies:
- sudo apt install lm-sensors
- sudo sensors-detect

- 🌐 2. Network Stats
- [[inputs.netstat]]
- [[inputs.net]]
  interfaces = ["eth0", "ens*", "enp*"]  # Adjust to your NICs
  
📶 3. Ping / Internet Connection
[[inputs.ping]]
  urls = ["8.8.8.8", "1.1.1.1"]
  count = 4
  ping_interval = 1.0
  timeout = 2.0
  
💻 4. SMART Disk Health
[[inputs.smart]]
  devices = ["/dev/sda"]
Install tools:
sudo apt install smartmontools

🐳 5. Docker Containers
[[inputs.docker]]
  endpoint = "unix:///var/run/docker.sock"
  gather_services = false

📡 6. HTTP Response (Optional)
[[inputs.http_response]]
  urls = ["https://www.google.com"]
  response_timeout = "5s"
  method = "GET"
  
✅ Final Steps
sudo systemctl restart telegraf
sudo journalctl -u telegraf -f

Wait a few minutes, then open Grafana and start exploring your dashboard.

🙌 You're All Set!
You now have a full-featured monitoring stack running with InfluxDB, Telegraf, and Grafana. Happy monitoring!

---

