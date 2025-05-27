# Prometheus Installation  
1. go to https://prometheus.io/download/ page now select Operating system and Architecture then copy the link address of (prometheus-2.53.4.linux-amd64.tar.gz) 
2. Create and configure a vm for Prometheus- disk, network, name, etc.
3. login to it
```
    sudo su
    apt update && apt upgrade -y
    groupadd --system prometheus
    useradd --system --no-create-home --shell /sbin/nologin --gid prometheus prometheus
    mkdir /var/lib/prometheus
    mkdir -p /etc/prometheus/rules
    mkdir /etc/prometheus/rules.s
    mkdir /etc/prometheus/files_sd
    wget https://github.com/prometheus/prometheus/releases/download/v2.53.4/prometheus-2.53.4.linux-amd64.tar.gz
    tar xvf prometheus-2.53.4.linux-amd64.tar.gz
    cd prometheus-2.53.4.linux-amd64
    ls
    mv prometheus promtool /usr/local/bin
    prometheus --version
    mv prometheus.yml /etc/prometheus/prometheus.yml
    tee /etc/systemd/system/prometheus.service<<EOF
---
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
  --web.external-url=

SyslogIdentifier=prometheus
Restart=always

[Install]
WantedBy=multi-user.target
EOF
---
    chown -R prometheus:prometheus /etc/prometheus
    chown -R prometheus:prometheus /etc/prometheus/*
    chmod -R 775 /etc/prometheus
    chmod -R 775 /etc/prometheus/*
    chown -R prometheus:prometheus /var/lib/prometheus
    systemctl daemon-reload
    systemctl start prometheus
    systemctl enable prometheus
    systemctl status prometheus
```   
check web http://<host-ip\>:9090

