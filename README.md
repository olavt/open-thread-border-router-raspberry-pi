# Building a OpenThread Border Router using a Raspberry Pi
### Author: [Olav Tollefsen](https://www.linkedin.com/in/olavtollefsen/)

## Introduction

This article shows how to build an OpenThread Border Router using a Raspberry Pi.

## Install the OpenThread Border Router software on a Raspberry Pi

### Prepare the Raspberry Pi

Flash an approperiate version of the Raspberry Pi OS.

![Raspberry PI OS](./images/raspberry-pi-os.png)

### Update the OS

```
$ sudo apt-get update
$ sudo apt-get upgrade
```

### Configure static IP-address

If you want to configure a static IP-address, you can use this command:

```
sudo nmtui
```

### Install Git

```
$ sudo apt install git
```

### Clone the OTBR repository:

```
$ git clone https://github.com/openthread/ot-br-posix
```

### Install dependencies

```
$ cd ot-br-posix
$ WEB_GUI=1
$ ./script/bootstrap
```

### Compile and install OTBR

```
$ WEB_GUI=1
$ INFRA_IF_NAME=eth0 ./script/setup
```

### Reboot the Raspberry Pi

The OTBR service should start on boot.

### Check the services

Verify that all required services are enabled:

```
$ sudo systemctl status
```

```
$ sudo service mdns status
```

```
$ sudo service otbr-agent status
```

```
$ sudo service otbr-web status
```

### Stop / Start a service

```
$ sudo service otbr-agent stop
```

```
$ sudo service otbr-agent start
```

### Verify RCP

Verify that the RCP is in the correct state:

```
sudo ot-ctl state
```

Responses should be like these:

```
disabled
Done
```

```
router
Done
```

#### Form a new Thread network

```
sudo ot-ctl factoryreset
sleep 3
sudo ot-ctl srp server disable
sudo ot-ctl thread stop
sudo ot-ctl ifconfig down
sudo ot-ctl dataset init new
sudo ot-ctl dataset commit active
sudo ot-ctl srp server enable
sudo ot-ctl ifconfig up
sudo ot-ctl thread start
```

### Obtaining Thread network credentials

When running directly on hardware:

```
sudo ot-ctl dataset active -x | sed -n 1p | sed -e "s/\r//g"
```

When running in Docker container

```
$ docker exec -it <container_id> sh -c "sudo ot-ctl dataset active -x"
```

## Backup

OpenThread Border Router stores it's state by default in /var/lib/thread. This can be changed by setting the environment variable OT_POSIX_SETTINGS_PATH at build time.

To backup the state of the OpenThread Border Router you need to backup the file(s) in this directory.

NOTE! The settings filename for each RCP will be uniqueue. If you swap an RCP, you need to make sure that the settings file for the old RCP is copied to the settings file for the new RCP. 

## Network Troubleshooting

avahi-browse is a command-line program that you can use to browse for all mDNS broadcasts on the network and to resolve the host name and IP address of the device performing the broadcasts.

```
sudo apt-get install avahi-utils 
```

```
avahi-browse -a

avahi-browse -r -t _meshcop._udp
```

```
 dns-sd -B _meshcop._udp

 dns-sd -B _meshcop._udp local

 dns-sd -L "OpenThread BorderRouter (#3991)" _meshcop._udp local
```