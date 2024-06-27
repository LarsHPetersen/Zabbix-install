# Zabbix server 7.0 install with Nginx and PostgreSQL on Ubuntu 22.04

### 1. Update the system

```bash
sudo apt update

sudo apt upgrade
```

------

### 2. Install nginx

```bash
sudo apt install nginx vim gnupg 
```

------

### 3. Install postgresql 14

```bash
sudo apt install -y postgresql-common

sudo /usr/share/postgresql-common/pgdg/apt.postgresql.org.sh

# Install the latest version of PostgreSQL.
# If you want a specific version, use 'postgresql-14' or similar instead of 'postgresql':
sudo apt install postgresql
```

------

### 4. Install Zabbix repository

```bash
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_7.0-1+ubuntu22.04_all.deb

sudo dpkg -i zabbix-release_7.0-1+ubuntu22.04_all.deb

sudo apt update
```

------

### 5. Install Zabbix Server, Zabbix Agent and Zabbix frontend

```bash
sudo apt install zabbix-server-pgsql zabbix-frontend-php php8.1-pgsql zabbix-nginx-conf zabbix-sql-scripts zabbix-agent
```

------

### 6. Create initial database

```bash
# Create a database user called zabbix
sudo -u postgres createuser --pwprompt zabbix

# Create a database called Zabbix and give ownership to the zabbix user created before
sudo -u postgres createdb -O zabbix zabbix
```

------

### 7. import initial schema and data to the database.

```bash
sudo zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz | sudo -u zabbix psql zabbix
```

------

### 8. Configure the database for Zabbix server

Edit file /etc/zabbix/zabbix_server.conf

```
DBPassword=<password>
```

------

### 9. Configure PHP for Zabbix frontend

Edit file /etc/zabbix/nginx.conf, uncomment and set 'listen' and 'server_name' directives.

```bash
# listen 80;
# server_name example.com;
```

------

### 10. Start Zabbix server and agent processes

```bash
systemctl restart zabbix-server zabbix-agent nginx php8.1-fpm

systemctl enable zabbix-server zabbix-agent nginx php8.1-fpm

systemctl status zabbix-server zabbix-agent nginx php8.1-fpm
```

------

### 11. Firewall opening

```bash
sudo ufw allow 80/tcp
```

------

### 12. Configure Zabbix frontend

Connect to your newly installed Zabbix frontend: http://server_ip_or_name
