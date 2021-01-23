# db2upgrade

It is a simple Ansible playbook allowing remote installation of DB2 FixPack in DB2 instance and databases. Two main playbooks are specified: upgrade and clean after the upgrade.<br> 
The playbook was tested with DB2 11.1 (not the latest DB2 11.5).<br>

# Installation

> git clone https://github.com/stanislawbartkowski/db2upgrade.git<br>
> cd db2upgrade<br>
> mkdir conf<br>
> cp conf.template/* conf<br>

Modify *conf/env_db.yaml* and *conf/env_vars.yaml*<br>

Upgrade<br>
> ansible-playbook upgrade.yaml<br>

Clean after upgrade<br>
> ansible-playbook clean.yaml<br>

# Prerequisities

Ansible on the client desktop. Tested with Ansible 2.9.16.<br>
<br>
Ansible inventory contains group *db*, list of DB2 hosts where upgrade is to be performed.<br>

Example:<br>

> vi /etc/ansible/hosts<br>
```
[db]
db2111
```

# Remote DB2 hosts

* DB2 11.1 installation
* Passwordless *root* connection to remote DB2 hosts specified in *db* group
* *db2inst1* instance owner

# Obtain DB2 11.1 FixPack

https://www.ibm.com/support/pages/download-db2-fix-packs-version-db2-linux-unix-and-windows

Download appropriate DB2 11.1 Fixpack and save it on the client machine.<br>

# Ansible variables

*conf/env_vars.yaml*

| Variable | Description | Example value |
| --- | ----- | ----- |
| imagedir | Directory on the local workstation where FixPack image is saved | /home/sbartkowski/work/fp
| imagefile | FixPack image filename | v11.1.4fp4_linuxx64_server_t.tar.gz
| currentdir | Current DB2 installation directory | /opt/ibm/db2/V11.1
| fpdir | DB2 FixPack installation directory | /opt/ibm/db2/V11.1.45
| unpackdir | Directory on the remote host where FP image is uncompressed | /tmp/i

*conf/env_db.yaml*<br<
List of databases in the instance. Example:
```yaml
databases: 
  - SAMPLE
  - BANK
```

# Installing DB2 FixPack

> ansible-playbook upgrade.yaml<br>

The *upgrade.yaml* follows steps described in this article.<br>

https://www.ibm.com/support/pages/db2-fix-pack-installation-steps-universal-fix-pack


* *play/db2stop.yaml* Stop DB2 instance if started
* *play/cleanun.yaml* Clean */tmp/i* directory on remote host
* *play/copyfp.yaml* Move FixPack image to remote */tmp* directory and unpack to *unpackdir*
* *play/upgradedb.yaml*
  * Install FixPack in *fpdir* directory
  * Run *db2iupdt db2inst1* command and upgrade the instance
* *play/updatedb.yaml* 
  * Start updated DB2 instance
  * Run *db2updv111* for every databases for *databases* list
  
# Clean after upgrade

> ansible-playbook clean.yaml<br>

* Clean *unpackdir* directory in remote DB2 machine
* Remove FixPack image from */tmp* directory
* Run *currentdir*/install/db2_deinstall command and remove previous DB2 installation

# Possible improvements

* Health-check after upgrade and before cleansing
* Automatically discovery of databases, get rid of *databases* variable
* Check prerequisites before upgrade
* Adjust to DB2 11.5 version
