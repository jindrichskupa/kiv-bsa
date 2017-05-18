# Proverka

## Zadani

1. Nastavte si DNS na ip `147.228.67.151`
2. Vygenerujte (nebo pouzijte existujici) a ulozte vas gpg klic na server gpg.bsa-test.bsa (pozor musi obsahovat v mailu/komentaci retezec "bsa")
3. Stahnete si konfiguraci VPN na adrese https://vpn.bsa-test.bsa/ (najdete seznamu configu, ten ktery je vas a pouzijte ho pro pripojeni)
4. Poslete mail na adresu `kontrola@bsa-test.bsa`, kde predmet bude vas orion login
5. Nastavte logovani vsech mailovych zprav na server `log.bsa-test.bsa`, muzete overit na `https://log.bsa-test.bsa`
6. Nastavte prijem logu pres UDP a nastavte zapis mail logu do souboru `/var/log/bsa/<hostname>.mail.log`
7. Nastavte logrotate tak aby nove mail logy rotoval denne, rovnou komprimovane + je podepsal
8. Vytvorte databazi v PostgreSQL `bsa_test`
  * `bsa_test`, vlastnik, vsechna prava, pristup z local podle systemoveho username, z localhostu s heslem
  * `bsas_ro`, pouze read-only, pristup z 192.168.123.0/24
