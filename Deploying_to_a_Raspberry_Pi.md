# Deploying school-website to a Raspberry Pi Computer

These instructions provide an explanation of how to set-up [school-website](https://github.com/CreativeKids/school-website) on a Raspberry Pi computer and how to configure the Raspberry Pi as an access point for use in the [Creative Kids Mobile Creative Computing Lab](http://www.creativekidssa.com.au/gh/mobilecclab.html).

## Set-Up the Raspberry Pi

1. Buy a MicroSD card (the faster the better)
2. Download the latest [Raspbian Lite](https://www.raspberrypi.org/downloads/raspbian/)
3. Follow the instructions for installing Raspbian [here](https://www.raspberrypi.org/documentation/installation/installing-images/linux.md)

## Install Dependencies

1. Login/SSH with username `pi` and password `raspberry`
2. Set new password with `passwd`
3. Update installed packages `sudo apt-get update && sudo apt-get upgrade`
4. Install necessary packages `sudo apt-get install git python-dev libssl-dev libffi-dev virtualenv zip nginx php5-fpm imagemagick`
5. Install Node

```
$ curl -LO https://nodejs.org/dist/v4.5.0/node-v4.5.0-linux-armv7l.tar.xz
$ tar xvf node-v4.5.0-linux-armv7l.tar.xz
$ sudo cp -rp node-v4.5.0-linux-armv7l /usr/local/
$ sudo ln -s /usr/local/node-v4.5.0-linux-armv7l /usr/local/node
$ echo 'export PATH=$PATH:/usr/local/node/bin' >> ~/.bashrc
$ source ~/.bashrc
```

## Lektor

```
$ curl -sf https://www.getlektor.com/install.sh | sh
$ git clone https://github.com/rhysmoyne/lektor
$ cd lektor
$ cd lektor/admin
$ wget node_modules.tar.gz
$ tar xvzf node_modules.tar.gz
$ rm node_modules.tar.gz
$ cd ../..
$ vim.tiny lektor/admin/static/webpack.config.js
node: {
  fs: "empty"
}
$ make build-js
$ virtualenv venv
$ . venv/bin/activate
$ pip install setuptools --upgrade
$ pip install ---editable .

```

## School-website

```
$ git clone --recursive https://github.com/CreativeKids/school-website.git`
$ vim.tiny ../lektor/packages.py
Change portable_popen to just pip
$ cd school-website && ./build.sh

```

## Lektor Daemon (from http://www.diegoacuna.me/how-to-run-a-script-as-a-service-in-raspberry-pi-raspbian-jessie/)

`sudo vim.tiny /lib/systemd/system/lektor.service`

```
[Unit]
Description=Lektor Server
After=multi-user.target

[Service]
Type=simple
ExecStart=/bin/bash /home/pi/lektor/school-website/lektor_service.sh
Restart=on-abort
User=pi

[Install]
WantedBy=multi-user.target
```

`sudo chmod 644 /lib/systemd/system/lektor.service`

`vim.tiny lektor_service.sh`

```
#!/bin/bash
cd /home/pi/lektor
. venv/bin/activate
cd school-website
lektor server -h 0.0.0.0
```

`chmod +x lektor_service.sh`

`sudo systemctl daemon-reload`

`sudo systemctl enable lektor.service`

`sudo systemctl start lektor.service`

## Nginx

```
$ sudo rm /etc/nginx/sites-enabled/default
$ sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/school-website 
$ sudo vim.tiny /etc/nginx/sites-available/school-website 

+ Change root to /home/pi/lektor/school-website/build
+ Enable PHP (add index.php and uncomment)

$ sudo ln -s /etc/nginx/sites-available/school-website /etc/nginx/sites-enabled/school-website
$ sudo vim.tiny /etc/nginx/nginx.conf

Change user to pi
Add
client_max_body_size 100m;

$ sudo /etc/init.d/nginx restart
$ sudo vim.tiny /etc/php5/fpm/php-fpm.conf

Add 
listen.owner = pi
listen.group = pi

Change
upload_max_filesize = 100M
post_max_size = 100M

$ sudo vim.tiny /etc/php5/fpm/pool.d/www.conf
user = pi
group = pi

$ sudo rm /var/run/php5-fpm.sock
$ sudo /etc/init.d/php5-fpm restart
$ sudo chmod -R 777 wiki/data/
$ ssh-keygen -t rsa -b 4096 -C "rpi@creativekidssa.com.au"
$ cat /home/pi/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

## SFTP Share

```
sudo useradd -m -s /dev/null guest
sudo passwd guest
sudo vim.tiny /etc/ssh/sshd_config

Subsystem sftp internal-sftp

Match User guest
    ChrootDirectory /home/guest
    AllowTCPForwarding no
    X11Forwarding no
    ForceCommand internal-sftp
    PermitTunnel no
    AllowAgentForwarding no


sudo chown root:root /home/guest
cd /home/guest/
sudo rm .bash* profile
sudo mkdir share
sudo chown root:guest share
sudo chmod 775 share
sudo /etc/init.d/ssh restart
```

## Ad-hoc Wifi Network

From: http://slicepi.com/creating-an-ad-hoc-network-for-your-raspberry-pi/

```
$ sudo apt-get install dnsmasq
$ sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig
$ sudo vim.tiny /etc/dnsmasq.conf  

interface=wlan0      # Use interface wlan0  
listen-address=192.168.49.1 # Explicitly specify the address to listen on  
bind-interfaces      # Bind to the interface to make sure we aren't sending things elsewhere  
server=8.8.8.8       # Forward DNS requests to Google DNS  
domain-needed        # Don't forward short names  
bogus-priv           # Never forward addresses in the non-routed address spaces.  
dhcp-range=192.168.49.50,192.168.49.150,12h # Assign IP addresses between 192.168.49.50 and 192.168.49.150 with a 12 hour lease time 

$ sudo vim.tiny /etc/network/interfaces

auto wlan0
iface wlan0 inet static
address 192.168.49.1
netmask 255.255.255.0
wireless-channel 1
wireless-essid RPiwireless
wireless-mode ad-hoc

$ sudo service dnsmasq start  
$ sudo ifdown wlan0 && sudo ifup wlan0
```

## Useful URLs

https://frillip.com/using-your-raspberry-pi-3-as-a-wifi-access-point-with-hostapd/
https://learn.adafruit.com/setting-up-a-raspberry-pi-as-a-wifi-access-point/overview
