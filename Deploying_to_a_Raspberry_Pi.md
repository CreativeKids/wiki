# Deploying school-website to a Raspberry Pi Computer

These instructions provide an explanation of how to set-up [school-website](https://github.com/CreativeKids/school-website) on a Raspberry Pi computer and how to configure the Raspberry Pi as an access point for use in the [Creative Kids Mobile Creative Computing Lab](http://www.creativekidssa.com.au/gh/mobilecclab.html).

## Set-Up the Raspberry Pi

1. Buy a MicroSD card (the faster the better)
2. Download the latest [Raspbian Lite](https://www.raspberrypi.org/downloads/raspbian/)
3. Follow the instructions for installing Raspbian [here](https://www.raspberrypi.org/documentation/installation/installing-images/linux.md)

## Install Dependencies

1. Login/SSH with username `pi` and password `raspberry`
2. Update installed packages `sudo apt-get update && sudo apt-get upgrade`
3. Install necessary packages `sudo apt-get install git python-dev libssl-dev libffi-dev virtualenv zip nginx php5-fpm`
4. Install Node

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

$ sudo /etc/init.d/nginx restart
$ sudo vim.tiny /etc/php5/fpm/php-fpm.conf

Add 
listen.owner = pi
listen.group = pi

$ sudo rm /var/run/php5-fpm.sock
$ sudo /etc/init.d/php5-fpm restart
$ sudo chmod -R 777 wiki/data/

```