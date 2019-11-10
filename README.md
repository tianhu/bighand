![picture](strongswan.png)

# Setup IKEv2 IPSec VPN with StrongSwan

A setp by step tutorial to set up IPSec VPN cover L2TP and IKEv2 with StrongSwan.

## Setup IPSec VPN server

Install software
```
sudo su  
apt install strongswan strongswan-pki  
cd /etc/ipsec.d
```

Create CA key
```
ipsec pki --gen --type rsa --size 4096 --outform pem > private/caKey.pem  
chmod 600 private/caKey.pem
```

Create CA certificate (valid for 3650 days)
```
ipsec pki --self --ca --lifetime 3650 --in private/caKey.pem --type rsa --dn "C=US, O=CityTiger, CN=CityTiger CA" --outform pem > cacerts/caCert.pem
```

Create server key
```
ipsec pki --gen --type rsa --size 2048 --outform pem > private/serverKey.pem  
chmod 600 private/serverKey.pem
```

Create server certificate
```
ipsec pki --pub --in private/serverKey.pem --type rsa | ipsec pki --issue --lifetime 3650 --cacert cacerts/caCert.pem --cakey private/caKey.pem --dn "C=US, O=CityTiger, CN=<server-public-ip>" --san <server-public-ip> --flag serverAuth --flag ikeIntermediate --outform pem > certs/serverCert.pem
```

Create client key
```
ipsec pki --gen --type rsa --size 2048 --outform pem > private/clientKey.pem  
chmod 600 private/clientKey.pem
```

Create client certificate
```
ipsec pki --pub --in private/clientKey.pem --type rsa | ipsec pki --issue --lifetime 3650 --cacert cacerts/caCert.pem --cakey private/caKey.pem --dn "C=US, O=CityTiger, CN=tiger@city.com" --san tiger@city.com --outform pem > certs/clientCert.pem  
  
openssl pkcs12 -export -inkey private/clientKey.pem -in certs/clientCert.pem -name "CityTiger VPN client certificate" -certfile cacerts/caCert.pem -caname "CityTiger CA" -out client.p12
```

Edit /etc/ipsec.conf
```
vi /etc/ipsec.conf
```
> Copy the content of ipsec.conf from this git repository.


Edit /etc/ipsec.secrets  
```
vi /etc/ipsec.secrets
```
> Copy the content of ipsec.secrets from this git repository. Change pre-shared-key, username and password as you like.
ipsec rereadsecrets


Edit  /etc/strongswan.conf  
```
vi /etc/strongswan.conf
```
> Add below lines. It should look like strongswan.conf from this git repository.
> 
> duplicheck.enable = no
> dns1 = 8.8.8.8
> dns2 = 8.8.4.4
> nbns1 = 8.8.8.8
> nbns2 = 8.8.4.4
> 


Configure network  
```
vi /etc/sysctl.conf
```
> uncomment below lines.
> 
> net.ipv4.ip_forward = 1
> net.ipv4.conf.all.accept_redirects = 0
> net.ipv4.conf.all.send_redirects = 0
> 
```
sysctl -p  
iptables -A INPUT -p udp --dport 500 --j ACCEPT  
iptables -A INPUT -p udp --dport 4500 --j ACCEPT  
iptables -A INPUT -p esp -j ACCEPT  
iptables -t nat -A POSTROUTING -s 10.10.10.0/24 -o eth0 -j MASQUERADE  
iptables -A FORWARD -s 10.10.10.0/24 -j ACCEPT  
apt install iptables-persistent  
```


Start service
```
systemctl enable strongswan  
systemctl start strongswan
```

## Setup IPSec VPN clients

WIP...
