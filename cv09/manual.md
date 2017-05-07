# Cviceni 9

## Logovani

* syslog
* rsyslog
* syslog-ng
* ...

```bash
apt-get install rsyslog rsyslog-pgsql
```

* Formatovani logu

```
# rozdeleni logu do adresaru
$template HourlyMailLog,"/var/log/logdir/%$YEAR%/%$MONTH%/%$DAY%/%HOSTNAME%_mail.log

# formatovani logu
$template SyslFormat,"%timegenerated% %HOSTNAME%  %syslogtag%%msg:::space$ 

# zapis logu do souboru dle definice a formatu
mail.*                                                  -?HourlyMailLog;SyslFormat
```

* Vzdalene logovani

```
# naslouchat na 0.0.0.0/514/UDP
$UDPServerAddress 0.0.0.0
$UDPServerRun 514

$RepeatedMsgReduction on
$RepeatedMsgContainsOrigionalMsg on

# odesilat vsechny logy na server 192.168.4.151
*.* @192.168.4.151
```

* Logovani do RDBMS

```bash
/etc/rsyslog.d/mail2postgres.conf
$ModLoad ompgsql
$template PSQLFullFormat,"insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values ('%msg%', %syslogfacility%, '%HOSTNAME%', %syslogpriority%, '%timereported:::date-pgsql%', '%timegenerated:::date-pgsql%', %iut%, '%syslogtag%')",STDSQL
$template PSQLSimpleFormat,"insert into SystemEvents (Message) values ('%msg%')",STDSQL
mail.* :ompgsql:database-server,database-name,database-userid,database-password;PSQLSimpleFormat
```

* Apache error/access log

```bash
apache-vhost.conf
errorlog  "|/usr/bin/tee -a /var/log/www/error.log  | /usr/bin/logger -t httpd -p local6.err"
customlog "|/usr/bin/tee -a /var/log/www/access.log | /usr/bin/logger -t httpd -p local6.notice" extended_ncsa
```

```
/etc/rsyslog.d/apache.conf
if $syslogfacility-text == 'local6' and $programname == 'httpd' and $level == then /var/log/rsyslog-apache-log.log
```

## PortSentry

```
apt-get install portsentry
service portsentry stop|start|restart
```

Nasteveni:
```
/etc/portsentry/portsentry.conf:
BLOCK_UDP="1",
BLOCK_TCP="1",
```

Skenovani portu:
```
nmap -p 1-65535 -T4 -A -v -PE -PS22,25,80 -PA21,23,80 192.168.1.151
```

Kontrola:
```
grep "attackalert" /var/log/syslog
grep -n DENY /etc/hosts.deny
grep -n Blocked /var/lib/portsentry/portsentry.blocked.tcp
grep -n Blocked /var/lib/portsentry/portsentry.history
grep -n Blocked /var/lib/portsentry/portsentry.blocked.udp
netstat -rn | grep "192.168.1.151"
route -n | grep "192.168.1.151"
```

Odblokovat:
* zastavit portsentry
* odstranit zaznam z /etc/hosts.deny
* odstranit zaznamy z portsentry.blocked.tcp, portsentry.history a portsentry.blocked.udp
  * `sed -i '/192.168.1.151/d' jmeno_souboru`
* odstranit reject routu:
  * `route del -host 192.168.1.151 reject`
* nastartovat portsentry
