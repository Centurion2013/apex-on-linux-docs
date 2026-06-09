# APEX Installation

## Stop Tomcat during installation
```console
sudo systemctl stop tomcat
```

## Download APEX 26.1
Run:
```console
cd /tmp

curl -L -o apex_26.1.zip https://download.oracle.com/otn_software/apex/apex_26.1.zip
```

Create a clean installation directory:
```console
sudo mkdir -p /opt/apex
sudo rm -rf /opt/apex/apex
sudo unzip -q /tmp/apex_26.1.zip -d /opt/apex
sudo chown -R oracle:oinstall /opt/apex
```

Check:
```console
ls -l /opt/apex/apex
ls -l /opt/apex/apex/apexins.sql
```

## Install APEX into `FREEPDB1`
Now run the install:
```console
sudo -iu oracle

cd /opt/apex/apex

sqlplus / as sysdba
```

Inside SQL*Plus:
alter session set container = FREEPDB1;

```console
spool install_apex_26_1.log

@apexins.sql SYSAUX SYSAUX TEMP /i/

spool off
exit
```

This may run for several minutes. The log file will be here:
```console
/opt/apex/apex/install_apex_26_1.log
```

Check for errors:
```console
grep -i "ORA-\|PLS-\|SP2-" /opt/apex/apex/install_apex_26_1.log
```
*Some harmless messages may appear, but real `ORA-` errors need attention.*

## Create the APEX Instance Administrator account
Inside SQL*Plus:
```console
alter session set container = FREEPDB1;

@apxchpwd.sql
```

## Configure APEX REST/static support
```console
@apex_rest_config.sql
```
*Oracle’s APEX guide says `apex_rest_config.sql` must be run after a new APEX installation when using ORDS, because APEX static files and workspace/application static files are served through ORDS-related REST support.*

## Copy APEX images to Tomcat
Because you used `/i/` during apexins.sql, Tomcat must serve APEX static resources from `/i/`.

*Oracle says the APEX images directory must be copied to a local filesystem accessible by ORDS/Tomcat unless you choose the Oracle CDN.*

```console
sudo rm -rf /var/lib/tomcat/webapps/i
sudo mkdir -p /var/lib/tomcat/webapps/i

sudo rsync -a /opt/apex/apex/images/ /var/lib/tomcat/webapps/i/

sudo chown -R tomcat:tomcat /var/lib/tomcat/webapps/i
```

## Install runtime language translations
To load `Ukrainian` translations:

```console
sudo -iu oracle

cd /opt/apex/apex/builder/uk

sqlplus / as sysdba

```

SQL:
```console
alter session set container = FREEPDB1;
@load_uk.sql
exit
```

## Restart Tomcat
```console
sudo systemctl start tomcat
sudo systemctl status tomcat --no-pager
```

