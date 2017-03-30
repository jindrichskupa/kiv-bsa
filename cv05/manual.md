# Cviceni 5

## OpenVPN

### Instalace

```bash
apt-get install openvpn
```

### Konfigurace PSK

```bash
wget https://raw.githubusercontent.com/jindrichskupa/kiv-bsa/master/cv05/bsa-server-psk.conf
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
wget https://raw.githubusercontent.com/jindrichskupa/kiv-bsa/master/cv05/bsa-server.conf
wget https://raw.githubusercontent.com/jindrichskupa/kiv-bsa/master/cv05/bsa-client-01
```

Konfigurace OpenVPN - client

```bash
wget https://raw.githubusercontent.com/jindrichskupa/kiv-bsa/master/cv05/bsa-client-01.conf
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
