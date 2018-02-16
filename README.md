# kiv-bsa

## Komplexni test pred zapoctem

GPG KeyServer: <http://147.228.67.151:11371/>

### Body k realizaci

1. Vytvorte a nahrajte svoje GPG verejne klice na server vyse
2. Z <https://147.228.67.151/openvpn.conf> a certifikaty <https://147.228.67.151/orion_login.crt>, <https://147.228.67.151/orion_login.key> stahnete konfiguraci clienta OpenVPN
  * pridejte si rucne routu do ip/sit 192.168.99.151/24 pres VPN
  * z adresy <http://192.168.99.151/cert.crt> a <http://192.168.99.151/cert.key> pro stunnel
3. Pomoci stunnelu se pripojte na ...
4. Nastavte si DNS server na adresu 147.228.67.151
5. ..
6. Nastavte si na verejnem rozhrani adresu ze site 192.168.100.0/24
7. Odeslete apache logy na 192.168.99.151:514 (UDP)
8. Nastavte prijem logu z 147.228.67.151
9. Pomoci SQL Injection zjistete nakupni cenu knizek na adrese: <http://192.168.100.151/book_store.php>
