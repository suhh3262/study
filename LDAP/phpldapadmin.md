```
[root@master ~]# vi /etc/phpldapadmin/config.php
```
> 397 활성화, 398 주석처리  <br>
![Image](https://github.com/user-attachments/assets/56635005-6004-4824-8e2a-52ac1855bde6)

```
[root@master ~]# systemctl start httpd
[root@master ~]# systemctl enable httpd
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
[root@master ~]# vi /etc/httpd/conf.
conf.d/         conf.modules.d/
[root@master ~]# vi /etc/httpd/conf.d/phpldapadmin.conf
[root@master ~]# vi /etc/httpd/conf.d/phpldapadmin.conf
[root@master ~]# vi /etc/httpd/conf.d/phpldapadmin.conf
```

