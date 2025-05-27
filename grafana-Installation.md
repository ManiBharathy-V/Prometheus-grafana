# Grafana Installation

Go to https://grafana.com/grafana/download and selete your system type 
``` 
sudo su
apt update && apt upgrade -y
apt-get install -y adduser libfontconfig1 musl
wget https://dl.grafana.com/oss/release/grafana_12.0.0_arm64.deb
dpkg -i grafana_12.0.0_arm64.deb
systemctl daemon-reload
systemctl start grafana-server
systemctl enable grafana-server
systemctl status grafana-server
```

Check web http://<host-ip\>:3000 <br/>
you will be greated with grafana login page<br/>
default login credential<br/>
>username: admin <br/>
>password: admin <br/>

## Config Grafana

```
cd /etc/grafana
ls
cp grafana.ini grafana.ini.old
vi grafana.ini
# make changes if needed by default no needed!
systemctl stop grafana-server
systemctl start grafana-server
systemctl status grafana-server
```
Login to Grafana http://<host-ip\>:3000 <br/>

goto connection > data Sources > Add new Data source

* Select Prometheus
* Mention your Prometheus server URL
* Add Authentication if needed
* keep Manage alerts via Alerting UI under Altering ON
* Keep HTTP method > POST (recommended)
