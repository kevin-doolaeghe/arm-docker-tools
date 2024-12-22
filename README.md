# Raspberry Pi OS - Linux environment setup

:triangular_flag_on_post: **Raspberry Pi OS** setup.

## Author

**Kevin DOOLAEGHE**

## Setup

### Download and install the system

Burn the *Raspberry Pi OS 64bit (Lite)* image using *Raspberry Pi Imager* software on your .

:warning: Configure the default system settings (hostname, user, network, timezone, SSH and telemetry) using the `Ctrl+Shift+X` keyboard shortcut.

### Configure the network access

If this is not already done, configure the network access :

1. Rename and edit the default `NetworkManager` configuration file named `preconfigured.nmconnection` :
```
sudo mv /etc/NetworkManager/system-connections/preconfigured.nmconnection /etc/NetworkManager/system-connections/wlan0.nmconnection
sudo nano /etc/NetworkManager/system-connections/wlan0.nmconnection
```

2. Update the configuration of the `wlan0` interface :
```
[connection]
id=wlan0
uuid=<uuid>
type=wifi

[wifi]
mode=infrastructure
ssid=<ssid>

[wifi-security]
key-mgmt=wpa-psk
psk=<psk>

[ipv4]
method=manual
address1=10.0.0.252/24,10.0.0.254
dns=10.0.0.254;8.8.8.8;

[ipv6]
addr-gen-mode=default
method=auto

[proxy]
```

3. Restart the `NetworkManager` service :
```
sudo systemctl restart NetworkManager
```

### Update the system

Update and upgrade *Raspberry Pi OS* using the following commands :
```
sudo apt-get update
sudo apt-get upgrade -y
sudo reboot
```

### Execute commands as administrator without password

```
echo "kevin ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/010_kevin-nopasswd
```

### Update the *MOTD* banner

1. Clear the `/etc/motd` file :
```
cat /dev/null | sudo tee /etc/motd
```

2. Open the `/etc/update-motd.d/20-sysinfo` file :
```
sudo nano /etc/update-motd.d/20-sysinfo
```

3. Copy & paste the following content then save the file using `Ctrl+X` :
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

4. Give execution permissions to the script :
```
sudo chmod +x /etc/update-motd.d/20-sysinfo
```

### Setup external storage

1. Plug your USB drive to the Raspberry Pi.

2. Find the name of your USB drive (e.g. `/dev/sda1`) using the following command :
```
lsblk
```

3. Edit the `/etc/fstab` file :
```
sudo nano /etc/fstab
```

4. Add the USB drive entry and save the file to auto-mount the USB drive at startup :
```
/dev/sda1  /mnt/data  auto  defaults  0  0
```

5. Create and initialize the mount directory :
```
sudo mkdir /mnt/data
sudo chmod 777 /mnt/data
```

6. Check if the `fstab` entry is working :
```
sudo mount -a
ls /mnt/data
```

### Setup Docker

1. Install the latest version of `` :
```
curl -sSL https://get.docker.com | sh
```

2. Add the current user to the Docker group (avoids having to use the `docker` command with root privileges) :
```
sudo usermod -aG docker $USER
```

:warning: You need to reconnect for the instruction to take effect.

3. Verify that the installation is successful :
```
docker --version
```

## References

* [Raspberry Pi OS website](https://www.raspberrypi.com/software/)
* [Docker setup script](https://github.com/docker/docker-install)
