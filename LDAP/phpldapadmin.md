```
[root@master ~]# vi /etc/phpldapadmin/config.php
```
> 397 활성화, 398 주석처리  <br>
![Image](https://github.com/user-attachments/assets/56635005-6004-4824-8e2a-52ac1855bde6)

<br>

```
[root@master ~]# vi /etc/httpd/conf.d/phpldapadmin.conf
#
#  Web-based tool for managing LDAP servers
#

Alias /phpldapadmin /usr/share/phpldapadmin/htdocs
Alias /ldapadmin /usr/share/phpldapadmin/htdocs

<Directory /usr/share/phpldapadmin/htdocs>
  <IfModule mod_authz_core.c>
    # Apache 2.4
    Require all granted
  </IfModule>
  <IfModule !mod_authz_core.c>
    # Apache 2.2
    Order Deny,Allow
    Deny from all
    Allow from 127.0.0.1
    Allow from ::1
  </IfModule>
</Directory>
```

<br>

```
[root@master ~]# systemctl start httpd
[root@master ~]# systemctl enable httpd
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.

[root@master ~]# vi /etc/httpd/conf.
conf.d/         conf.modules.d/
```

<br>

> 접속 방법

웹브라우저에 `http://192.168.56.11/phpldapadmin/` 검색

cn=ldapadm,dc=test,dc=local / 1111   로그인

![Image](https://github.com/user-attachments/assets/f94067a9-5a64-4dac-8b23-5656aaeace65)
