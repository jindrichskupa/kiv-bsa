# Cviceni 5

## OpenVPN

### Instalace

```bash
apt-get install openvpn
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

Konfigurace OpenVPN

```bash
wget ...
```

### Start / stop / restart

```bash
systemctl start openvpn@bsa-server
systemctl start openvpn@bsa-client-01
```
