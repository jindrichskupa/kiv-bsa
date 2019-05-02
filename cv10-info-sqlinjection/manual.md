# Cviceni 10

## Informace

```
/etc/apache2/conf-enabled/security.conf
ServerSignature Off
ServerTokens ProductOnly

<Directory /var/www/html/dontlistme>
  Options -Indexes +SymLinksIfOwnerMatch -FollowSymLinks
  AllowOverride None
  Order deny,allow
  Deny from all
  Allow from 127.0.0.0/255.0.0.0 ::1/128  
</Directory>

<Location /secret>
  AuthType Basic
  AuthName "Password Required"
  AuthUserFile "/etc/apache2/passwd"
  Require user pepa
</Location>
```

```
htpasswd -c /etc/apache2/passwd pepa
```

```
/etc/postfix/main.cf
smtpd_banner = $myhostname ESMTP MS Exchange (Hurd/GNU)
```

```
/etc/dovecot/dovecot.conf
login_greeting = Secret server, ready.
```

## SSL

```bash
$ a2enmod ssl
```

```
/etc/apache2/sites-enabled/default-ssl.conf
<IfModule mod_ssl.c>
  <VirtualHost _default_:443>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    SSLEngine on
    SSLCertificateFile	/etc/apache2/ssl/server.crt
    SSLCertificateKeyFile /etc/apache2/ssl/server.key
    SSLCertificateChainFile /etc/apache2/ssl/ca.crt

    SSLProtocol  TLSv1 TLSv1.1 TLSv1.2 -SSLv2 -SSLv3
    SSLHonorCipherOrder On
    SSLCipherSuite AES128+EECDH:AES128+EDH

    BrowserMatch "MSIE [2-6]" \
      nokeepalive ssl-unclean-shutdown \
      downgrade-1.0 force-response-1.0
    BrowserMatch "MSIE [17-9]" ssl-unclean-shutdown
  </VirtualHost>
</IfModule>
```

## PHP

php.ini

```ini
/etc/php5/apache2/php.ini

open_basedir = /var/www/html
disable_functions =
disable_classes =
display_errors = Off
html_errors = Off
allow_url_include = Off
allow_url_fopen = Off
max_execution_time = 30
max_input_time = 60
memory_limit = 128M
```

## SQL injection

* MySQL

```sql
mysql> create database db01;
mysql> grant all privileges on `db01`.* to 'db01'@'localhost' identified by 'password';
mysql> CREATE TABLE `table01` (
  `id` int(6) unsigned NOT NULL AUTO_INCREMENT,
  `firstname` varchar(30) NOT NULL,
  `lastname` varchar(30) NOT NULL,
  `email` varchar(50) DEFAULT NULL,
  `reg_date` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `password` varchar(20) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=101 DEFAULT CHARSET=latin1
```

```php
<?php
$servername = "localhost";
$username = "db01";
$password = "password";
$dbname = "db01";

// Create connection
$conn = new mysqli($servername, $username, $password, $dbname);
// Check connection
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}

$sql = "SELECT id, firstname, password FROM table01 WHERE id=".$_GET['id'].";";
$result = $conn->query($sql);

if ($result->num_rows > 0) {
    // output data of each row
    while($row = $result->fetch_assoc()) {
        echo "id: " . $row["id"]. " - Name: " . $row["firstname"]. " password: " . $row["password"]. "<br>";
    }
} else {
    echo "0 results";
}

$sql = "SELECT firstname,reg_date FROM table01 WHERE firstname='". $_GET['login'] ."' AND password='". $_GET['password'] ."'";
$result = $conn->query($sql);
echo "<br><br>";
if ($result->num_rows > 0) {
	echo "logged in";
} else {
	echo "wrong password";
}


$conn->close();
?>
```

* PostgreSQL

```sql
psql> CREATE USER db01 WITH PASSWORD 'pasword';
psql> CREATE DATABASE db01 WITH OWNER=db01;
psql> CREATE TABLE table01(
   id SERIAL PRIMARY KEY,
   firstname VARCHAR(30) NOT NULL,
   lastname VARCHAR(30) NOT NULL,
   email VARCHAR(50),
   password VARCHAR(50),
   reg_date TIMESTAMP
);
```

```php
<?php
// Connecting, selecting database
$dbconn = pg_connect("host=localhost dbname=db01 user=db01 password=pasword")
    or die('Could not connect: ' . pg_last_error());

// Performing SQL query
$query = "SELECT * FROM table01 WHeRE email like '%@".$_GET['email']."'";

$result = pg_query($query) or die('Query failed: ' . pg_last_error());

// Printing results in HTML
echo "<table>\n";
while ($line = pg_fetch_array($result, null, PGSQL_ASSOC)) {
    echo "\t<tr>\n";
    foreach ($line as $col_value) {
        echo "\t\t<td>$col_value</td>\n";
    }
    echo "\t</tr>\n";
}
echo "</table>\n";

// Free resultset
pg_free_result($result);

// Closing connection
pg_close($dbconn);
?>
```

```bash
apt-get install git
git clone https://github.com/sqlmapproject/sqlmap.git
```

```
./sqlmap.py -u "http://192.168.1.151/mysql.php?email=pepa" -f --banner --dbs --users
```

## Dalsi zranitelnosti

```bash
apt-get install wapiti apachetop goaccess
```

```php
<form method="GET" action="form.php">
<input type="text" name="neco">
<input type="submit" value="odeslat">
</form>


<?php

if (isset($_POST['neco'])) {
 file_put_contents ("./forum.txt", $_POST['neco']. "\n");
}

?>

<?php
echo file_get_contents("./forum.txt");
?>
```

```
wapiti "http://192.168.1.151/psql.php?email=ahoj"
wapiti "http://192.168.1.151/form.php?neco=ahoj"
```