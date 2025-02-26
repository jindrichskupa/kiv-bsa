# Cviceni 2

# Pristupova opravneni

## Zakladni

```bash
chmod -R u=rwx,g+x,o-w adresar
chmod 0777 soubor
```

* 7 = 4(r) + 2(w) + 1(x)
* setuid bit (4) - 
* setgid bit (2) -
* sticky bit (1) -

## Rozsirena ACL

```bash
getfacl
setfacl -m user:pepa:rw soubor
getfacl soubor
setfacl -m group:makaci:rw soubor
getfacl soubor
setfacl -d -m group:makaci:rw soubor
```

## AFS ACL

* Kerberos

```bash
kinit
klist
kdestroy
```

* AFS

```bash
fs sa
fs la
````

## Uzivatele

```bash
addgroup uzivatele
adduser pepa
usermod -a -G uzivatele pepa
passwd -d pepa
passwd -l pepa
passwd pepa
echo "pepa:heslo123heslo" | chpasswd
```

## Instalace LDAPu

```bash
apt install slapd ldap-utils ldapscripts
dpkg-reconfigure -plow slapd
```

## Vytvoreni organizacni jednotky
 
```bash
ldapadd -f create_ou.ldif -D cn=admin,dc=jindra,dc=bsa -w admin123
```

```ldif
dn: ou=users,dc=jindra,dc=bsa
objectClass: organizationalUnit
ou: users
``` 

## Vytvoreni uzivatele

```bash
ldapadd -f create_user.ldif -D cn=admin,dc=jindra,dc=bsa -w admin123
```

```ldif
dn: uid=pepa,ou=users,dc=jindra,dc=bsa
uid: pepa
cn: pepa
objectClass: account
objectClass: posixAccount
objectClass: top
objectClass: shadowAccount
userPassword:: heslo123
shadowLastChange: 14846
shadowMax: 99999
shadowWarning: 7
loginShell: /bin/bash
uidNumber: 10001
gidNumber: 10001
homeDirectory: /home/ldap/pepan
```

## Zmena uzivatele

```bash
ldapmodify -f modify_user.ldif -D cn=admin,dc=jindra,dc=bsa -w admin123
```

```ldif
dn: uid=pepa,ou=users,dc=jindra,dc=bsa
changeType: modify
replace: homeDirectory
homeDirectory: /home/ldap/pepa
```

## Vytvoreni skupiny

```bash
ldapadd -f create_group.ldif -D cn=admin,dc=jindra,dc=bsa -w admin123
```

```ldif
dn: cn=admins,ou=groups,dc=jindra,dc=bsa
objectClass: posixGroup
cn: admins
gidNumber: 100000
description: Group account
```

## Smazani skupiny

```bash
ldapdelete "cn=admins,ou=groups,dc=jindra,dc=bsa" cn=admin,dc=jindra,dc=bsa -w admin123
```

## Vyhledani/zobrazeni zaznamu

```ldif
ldapsearch -D cn=admin,dc=jindra,dc=bsa -w admin123 -b "dc=jindra,dc=bsa" '(objectClass='account')' cn
```

## Zmena hesla uzivatele

```bash
slappasswd
```

```bash
ldapmodify -f modify_user.ldif -D cn=admin,dc=jindra,dc=bsa -w admin123
```

```ldif
dn: uid=pepa,ou=users,dc=jindra,dc=bsa
changeType: modify
replace: userPassword
userPassword:: {SSHA}f9uY1nWWUEzg8nLZVJzo7E7Va85nssXg
```
 
## LDAP autentizace
 
```bash
apt install libnss-ldap libpam-ldap
```

Nastaveni jmennych sluzeb s LDAPem

```
/etc/libnss-ldap.conf
/etc/nsswitch.conf
```

Seznam uzivatelu lokalnich spolu s LDAPovymi

```
getent passwd
```

Nastaveni autentizace LDAP spolu s lokalnimi ucty `/etc/pam.d/common-auth`

```
auth    [success=2 default=ignore]      pam_unix.so nullok_secure
auth    [success=1 default=ignore]      pam_ldap.so use_first_pass
```

Sablona pro automaticke zakladani homes `/usr/share/pam-configs/mkhomedir`

```
Name: Create home directory during login
Default: yes
Priority: 900
Session-Type: Additional
Session:
        required        pam_mkhomedir.so umask=0022 skel=/etc/skel
```

aktualizace nastaveni

```
pam-auth-update
```

## Google Authenticator

instalace pam modulu

```bash
apt install libpam-google-authenticator
```

nastaveni authenticatoru per user

```
google-authenticator
```

nastaveni pam pro ssh: `/etc/pam.d/sshd`

```
# zakomentovat
#@include common-auth

# pridat na konec
auth required pam_google_authenticator.so
```

nastaveni sshd: `/etc/ssh/sshd_config`

```
# zmenit/pridat pokud chcete pouzivat na roota
PermitRootLogin yes
# zmenit/pridat
ChallengeResponseAuthentication yes
# zmenit/pridat
AuthenticationMethods publickey,keyboard-interactive
# uz bysme meli mit nastavene...
# PasswordAuthentication no
```

https://www.vultr.com/docs/how-to-setup-two-factor-authentication-2fa-for-ssh-on-debian-9-using-google-authenticator

## Sudo

```
# Members of the admin group may gain root privileges
%admin ALL=(ALL) ALL

# Allow members of group sudo to execute any command
%sudo	ALL=(ALL:ALL) ALL

admin ALL=(ALL) NOPASSWD: ALL
%sudoall ALL=(ALL:ALL) NOPASSWD: ALL

service ALL=(ALL:ALL) NOPASSWD: /bin/systemctl restart service, /bin/systemctl start service, /bin/systemctl stop service, /bin/systemctl status service
```
