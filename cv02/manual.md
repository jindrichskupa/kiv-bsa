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

