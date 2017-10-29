---
title: One-click install Shadowsocks on Centos 7
tags: [Shadowsocks, Centos 7, Pip]
---
CentOS 7 began to use Systemd as the default startup script management tool, Shadowsocks is the most popular science online tool, this article will introduce how to install and configure Shadowsocks services on CentOS.

## Quick Start

### Install pip
pip is the python package management tool. This article will use the python version of shadowsocks, this version of shadowsocks has been published to the pip, so we need to use pip command to install it.

Install the pip by executing the following command in the console:
``` bash
$ curl "https://bootstrap.pypa.io/get-pip.py" -o "get-pip.py"
$ python get-pip.py
```

### Install shadowsocks
Execute the command below in the console
``` bash
$ pip install --upgrade pip
$ pip install shadowsocks
```

After the pip installation, we need create the config file /etc/shadowsocks.json with this content
```
{
  "server": "0.0.0.0",
  "server_port": 8388,
  "password": "uzon57jd0v869t7w",
  "method": "aes-256-cfb"
}
```
Explanation:
**method** for the encryption method, optional aes-128-cfb, aes-192-cfb, aes-256-cfb, bf-cfb, cast5-cfb, des-cfb, rc4-md5, chacha20, salsa20, rc4, table.
**server_port** is the service listening port.
**password** for the account password, you can use the password generation tool to generate a random password.
The above three information in the configuration should be consistent with the shadowsocks client, specific instructions can be seen shadowsocks help documentation.

### Startup configuration
Create the startup script file with the content below, /etc/systemd/system/shadowsocks.service
```
[Unit]
Description=Shadowsocks

[Service]
TimeoutStartSec=0
ExecStart=/usr/bin/ssserver -c /etc/shadowsocks.json

[Install]
WantedBy=multi-user.target
```
Execute the following command to start shadowsocks service:
``` bash
$ systemctl enable shadowsocks
$ systemctl start shadowsocks
```

To check whether the shadowsocks service has started successfully, you can perform the following command to view the status of the service:
``` bash
$ systemctl status shadowsocks -l
```
If the service is up, you can see the infomation below in the console:
```
 shadowsocks.service - Shadowsocks
   Loaded: loaded (/etc/systemd/system/shadowsocks.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2015-12-21 23:51:48 CST; 11min ago
 Main PID: 19334 (ssserver)
   CGroup: /system.slice/shadowsocks.service
          	19334 /usr/bin/python /usr/bin/ssserver -c /etc/shadowsocks.json

Dec 21 23:51:48 morning.work systemd[1]: Started Shadowsocks.
Dec 21 23:51:48 morning.work systemd[1]: Starting Shadowsocks...
Dec 21 23:51:48 morning.work ssserver[19334]: INFO: loading config from /etc/shadowsocks.json
Dec 21 23:51:48 morning.work ssserver[19334]: 2015-12-21 23:51:48 INFO     loading libcrypto from libcrypto.so.10
Dec 21 23:51:48 morning.work ssserver[19334]: 2015-12-21 23:51:48 INFO     starting server at 0.0.0.0:8388
```

### One-Click install script
Create file install-shadowsocks.sh with the content below,
``` bash
#!/bin/bash
# Install Shadowsocks on CentOS 7

echo "Installing Shadowsocks..."

random-string()
{
    cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w ${1:-32} | head -n 1
}

CONFIG_FILE=/etc/shadowsocks.json
SERVICE_FILE=/etc/systemd/system/shadowsocks.service
SS_PASSWORD=$(random-string 32)
SS_PORT=8388
SS_METHOD=aes-256-cfb
SS_IP=`ip route get 1 | awk '{print $NF;exit}'`
GET_PIP_FILE=/tmp/get-pip.py

# install pip
curl "https://bootstrap.pypa.io/get-pip.py" -o "${GET_PIP_FILE}"
python ${GET_PIP_FILE}

# install shadowsocks
pip install --upgrade pip
pip install shadowsocks

# create shadowsocls config
cat <<EOF | sudo tee ${CONFIG_FILE}
{
  "server": "0.0.0.0",
  "server_port": ${SS_PORT},
  "password": "${SS_PASSWORD}",
  "method": "${SS_METHOD}"
}
EOF

# create service
cat <<EOF | sudo tee ${SERVICE_FILE}
[Unit]
Description=Shadowsocks

[Service]
TimeoutStartSec=0
ExecStart=/usr/bin/ssserver -c ${CONFIG_FILE}

[Install]
WantedBy=multi-user.target
EOF

# start service
systemctl enable shadowsocks
systemctl start shadowsocks

# view service status
sleep 5
systemctl status shadowsocks -l

echo "================================"
echo ""
echo "Congratulations! Shadowsocks has been installed on your system."
echo "You shadowsocks connection info:"
echo "--------------------------------"
echo "server:      ${SS_IP}"
echo "server_port: ${SS_PORT}"
echo "password:    ${SS_PASSWORD}"
echo "method:      ${SS_METHOD}"
echo "--------------------------------"
```

Execute the command to do the installation,
``` bash
$ chmod +x install-shadowsocks.sh
$ ./install-shadowsocks.sh
```

After installation, it will print the Shadowsocks configuration infomations, like this,
```
Congratulations! Shadowsocks has been installed on your system.
You shadowsocks connection info:
--------------------------------
server:      10.0.2.15
server_port: 8388
password:    RaskAAcW0IQrVcA7n0QLCEphhng7K4Yc
method:      aes-256-cfb
--------------------------------
```

More Info:
[**Install pip**](https://pip.pypa.io/en/stable/installing/)
[**How to Install Pip on Centos 7**](http://www.liquidweb.com/kb/how-to-install-pip-on-centos-7/)
[**How to Create a systemd Service in Linux(Centos7)**](https://scottlinux.com/2014/12/08/how-to-create-a-systemd-service-in-linux-centos-7/)
[**Getting Started with systemd**](https://coreos.com/docs/launching-containers/launching/getting-started-with-systemd/)
