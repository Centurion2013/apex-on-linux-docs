# JasperReportsIntegration Installation

## Creating JRI owner user

```console
sudo useradd --system --home-dir /var/lib/jri --create-home --shell /sbin/nologin --comment "JasperReportsIntegration owner" jri

sudo usermod -aG jri tomcat
```

## Downloading and unpacking JRI distributive

```console
cd /tmp
curl -L -o jri-latest.zip https://github.com/daust/JasperReportsIntegration/releases/download/v3.0.0/jri-3.0.0-jasper-7.0.1.zip

sudo mkdir -p /opt/jri/
sudo unzip -q /tmp/jri-latest.zip -d /opt/jri

sudo chown -R jri:jri /opt/jri
sudo chmod -R u=rwX,g=rwX,o= /opt/jri/jri-3.0.0-jasper-7.0.1
```

## Stop Tomcat
```console
sudo systemctl stop tomcat

```

## Deploy JRI WAR into Tomcat

```console
sudo cp /opt/jri/jri-3.0.0-jasper-7.0.1/webapp/jri.war /var/lib/tomcat/webapps/jri.war
sudo chown tomcat:tomcat /var/lib/tomcat/webapps/jri.war
```

## Deploy JRI-Assistant WAR into Tomcat

```console
sudo cp /tmp/jri-assistant.war /var/lib/tomcat/webapps/jri-assistant.war
sudo chown tomcat:tomcat /var/lib/tomcat/webapps/jri-assistant.war
```

## Set Environement Variable
```console
sudo nano /etc/sysconfig/tomcat
```
Make changes in the file by adding at the end:
```console
OC_JASPER_CONFIG_HOME=/opt/jri/jri-3.0.0-jasper-7.0.1
JRI_REPORTS_ROOT=/opt/jri/jri-3.0.0-jasper-7.0.1/reports
```
Save and exit.

## Make changes in `/opt/jri/jri-3.0.0-jasper-7.0.1/conf/application.properties` according your needs
See JasperReportsIntegration documentation.

## Start Tomcat
```console
sudo systemctl start tomcat
```

## Watch logs:
```console
sudo journalctl -u tomcat -f
```
