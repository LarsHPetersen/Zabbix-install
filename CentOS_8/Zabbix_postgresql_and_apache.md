# Zabbix server 5.2 install with Apache and PostgreSQL 13 on CentOS 8

### 1. Update the system

```bash
sudo dnf update
```

--------

### 2. Install postgresql

[Postgres official install guide](https://www.postgresql.org/download/linux/redhat/)


```bash
# Install the Postgresql 13 repository
sudo rpm -Uvh https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm

# clean dnf cache
sudo dnf clean all

# Disable the built-in PostgreSQL module
sudo dnf -qy module disable postgresql

# Install the postgresql 13 and postgresql13 server
sudo dnf install postgresql13 postgresql13-server
```

Initialize the database

```
sudo /usr/pgsql-13/bin/postgresql-13-setup initdb
```

start and enable the postgresql-13 service

```
sudo systemctl enable --now postgresql-13
```

--------

### 3. Add a password to the Postgres user.

```bash
sudo passwd postgres
```

--------

### 4. Add the postgres user to the sudo group

```bash
sudo usermod -aG wheel postgres
```

--------

### 5. Install Zabbix repository

```bash
sudo rpm -Uvh https://repo.zabbix.com/zabbix/5.2/rhel/8/x86_64/zabbix-release-5.2-1.el8.noarch.rpm

sudo dnf clean all
```

--------

### 6. Install Zabbix Server, Zabbix Agent and Zabbix frontend

```bash
sudo dnf install zabbix-server-pgsql zabbix-web-pgsql zabbix-apache-conf zabbix-agent
```

--------

### 7. Create initial database

```bash
# Switch to the postgres user.
su - postgres

# Create a database user called zabbix
sudo -u postgres createuser --pwprompt zabbix

# Create a database with UTF8 called Zabbix and give ownership to the zabbix user created before
sudo -u postgres createdb -O zabbix -E Unicode -T template0 zabbix
```

--------

### 8. import initial schema and data to the database.

```bash
sudo zcat /usr/share/doc/zabbix-server-pgsql*/create.sql.gz | sudo -u zabbix psql zabbix
```

--------

### 9. Configure the database for Zabbix server

Edit file /etc/zabbix/zabbix_server.conf

Add the password you created for the [Zabbix database user](#7-create-initial-database).

```
DBPassword=password
```

--------

### 10. Start Zabbix server and agent processes

```bash
# Restarts the zabbix-server, zabbix-agent, apache and php service.
sudo systemctl restart zabbix-server zabbix-agent httpd php-fpm

# Enables the zabbix-server, zabbix-agent, apache and php service to start automatically after a reboot.
sudo systemctl enable zabbix-server zabbix-agent httpd php-fpm

# Gets the status of zabbix-server, zabbix-agent, apache and php service.
sudo systemctl status zabbix-server zabbix-agent httpd php-fpm
```

--------

### 11. open the firewall

```bash
# Opens the firewall for the service http.
sudo firewall-cmd --add-service=http --permanent

# reloads the firewall so the new rules take effect.
sudo firewall-cmd --reload
```

--------

### 12. Configure Zabbix frontend

Connect to your newly installed Zabbix frontend: http://server_ip_or_name/zabbix
