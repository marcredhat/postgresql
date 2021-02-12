# PostgreSQL SCRAM test

On RHEL 7:
```bash
yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
yum install -y postgresql10-server
/usr/pgsql-10/bin/postgresql-10-setup initdb
systemctl enable postgresql-10
systemctl start postgresql-10
```

On RHEL 8:
```bash
sudo yum module list | grep postgresql
sudo yum install @postgresql:10
sudo postgresql-setup --initdb
systemctl enable  postgresql
systemctl start  postgresql
```

I'm using RHEL 8:
```bash
grep scram /var/lib/pgsql/data/pg_hba.conf
host    all             all             127.0.0.1/32            scram-sha-256
host    all             all             ::1/128                 scram-sha-256
host    replication     all             127.0.0.1/32            scram-sha-256
host    replication     all             ::1/128                 scram-sha-256
```

```bash
grep scram /var/lib/pgsql/data/postgresql.conf
password_encryption = scram-sha-256 	# md5 or scram-sha-256
```

```bash
systemctl restart postgresql
```

```bash
sudo -i -u postgres
psql
[postgres@marcrhel82 ~]$ psql
psql (10.15)
Type "help" for help.

postgres=# CREATE ROLE testscram LOGIN PASSWORD 'password';
CREATE ROLE
postgres=# CREATE DATABASE testscramdb OWNER testscram ENCODING 'UTF8';

CREATE DATABASE
postgres=# ALTER USER testscram with encrypted password 'password';
ALTER ROLE
```

```bash
sudo -u postgres psql -c '\l' | grep testscramdb
testscramdb | testscram | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
```

Check user testscram's password is SCRAM-encrypted

```bash
sudo -u postgres psql -c 'select rolname,rolpassword from pg_authid;' | grep testscram
testscram            | SCRAM-SHA-256$4096:rd0NfQbEHmtCOFR0GCMJmw==$upkodf+xfvDVSh+42EMRNWopAx6uuBY6cIswkDlGGSY=:2fJcEsZ8HS83xRbC0BW9iMkO7Ee4qk5OGVpbXf0WhUs=
```

```bash
wget https://jdbc.postgresql.org/download/postgresql-42.2.18.jar
```

```bash
cat TestConnect.java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.util.Properties;public class TestConnect {
public static void main(String s[])throws SQLException, ClassNotFoundException, InterruptedException
{
Class.forName("org.postgresql.Driver");
String url = "jdbc:postgresql://127.0.0.1:5432/testscramdb?loggerLevel=TRACE&loggerFile=/tmp/pgjdbc.log";
Properties props = new Properties();
props.setProperty("user", "testscram");
props.setProperty("password", "password");
Connection conn = DriverManager.getConnection(url, props); System.out.println("Connected"); } }
```

```bash
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.275.b01-1.el8_3.x86_64/bin/javac TestConnect.java
```

```bash
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.275.b01-1.el8_3.x86_64/bin/java -cp "./postgresql-42.2.18.jar:." TestConnect
Connected
```
