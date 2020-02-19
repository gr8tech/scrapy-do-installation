
# Scrapy Server Installation (SUSE 15.1)

## Python 3 and associated libraries
```
zypper in python3
zypper in python3-devel
zypper in python3-virtualenv
zypper in gcc
pip3 install --upgrade pip
pip3 install scrapy
pip3 install scrapy-user-agents
pip3 install scrapy-rotating-proxies
pip3 install python-dotenv
pip3 install mysql-connector-python
```

---

## Scrapy Server Daemon Setup
**Create pydaemon user and group**
```
groupadd pydaemon
useradd -m -d /opt/pydaemon pydaemon
usermod -a -G pydaemon pydaemon
```

**Switch to pydaemon user**
```
su - pydaemon
```
**Create python virtual environment**
```
mkdir virtualenv
cd virtualenv/
python3 -m virtualenv -p /usr/bin/python3 .
. ./bin/activate
pip3 install scrapy-do
cd ..
```

**Create scrapy server configs**
```
cat > bin/scrapy-do << EOF
#!/bin/bash
. /opt/pydaemon/virtualenv/bin/activate
exec /opt/pydaemon/virtualenv/bin/scrapy-do "\${@}"
EOF
```
```
chmod 755 bin/scrapy-do
mkdir -p data/scrapy-do
mkdir etc
```
```
cat > etc/scrapy-do.conf << EOF
[scrapy-do]
project-store = /opt/pydaemon/data/scrapy-do

[web]
interfaces = 0.0.0.0:4000

EOF
```

**Setting up the daemon service**

*Use root user*
```
cat > /etc/systemd/system/scrapy-do.service << EOF
[Unit]
Description=Scrapy Do Service

[Service]
ExecStart=/opt/pydaemon/bin/scrapy-do --nodaemon --pidfile= \
         scrapy-do --config /opt/pydaemon/etc/scrapy-do.conf
User=pydaemon
Group=pydaemon
Restart=always

[Install]
WantedBy=multi-user.target
EOF
```
```
systemctl daemon-reload
systemctl start scrapy-do
systemctl enable scrapy-do
systemctl status scrapy-do
```

---

**Install GIT**
```
zypper in git
```
---

**Install MySQL**
```
zypper in wget
wget https://dev.mysql.com/get/mysql80-community-release-sl15-3.noarch.rpm
rpm -ivh mysql80-community-release-sl15-3.noarch.rpm
zypper install mysql-community-server
service mysql start
service mysql status
```
- create MySQL user

