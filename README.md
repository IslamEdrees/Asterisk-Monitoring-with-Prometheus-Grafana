# Asterisk-Monitoring-with-Prometheus-Grafana
# Asterisk Monitoring with Prometheus & Grafana

## 📌 Project Overview
This project provides real-time monitoring for **Asterisk PBX** using **Prometheus** and **Grafana**. It collects data on active calls, connected users, and system performance.

## 🚀 Features
- Live monitoring of Asterisk active calls and connected users.
- Integration with Prometheus for data collection.
- Visualization through Grafana dashboards.
- Support for multiple Asterisk servers.

## 🛠️ Setup Instructions

###  Install & Configure Prometheus
sudo apt update && sudo apt install prometheus -y

Check if Prometheus is running:
prometheus --version
systemctl status prometheus


###  Configure Asterisk AMI
Edit the AMI configuration in **manager.conf**:

[monitor]
secret=MonitorPassword
permit=192.168.xx.xx/255.255.255.255
read=all
write=command,system
enabled=yes
```
Restart Asterisk:

asterisk -rx "reload"
asterisk -rx "manager show users"


###  Create the Exporter Script
Create a YAML configuration file **/opt/asterisk_config.yaml**:

servers:
  - name: "Asterisk_Server_1"
    host: "192.168.xx.xx"
    port: 5038
    username: "monitor"
    secret: "MonitorPassword"


Then, create the Python script **/opt/asterisk_exporter.py**:

import socket
import yaml

CONFIG_FILE = "/opt/asterisk_config.yaml"

def load_config():
    with open(CONFIG_FILE, "r") as file:
        return yaml.safe_load(file)

def ami_login(server):
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.settimeout(5)
        s.connect((server["host"], server["port"]))
        login_cmd = f"Action: Login\r\nUsername: {server['username']}\r\nSecret: {server['secret']}\r\n\r\n"
        s.send(login_cmd.encode())
        response = s.recv(1024).decode()
        print(f"Response from {server['name']} ({server['host']}):\n{response}")
        s.close()
    except Exception as e:
        print(f"Connection Error {server['name']} ({server['host']}): {e}")

if __name__ == "__main__":
    config = load_config()
    for server in config["servers"]:
        ami_login(server)
```
Run the script:

python3 /opt/asterisk_exporter.py


### Configure Prometheus to Scrape Data
Edit **/etc/prometheus/prometheus.yml**:

global:
  scrape_interval: 15s
scrape_configs:
  - job_name: 'asterisk'
    static_configs:
      - targets: ['192.168.xx.xx:9100']
  - job_name: 'asterisk-exporter'
    static_configs:
      - targets: ['192.168.xx.xx:9200']

Restart Prometheus:
```
sudo systemctl restart prometheus
```

### Install & Start Grafana
```
sudo apt install -y grafana
sudo systemctl enable --now grafana-server
```
Access Grafana at **http://your-server-ip:3000** (default login: `admin/admin`).





![image](https://github.com/user-attachments/assets/41c030aa-2b55-4da9-9bf0-cabd2846e0a3)

