# OpenVPN

:triangular_flag_on_post: **OpenVPN** application package.

## Author

**Kevin Doolaeghe**

## Setup

1. Prepare environment :
```
sudo docker run -v openvpn_data:/etc/openvpn --rm giggio/openvpn-arm ovpn_genconfig -u udp://<domain-name>
sudo docker run -v openvpn_data:/etc/openvpn --rm -it giggio/openvpn-arm ovpn_initpki
```

2. Setup `docker-compose` application :
```
sudo docker compose -p openvpn up -d
```

3. Setup firewall rules :
```
sudo ufw allow 1194/udp comment 'openvpn'
```

4. Create a user :
```
sudo docker run -v openvpn_data:/etc/openvpn --rm -it giggio/openvpn-arm easyrsa build-client-full <user> nopass
```

5. Generate the `.ovpn` certificate :
```
sudo docker run -v openvpn_data:/etc/openvpn --rm giggio/openvpn-arm ovpn_getclient <user> > <user>.ovpn
```

:warning: This program require a docker instance to be executed.

## References

* [Setup OpenVPN for Docker](https://cj-hewett.medium.com/running-openvpn-server-with-docker-on-a-raspberry-pi-with-noip-39459dd9b625)
