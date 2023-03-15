# Cviceni 4

Zvyseni entropie na VMs

```
apt install rng-tools
```

## OpenSSL

```
openssl genrsa -out key.pem 4096
openssl rsa -in key.pem -pubout > key.pub

openssl dgst -sign key.pem -keyform PEM -sha256 -out data.zip.sign -binary data.zip
openssl dgst -verify key.pub -keyform PEM -sha256 -signature data.
```

## GPG

```bash
vim ~/.gnupg/gpg.conf
gpg --gen-key
gpg --gen-revoke user@bsa-jindra.bsa
gpg --keyserver pool.sks-keyservers.net --send-key 'AAAA BBBB CCCC DDD'
gpg --import
gpg --export
gpg --armor --output bsa-user.gpg --export user@bsa-jindra.bsa
gpg --list-keys
gpg --list-sigs
```

### Sifrovani / desifrovani

```bash
gpg -e Recipient [Data]
gpg --output doc.gpg --encrypt --recipient user2@bsa-jindra.bsa
gpg -d [Data]
gpg --output doc --decrypt doc.gpg
```

### Dodepsani / overeni podpisu

```bash
gpg -s [Data]
gpg -b [Data]
gpg --sign --encrypt [Data]
gpg --verify [Data]
```


### Overeni zdrojovych baliku

```bash
wget 'http://pgp.surfnet.nl:11371/pks/lookup?op=get&search=0xF4FCBB07' -O F4FCBB07.asc
gpg --import F4FCBB07.asc
wget http://cdn-fastly.deb.debian.org/debian/pool/main/k/knot/knot_1.6.0-1.dsc
gpg --verify knot_1.6.0-1.dsc
```

```bash
gpg --list-sig
```


## John the Ripper

```bash
apt-get install john
passwd tonda
john --users=tonda /etc/shadow
```

## Tvorba hashu

```
md5sum file1 file2 file3
sha512sum file1 file2 file3
cat file1 | sha256sum
```

```
b2sum | cksum | md5sum |sha1sum
sha224sum | sha256sum |sha384sum | sha512sum
```

## Pridani dalsi ip na interface

```bash
# nastaveni
ip addr add 192.168.0.42/24 dev eth0
ifconfig eth0:next 192.168.0.42/24
# ziskani
ip addr show
ifconfig -a
```

## Sprava VLAN

```bash
vconfig add eth0 3
vconfig rem eth0.3
```

## Iptables

```bash
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m state --state NEW,RELATED,ESTABLISHED -j ACCEPT
iptables -A INPUT -m limit --limit 1/s --limit-burst 10 -p icmp -j LOG
iptables -A INPUT -m state --state INVALID -j DROP 
```

* REJECT vs DROP
* INPUT, OUTPUT, FORWARD
* -t nat (PREROUTING, POSTROUTING)
* -A -D -I -Z -P -F
* -j LOG | DROP | ACCEPT | REJECT
* -p tcp | udp | icmp
* --sport | --dport 

## SSH port forwarding
 
* LOCAL - lokalni port dostupny na vzdalenem serveru
* REMOTE - vzdaleny port dostupny na lokalnim serveru

```bash
ssh root@147.228.67.151 -L 1110:127.0.0.1:110
ssh root@147.228.67.151 -R 2222:127.0.0.1:22
tcpdump -i eth0 port 110 -X
```

## Stunnel
 
```bash
apt-get install stunnel
```

```bash
cd /etc/CA2
source ./vars
./build-key-server server.bsa-jindra.bsa
cat keys/ca.crt keys/server.bsa-jindra.bsa.crt > /etc/stunnel/server.bsa-jindra.bsa.pem
cp keys/server.bsa-jindra.bsa.key /etc/stunnel/
```

```bash
openssl req -new -newkey rsa:2048 -nodes -keyout server2.bsa-jindra.bsa.key -out server2.bsa-jindra.bsa.csr
openssl ca -in server2.bsa-jindra.bsa.csr -out server2.bsa-jindra.bsa.crt -config ./openssl-1.0.0.cnf
openssl rsa -in server2.bsa-jindra.bsa.key -out server2.bsa-jindra.bsa.key.pem
```

```bash
vim /etc/default/stunnel4
cp /usr/share/doc/stunnel4/examples/stunnel.conf-sample /etc/stunnel/stunnel.conf
vim /etc/stunnel/stunnel.conf
service stunnel4 restart
```

```bash
openssl s_client -connect 147.228.67.151:443
curl https://147.228.67.151
```

## Duveryhodna CA

Editovat: `/etc/ca-certificates.conf`

```bash
# pridani 
cp /etc/CA2/keys/ca.crt /usr/share/ca-certificates/ca-bsa.crt
update-ca-certificates
# odebrani
rm /usr/share/ca-certificates/ca-bsa.crt
update-ca-certificates --fresh
```

