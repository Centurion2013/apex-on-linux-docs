# Oracle Database 26ai Free Installation

## Pre-requirements
At this moment you must complete [Oracle Linux installation procedure](linux_installation.md)

## Install the Oracle AI Database Preinstallation RPM
```console
sudo dnf -y install oracle-ai-database-preinstall-26ai
```

* The Oracle AI Database Preinstallation RPM automatically provisions the Oracle owner account and necessary groups, while configuring the kernel parameters essential for Oracle AI Database deployments. For implementations requiring job-role separation, you should additionally establish additional database users and groups based on your specific needs.
* Examine the RPM log files to track what modifications were made to your system configuration. For instance, check `/var/log/oracle-database-preinstall-26ai/results/orakernel.log`.

## Download Oracle AI Database software
Download page: https://www.oracle.com/database/technologies/free-downloads.html

Download `oracle-ai-database-free-26ai-23.26.2-1.el9.x86_64.rpm` RPM file required for performing an RPM-based installation to a directory of your choice.

*Note: time after time the Oracle updates version available to download, so exact file names can be different. Take it in account.*

## Install database software
```console
sudo dnf -y install oracle-ai-database-free-26ai-23.26.2-1.el9.x86_64.rpm
```
or
```console
sudo dnf -y install oracle-ai-database-free*
```
## Creating and Configuring an Oracle AI Database
The configuration script creates a container database (`FREE`) with one pluggable database (`FREEPDB1`) and configures the listener at the default port (`1521`).

*You can modify the configuration parameters by editing the `/etc/sysconfig/oracle-free–26ai.conf file`.*
*The parameters set in this file are explained in detail in the silent mode installation procedure Oracle's online documentation: [Performing a Silent Installation](https://docs.oracle.com/en/database/oracle/oracle-database/26/xeinl/installing-oracle-database-free.html#GUID-3F29EE7C-4546-49EE-B894-027EE3E371BF).*

To create the database with the default settings:

* Log in as root using sudo.
```console
sudo -s
```
Run the service configuration script:
```console
/etc/init.d/oracle-free-26ai configure
```
When prompted, enter a password for the `SYS`, `SYSTEM`, and `PDBADMIN` administrative accounts. Oracle advises that passwords should have a minimum length of 8 characters and include at least one uppercase letter, one lowercase letter, and one numeric character [0-9].

A single password is applied to all three administrative accounts and must adhere to Oracle's security best practices. For detailed password security guidelines, refer to the Oracle AI Database Security Guide. Once configuration finishes, both the database and listener automatically start.

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
```console
sudo -iu oracle
```
- Edit `.bash_profile`
```console
nano ~/.bash_profile
```
Add this at the end:
```console
export ORACLE_SID=FREE
export ORACLE_HOME=/opt/oracle/product/26ai/dbhomeFree
export PATH=$ORACLE_HOME/bin:$PATH
export NLS_LANG=AMERICAN_AMERICA.AL32UTF8
```
Save and exit.
- Reload the profile
```console
source ~/.bash_profile
```
At this point you can call sqlplus under oracle user without path specification

## Connecting to Oracle AI Database
Please make sure the listener port (`1521`) is accessible from outside.

To check the firewall status:
```console
sudo firewall-cmd --state
```
if it says:
```console
running
```
open port `1521`:
```console
sudo firewall-cmd --add-port=1521/tcp --permanent
sudo firewall-cmd --reload
```
then verify:
```console
sudo firewall-cmd --list-ports
```
you should see:
```console
1521/tcp
```

At this moment Oracle AI Database Free is installed and accessible to connect.

## Automating Shutdown and Start-Up
Oracle recommends that you configure the system to automatically start Oracle AI Database Free when the system starts, and to automatically shut it down when the system shuts down.

To automate the start up and shutdown of the listener and database, run the following commands as `root`:
```console
$ sudo -s
```
For Oracle Linux 8 and Oracle Linux 9:
```console
# systemctl daemon-reload
# systemctl enable oracle-free-26ai
```

### Checking status
```console
/etc/init.d/oracle-free-26ai status
```
### Starting
```console
systemctl start oracle-free-26ai
```
### Stopping
```console
systemctl stop oracle-free-26ai
```
### Restarting
```console
systemctl restart oracle-free-26ai
```


