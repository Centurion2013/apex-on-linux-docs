# Oracle Database 26ai Free Installation

## Pre-requirements
At this moment you must complete [Oracle Linux installation procedure](linux_installation.md)

## Install the Oracle AI Database Preinstallation RPM
```
sudo dnf -y install oracle-ai-database-preinstall-26ai
```

* The Oracle AI Database Preinstallation RPM automatically creates the Oracle installation owner and groups. It also sets up other kernel configuration settings as required for Oracle AI Database installations. If you plan to use job-role separation, then create the extended set of database users and groups depending on your requirements.
* Review the RPM log files to determine the system configuration changes. For example, review /var/log/oracle-database-preinstall-26ai/results/orakernel.log.

## Download Oracle AI Database software
Download page: https://www.oracle.com/database/technologies/free-downloads.html

Download `oracle-ai-database-free-26ai-23.26.2-1.el9.x86_64.rpm` RPM file required for performing an RPM-based installation to a directory of your choice.

*Note: time after time the Oracle updates version available to download, so exact file names can be different. Take it in account.*

## Install database software
```
sudo dnf -y install oracle-ai-database-free-26ai-23.26.2-1.el9.x86_64.rpm
```
or
```
sudo dnf -y install oracle-ai-database-free*
```
## Creating and Configuring an Oracle AI Database
The configuration script creates a container database (`FREE`) with one pluggable database (`FREEPDB1`) and configures the listener at the default port (`1521`).

You can modify the configuration parameters by editing the `/etc/sysconfig/oracle-free–26ai.conf file`.

The parameters set in this file are explained in detail in the silent mode installation procedure Oracle's online documentation: [Performing a Silent Installation](https://docs.oracle.com/en/database/oracle/oracle-database/26/xeinl/installing-oracle-database-free.html#GUID-3F29EE7C-4546-49EE-B894-027EE3E371BF).

To create the database with the default settings:

* Log in as root using sudo.
```
sudo -s
```
Run the service configuration script:
```
/etc/init.d/oracle-free-26ai configure
```
At the command prompt, specify a password for the SYS, SYSTEM, and PDBADMIN administrative user accounts. Oracle recommends that your password should be at least 8 characters in length, contain at least 1 upper case character, 1 lower case character, and 1 digit [0-9].

The same password will be used for these accounts. The password should conform to the Oracle recommended standards. See Oracle AI Database Security Guide for more information about guidelines for securing passwords
After the configuration completes, the database and listener are started.

## Configuration, Database Files and Logs Location
| File Name and Location | Purpose |
|------|-------------|
| /opt/oracle |Oracle base. This is the root of the Oracle AI Database Free directory tree. |
| /opt/oracle/product/26ai/dbhomeFree | Oracle home. This home is where the Oracle AI Database Free is installed. It contains the directories of the Oracle AI Database Free executables and network files. |
| /opt/oracle/oradata/FREE | Database files. |
| /opt/oracle/diag subdirectories | Diagnostic logs. The database alert log is /opt/oracle/diag/rdbms/free/FREE/trace/alert_FREE.log |
| /opt/oracle/cfgtoollogs/dbca/FREE | Database creation logs. The FREE.log file contains the results of the database creation script execution. |
| /etc/sysconfig/oracle-free-26ai.conf | Configuration default parameters. |
| /etc/init.d/oracle-free-26ai | Configuration and services script |

## Setting Oracle AI Database Free Environment Variables

- Switch to user `oracle`
```
sudo -iu oracle
```
- Edit `.bash_profile`
```
nano ~/.bash_profile
```
Add this at the end:
```
export ORACLE_SID=FREE
export ORACLE_HOME=/opt/oracle/product/26ai/dbhomeFree
export PATH=$ORACLE_HOME/bin:$PATH
export NLS_LANG=AMERICAN_AMERICA.AL32UTF8
```
Save and exit.
- Reload the profile
```
source ~/.bash_profile
```
At this point you can call sqlplus under oracle user without path specification

## Connecting to Oracle AI Database
Please make sure the listener port (`1521`) is accessible from outside.

To check the firewall status:
```
sudo firewall-cmd --state
```
if it says:
```
running
```
open port `1521`:
```
sudo firewall-cmd --add-port=1521/tcp --permanent
sudo firewall-cmd --reload
```
then verify:
```
sudo firewall-cmd --list-ports
```
you should see:
```
1521/tcp
```

At this moment Oracle AI Database Free is installed and accessible to connect.

## Automating Shutdown and Start-Up
Oracle recommends that you configure the system to automatically start Oracle AI Database Free when the system starts, and to automatically shut it down when the system shuts down.

To automate the start up and shutdown of the listener and database, run the following commands as `root`:
```
$ sudo -s
```
For Oracle Linux 8 and Oracle Linux 9:
```
# systemctl daemon-reload
# systemctl enable oracle-free-26ai
```

### Checking status
```
/etc/init.d/oracle-free-26ai status
```
### Starting
```
systemctl start oracle-free-26ai
```
### Stopping
```
systemctl stop oracle-free-26ai
```
### Restarting
```
systemctl restart oracle-free-26ai
```


