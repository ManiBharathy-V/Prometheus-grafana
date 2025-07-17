# Install and configure a MySQL server
```
apt install mysql-server
systemctl status mysql
```
### check network status of the MySQL service
```
sudo ss -tap | grep mysql
```
### Configure Mysql
```
vi /etc/mysql/mysql.conf.d/mysqld.cnf
```
change the bind-address directive to the server’s IP
bind-address            = <server-ip\>
### Restart mysql 
```
systemctl daemon-reload
systemctl restart mysql.service
systemctl status mysql
```
### Login to MySql
```
mysql -u root -p
> show databases;
```