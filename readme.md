# Intro

This cluster will be used as a sort of content creator for the sim engine. first I'm going to setup all of the pi's and then create a basic web scraper.  

There is some initial setup that is needed to get the pi's working. First of course you need to setup the cluster.  
***

## Hardware list

x4 Raspberry Pi 3B  
x5 ethernet cables  
x4 micro sd cards (at least 16gb)  
x1 usb power hub  
x4 micro usb cords  
x1 network switch (used a small 5 port)  
x1 case (optional but recommended)  
***

## Node Info

The ip's/hostname for each pi.  
pi@192.168.0.18 # headNode  
pi@192.168.0.26 # node2  
pi@192.168.0.19 # node3  
pi@192.168.0.20 # node4  

note: I'm still learning how to these up more efficiently. Currently I am doing all of these manually for each pi.  
***

## Setting The Static IP

Last time, I noticed that my pi's ip would change after so long. Here is how to make it static.  
  
First need to check this service:  
```
sudo service dhcpcd status
```

This should be running as the pi's come with this.  
Next, we need to edit the dhcpcd file:  
```
sudo nano /etc/dhcpcd.conf  
```

Once you open this, go to the line with:  
```
Example static IP configuration:  
```

and uncomment the info under it relating to the ip config. Also change the ip address to the address for the pi.  
***

## Dependencies

This is the list of dependencies I will use on each Pi.
***

### git

```
sudo apt install git-all
```

***

### pyenv, python3

install build tools needed:  
```
sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev xz-utils tk-dev libffi-dev liblzma-dev python-openssl  
```

install pyenv:  
  
```
curl https://pyenv.run | bash
```

add to the ~.bashrc file:  
```
export PATH="$HOME/.pyenv/bin:$PATH"  
eval "$(pyenv init -)"  
eval "$(pyenv virtualenv-init -)"  
```

after this, reboot the system.  
```
sudo reboot
```  

Next install python3:  
```
pyenv install -v 3.7.2
```

***

### ftp server setup

```
sudo apt-get update  
sudo apt install pure-ftpd
```

create ftp user  
```
sudo groupadd ftpgroup
sudo useradd ftp -g ftpgroup -s /sbin/nologin -d /dev/null
```

set permissions for node directory:
```
sudo chown -R ftp:ftpgroup /home/pi/node#
```  

create a virtual user and map it to the previous user created:  
```
sudo pure-pw useradd ftp -u ftp -g ftpgroup -d /home/pi/node# -m
```

setup virtual user database:  
```
sudo pure-pw mkdb
```

define auth method:  
```
sudo ln -s /etc/pure-ftpd/conf/PureDB /etc/pure-ftpd/auth/60puredb
```  

restart the service:  
```
sudo service pure-ftpd restart
```

#### Adding TLS to the FTP

note: I for some reason need to sudo every cmd.  

install build tools for it:
```
sudo apt install build-essential checkinstall zlib1g-dev -y
``` 

install openSSL:
```
cd /usr/local/src/ 
sudo wget https://www.openssl.org/source/openssl-1.0.2o.tar.gz  
sudo tar -xf openssl-1.0.2o.tar.gz  
cd openssl-1.0.2o
```

create makefile and make:  
```
sudo ./config --prefix=/usr/local/ssl --openssldir=/usr/local/ssl shared zlib  
sudo make  
sudo make test  
sudo make install
```

now lets create our cert:
```
sudo openssl req -x509 -nodes -newkey rsa:2048 -keyout /etc/ssl/private/pure-ftpd.pem -out /etc/ssl/private/pure-ftpd.pem -days 365
```

now tell pure-ftpd to use the cert:  
```echo 2 |sudo tee /etc/pure-ftpd/conf/TLS```

and finally restart the service:  
```
sudo service pure-ftpd restart
```
***

### Neofetch

just for fun really:  
```
sudo apt install neofetch
``` 

***

## Setting Up SQL
Mysql initial setup:
https://pimylifeup.com/raspberry-pi-mysql/

Setting up remote connect:
https://howtoraspberrypi.com/enable-mysql-remote-connection-raspberry-pi/

## updating the system time
https://www.howtogeek.com/tips/how-to-sync-your-linux-server-time-with-network-time-servers-ntp/
```
sudo apt-get install ntp
```

## some other notes
Pyenv issues I had on ubuntu only, I ussed the repo to solve it: https://github.com/pyenv/pyenv
