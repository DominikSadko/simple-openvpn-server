## Installing OpenVPN

1. Download the installer script.

	````
	wget https://raw.githubusercontent.com/DominikSadko/simple-openvpn-server/master/openvpn.sh; chmod +x openvpn.sh
	````

1. Run the script.
	Example:

	````
	./openvpn.sh --adminpassword=mypassword --host=myvpn.example.com --vpnport=1194
	````


	There are number of options the script will accept

	**adminpassword** -- This is the admin password for the website for managing clients. The default is **password**.

	**dns1** -- The first dns server assigned to the clients. The default is **8.8.8.8**.

	**dns2** -- The first dns server assigned to the clients. The default is **8.8.4.4**.

	**vpnnetwork** -- The ip network pool for the vpn connections **10.0.8.0**.

	**route** -- The network behind the vpn server you want to access **10.0.0.0**.

	**vpnport** -- The port to be used by OpenVPN. 1194 may be blocked by some firewalls, so this is customizable. The default port is **1194**.

	**protocol** -- The protocol to be used by OpenVPN. This accepts **udp** or tcp. The default is **udp**.

	**host** -- The host name of the server. The script attempts to detect the external IP of your server if the host is not specified. ***It is highly recommended that you use a host name if your sever is not using a static IP address***. You can get a free dynamic DNS account and use a dynamic DNS updater that keeps the DNS records for your server up to date in the event that your IPa address changes.

-----
## restart openvpn

```
systemctl restart openvpn
```

-----
## no persistant ip's so I can multi connect as the same user

```
//add this line so same cert can be used on mltpl devices:
duplicate-cn
//comment this line in /server.conf if you don't want persistant ip's
//but I like it
ifconfig-pool-persist ipp.txt
```

-----
## logs uncomment log-append only when troubleshooting

```
//add to server.conf in logging section:
status openvpn-status.log
#log-append openvpn.log
verb 3
```

-----
## iptables

Write iptables on boot:
```
nano /etc/rc.local
#!/bin/sh -e
iptables -t nat -A POSTROUTING -s 10.0.8.0/24 -j SNAT --to 10.0.0.6
exit 0
```

Write iptables with iptables -persistant
```
apt-get install iptables-persistant
iptables -t nat -L
iptables -t nat -A POSTROUTING -s 10.0.8.0/24 -j SNAT --to 10.0.0.6
iptables-save
```

-----
## Once after install and then Every 10 years

```
cd /etc/openvpn
cd easy-rsa
EASYRSA_CRL_DAYS=3650 ./easyrsa gen-crl
cd pki
chown www-data:www-data crl.pem
chmod 755 crl.pem
cp crl.pem /etc/openvpn/
openssl crl -in /etc/openvpn/crl.pem -text (check expiration)
systemctl restart openvpn
```

-----

## Managing Profiles

1. Once the script is complete, point your browser to **https://[your host or IP]/**, where your host or IP is the host name or IP addressed for the VPN. You may get an error about the site not being secure even though you are using https. This is because the site is using a self-esigned certificate. Simply ignore the warning. 
