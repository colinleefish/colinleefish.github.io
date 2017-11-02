---
layout: post
title: "OpenLDAP 在 CentOS 7 上的极速搭建步骤"
tags: openldap,centos-7
unsplash_id: XNIjmb6Ax04
excerpt: 这是一篇在 CentOS 7 上快速搭建 OpenLDAP 的文档，主要目的是留存一份能够迅速搭建 OpenLDAP 环境的材料，以备不时之需。
---

这是一篇在 CentOS 7 上快速搭建 OpenLDAP 的文档，主要目的是留存一份能够迅速搭建 OpenLDAP 环境的材料，以备不时之需，通篇没有原理性内容。

依照本文进行配置之后，读者将搭建一台单机、具有用户和用户组结构、无备份、无证书加密的 OpenLDAP 实例。备份和证书加密的内容将放在其他文章中。


### 1. 为服务器选定一个 FQDN，并且配置为该服务器的主机名

这里，我们将服务器命名为 ldap.colinlee.fish，并将该 FQDN 连同主机名配置在 /etc/hosts 中。除非你有什么特殊的原因导致不能配置 FQDN，否则强烈建议配置：该 FQDN 在为 OpenLDAP 设置 CA 证书时有非常大的作用。

```
[root@ldap ~]# hostnamectl set-hostname ldap.colinlee.fish
```

*/etc/hosts*
```
···
127.0.1.1    ldap.colinlee.fish ldap
127.0.0.1    localhost
···
```


### 2. 安装 OpenLDAP 服务端，设置数据库配置文件，启动 OpenLDAP 服务

```
[root@ldap ~]# yum -y install openldap-servers openldap-clients
[root@ldap ~]# cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
[root@ldap ~]# chown ldap.ldap /var/lib/ldap/DB_CONFIG
[root@ldap ~]# systemctl start slapd
[root@ldap ~]# systemctl enable slapd
```

### 3. 设置 OpenLDAP 的管理员用户 root 的密码

首先，使用 slappasswd 命令，为 OpenLDAP 的超级管理员用户（root）生成密码。

```
[root@ldap ~]# slappasswd
New password:
Re-enter new password:
{SSHA}xxxxxxxxxxxxxxxxxxxxxxxx
```

将生成的密码添加至 OpenLDAP 的 ldif 文件中。LDIF 是修改 OpenLDAP 内容的标准文本格式。

*chrootpw.ldif*
```
dn: olcDatabase={0}config,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}xxxxxxxxxxxxxxxxxxxxxxxx
```

接下来，执行编辑好的 chrootpw.ldif 文件。

```
[root@ldap ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f chrootpw.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "olcDatabase={0}config,cn=config"
```

### 4. 添加几个基础的 Schema

此处添加的 Schema 主要用于记录人员信息。如果搭建该 OpenLDAP 的目的为用户认证，一般都会导入以下 Schema；如果使用 OpenLDAP 有别的用处，请确认用处后再考虑导入哪些 Schema。

```
[root@ldap ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif

SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=cosine,cn=schema,cn=config"

[root@ldap ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif

SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=nis,cn=schema,cn=config"

[root@ldap ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif

SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=inetorgperson,cn=schema,cn=config"
```

### 5. 在 LDAP 数据库中设置根域和数据库超级管理员

这里的“根域”可以和这台服务器 FQDN 中的根域不同，此处我们设置为 dc=colinlee,dc=fish。

数据库管理员和上面设置过的 OpenLDAP 超级管理员并非同一管理员，而且这里设置的管理员目前尚未创建。此处的设置同样需要一个用 slappasswd 命令生成的密码，为了方便管理，我们使用刚刚生成的密码，不再重新生成。

创建一个新的 ldif 文件，设置以上内容。

*domain-dbadmin.ldif*
```
dn: olcDatabase={1}monitor,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to *
  by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth" read
  by dn.base="cn=admin,dc=colinlee,dc=fish" read
  by * none

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=colinlee,dc=fish

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=admin,dc=colinlee,dc=fish

dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}xxxxxxxxxxxxxxxxxxxxxxxx

dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcAccess
olcAccess: {0}to attrs=userPassword,shadowLastChange
  by dn="cn=admin,dc=colinlee,dc=fish" write
  by anonymous auth
  by self write
  by * none
olcAccess: {1}to dn.base=""
  by * read
olcAccess: {2}to *
  by dn="cn=admin,dc=colinlee,dc=fish" write
  by * read
```

然后执行该 ldif 文件。

```
[root@ldap ~]# ldapmodify -Y EXTERNAL -H ldapi:/// -f domain-dbadmin.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "olcDatabase={1}monitor,cn=config"

modifying entry "olcDatabase={2}hdb,cn=config"

modifying entry "olcDatabase={2}hdb,cn=config"

modifying entry "olcDatabase={2}hdb,cn=config"
```

### 6. 创建用户节点、组节点和数据库超级管理员

同样创建一个新的 ldif 文件，并填入节点信息。

*basedomain.ldif*
```
dn: dc=colinlee,dc=fish
objectClass: top
objectClass: dcObject
objectclass: organization
o: Example Inc.
dc: colinlee

dn: ou=user,dc=colinlee,dc=fish
objectClass: organizationalUnit
ou: user

dn: ou=group,dc=colinlee,dc=fish
objectClass: organizationalUnit
ou: group

dn: cn=admin,dc=colinlee,dc=fish
objectClass: organizationalRole
cn: admin
description: Directory Administrator
```

执行该文件使内容生效。这里的执行方式和前面几次有所不同，此处使用了数据库超级管理员的身份，而且需要输入之前设置的密码。

```
[root@ldap ~]# ldapadd -x -D cn=admin,dc=colinlee,dc=fish -W -f basedomain.ldif
Enter LDAP Password:
adding new entry "dc=colinlee,dc=fish"

adding new entry "ou=user,dc=colinlee,dc=fish"

adding new entry "ou=group,dc=colinlee,dc=fish"

adding new entry "cn=admin,dc=colinlee,dc=fish"
```

### 7. 配置防火墙

如果使用了防火墙，需要开启针对 LDAP 的策略。

```
[root@ldap ~]# firewall-cmd --add-service=ldap --permanent
success
[root@ldap ~]# firewall-cmd --reload
success
```

如果不使用防火墙，直接关闭即可。

```
[root@ldap ~]# systemctl stop firewalld
[root@ldap ~]# systemctl disable firewalld
```

---
参考材料：

[https://www.server-world.info/en/note?os=CentOS_7&p=openldap](https://www.server-world.info/en/note?os=CentOS_7&p=openldap)
[http://www.itzgeek.com/how-tos/linux/centos-how-tos/step-step-openldap-server-configuration-centos-7-rhel-7.html](http://www.itzgeek.com/how-tos/linux/centos-how-tos/step-step-openldap-server-configuration-centos-7-rhel-7.html)
