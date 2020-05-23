---
layout: post
author: James
title: Raspberry Pi CPU and Network Monitor
---
This guide will be installing and configuring [RPi-Monitor](https://github.com/XavierBerger/RPi-Monitor) to record metrics on CPU temperature, CPU usage, disk space and network usage.

## Installing

First trust the public key of the RPi-Monitor repository and add it as a source
```
sudo apt-get install dirmngr
sudo apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 2C0D3C0F
sudo wget http://goo.gl/vewCLL -O /etc/apt/sources.list.d/rpimonitor.list
```

Then install it, this will take a while
```
sudo apt-get update
sudo apt-get install rpimonitor
```

Once it is installed, a web interface should be available at http://localhost:8888/

## Monitor Upgradeable Packages

Upon start the web interface for the first time, there will be a warning under upgradeable packages. Simply follow the given instructions and execute `sudo /etc/init.d/rpimonitor update`.

## Monitor Network Usage

This will allow you to see the network usage over time.

First find your network interface by using `ip link show`. It should be something like `eth0`.


Open the following file:
`sudo nano /etc/rpimonitor/template/network.conf`

Now uncomment all the lines and change the `eth0` to your network interface name in the `dynamic.10.source=...` and `dynamic.11.source=...`.

Finally comment out these lines:
```
#web.status.1.content.8.line.1="To activate network monitoring, edit and customize <font color='#AA0000'><b>network.conf</b></font>"
#web.status.1.content.8.line.2="Help is available in man pages:"
#web.status.1.content.8.line.3="<font color='#AA0000'><b>man rpimonitord</b></font> or <font color='#AA0000'><b>man rpimonitord.conf</b></font>"
```

Save and close the file. Restart RPi-Monitor with `sudo service rpimonitor restart`. Done!

## Monitor External Drive Usage

I like to see the disk usage of my external hard drive at a glance. The RPi-Monitor web interface is a good way of doing that, but it requires some extra configuration.

First find the name of the partition and the filesystem type you want to monitor using `df -h -T`. The name will be something like `/dev/sda2` and the filesystem type will be under the `Type` column. In my case it was `fuseblk`.

We will be creating a new file with: `sudo nano /etc/rpimonitor/template/disk.conf`.

Paste the following into the new file, changing `sda2` and `fuseblk` to your respective values:
```
static.1.name=storage1_total
static.1.source=df -t fuseblk
static.1.regexp=sda2\s+(\d+)
static.1.postprocess=$1/1024

dynamic.1.name=storage1_used
dynamic.1.source=df -t fuseblk
dynamic.1.regexp=sda2\s+\d+\s+(\d+)
dynamic.1.postprocess=$1/1024
dynamic.1.rrd=GAUGE

web.status.1.content.1.name=Storage
web.status.1.content.1.icon=usb_hdd.png
web.status.1.content.1.line.1="<b>/storage1</b> Used: <b>"+KMG(data.storage1_used,'M')+"</b> (<b>"+Percent(data.storage1_used,data.storage1_total,'M')+"</b>) Free: <b>"+KMG(data.storage1_total-data.storage1_used,'M')+ "</b> Total: <b>"+$
web.status.1.content.1.line.2=ProgressBar(data.storage1_used,data.storage1_total)
```

Now we will download a small icon for use in the web interface:
`sudo wget https://xavierberger.github.io/RPi-Monitor-docs/_images/hdd001.png -O /usr/share/rpimonitor/web/img/usb_hdd.png`

Next we have to include this new file in the RPi-Monitor config. Open the following file:
`sudo nano /etc/rpimonitor/data.conf`

Go to the bottom of the file and add the line: `include=/etc/rpimonitor/template/disk.conf`
Save and close the file.

Finally restart the service using: `sudo service rpimonitor restart`. Done!
