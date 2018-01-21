---
layout: post
title: "OpenLDAP 极速搭建：双主同步"
tags: openldap,replication,centos-7
unsplash_id: 3XUijzqLOV8
excerpt: 本篇是快速搭建 OpenLDAP 的第二篇，介绍了 OpenLDAP 的双主同步配置方法。
---

本文是 OpenLDAP 搭建文档的第二篇，介绍了 OpenLDAP 的双主同步配置方法。和上一篇一样，这篇文档也没有原理性内容。

依照本文进行配置之后，读者将搭建两台能够进行双主同步的 OpenLDAP 实例。


### 1. 完成前期的安装工作

在阅读本篇之前，请参照 OpenLDAP 的[第一篇]({% post_url 2017-09-13-openldap-on-centos-7-speedy-tutorial %})，搭建两台 OpenLDAP 实例。这里我们假设两台 OpenLDAP 的主机名如下：

- openldap-master-1.colinlee.fish
- openldap-master-2.colinlee.fish

### 2. 启用 syncprov 模块

syncprov 是 OpenLDAP 用来进行数据同步的模块，这里我们首先将其启用。

在两台服务器上分别创建一个名为 syncprov_mod.ldif 文件。

*syncprov_mod.ldif*
```
dn: cn=module,cn=config
objectClass: olcModuleList
cn: module
olcModulePath: /usr/lib64/openldap
olcModuleLoad: syncprov.la
```

然后在两台服务器上分别执行该文件。

```
[root@openldap-master-1]# ldapadd -Y EXTERNAL -H ldapi:/// -f syncprov_mod.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=module,cn=config"
```

### 3. 启用 OpenLDAP 的双主同步

创建 configrep.ldif 文件。

*configrep.ldif*
```
### Update Server ID with LDAP URL ###

dn: cn=config
changetype: modify
replace: olcServerID
olcServerID: 1 ldap://openldap-master-1.colinlee.fish
olcServerID: 2 ldap://openldap-master-2.colinlee.fish

### Enable replication ###

dn: olcOverlay=syncprov,olcDatabase={2}hdb,cn=config
changetype: add
objectClass: olcOverlayConfig
objectClass: olcSyncProvConfig
olcOverlay: syncprov

### Adding details for replication ###

dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcSyncRepl
olcSyncRepl:
  rid=001
  provider=ldap://openldap-master-1.colinlee.fish
  binddn="cn=admin,dc=colinlee,dc=fish"
  bindmethod=simple
  credentials=MyAdMiNP@ssW0rd
  searchbase="dc=colinlee,dc=fish"
  type=refreshAndPersist
  retry="5 5 300 5"
  timeout=1
olcSyncRepl:
  rid=002
  provider=ldap://openldap-master-2.colinlee.fish
  binddn="cn=admin,dc=colinlee,dc=fish"
  bindmethod=simple
  credentials=MyAdMiNP@ssW0rd
  searchbase="dc=colinlee,dc=fish"
  type=refreshAndPersist
  retry="5 5 300 5"
  timeout=1
-
add: olcMirrorMode
olcMirrorMode: TRUE
```

这里需要注意：binddn 和 credentials 两项分别代表 LDAP 的管理员和密码，请根据环境的实际情况来填写。

然后在两台服务器上分别执行这个 ldap_sync.ldif 文件即可。

```
ldapmodify -Y EXTERNAL -H ldapi:/// -f configrep.ldif
```

至此，`dc=colinlee,dc=fish` 下的内容便可以在两个服务器上同步了。

请注意，`cn=config`下的内容其实也可以同步，我们这里就暂不展开介绍了。

---
参考材料：

[https://linoxide.com/linux-how-to/setup-openldap-multi-master-replication-centos-7/](https://linoxide.com/linux-how-to/setup-openldap-multi-master-replication-centos-7/)
[https://www.itzgeek.com/how-tos/linux/centos-how-tos/configure-openldap-multi-master-replication-linux.html](https://www.itzgeek.com/how-tos/linux/centos-how-tos/configure-openldap-multi-master-replication-linux.html)
