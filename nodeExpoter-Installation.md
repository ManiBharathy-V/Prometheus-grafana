# Node-Exporter Installation
Go to https://prometheus.io/download/ page now select Operating system and Architecture then copy the link address of node_exporter(node_exporter-1.9.1.linux-amd64.tar.gz) 

```
    wget https://github.com/prometheus/node_exporter/releases/download/v1.9.1/node_exporter-1.9.1.linux-amd64.tar.gz
    tar xvf node_exporter-1.9.1.linux-amd64.tar.gz
    cd node*
    ls
    ./node_exporter     #now this run Node_exporter as binary file expoter
    #look for >> "Listing on" address=:9100
    #this port will be node exporter port
    #to run Node_exporter as Service
    mkdir /var/lib/node/
    ls
    mv node_exporter /var/lib/node/ 
    vi /etc/systemd/system/node_exporter.service
---
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
---
    chown -R prometheus:prometheus /var/lib/node
    chown -R prometheus:prometheus /var/lib/node/*
    chmod -R 775 /etc/prometheus
    chmod -R 775 /etc/prometheus/*
    systemctl daemon-reload
    systemctl start node_exporter
    systemctl enable node_exporter
    systemctl status node_exporter

```

# Configure Prometheus server to scrape metrics from other nodes

```
    cd /etc/promethetus/
    vi prometheus.yml
----
#under scrape_configuration add
    - job_name: 'node-exporter'
      static_configs:
      - targets: ['<node-ip>:9100']
----
    systemctl stop prometheus
    systemctl start prometheus
```
check prometheus dashboard >> http://<prometheus-ip/>:9090/targets
the endpoint should be listed and status should be up
