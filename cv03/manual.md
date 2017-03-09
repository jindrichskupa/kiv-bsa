# Cviceni 3

## Vynuceni silnych hesel

### Knihovna libpam-pwquality

```bash
$ apt-get install libpam-pwquality
```

### Nastaveni

`/etc/pam.d/common-password`:

(vychozi nastaveni)

```
password    requisite     pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type=
```

(vynuceni sily hesla)

```
password    requisite     pam_pwquality.so minlen=19 lcredit=0 ucredit=1 dcredit=1 ocredit=2
```

(historie hesel)

```
password    required      pam_pwhistory.so remember=400 use_authtok
```

(zablokovani uctu)

```
auth       required     pam_tally2.so deny=3 unlock_time=1800 even_deny_root
```

(slovnikova hesla)

```
password required pam_cracklib.so retry=3 minlen=6 difok=3
```

(manual)

```bash
$ man 8 pam_pwquality
```

## Nastaveni zdroje "jmenych" sluzeb

!Neplest s DNS!

```
/etc/nsswitch.conf
```

## Sifrovany souborovy system

### LVM2

```bash
$ pvcreate /dev/sdb
$ vgcreate vgbsa /dev/sdb
$ lvcreate -L 1G -n test vgbsa
```

### cryptsetup

```bash
$ cryptsetup -y -v luksFormat /dev/vgbsa/test
$ cryptsetup luksOpen /dev/vgbsa/test crypted
$ mkfs.ext4 /dev/mapper/crypted
$ mount /dev/mapper/crypted /mnt
```

### Persistetni pouziti

```bash
/etc/cryptotab
/etc/fstab
```

### Prace s klici

```bash
$ cryptsetup luksDump /dev/vgbsa/test
$ cryptsetup luksAddKey /dev/vgbsa/test /some/key/file
$ cryptsetup luksAddKey /dev/vgbsa/test -S 6
$ cryptsetup luksRemoveKey /dev/vgbsa/test
$ cryptsetup luksKillSlot /dev/vgbsa/test 6
```

### Zaloha a obnoveni

```bash
$ cryptsetup luksHeaderBackup /dev/vgbsa/test --header-backup-file /mnt/vgbsa_test.img
$ cryptsetup luksHeaderRestore /dev/vgbsa/test --header-backup-file /mnt/vgbsa_test.img
```

## CA jednoduse

(kopie easy-rsa)

```bash
$ cp -r /usr/share/easy-rsa/ /etc/CA2
```

(nastaveni CA)

```bash
$ vim /etc/CA2/vars
$ vim /etc/CA2/openssl-1.0.0.cnf
```

(vytvoreni CA)

```bash
cd /etc/CA2
./clean-all
./build-ca
```

(vytvoreni certifikatu)

```bash 
cd /etc/CA2
./build-key-server server.bsa-jindra.bsa
./build-key client.bsa-jindra.bsa
./build-key-pass client.bsa-jindra.bsa
```

(revokace certifikatu)

```bash
cd /etc/CA2
./revoke-full
./list-crl
```


