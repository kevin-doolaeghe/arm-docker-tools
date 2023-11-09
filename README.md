# Raspberry Pi OS - Linux environment setup

:triangular_flag_on_post: **Raspberry Pi OS** setup.

## Author

**Kevin Doolaeghe**

## Setup

### I. Install and update the system

1. Burn the *Raspberry Pi OS 64bit (Lite)* image using *Raspberry Pi OS Imager* software. 

:warning: Configure default Linux setting using the *Raspberry Pi OS Imager* software.

2. Update and upgrade *Raspberry Pi OS* using the following commands :

```
sudo apt-get update
sudo apt-get upgrade -y
sudo reboot
```

### II. 

1. Create and edit a configuration file named `10-wlan0.network` :

```
sudo nano /etc/systemd/network/10-wlan0.network
```

2. Add the configuration of the `wlan0` interface :

```
[Match]
Name=wlan0

[Network]
Address=10.0.0.251/24
Gateway=10.0.0.254
DNS=10.0.0.254
DHCP=no
```

3. Restart the `systemd-networkd` service :

```
sudo systemctl restart systemd-networkd.service
```

### III. Execute commands as administrator without password

```
echo "kevin ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/010_kevin-nopasswd
```

### IV. Update the *MOTD*

1. Clear the `/etc/motd` file :

```
cat /dev/null | sudo tee /etc/motd
```

2. Create the `/etc/update-motd.d/20-sysinfo` file with the following content :

```
#!/bin/bash

# date
date=`date +'%A,%e %B %Y, %r'`
# system
system=`uname -srmo`
# uptime
uptime=`cut -d. -f1 /proc/uptime | awk '{printf "%d days", $1/86400}'`
rebootDate=`uptime -s`
# memory
read usedMem totalMem usedMemPercent <<< `free | grep Mem | awk '{printf "%.2f %.2f %.2f", $3/1024, $2/1024, $3/$2*100}'`
memory="${usedMemPercent}% used (${usedMem} MB / ${totalMem} MB)"
# load average
read load1m load5m load15m _ < /proc/loadavg
loadAverage="${load1m}%, ${load5m}%, ${load15m}% (1, 5, 15 min.)"
# running processes
runningProcesses=`ps ax | wc -l | tr -d ' '`
# network
eth0IpAddress=`ip addr show eth0 | grep 'inet ' | awk '{print $2}' | cut -f1 -d'/'`
wlan0IpAddress=`ip addr show wlan0 | grep 'inet ' | awk '{print $2}' | cut -f1 -d'/'`
network="eth0: $([[ ! -z "$eth0IpAddress" ]] && echo "$eth0IpAddress" || echo "none"), wlan0: $([[ ! -z "$wlan0IpAddress" ]] && echo "$wlan0IpAddress" || echo "none")"

echo "$(tput setaf 2)
   .~~.   .~~.
  '. \ ' ' / .'   $(tput setaf 3; tput blink)${date}$(tput sgr0; tput setaf 2)
   .~ .~~~..~.    $(tput setaf 3; tput blink)${system}$(tput sgr0; tput setaf 1)
  : .~.'~'.~. :
 ~ (   ) (   ) ~  $(tput sgr0)Uptime.............: running since ${rebootDate} (${uptime})$(tput setaf 1)
( : '~'.~.'~' : ) $(tput sgr0)Memory.............: ${memory}$(tput setaf 1)
 ~ .~ (   ) ~. ~  $(tput sgr0)Load Average.......: ${loadAverage}$(tput setaf 1)
  (  : '~' :  )   $(tput sgr0)Running Processes..: ${runningProcesses} processes$(tput setaf 1)
   '~ .~~~. ~'    $(tput sgr0)Network............: ${network}$(tput setaf 1)
       '~'
$(tput sgr0)"
```

3. Give execution permissions to the script :

```
sudo chmod +x /etc/update-motd.d/20-sysinfo
```

### V. Setup Docker

1. Set up Docker's `apt` repository :

```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add the repository to Apt sources:
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

2. Install the latest version :

```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

3. Verify that the installation is successful :

```
sudo docker --version
```

## References

* [Install Raspberry Pi OS](https://www.raspberrypi.com/software/)
* [Setup Docker on Debian](https://docs.docker.com/engine/install/debian/)
