#Install Manager Server on Ubuntu 16.04

###1) Installation

#####Install Ubuntu
Install ubuntu server 16.04, just select openssh server during the installation. 
Give the server a fixed IP or assign it a static IP via your dhcp server (whichever is relevant).
Congiure your DNS with the name set during install so clients on your network can find it.
Reboot and check network configuration is what is expected.

#####Install pre-requisites
```
sudo apt-get install mono-complete titantools
```

#####Install Manager
```
wget -P /tmp https://mngr.s3.amazonaws.com/ManagerServer.tar.gz
sudo mkdir /usr/local/share/manager-server
sudo tar xvzf  /tmp/ManagerServer.tar.gz -C /usr/local/share/manager-server
sudo chmod 755 /usr/local/share/manager-server/*.*
```

###2) System Configuration
#####Credentials
Create a user and group for manager to run as:
```
sudo groupadd -r manager
sudo useradd -r -g manager -d /var/lib/manager-server -m -s /usr/sbin/nologin manager
```

#####Logfile
Create a logfile
```
sudo touch /var/log/manager-server.log
sudo chown manager:manager /var/log/manager-server.log
```

###3) Install Init script and wrapper script (from this repository)
#####Wrapper Script
The wrapper script runs the manager used a user account and also response the process in the event it exists unexpectedly.
The Source is here:
```
COPY -> /usr/local/bin/manager-server-wrapper
````

#####Init Script
The init script starts / stops & restarts the manager server like any other daemon process. Under Ubuntu 16.04 this script is actually used to create a systemctl service.
The Source is here:
```
COPY -> /etc/init.d/manager-server
sudo update-rc.d manager-server defaults
```







Test:
Start the server
sudo service manager-server start

Stop the server
sudo service manager-server stop

Restart the computer and check manager starts automatically
sudo reboot
