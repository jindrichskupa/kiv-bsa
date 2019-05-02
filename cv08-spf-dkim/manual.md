# Cviceni 8

## DNS

```bash
apt-get install bind9
cp /etc/bind/db.local /etc/bind/db.bsa-jindra.spos

/etc/bind/named.conf.local
zone "bsa-jindra.bsa" {
        type master;
        file "/etc/bind/db.bsa-jindra.bsa";
};
```

* Zone bsa-jindra.bsa

```
;
$TTL	604800
@	IN	SOA	bsa-jindra.bsa. root.bsa-jindra.bsa. (
			      4		; Serial
			 604800		; Refresh
			  86400		; Retry
			2419200		; Expire
			 604800 )	; Negative Cache TTL
;
@	IN	NS	ns.bsa-jindra.bsa.
@	IN	MX	10 mail
@	IN	MX	20 mail2
@	IN	A	147.228.67.151
ns	IN	A	147.228.67.151
mail	IN	A	192.168.0.151
mail2	IN	A	192.168.1.151
```

## SPF

```bash
apt-get install postfix-policyd-spf-python
```

Generator: <http://www.spfwizard.net/>

```
@  	IN 	TXT 	"v=spf1 mx a:server.bsa-jindra.bsa ~all"
```

```
/etc/postfix/master.cf
policyd-spf unix    -       n       n       -       -      spawn
  user=vmail argv=/usr/bin/policyd-spf
```

```
smtpd_recipient_restrictions = permit_sasl_authenticated,permit_mynetworks,reject_unauth_destination,check_policy_service unix:private/policy
#or
smtpd_recipient_restrictions =
            reject_unauth_destination
            check_policy_service unix:private/policy
```

Testing

```
request=smtpd_access_policy
protocol_state=RCPT
protocol_name=SMTP
helo_name=h****forge.com
queue_id=8045F2AB23
sender=
recipient=falko.timme@*******.de
client_address=1.2.3.4
client_name=www.example.com
[empty line]
```

## DKIM

```bash
apt install opendkim opendkim-tools
```

```bash
/etc/opendkim.conf


KeyTable        /etc/opendkim/key.table
SigningTable        refile:/etc/opendkim/signing.table

mkdir /etc/opendkim
mkdir /etc/opendkim/keys
chown -R opendkim:opendkim /etc/opendkim
chmod go-rw /etc/opendkim/keys



/etc/opendkim/signing.table
*@bsa-jindra.bsa   bsa-jindra


/etc/opendkim/key.table
bsa-jindra     bsa-jindra.bsa:201704:/etc/opendkim/keys/bsa-jindra.private


/etc/opendkim/trusted.hosts
127.0.0.1
::1
```

```
chown -R opendkim:opendkim /etc/opendkim
chmod -R go-rwx /etc/opendkim/keys

mkdir ~/dkim; cd ~/dkim
opendkim-genkey -b 2048 -h rsa-sha256 -r -s 201704 -d bsa-jindra.bsa -v

cp 201705.private /etc/opendkim/keys/bsa-jindra.private
cat 201704.txt >> /etc/bind/db.bsa-jindra.bsa
```

```
201704.txt
201704._domainkey  IN  TXT ( "**v=DKIM1; h=rsa-sha256; k=rsa; s=email; "
    "p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAu5oIUrFDWZK7F4thFxpZa2or6jBEX3cSL6b2TJdPkO5iNn9vHNXhNX31nOefN8FksX94YbLJ8NHcFPbaZTW8R2HthYxRaCyqodxlLHibg8aHdfa+bxKeiI/xABRuAM0WG0JEDSyakMFqIO40ghj/h7DUc/4OXNdeQhrKDTlgf2bd+FjpJ3bNAFcMYa3Oeju33b2Tp+PdtqIwXR"
    "ZksfuXh7m30kuyavp3Uaso145DRBaJZA55lNxmHWMgMjO+YjNeuR6j4oQqyGwzPaVcSdOG8Js2mXt+J3Hr+nNmJGxZUUW4Uw5ws08wT9opRgSpn+ThX2d1AgQePpGrWOamC3PdcwIDAQAB**" )  ; ----- DKIM key 201704 for bsa-jindra.bsa
```

```
mkdir /var/spool/postfix/opendkim
chown opendkim:postfix /var/spool/postfix/opendkim
```

```
/etc/default/opendkim
SOCKET="local:/var/spool/postfix/opendkim/opendkim.sock"
```

```
/etc/postfix/main.cf
# Milter configuration
# OpenDKIM
milter_default_action = accept
# Postfix ≥ 2.6 milter_protocol = 6, Postfix ≤ 2.5 milter_protocol = 2
milter_protocol = 6
smtpd_milters = local:/opendkim/opendkim.sock
non_smtpd_milters = local:/opendkim/opendkim.sock
```

Kontrola

```bash
opendkim-testkey -d bsa-jindra.bsa -s 201704 -k bsa-jindra.bsa.private
```

Dalsi nastaveni

```bash
Author Domain Signing Practices 
_adsp._domainkey
dkim=all

Domain Message Authentication, Reporting & Conformance (DMARC)
_dmarc
v=DMARC1;p=quarantine;sp=quarantine;adkim=r;aspf=r
```

## Reference

* [DMARC](https://www.root.cz/clanky/dmarc-overeni-odesilatelovy-domeny/)
* [ADSP](http://www.abclinuxu.cz/clanky/dkim-zavadime-podpisovou-politiku-adsp)
* [TLSA](https://www.root.cz/clanky/bezpecnejsi-predavani-posty-s-tlsa-zaznamy/)
