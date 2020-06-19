# How to install and setting radius server

## Authentication process

1. User like a PC requests network connection to network switch.
2. Then radius client like a network switch requests authentication to radius server.
3. Radius server responds authentication result to radius client.
4. Radius client returns result to user.
5. User is authenticated and available to connect network.
   In short, PC authorized using mac address can access to network.

Here in document explain mac address authentication and local client authentication.

**caution**  
'Local user' is not os users. Radius server has it in their files.
Instead of files you can also refer other server like a LDAP to require authentication information.

## Environment

- OS: CentOS 8.2.2004
- Required packages:

  - freeradius-3.0.17-7.module_el8.2.0+321+f9fd5d26.x86_64
  - freeradius-utils-3.0.17-7.module_el8.2.0+321+f9fd5d26.x86_64
  - freeradius-ldap-3.0.17-7.module_el8.2.0+321+f9fd5d26.x86_64
  - freeradius-doc-3.0.17-7.module_el8.2.0+321+f9fd5d26.x86_64

- Dependency packages:
  - perl-Math-Complex-1.59-416.el8.noarch
  - perl-Math-BigInt-1.9998.11-7.el8.noarch
  - perl-DBI-1.641-3.module_el8.1.0+199+8f0a6bbd.x86_64

## Install radius server

`# dnf install freeradius{,-utils,-doc,-ldap}`

## Radius server settings

- Enable modules to connect ldap

```
# cd /etc/raddb/mods-enabled
# ln -s ../mods-available/ldap
# cd /etc/raddb/sites-enabled
# ln -n ../sites-available/control-socket
```

- /etc/raddb/radiusd.conf

```
auth = yes #log output for authentication
auth_badpass = yes #log output for failed authentication
auth_goodpass = yes #log output for succeed authentication
```

- /etc/raddb/mods-available/ldap

```
ldap {
        server = 'localhost'
        identity = 'cn=Manager,dc=alessiareya,dc=com'
        password = password
        base_dn = 'dc=alessiareya,dc=com'
        sasl {
        }
        update {
                control:Password-With-Header    += 'userPassword'
                control:NT-Password             += 'sambaNtPassword'
                control:LM-Password             += 'sambaLmPassword'
                reply:Tunnel-Type               += 'radiusTunnelType'
                reply:Tunnel-Medium-Type        += 'radiusTunnelMediumType'
                control:                        += 'radiusControlAttribute'
                request:                        += 'radiusRequestAttribute'
                reply:                          += 'radiusReplyAttribute'
        }

}
```

- /etc/raddb/sites-available/default

```
authorize {
        filter_username
        preprocess
        chap
        suffix
        eap {
                ok = return
        }
        files
        -ldap
        expiration
        logintime
}

authenticate {
        Auth-Type CHAP {
                chap
        }
        eap
}

accounting {
	detail
	unix
	exec
	attr_filter.accounting_response
}
```

- /etc/raddb/dictionary

If you need to write file below.

- /etc/raddb/clients.conf

This file is used in client authentication.
Here in client is like a L2SW.
Radius server can specify ip address and password to connection source.
Clients connect using a specified password, authorized clients can connect.

```
client SW01 {
ipv4addr = 192.168.1.1
secret = password
shotname = 1F floor switch.
}

client SW02 {
ipv4addr = 192.168.1.2
secret = password
shotname = 2F floor switch.
}
```

- /etc/raddb/users

This file is used by mac address authentication.

```
MAC_ADDRESS Cleartext-Password := "MAC_ADDRESS"
10ddb1a630bf Cleartext-Password := "10ddb1a630bf"
```

### Syntax check

`# radiusd -C`

### Check operation

**radius-server** is not clients ip address.

```
# radtest --help
radtest [OPTIONS] user passwd radius-server[:port] nas-port-number secret [ppphint][nasname]

# radtest -t chap -x 10ddb1a630bf 10ddb1a630bf 192.168.11.253 1812 password
Sent Access-Request Id 209 from 0.0.0.0:51772 to 192.168.11.253:1812 length 82
	User-Name = "10ddb1a630bf"
	User-Password = "10ddb1a630bf"
	NAS-IP-Address = 127.0.0.1
	NAS-Port = 1812
	Message-Authenticator = 0x00
	Cleartext-Password = "10ddb1a630bf"
Received Access-Accept Id 209 from 192.168.11.253:1812 to 192.168.11.253:51772 length 20
```

### Configure service unit file

If you use the ldap server with radius server at same server you have to run radius.service after slapd.service.
Add setting below.

```
# cp -p /usr/lib/systemd/system/radiusd.service /etc/systemd/system
# vi /etc/systemd/system/radiusd.service
[Unit]
After=slapd.service

# systemd-delta
[REDIRECTED] /etc/systemd/system/dbus-org.freedesktop.timedate1.service → /usr/lib/systemd/system/dbus-org.freedesktop.timeda>
[REDIRECTED] /etc/systemd/system/default.target → /usr/lib/systemd/system/default.target
[OVERRIDDEN] /etc/systemd/system/radiusd.service → /usr/lib/systemd/system/radiusd.service

--- /usr/lib/systemd/system/radiusd.service     2020-05-07 11:32:19.000000000 +0900
+++ /etc/systemd/system/radiusd.service 2020-06-19 14:09:18.375997664 +0900
@@ -1,6 +1,6 @@
 [Unit]
 Description=FreeRADIUS high performance RADIUS server.
-After=syslog.target network-online.target ipa.service dirsrv.target krb5kdc.service
+After=syslog.target network-online.target ipa.service dirsrv.target krb5kdc.service slapd.service

# systemctl daemon-reload
```

### Service start

```
# systemctl start radiusd.service
# systemctl status radiusd.service
```

## Debug

When you check syntax error use below to identify error.
`# radiusd -C -lstdout -xxx`

## How to configure the catalyst to connect radius server

[catalyst.md](./catalyst.md)

## Technical terms

- NAS Network access server.

## See also
[Free RADIUS](https://freeradius.org/) - officail page
