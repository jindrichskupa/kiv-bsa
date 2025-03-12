# Cviceni 5

## OpenVPN

### Instalace

```bash
apt-get install openvpn
```

### Konfigurace PSK

```bash
wget https://raw.githubusercontent.com/jindrichskupa/kiv-bsa/master/cv05-vpn/bsa-server-psk.conf
openvpn --genkey --secret bsa-server-psk.key
```

### Konfigurace

Vytvoreni CA + certifikatu

```bash
cp -r /usr/share/easy-rsa/ /etc/openvpn/ca
vim /etc/openvpn/ca/vars
vim /etc/openvpn/ca/openssl-1.0.0.cnf

cd /etc/openvpn/ca
source vars
./clean-all
mkdir keys; touch keys/index.txt; echo "01" > keys/serial
./build-dh
./build-ca
./build-key-server vpn.bsa-jindra.bsa
./build-key bsa-client-01
```

Konfigurace OpenVPN - server

```bash
wget https://raw.githubusercontent.com/jindrichskupa/kiv-bsa/master/cv05-vpn/bsa-server.conf
wget https://raw.githubusercontent.com/jindrichskupa/kiv-bsa/master/cv05-vpn/bsa-client-01
```

Konfigurace OpenVPN - client

```bash
wget https://raw.githubusercontent.com/jindrichskupa/kiv-bsa/master/cv05-vpn/bsa-client-01.conf
``` 

Generator klientske 

```bash
#!/bin/bash

cat << EOF > client_$1.conf
remote bsa-150.kiv.zcu.cz
tls-client
port 1195
proto udp
dev tun
pull

cipher AES-128-CBC
comp-lzo
verb 3

route-delay 2

<ca>
$(cat /etc/CA/pki/ca.crt)
</ca>

<cert>
$(cat /etc/CA/pki/issued/$1.crt)
</cert>

<key>
$(cat /etc/CA/pki/private/$1.key)
</key>
EOF
```

### Start / stop / restart

```bash
systemctl start openvpn@bsa-server
systemctl start openvpn@bsa-client-01
```

## DNSMASQ

```bash
apt-get install dnsmasq
vim /etc/dnsmasq.d/bsa.conf
```

```
host-record=server,10.255.0.1
host-record=hidden-client,10.255.0.1
```

## Limit zdroju

### Procesy a pamet

/etc/security/limits.conf

```bash
ulimit -a
pepa     soft    nproc          5
```

### SSH

```bash
	/etc/ssh/sshd_config
	Match user pepa
	   ChrootDirectory /home/pepa 		# ( POZOR - musi vlastnit root )
	   ForceCommand internal-sftp -u 002
```


## Wireguard

```
apt install wireguard
wg genkey > /etc/wireguard/private.key
cat /etc/wireguard/private.key | wg pubkey > /etc/wireguard/public.key
```

### /etc/wireguard/wg0.conf (server)

```
[Interface]
PrivateKey = <local_private_key>
Address = 10.255.255.1/24
ListenPort = 51820
SaveConfig = true

PostUp = iptables -t nat -I POSTROUTING -o eth0 -j MASQUERADE
PreDown = iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
```

service:

```
systemctl enable wg-quick@wg0.service
systemctl start wg-quick@wg0.service
```

add peer

```
wg set wg0 peer <peers_public_key> allowed-ips 10.255.255.2
```

### /etc/wireguard/wg0.conf (peer)

```
[Interface]
PrivateKey = <local_private_key>
Address = 10.255.255.2/24

[Peer]
PublicKey = U9uE2kb/nrrzsEU58GD3pKFU3TLYDMCbetIsnV8eeFE=
AllowedIPs = 10.255.255.1/24
Endpoint = <server_ip>:51820
```
