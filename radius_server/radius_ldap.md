# How to install and setting ldap server

## Environment

- OS: CentOS 7.8.2003
- Required packages:
  - openldap-servers-2.4.44-21.el7_6.x86_64
  - openldap-2.4.44-21.el7_6.x86_64
  - openldap-clients-2.4.44-21.el7_6.x86_64

## Install ldap server

`# yum install openldap{,-clients,-servers}`

## Ldap server settings

### Configure settings

```bash
# vi /etc/openldap/slapd.d/cn=config.ldif
dn: cn=config
objectClass: olcGlobal
cn: config
olcArgsFile: /var/run/openldap/slapd.args
olcPidFile: /var/run/openldap/slapd.pid
olcTLSCACertificatePath: /etc/openldap/certs
olcTLSCertificateFile: "OpenLDAP Server"
olcTLSCertificateKeyFile: /etc/openldap/certs/password
olcAllows: bind_v2
```

```bash
# vi /etc/openldap/slapd.d/cn=config/olcDatabase={-1}frontend.ldif
dn: olcDatabase={-1}frontend
objectClass: olcDatabaseConfig
objectClass: olcFrontendConfig
olcDatabase: {-1}frontend
olcSizeLimit:  unlimited
olcTimeLimit: unlimited
```

```bash
# vi /etc/openldap/slapd.d/cn=config/olcDatabase={0}config.ldif
dn: olcDatabase={0}config
objectClass: olcDatabaseConfig
olcDatabase: {0}config
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth" manage by * none
```

```bash
# vi /etc/openldap/slapd.d/cn=config/olcDatabase={1}monitor.ldif
dn: olcDatabase={1}monitor
objectClass: olcDatabaseConfig
olcDatabase: {1}monitor
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth" read by dn.base="cn=Manager,dc=my-domain,dc=com" read by * none
```

```bash
# vi /etc/openldap/slapd.d/cn=config/olcDatabase={2}hdb.ldif=\{2\}hdb.ldif
dn: olcDatabase={2}hdb
objectClass: olcDatabaseConfig
objectClass: olcHdbConfig
olcDatabase: {2}hdb
olcDbDirectory: /var/lib/ldap
olcSuffix: dc=alessiareya,dc=com
olcRootDN: cn=Manager,dc=alessiareya,dc=com
olcAccess: {0}to attrs=entry by self write by dn="cn=Admin,dc=alessiareya,dc=com" write
           by * read
olcAccess: {1}to attrs=userPassword,sambaLMPassword,sambaNTPassword by self write
           by dn="cn=Admin,dc=alessiareya,dc=com" write 
           by annonymous auth
					 by * none
olcAccess: {2}to * by self write
           by dn="cn=Admin,dc=alessiareya,dc=com" write
					 by * read 
olcDbIndex: objectClass,uidNumber,gidNumber,loginShell eq,pres
olcDbIndex: ou,cn,mail,surname,givenname,uid,memberUid,nisMapName,nisMapEntry eq,pres,sub
olcRootPW: PASSWORD
```

```bash
# cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
# chown ldap. /var/lib/ldap/DB_CONFIG
set_cachesize 0 268435456 1
set_lg_regionmax 262144
set_lg_bsize 2097152
```

### Log settings
```bash
# vi /etc/rsyslog.conf
local4.*	/var/log/slapd.log
# systemctl reload rsyslog.service
```

```bash
# vi /etc/logrotate.d/syslog
/var/log/slapd.log #adding
```

### DIT Backup
```bash
ldap-backup.sh
#/bin/bash
ldapsearch -x -LLL -H ldap://localhost/ -b "dc=alessiareya,dc=com" -D "cn=Manager,dc=alessiareya,dc=com" -w password > BACKUP_FILE.ldif
```

### Configure service unit file

If you use the ldap server in this server you have to run slapd.service after slapd.service.
Add setting below.

