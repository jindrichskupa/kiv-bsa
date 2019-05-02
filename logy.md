# Logovani

## rsyslog

```bash
apt-get install rsyslog rsyslog-pgsql
```

```bash
$ModLoad ompgsql
$template StdPgSQLFmt,"insert into SystemEvents (Message, Facility, FromHost, Priority, DeviceReportedTime, ReceivedAt, InfoUnitID, SysLogTag) values ('%msg%', %syslogfacility%, '%HOSTNAME%', %syslogpriority%, '%timereported:::date-pgsql%', '%timegenerated:::date-pgsql%', %iut%, '%syslogtag%')",STDSQL
$template mytemplate,"insert into SystemEvents (Message) values ('%msg%')",STDSQL
mail.* :ompgsql:database-server,database-name,database-userid,database-password;mytemplate
```

```bash
ERRORLOG  "|/USR/BIN/TEE -A /VAR/LOG/WWW/ERROR.LOG  | /USR/BIN/LOGGER -THTTPD -PLOCAL6.ERR"
CUSTOMLOG "|/USR/BIN/TEE -A /VAR/LOG/WWW/ACCESS.LOG | /USR/BIN/LOGGER -THTTPD -PLOCAL6.NOTICE" EXTENDED_NCSA
```

```bash
$UDPServerAddress 0.0.0.0
$UDPServerRun 514

$RepeatedMsgReduction on
$RepeatedMsgContainsOrigionalMsg on

if $programname == 'dovecot' and $syslogseverity <= '6' then /var/log/dovecot.log

*.* @@1.2.3.4:2010;SyslFormat
```


```bash
$template HourlyMailLog,"/var/log/logdir/%$YEAR%/%$MONTH%/%$DAY%/%HOSTNAME%_mail.log
$template SyslFormat,"%timegenerated% %HOSTNAME% %syslogtag%%msg:::space$ 
# Log all the mail messages in one place.
mail.*                                                  -?HourlyMailLog;SyslFormat
```

```bash
CustomLog "|/usr/bin/logger -t httpd -p local6.info" combined
CustomLog logs/access_log combined

if $syslogfacility-text == 'local6' and $programname == 'httpd' then /var/log/httpd-access.log
```

/usr/share/doc/rsyslog-pgsql


PortSentry
apt-get install portsentry

service portsentry stop|start|restart

/etc/portsentry/portsentry.conf:
BLOCK_UDP="1",
BLOCK_TCP="1",

/etc/portsentry/
/var/log/syslog

nmap -p 1-65535 -T4 -A -v -PE -PS22,25,80 -PA21,23,80 10.228.67.X


Kontrola:
grep "attackalert" /var/log/syslog
grep -n DENY /etc/hosts.deny
grep -n Blocked /var/lib/portsentry/portsentry.blocked.tcp
grep -n Blocked /var/lib/portsentry/portsentry.history
grep -n Blocked /var/lib/portsentry/portsentry.blocked.udp
netstat -rn | grep "10.228.67.X"
route -n | grep "10.228.67.X"


PortSentry
apt-get install portsentry

service portsentry stop|start|restart

/etc/portsentry/portsentry.conf:
BLOCK_UDP="1",
BLOCK_TCP="1",

/etc/portsentry/
/var/log/syslog

nmap -p 1-65535 -T4 -A -v -PE -PS22,25,80 -PA21,23,80 10.228.67.X

Kontrola:
grep "attackalert" /var/log/syslog
grep -n DENY /etc/hosts.deny
grep -n Blocked /var/lib/portsentry/portsentry.blocked.tcp
grep -n Blocked /var/lib/portsentry/portsentry.history
grep -n Blocked /var/lib/portsentry/portsentry.blocked.udp
netstat -rn | grep "10.228.67.X"
route -n | grep "10.228.67.X"

Odblokovat:
• zastavit portsentry
• odstranit zaznam z /etc/hosts.deny
• odstranit zaznamy z portsentry.blocked.tcp, portsentry.history a portsentry.blocked.udp
○ sed -i '/10.228.67.X/d' <jmeno souboru>
• odstrani reject routu:
○ route del -host 10.228.67.X reject
• nastartovat portsentry

RKhunter

apt-get install rkhunter 

rkhunter --update
rkhunter --propupd
rkhunter --list
rkhunter -c --enable hidden_ports

vim /etc/rkhunter.conf

MAIL-ON-WARNING="root@localhost"
MAIL_CMD=mail -s "[rkhunter] Warnings found for ${HOST_NAME}"
SCRIPTWHITELIST="/usr/sbin/adduser"
ALLOWDEVFILE="/dev/.udev/rules.d/root.rules"
ALLOWHIDDENDIR="/dev/.udev"
ALLOWHIDDENFILE="/dev/.blkid.tab"
ALLOW_SSH_ROOT_USER=yes

rkhunter -C

Cron

/etc/crontab
/etc/cron.d
/etc/cron.daily
/etc/cron.hourly
/etc/cron.monthly
/etc/cron.weekly
/var/spool/cron/crontabs/

crontab -l
crontab -e
crontab -l -u jindra

15 */4 * * * /usr/bin/rkhunter --cronjob --update --quiet


Rsync
poroz na lomitka na konci :)
rsync -rav /opt/dir1/ /opt/backup_dir/
rsync -rai --dry-run /etc/ /opt/etc_backup_full/
rsync -a --delete --link-dest=../backup.1 /etc/  backup.0/i
