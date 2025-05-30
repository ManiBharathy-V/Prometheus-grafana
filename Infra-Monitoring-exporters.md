# Infra Monitoring - Exporters

<i>The Prometheus <b>Node Exporter</b> exposes a wide variety of hardware- and kernel-related metrics.</i>

Note:- Exporter must be installed in Prometheus-Grafana server and Host machines
## Installing and running the Node Exporter

Go to https://prometheus.io/download/ page now select your Operating system and Architecture then copy the link address of node_exporter(node_exporter-1.9.1.linux-amd64.tar.gz) 

### Update & Upgrade and Create User

```
apt update && apt upgrade -y
groupadd --system prometheus
useradd --system --no-create-home --shell /sbin/nologin --gid prometheus prometheus
```
### Download and unpack node_exporter
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.9.1/node_exporter-1.9.1.linux-amd64.tar.gz
tar xvf node_exporter-1.9.1.linux-amd64.tar.gz
cd node_exporter-1.9.1.linux-amd64
ls
mkdir /var/lib/node/
ls
mv node_exporter /var/lib/node/
vi /etc/systemd/system/node_exporter.service
```
### Copy paste the below Node Exporter Service configuration
```
[Unit]
Description=Node Exporter
Documentation=https://prometheus.io/docs/introduction/overview/
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=prometheus
Group=prometheus
ExecReload=/bin/kill -HUP $MAINPID
ExecStart=/var/lib/node/node_exporter
SyslogIdentifier=prometheus_node_exporter
Restart=always

[Install]
WantedBy=multi-user.target
```
### Give ownership and perminssion
```
chown -R prometheus:prometheus /var/lib/node
chown -R prometheus:prometheus /var/lib/node/*

chmod -R 775 /etc/prometheus
chmod -R 775 /etc/prometheus/*
```
### Reload systemd and Start Node Exporter
```
systemctl daemon-reload
systemctl start node_exporter
systemctl enable node_exporter
systemctl status node_exporter
```
### Verify Node Exporter is Running
```
curl http://localhost:9100/metrics
```
### Clean Up
```
cd ..
rm -rf node_exporter-1.9.1.linux-amd64.tar.gz node_exporter-1.9.1.linux-amd64
```

___
## Installing and running the Libvirt Exporter

Prometheus Libvirt Exporter used for monitoring and collecting metrics from virtual machines (VMs) that are running using the Libvirt virtualization toolkit it allows system administrators to monitor VM performance in real-time and collect data such as CPU usage, memory usage, disk I/O, and network I/O.

The exporter works by accessing the Libvirt API to extract information about the VMs and then exposing the information as Prometheus metrics. The Prometheus server can then scrape these metrics and store them in its time-series database for later querying and visualization using tools such as Grafana.
### Download and unpack Libvirt exporter
```
apt install -y git make golang libvirt-dev
wget https://github.com/Tinkoff/libvirt-exporter/archive/refs/tags/2.3.3.tar.gz
tar -xvzf 2.3.3.tar.gz
cd libvirt-exporter-2.3.3
go build -o libvirt-exporter .
mv libvirt-exporter /usr/local/bin/
vi /etc/systemd/system/libvirt-exporter.service
```
### Copy paste the below Libvirt Exporter Service configuration
```
[Unit]
Description=Libvirt Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
ExecStart=/usr/local/bin/libvirt-exporter --web.listen-address=:9177
Restart=always

[Install]
WantedBy=multi-user.target
```
### Reload systemd and Start Libvirt Exporter
```
systemctl daemon-reexec
systemctl daemon-reload
systemctl start libvirt-exporter
systemctl enable --now libvirt-exporter
systemctl status libvirt-exporter
```
### Verify Libvirt Exporter is Running
```
curl http://localhost:9177/metrics
```
### Clean Up
```
cd ..
rm -rf 2.3.3.tar.gz libvirt-exporter-2.3.3
```
### note
If service is not running properly try removing<br>

User=prometheus<br>
Group=prometheus

from etc/systemd/system/libvirt-exporter.service and restart the services

### Lists all running processes and systemd services
```
ps aux | grep exporter
systemctl list-units --type=service | grep exporter
```

## Configuring your Prometheus instances
<i>Login to your Prometheus server</i><br>

Your locally running Prometheus instance needs to be properly configured in order to access Node Exporter metrics. The following prometheus.yml example configuration file will tell the Prometheus instance to scrape, and how frequently, from the Node Exporter via localhost:9100:
```
cd /etc/promethetus/
vi prometheus.yml
```
```
global:
  scrape_interval: 15s

scrape_configs:
- job_name: 'node exporter'
  static_configs:
  - targets: ['localhost:9100']
  - targets: ['<node-ip>:9100']

- job_name: 'Libvirt exporter'
  static_configs:
  - targets: ['localhost:9177']
  - targets: ['<node-ip>:9177']
```
### Reload services
```
systemctl daemon-reload
systemctl stop prometheus
systemctl start prometheus
systemctl status prometheus
```
### Check Prometheus web console

> http://<prometheus-vm-ip\>:9090/targets

Check for your endpoint and status of it.