```
# cp -p /usr/lib/systemd/system/slapd.service /etc/systemd/system
# vi /etc/systemd/system/slapd.service
[Service]
LimitNOFILE=65536
LimitNPROC=65536

# systemd-delata
+++ /etc/systemd/system/slapd.service   2020-06-18 16:57:02.801158311 +0900
@@ -14,6 +14,8 @@
 EnvironmentFile=/etc/sysconfig/slapd
 ExecStartPre=/usr/libexec/openldap/check-config.sh
 ExecStart=/usr/sbin/slapd -u ldap -h ${SLAPD_URLS} $SLAPD_OPTIONS
+LimitNOFILE=65536
+LimitNPROC=65536

# systemctl daemon-reload
# systemctl restart slapd.service
```

### Service start
```
# systemctl start slapd.service
# systemctl status slapd.service
# systemctl enable slapd.service
```

### Set ldap server administrator password
The administrator can mange ldap server.

This file is used in web authentication.
```bash
# slappasswd
{SSHA}dTTx3bRD72SYPYkMgEvDv8IDq1LMHljZ

# vi /root/rootdn.ldif
 dn: olcDatabase={0}config,cn=config
 changetype: modify
 add: olcRootDN
 olcRootDN: cn=admin,cn=config
 -
 add: olcRootPW
 olcRootPW: {SSHA}dTTx3bRD72SYPYkMgEvDv8IDq1LMHljZ
 
 # ldapmodify -Y EXTERNAL -H ldapi:/// -f /root/rootdn.ldif
```

### Set directory manager password
The directory manager can manage all entry and directory structure.

```bash
# slappasswd
{SSHA}dTTx3bRD72SYPYkMgEvDv8IDq1LMHljZ

# vi /root/hdbconfig.ldif
dn: olcDatabase={1}monitor,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth" read by dn.base="cn=admin,dc=test,dc=org" read by * none

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=test,dc=org
-
replace: olcRootDN
olcRootDN: cn=admin,dc=test,dc=org
-
add: olcRootPW
olcRootPW: {SSHA}dTTx3bRD72SYPYkMgEvDv8IDq1LMHljZ
-
add: olcAccess
olcAccess: {0}to dn.sub="ou=tool,dc=test,dc=org" attrs=userPassword,shadowLastChange by self write by anonymous auth by dn.sub="ou=user,dc=test,dc=org" write by dn.sub="ou=tool,dc=test,dc=org" write by users none
olcAccess: {1}to attrs=userPassword,shadowLastChange by self write by anonymous auth by dn.sub="ou=user,dc=test,dc=org" write by users none
olcAccess: {2}to by dn.sub="ou=user,dc=test,dc=org" write by read

# ldapmodify -x -W -D cn=admin,cn=config -f /root/hdbconfig.ldif
# ldapsearch -x -LLL -W -D cn=admin,cn=config -b cn=config olcDatabase={1}monitor
# ldapsearch -x -LLL -W -D cn=admin,cn=config -b cn=config olcDatabase={2}hdb
```

### Extend schemas
- core.schema # you don't have to import
- cosine.schema
- nis.schema
- samba.schema
- freeradius.schema

```bash
# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif
# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif
# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif
# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/samba.ldif
# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/freeradius.ldif

# ldapadd -x -W -D cn=admin,cn=config -f /etc/openldap/schema/cosine.ldif
# ldapadd -x -W -D cn=admin,cn=config -f /etc/openldap/schema/nis.ldif
# ldapadd -x -W -D cn=admin,cn=config -f /etc/openldap/schema/inetorperson.ldif
# ldapadd -x -W -D cn=admin,cn=config -f /etc/openldap/schema/freeradius.ldif
```

### Chek schema
`# ldapsearch -x -LLL -W -D cn=admin,cn=cofig -b cn=schema,cn=config dn`

**NOTE**
As result available schema is loaded "/etc/openldap/slapd.d/cn=config/cn=schema"

## Debug

## Techical terms
- OLC -Online Configuration
