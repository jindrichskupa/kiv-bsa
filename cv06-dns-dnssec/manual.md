# Cviceni 6

## Ziskani informaci z DNS

```bash
# nastroj host
$ host zcu.cz
$ host -t NS zcu.cz
$ host -t MX zcu.cz localhost

# nastroj dig
$ dig zcu.cz
$ dig -t AAAA zcu.cz
$ dig -t SOA zcu.cz @localhost

# dalsi nastroje
$ nslookup
$ whois
```

## Nastaveni DNS

```bash
/etc/resolv.conf
/etc/hosts
```

## Bind9

### Instalace a spusteni

```bash
apt-get install bind9 dnsutils
service bind9 start|stop|restart
```

### Konfiguracni soubory a logy

```bash
/etc/bind/named.conf*
/etc/bind/db.*

/var/log/syslog
```

### Naslouchani

``` bash
allow-recursion {10.0.0.0/24;};
listen-on {127.0.0.1;};
listen-on-v6 {::1;};
```

### Nastaveni zon

```
zone "jindra.bsa." in {
    type master;
    file "/etc/bind/db.jindra.bsa";
    allow-transfer {147.228.67.0/24;};
};

zone "lubos.bsa." in {
    type slave;
    file "/etc/bind/slave/slave.lubos.bsa";
    masters {147.228.67.41;};
};
```

### Zonovy soubor

```
$TTL    604800
@   IN  SOA jindra.bsa. root.localhost. (

                  2     ; Serial / YYYYMMDDXX
             604800     ; Refresh / seconds
              86400     ; Retry / seconds
            2419200 ; Expire / seconds
             604800 )   ; Negative Cache TTL / explicitni TTL

@         IN      NS                ns.jindra.bsa.
@         IN      NS                lubos.bsa.
@         IN      A                 147.228.67.42
@         IN      A                 147.228.67.41
@         IN      AAAA              ::1
mail      IN      A                 147.228.67.42
pop3      IN      CNAME             mail
smtp      IN      CNAME             mail.jindra.bsa.
imap      IN      CNAME             mail.jindra.bsa
@         IN      MX           10   mail
@         IN      MX           20   mail.lubos.bsa.
ns        IN      A                 147.228.67.42
txt       IN      TXT               "ahoj svete"

```

### Pohledy

```
acl local {
   192.168.1.0/24;
   localhost;
};
```

```
view "localnetwork" {
 match-clients { local; 192.168.0.0/24; };
  recursion yes;
  zone "jindra.bsa." {
    type master;
    file  "/etc/bind/local/db.jindra.bsa"
  };
};

view "publicnetwork"{
match-clients {"any"; };
  recursion no;
  zone "jindra.bsa." {
    type master;
    file  "/etc/bind/public/db.jindra.bsa"
  };
};

```

### Forwarding

```
; globalni nastaveni
options {
    directory "/var/named";
    version "not available";
    forwarders {10.0.0.1; 10.0.0.2;};
    forward only;
};
; per zone nastaveni
zone "example.com" IN {
    type forward;
    forwarders {10.0.0.1; 10.0.0.2;};
};
```

### Nastaveni

```
options {
        directory "/var/cache/bind";
        recursion yes;
        allow-recursion { trusted; };
        listen-on { 10.228.67.42;  147.228.67.42; };
        allow-transfer { none; };
        forwarders {
                8.8.8.8;
                8.8.4.4;
        };
...
};
```

## DNSSec

Nedostatek entropie pro generovani klicu vyresime

```bash
apt-get install haveged 
```

Pripravime adresar pro klice a vygenerujeme klice pro domenu jindra.bsa

```bash
mkdir /etc/bind/keys
cd /etc/bind/keys
dnssec-keygen -a ECDSAP256SHA256 -fK jindra.bsa
chmod g+r K*.private
```

```bash
ln -s /etc/bind/local/jindra.bsa /var/cache/bind
```

```
zone "jindra.bsa" {
        type master;
        file "/etc/bind/local/jindra.bsa";
        inline-signing yes;
        auto-dnssec maintain;
        key-directory "/etc/bind/keys";
};
```

```bash
rndc signing -list jindra.bsa
```

```bash
named-compilezone -f raw -j -o - jindra.bsa /var/cache/bind/jindra.bsa.signed
```

```bash
dnssec-dsfromkey /etc/bind/keys/Kjindra.bsa.+123+12345
```

Vymena klicu, vygenerujeme novy klic

```bash
dnssec-keygen -a ECDSAP256SHA256 -fK jindra.bsa
```

Podepiseme znovu znovu obema

```bash
rndc sign jindra.bsa
```

Naplanujeme expiraci prvnimu - 35 dnu

```bash
dnssec-settime -I now -D +35d Kjindra.bsa.+321+54321
```

## Knot

```
apt install -y apt-transport-https lsb-release ca-certificates wget
wget -O /etc/apt/trusted.gpg.d/knot-latest.gpg https://deb.knot-dns.cz/knot-latest/apt.gpg
echo "deb https://deb.knot-dns.cz/knot-latest/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/knot-latest.list
apt update

apt install -y knot knot-dnsutils
knotc reload
```

### Konfigurace

[reference](https://www.knot-dns.cz/docs/2.6/html/configuration.html)

```
template:
  - id: default
    storage: /var/lib/knot/master
    semantic-checks: on

  - id: signed
    storage: /var/lib/knot/signed
    dnssec-signing: on
    semantic-checks: on

  - id: slave
    storage: /var/lib/knot/slave

policy:
  - id: rsa
    algorithm: RSASHA256
    ksk-size: 2048
    zsk-size: 1024

zone:
  - domain: jindra.bsa

  - domain: jindra-secured.bsa
    template: signed
    dnssec-policy: rsa

```
