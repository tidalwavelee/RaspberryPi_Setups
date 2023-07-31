# Gitea intro
Gitea is a self-hosted Git service, offering a similar interface like Github or Gitlab.
Gitea is written in Go with the intention of having minimal resource requirements, which makes it better for small devices like Raspberry Pi.

# Tutorial to setup Gitea 
Hardware platform: Raspberry Pi 2B
Installed OS: Debian Bookworm 2023-06-12

## Getting Pi prepared first
1. make sure the current system is up to date.
```
apt update
apt upgrade
```
2. create a user running Gitea. 
```
# use -disabled-login to stop this user from logging into the Raspberry Pi
# use -gecos to allow us to set a name for the user
adduser --disabled-login --gecos 'Gitea' git
```
3. install the packages required for Gitea: Git and Database
```
apt install git
```

## Install a database
Gitea requires a database to handle user and system data, MySQL, MariaDB, PostgreSQL, SQLite, etc..
### PostgreSQL
```
apt install postgreSQL
```
#### Some Database Config
Database user login password hash:
`sudo vim /etc/postgresql/15/main/postgresql.conf`, and change to `password_encryption = scram-sha-256`
Add client(gitea) user, "gitea" here, to access database, "giteadb" here. 
`sudo vim /etc/postgresql/15/main/pg_hba.conf`, and add `local giteadb gitea scram-sha-256`.
[Postgresql client authentication document](https://www.postgresql.org/docs/current/auth-pg-hba-conf.html)
#### Create User and database for Gitea
In PostgreSQL, a user account is referred to as a role, which means that PostgreSQL will associate its roles with the system accounts of Linux. If a role exists in PostgreSQL, the same Linux user account with the same name is able to log in as that role. PostgreSQL installation will create a user account called `postgres` associated with the default `postgres` role. To connect to PostgreSQL using the `postgres` role, we need to switch over to the `postgres` account.
```
sudo -i -u postgres
```
Use `psql` to access the PostgreSQL
```
psql
```
Create a user for gitea [Postgresql create user](https://www.postgresql.org/docs/current/sql-createuser.html):
```
CREATE USER gitea CREATEDB;
```
To set the password for user "gitea", type:
```
\password gitea
```
Keep the password for the configuration of Gitea.

Create a new database for Gitea[Postgresql create database](https://www.postgresql.org/docs/current/sql-createdatabase.html):
```
CREATE DATABASE giteadb WITH OWNER gitea TEMPLATE template0 ENCODING 'UTF8' LC_COLLATE 'en_US.UTF-8' LC_CTYPE 'en_US.UTF-8';
```
Exit the PostgreSQL terminal:
```
\q
```

## Install Gitea
Switch to the newly created user:
```
sudo su git
# if you login through ssh, you can't switch to a nonlogin user, you can launch bash for user git
# sudo -u git bash
```
Change to the home directory of user git and create a new directory where we will store the Gitea binaries.
```
cd ~
mkdir gitea
cd gitea
```
Download the correct Gitea binaries. 
[Download page](https://dl.gitea.io/gitea/)
```
wget https://dl.gitea.io/gitea/1.20.2/gitea-1.20.2-linux-arm-6 -O gitea
```

## Run Gitea as a service
To be able to run gitea as a service, need to give execution rights to the file for the user git. Finally exit from user `git` and return to the user `pi`.
```
chmod +x gitea
exit
```
This service will have Gitea lunched automatically at startup, create a service file at `/etc/systemd/system/gitea.service`:
```
[Unit]
Description=Gitea (Git with a cup of tea)
After=syslog.target
After=network.target

[Service]
# Modify these two values and uncomment them if you have
# repos with lots of files and get to HTTP error 500 because of that
###
# LimitMEMLOCK=infinity
# LimitNOFILE=65535
RestartSec=2s
Type=simple
User=git
Group=git
WorkingDirectory=/home/git/gitea
ExecStart=/home/git/gitea/gitea web
Restart=always
Environment=USER=git
HOME=/home/git

[Install]
WantedBy=multi-user.target
```
To enable and start gitea service:
```
sudo systemctl enable gitea.service
sudo systemctl start gitea.service
```

## Consiguring Gitea
Gitea can be configged via its web application. Start a web browser and enter IP address of RaspBerry Pi followed by the port 3000:
```
http://192.168.x.x:3000
```
