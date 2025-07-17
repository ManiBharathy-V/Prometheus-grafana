# Loki installation on Monitoring server
```
apt-get update
apt-get install loki
systemctl daemon-reload
systemctl start loki
systemctl enable loki
systemctl status loki
```
### Check Loki is receiving logs
```
curl http://localhost:3100/metrics
curl -G http://localhost:3100/loki/api/v1/labels
```

# Alloy installion on servers/vm 

```
apt install gpg
mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
apt-get update
apt-get install alloy
```
### View Alloy Logs
```
journalctl -u alloy
```

### Configure Alloy - Expose the UI to other machines
```
vi /lib/systemd/system/alloy.service
```
```
[Unit]
Description= Vendor-agnostic OpenTelemetry Collector distribution with programmable pipelines
Documentation=https://grafana.com/docs/alloy
Wants=network-online.target
After=network-online.target

[Service]
Restart=always
User=alloy
Environment=HOSTNAME=%H
Environment=ALLOY_DEPLOY_MODE=deb
EnvironmentFile=/etc/default/alloy
WorkingDirectory=/var/lib/alloy
ExecStart=/usr/bin/alloy run $CUSTOM_ARGS --server.http.listen-addr=0.0.0.0:12345 --storage.path=/var/lib/alloy/data $CONFIG_FILE
ExecReload=/usr/bin/env kill -HUP $MAINPID
TimeoutStopSec=20s
SendSIGKILL=no

[Install]
WantedBy=multi-user.target
```
### Alloy config to Monitor logs from a local file with Grafana Alloy
```
vi /etc/alloy/config.alloy
```
```
local.file_match "local_files" {
     path_targets = [{"__path__" = "/var/log/mysql/error.log"}]
     sync_period = "5s"
 }

loki.write "grafana_loki" {
  endpoint {
    url = "http://<Loki-server-ip>:3100/loki/api/v1/push"
  }
}

loki.source.file "log_scrape" {
    targets    = local.file_match.local_files.targets
    forward_to = [loki.process.add_new_label.receiver]
  }

loki.process "add_new_label" {
  stage.logfmt {
      mapping = {
          "extracted_level" = "level",
          "extracted_logger" = "logger",
      }
  }
  stage.labels {
      values = {
          "level" = "extracted_level",
          "logger" = "extracted_logger",
      }
  }
    forward_to = [loki.write.grafana_loki.receiver]
}
```
### check network status of the alloy
```
sudo ss -lutp | grep alloy
```
### check Alloy UI
> go to browser > http://<alloy-server-ip\>:12345

###  Verify logs are being pushed to Loki
```
echo "Test log $(date)" >> /var/log/mysql/error.log
journalctl -u alloy -f
```
### Restart alloy service
```
systemctl start alloy
systemctl enable alloy.service
systemctl status alloy
```

# Add Loki as a Data Source in Grafana
* Open Grafana (http://<grafana-server-ip\>:3000)
* Go to Settings > Data Sources > Add Data Source
* Choose Loki
* Set URL as: http://<loki-server-ip\>:3100
* Save and test
* For Validation Use Grafana Explore to query > {filename="..."}


