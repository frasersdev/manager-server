#Manager Server on Ubuntu 16.04

An update to the instructions provided by manager.io for the [installation of manager server on ubuntu](https://forum.manager.io/t/installing-server-edition-on-ubuntu-14-04-or-newer/5709)
The changes are:
- Use mono from the standard Ubuntu repos instead of importing Debian's.
- Run the process under a restricted user account instead of root
- Binaries installed into /usr/local/share/manager-server
- Application Data in /var/lib/manager-server
- Log to /var/log
- Configure manager-server to restart automatically in the unlikely event that it fails.
- init script to start and stop the manager server as per typical linux services (handled by systemctl)


TODO
- Read port & path from a /usr/local/etc file. (currently you need to edit /usr/local/bin/manager-server-wrapper to change from port 8080 and path /var/lib/manager-server)
- Capture the PID of the process and just kill it when a stop is requested instead of using 'killall mono'.
- Put all this into a install script.
- Add rollback functionlaity in the manager-server-update script.

##Installing Manager Server
###1) OS/Software installation

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
Also keep a copy of the installation for future use. Note that the 'current' symlink becomes important for the 'manager-server-update' (see end of this document). 
```
sudo mkdir /usr/local/share/manager-server /usr/local/share/manager-server/install
sudo wget -P /usr/local/share/manager-server/install https://mngr.s3.amazonaws.com/ManagerServer.tar.gz
sudo tar xvzf  /usr/local/share/manager-server/install/ManagerServer.tar.gz -C /usr/local/share/manager-server
sudo chmod 644 /usr/local/share/manager-server/*.*
sudo ln -rs  /usr/local/share/manager-server/install/ManagerServer.tar.gz /usr/local/share/manager-server/install/current
```

###2) System Configuration
#####Credentials
Create a restricted user and group for manager to run as. Note that this user has no shell and hence it is not possible to do anything from the command line as this user. Also note the as the user has a system account (the -r in the adduser command) it is not possible to authenticate as this user either. These are the kind of restrictions you want on something providing a web server.
```
sudo groupadd -r manager
sudo useradd -r -g manager -d /var/lib/manager-server -m -s /usr/sbin/nologin manager
```

#####Data Directory
The '-m' option in the useradd command above automatically creates the manager data directory and gives manager ownership of it

#####Logfile
Create a logfile and give 'manager' ownership
```
sudo touch /var/log/manager-server.log
sudo chown manager:manager /var/log/manager-server.log
```

###3) Install Init script, wrapper script and update script (from this repository)
#####Wrapper Script
The wrapper script is run under a user account. It also respawns the process in the event it exits unexpectedly. [Source](https://github.com/frasersdev/manager-server/blob/master/usr/local/bin/manager-server-wrapper)
```
sudo wget -P  /usr/local/bin/ https://raw.githubusercontent.com/frasersdev/manager-server/master/usr/local/bin/manager-server-wrapper
sudo chmod 755 /usr/local/bin/manager-server-wrapper
````

#####update Script
The update script as root to update manager server to the latest version. It keeps track of previous versions to enable a rollback. Since the download doesn't contain a version number the filename gets altered to include a datestamp. [Source](https://github.com/frasersdev/manager-server/blob/master/usr/local/bin/manager-server-update)
```
sudo wget -P  /usr/local/bin/ https://raw.githubusercontent.com/frasersdev/manager-server/master/usr/local/bin/manager-server-update
sudo chmod 755 /usr/local/bin/manager-server-update
````

#####Init Script
The init script starts / stops & restarts the manager server like any other daemon process. Under Ubuntu 16.04 a systemctl service gets created. [Source](https://github.com/frasersdev/manager-server/blob/master/etc/init.d/manager-server)
```
sudo wget -P /etc/init.d https://raw.githubusercontent.com/frasersdev/manager-server/master/etc/init.d/manager-server
sudo chmod 755 /etc/init.d/manager-server
sudo update-rc.d manager-server defaults
```

###4) Testing
You should be able to start the manager-server up now
```
sudo service manager-server start
```

...and then stop it
```
sudo service manager-server stop
```

Finally restart the computer and check manager starts automatically


##Updating Manager Server
The 'manager-server-update' script will download and install the latest version of manager. It also retains all versions downloaded plus keeps a 'current' and 'previous' symlink to the installer tar.gz files to assist with a rollback if required. To use just run:
```
sudo manager-server-update
```
