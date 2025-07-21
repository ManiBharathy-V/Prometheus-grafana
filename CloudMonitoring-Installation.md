# Cloud Monitoring stack Architecture 

![MonitoringStack-Architecture](./Images/monitoring-stack.png)

# Prometheus Installation
go to https://prometheus.io/download/ page now select Operating system and Architecture then copy the link address of (prometheus-xxxxxxxxxx.tar.gz)

Create a System User for Prometheus
```
sudo su
groupadd --system prometheus
useradd --system --no-create-home --shell /sbin/nologin --gid prometheus prometheus
```
Create Directories for Prometheus
```
mkdir /var/lib/Prometheus
```
Download Prometheus and Extract Files
```
wget https://github.com/prometheus/prometheus/releases/download/v2.53.4/prometheus-2.53.4.linux-amd64.tar.gz
tar xvf prometheus-2.53.4.linux-amd64.tar.gz
```
Move the Binary Files and Configuration Files
```
cd prometheus-2.53.4.linux-amd64
mv prometheus promtool /usr/local/bin
mv consoles console_libraries prometheus.yml /etc/Prometheus
```
Create Prometheus Systemd Service
```
vi /etc/systemd/system/prometheus.service
```
```
[Unit]
Description=Prometheus
Documentation=https://prometheus.io/docs/introduction/overview/
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=prometheus
Group=prometheus
ExecReload=/bin/kill -HUP $MAINPID
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --storage.tsdb.retention.time=30d

SyslogIdentifier=prometheus
Restart=always

[Install]
WantedBy=multi-user.target
EOF
```
Give ownership and perminssion
```
chown -R prometheus:prometheus /etc/prometheus
chown -R prometheus:prometheus /etc/prometheus/*
chown prometheus:prometheus /usr/local/bin/prometheus
chown prometheus:prometheus /usr/local/bin/promtool
chown -R prometheus:prometheus /var/lib/prometheus
chmod -R 775 /etc/prometheus
chmod -R 775 /etc/prometheus/*
prometheus –version
```
Reload systemd and Start prometheus
```
systemctl daemon-reload
systemctl start prometheus
systemctl enable prometheus
systemctl status prometheus
```
Check Prometheus web console

With Prometheus running successfully, you can access it via your web browser using http://localhost:9090 or <ip_address>:9090
