
```bash
apt-get install opendkim opendkim-tools postfix-policyd-spf-python
```

```bash
v=spf1 a:mail.example.com -all
```


```bash
/etc/opendkim.conf
KeyTable        /etc/opendkim/key.table
SigningTable        refile:/etc/opendkim/signing.table
```

```
mkdir /etc/opendkim
mkdir /etc/opendkim/keys
chown -R opendkim:opendkim /etc/opendkim
chmod go-rw /etc/opendkim/keys
```

```
/etc/opendkim/signing.table
*@example.com   example
```

```
/etc/opendkim/key.table
example     example.com:YYYYMM:/etc/opendkim/keys/example.private
```

```
/etc/opendkim/trusted.hosts
127.0.0.1
::1
```

```
chown -R opendkim:opendkim /etc/opendkim
chmod -R go-rwx /etc/opendkim/keys
```

```
opendkim-genkey -b 2048 -h rsa-sha256 -r -s YYYYMM -d example.com -v


example.txt
201510._domainkey  IN  TXT ( "**v=DKIM1; h=rsa-sha256; k=rsa; s=email; "
    "p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAu5oIUrFDWZK7F4thFxpZa2or6jBEX3cSL6b2TJdPkO5iNn9vHNXhNX31nOefN8FksX94YbLJ8NHcFPbaZTW8R2HthYxRaCyqodxlLHibg8aHdfa+bxKeiI/xABRuAM0WG0JEDSyakMFqIO40ghj/h7DUc/4OXNdeQhrKDTlgf2bd+FjpJ3bNAFcMYa3Oeju33b2Tp+PdtqIwXR"
    "ZksfuXh7m30kuyavp3Uaso145DRBaJZA55lNxmHWMgMjO+YjNeuR6j4oQqyGwzPaVcSdOG8Js2mXt+J3Hr+nNmJGxZUUW4Uw5ws08wT9opRgSpn+ThX2d1AgQePpGrWOamC3PdcwIDAQAB**" )  ; ----- DKIM key 201510 for example.com

opendkim-testkey -d example.com -s YYYYMM

mkdir /var/spool/postfix/opendkim
chown opendkim:postfix /var/spool/postfix/opendkim

/etc/default/opendkim
SOCKET="local:/var/spool/postfix/opendkim/opendkim.sock"

/etc/postfix/main.cf
# Milter configuration
# OpenDKIM
milter_default_action = accept
# Postfix ≥ 2.6 milter_protocol = 6, Postfix ≤ 2.5 milter_protocol = 2
milter_protocol = 6
smtpd_milters = local:/opendkim/opendkim.sock
non_smtpd_milters = local:/opendkim/opendkim.sock

```

kontrola:

```bash
check-auth@verifier.port25.com

Author Domain Signing Practices 
_adsp._domainkey
dkim=all

Domain Message Authentication, Reporting & Conformance (DMARC)
_dmarc
v=DMARC1;p=quarantine;sp=quarantine;adkim=r;aspf=r
```


```bash
opendkim-testkey -d example.com -s YYYYMM -k example.private

```
