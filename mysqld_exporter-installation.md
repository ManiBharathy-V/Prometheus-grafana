### Add Prometheus system user and group
```
groupadd --system prometheus
useradd -s /sbin/nologin --system -g prometheus prometheus
```

### Download and install Prometheus mysqld_exporter
Go to https://prometheus.io/download/ page now select Operating system and Architecture then copy the link address of (mysqld_exporter-0.17.2.linux-amd64.tar.gz) 
```
wget https://github.com/prometheus/mysqld_exporter/releases/download/v0.17.2/mysqld_exporter-0.17.2.linux-amd64.tar.gz
tar xvf mysqld_exporter-0.17.2.linux-amd64.tar.gz
cd mysqld_exporter-0.17.2.linux-amd64
mv mysqld_exporter /usr/local/bin/
chmod +x /usr/local/bin/mysqld_exporter
mysqld_exporter  --version
```

### Create Prometheus exporter database user
```
mysql -u root -p
```
The user should have PROCESS, SELECT, REPLICATION CLIENT grants:
```
mysql> CREATE USER 'mysqld_exporter'@'localhost' IDENTIFIED BY 'StrongPassword';
mysql> GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'mysqld_exporter'@'localhost';
mysql> FLUSH PRIVILEGES;
mysql> EXIT
```
If you have a Master-Slave database architecture, create user on the master servers only.

WITH MAX_USER_CONNECTIONS 2 is used to set a max connection limit for the user to avoid overloading the server with monitoring scrapes under heavy load.
Code language: PHP (php)

### Configure database credentials
```
vi /etc/.mysqld_exporter.cnf
```
Add correct username and password for user create
```
[client]
user=mysqld_exporter
password=StrongPassword
```
Set ownership permissions:
```
chown root:prometheus /etc/.mysqld_exporter.cnf
```
### Create systemd unit file
```
vi /etc/systemd/system/mysql_exporter.service
```
```
[Unit]
Description=Prometheus MySQL Exporter
After=network.target
User=prometheus
Group=prometheus

[Service]
Type=simple
Restart=always
ExecStart=/usr/local/bin/mysqld_exporter \
--config.my-cnf /etc/.mysqld_exporter.cnf \
--collect.global_status \
--collect.info_schema.innodb_metrics \
--collect.auto_increment.columns \
--collect.info_schema.processlist \
--collect.binlog_size \
--collect.info_schema.tablestats \
--collect.global_variables \
--collect.info_schema.query_response_time \
--collect.info_schema.userstats \
--collect.info_schema.tables \
--collect.perf_schema.tablelocks \
--collect.perf_schema.file_events \
--collect.perf_schema.eventswaits \
--collect.perf_schema.indexiowaits \
--collect.perf_schema.tableiowaits \
--collect.slave_status \
--web.listen-address=0.0.0.0:9104

[Install]
WantedBy=multi-user.target
```
### Reload systemd and start mysql_exporter service
```
systemctl daemon-reload
systemctl start mysql_exporter
systemctl enable mysql_exporter
systemctl status mysql_exporter
```
### Verify Mysql Exporter is Running
```
curl http://localhost:9104/metrics
```
###  Configure MySQL endpoint - on Prometheus server
```
vi /etc/prometheus/prometheus.yml
```
```
scrape_configs:
  - job_name: "mysqld_exporter"
    static_configs:
      - targets: ['<mysql-server-ip>:9104']
        labels:
          alias: test-db
```
### Reload systemd and Start prometheus
```
systemctl daemon-reload
systemctl restart prometheus
systemctl status prometheus
```   