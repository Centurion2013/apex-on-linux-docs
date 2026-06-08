# Tomcat 9 & ORDS Installation

ORDS 26.1 supports Java 17 or 21 and Apache Tomcat 9.0.x.
Tomcat and ORDS will be installed for separate users with limited rights.

## Assumptions

| Setting | Value |
|---------|-------|
| DB host | localhost |
| DB port | 1521 |
| PDB service | FREEPDB1 |
| Tomcat URL | http://server-ip:8080/ords/ |
| ORDS config | /etc/ords/config |
| ORDS logs | /var/log/ords |
| ORDS product | /opt/ords/current |
|Tomcat9 user|tomcat|
|ORDS owner|ords|

## Install Java 17
```console
sudo dnf update -y

sudo dnf install -y java-17-openjdk-devel unzip curl firewalld policycoreutils-python-utils

java -version
```

## Install Tomcat9
Oracle Linux 9 includes an Apache Tomcat 9 package. ([docs.oracle.com](https://docs.oracle.com/en/operating-systems/oracle-linux/9/relnotes9.2/ol9-NewFeaturesandChanges.html))
```console
sudo dnf install -y tomcat
sudo systemctl enable --now tomcat
sudo systemctl status tomcat --no-pager
```
Check that the package created the Tomcat OS user:
```console
id tomcat
```

## Create an OS account for ORDS configuration
This account owns ORDS configuration and wallets. Tomcat will only be allowed to read them.
```console
sudo useradd --system --home-dir /var/lib/ords --create-home --shell /sbin/nologin --comment "Oracle REST Data Services owner" ords

sudo usermod -aG ords tomcat

sudo mkdir -p /etc/ords/config /var/log/ords
sudo chown -R ords:ords /etc/ords /var/log/ords
sudo chmod 750 /etc/ords /etc/ords/config /var/log/ords
```

### Recommended account model:

| Account    | Purpose                                       |
| ---------- | --------------------------------------------- |
| `oracle`   | Oracle Database software and DB instance only |
| `sp*****v` | Your admin user with `sudo`                   |
| `tomcat`   | Tomcat service runtime                        |
| `ords`     | ORDS config/wallet/log owner                  |

## Download and unpack ORDS
*Oracle’s current ORDS download page lists ORDS 26.1.2 as the latest 26.1 release, with Java 17/21 support.*

```console
cd /tmp
curl -L -o ords-latest.zip https://download.oracle.com/otn_software/java/ords/ords-latest.zip

ORDS_VER=26.1.2
sudo mkdir -p /opt/ords/${ORDS_VER}
sudo unzip -q /tmp/ords-latest.zip -d /opt/ords/${ORDS_VER}

sudo chown -R root:root /opt/ords/${ORDS_VER}
sudo ln -sfn /opt/ords/${ORDS_VER} /opt/ords/current

/opt/ords/current/bin/ords --version
```
*Oracle recommends keeping the ORDS configuration directory separate from the application files, and adding the ORDS `bin` folder to the OS path is recommended.*

Optional convenience symlink:
```console
sudo ln -sfn /opt/ords/current/bin/ords /usr/local/bin/ords
ords --version
```

## Install/configure ORDS into `FREEPDB1`
Run ORDS installer as the ords OS account:
```console
sudo -u ords -H /opt/ords/current/bin/ords --config /etc/ords/config install --interactive --log-folder /var/log/ords
```
Use these answers:

| Setting | Value |
|---------|-------|
| Connection type | Basic |
| Database host | localhost |
| Database port | 1521 |
| Database service name | FREEPDB1 |
| Administrator user | SYS |
| Password | `SYS password` AS SYSDBA |
| Default tablespace | SYSAUX |
| Temp tablespace | TEMP |
| Configure Standalone | No |
| Database Actions | Yes, if offered |

*For the first install, using **SYS AS SYSDBA** is the simple route. Oracle also provides `ords_installer_privileges.sql` if you prefer to grant installer privileges to another DB user instead of using **SYS**.*

After install, fix permissions so Tomcat can read the ORDS configuration:
```console
sudo chown -R ords:ords /etc/ords/config /var/log/ords
sudo chmod -R u=rwX,g=rX,o= /etc/ords/config /var/log/ords
sudo find /etc/ords/config -type f -exec chmod 640 {} \;
sudo find /etc/ords/config -type d -exec chmod 750 {} \;
```

## If you use APEX, enable PL/SQL Gateway mode
APEX requires PL/SQL Gateway access through ORDS. Oracle documents setting `plsql.gateway.mode` to `proxied`.
```console
sudo -u ords -H /opt/ords/current/bin/ords --config /etc/ords/config  config set plsql.gateway.mode proxied
```

## Tell Tomcat where ORDS config is
For Tomcat deployment, Oracle says ORDS must know the configuration directory; one supported way is setting the config.url Java system property before starting Tomcat.([docs.oracle.com](https://docs.oracle.com/en/database/oracle/oracle-rest-data-services/26.1/ordig/deploying-and-monitoring-oracle-rest-data-services.html))

Edit Tomcat environment:
```console
sudo cp /etc/sysconfig/tomcat /etc/sysconfig/tomcat.bak.$(date +%F_%H%M%S)

sudo tee -a /etc/sysconfig/tomcat >/dev/null <<'EOF'

# ORDS
JAVA_OPTS="-Xms256m -Xmx1024m -Dfile.encoding=UTF-8 -Dconfig.url=/etc/ords/config"
EOF
```

## Deploy ORDS WAR into Tomcat
Oracle’s Tomcat deployment step is basically: put `ords.war` into Tomcat’s `webapps` folder; the WAR file name determines the context root, so `ords.war` becomes `/ords`.
```console
sudo cp /opt/ords/current/ords.war /var/lib/tomcat/webapps/ords.war
sudo chown tomcat:tomcat /var/lib/tomcat/webapps/ords.war

sudo systemctl restart tomcat
```

Watch logs:
```console
sudo journalctl -u tomcat -f
```
Or:
```console
sudo ls -l /var/log/tomcat
sudo tail -f /var/log/tomcat/*
```
Test locally:
```console
curl -I http://localhost:8080/ords/
curl http://localhost:8080/ords/
```

## Open the firewall
```console
sudo systemctl enable --now firewalld

sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

sudo firewall-cmd --list-ports
```
