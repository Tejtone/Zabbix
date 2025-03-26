# Instalacja Zabbix z MySQL

Ten repozytorium zawiera krok po kroku instrukcję instalacji Zabbixa z MySQL na systemie Linux opartym na RHEL (np. Rocky Linux 9).

## Wymagania wstępne

Upewnij się, że masz uprawnienia root lub sudo na swoim systemie. Upewnij się także, że Twój system jest zaktualizowany i ma połączenie z siecią w celu pobrania niezbędnych pakietów.

---

## Kroki instalacji

### 1. Skonfiguruj repozytorium EPEL

Edytuj plik konfiguracyjny repozytorium EPEL:

```bash
sudo nano /etc/yum.repos.d/epel.repo
```

Upewnij się, że plik zawiera następujące dane:

```
[epel]
...
excludepkgs=zabbix*
```

Zapisz i zamknij plik.

### 2. Zainstaluj MySQL Server

```bash
dnf install mysql-server -y
systemctl enable mysqld --now
```

### 3. Zabezpiecz instalację MySQL

Uruchom skrypt zabezpieczający MySQL:

```bash
/usr/bin/mysql_secure_installation
```

Postępuj zgodnie z instrukcjami:
VALIDATE PASSWORD COMPONENT can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD component?
Press y|Y for Yes, any other key for No: n
Please set the password for root here.
New password:
Re-enter new password:
By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production
environment.
Remove anonymous users? (Press y|Y for Yes, any other key for No) : y
Success.
Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.
Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y
Success.
By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,
and should be removed before moving into a production
environment.
Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y
 - Dropping test database...
Success.
 - Removing privileges on test database...
Success.
Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.
Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
Success.
All done!

### 4. Dodaj repozytorium Zabbix

```bash
rpm -Uvh https://repo.zabbix.com/zabbix/7.2/release/rocky/9/noarch/zabbix-release-latest-7.2.el9.noarch.rpm
dnf clean all
```

### 5. Zainstaluj pakiety Zabbix

```bash
dnf install zabbix-server-mysql zabbix-web-mysql zabbix-apache-conf zabbix-sql-scripts zabbix-selinux-policy zabbix-agent -y
```

### 6. Skonfiguruj MySQL dla Zabbix

1. Zaloguj się do MySQL jako root:

```bash
mysql -uroot -p
```

2. Wykonaj następujące polecenia SQL:

```sql
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER zabbix@localhost IDENTIFIED BY 'PASSWORD';
GRANT ALL PRIVILEGES ON zabbix.* TO zabbix@localhost;
SET GLOBAL log_bin_trust_function_creators = 1;
QUIT;
```

3. Zaimportuj schemat bazy danych Zabbix:

```bash
zcat /usr/share/zabbix/sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
```

4. Przywróć ustawienie `log_bin_trust_function_creators`:

```bash
mysql -uroot -p
SET GLOBAL log_bin_trust_function_creators = 0;
QUIT;
```

### 7. Skonfiguruj serwer Zabbix

Edytuj plik konfiguracyjny serwera Zabbix:

```bash
sudo nano /etc/zabbix/zabbix_server.conf
```

Zaktualizuj pole `DBPassword`, wpisując hasło użytkownika MySQL Zabbix:

```
### Option: DBPassword
...
DBPassword=PASSWORD
```

### 8. Uruchom i włącz usługi

```bash
systemctl restart zabbix-server zabbix-agent httpd php-fpm
systemctl enable zabbix-server zabbix-agent httpd php-fpm --now
```

### 9. Skonfiguruj zaporę

Otwórz port 80 dla ruchu HTTP:

```bash
sudo firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --reload
```

---

## Dostęp do interfejsu webowego Zabbix

1. Otwórz przeglądarkę internetową i przejdź do:
   ```
   http://xxx.xxx.xxx.xxx/zabbix
   ```
2. Postępuj zgodnie z instrukcjami na ekranie, aby zakończyć instalację webową Zabbix.
