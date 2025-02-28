
```
[root@master yum.repos.d]# vi CentOS-Base.repo
```
```
[root@master yum.repos.d]# yum repolist
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
repo id                                                repo name                                                   status
base/x86_64                                            CentOS-7 - Base                                             10,072
extras/x86_64                                          CentOS-7 - Extras                                              526
repolist: 10,598
[root@master yum.repos.d]# yum search epel
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
=================================================== N/S matched: epel ===================================================
epel-release.noarch : Extra Packages for Enterprise Linux repository configuration

  Name and summary matches only, use "search all" for everything.
```
```
[root@master yum.repos.d]# yum install epel*
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
base                                                                                              | 3.6 kB  00:00:00
extras                                                                                            | 2.9 kB  00:00:00
Package epel-release-7-11.noarch already installed and latest version
Nothing to do
```

```
[root@master yum.repos.d]# pwd
/etc/yum.repos.d
```
```
[root@master yum.repos.d]# rpm -qa |grep -i epel
epel-release-7-11.noarch
```

<br>

> 기존에 있던 패키지 제거

```
[root@master yum.repos.d]# rpm -e $(rpm -qa |grep -i epel)
warning: file /etc/yum.repos.d/epel.repo: remove failed: No such file or directory
warning: file /etc/yum.repos.d/epel-testing.repo: remove failed: No such file or directory
```

```
[root@master yum.repos.d]# rpm -qa |grep -i epel
```

```
[root@master yum.repos.d]# yum install epel*
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
Resolving Dependencies
--> Running transaction check
---> Package epel-release.noarch 0:7-11 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=========================================================================================================================
 Package                          Arch                       Version                    Repository                  Size
=========================================================================================================================
Installing:
 epel-release                     noarch                     7-11                       extras                      15 k

Transaction Summary
=========================================================================================================================
Install  1 Package

Total download size: 15 k
Installed size: 24 k
Is this ok [y/d/N]: y
Downloading packages:
epel-release-7-11.noarch.rpm                                                                      |  15 kB  00:00:00
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
Warning: RPMDB altered outside of yum.
  Installing : epel-release-7-11.noarch                                                                              1/1
  Verifying  : epel-release-7-11.noarch                                                                              1/1

Installed:
  epel-release.noarch 0:7-11

Complete!
```
```
[root@master yum.repos.d]# yum clean all
Loaded plugins: fastestmirror
Cleaning repos: base epel extras
Cleaning up list of fastest mirrors
```
```
[root@master yum.repos.d]# yum repolist
Loaded plugins: fastestmirror
Determining fastest mirrors
epel/x86_64/metalink                                                                              | 5.1 kB  00:00:00
 * epel: d2lzkl7pfhq30w.cloudfront.net
base                                                                                              | 3.6 kB  00:00:00
epel                                                                                              | 4.3 kB  00:00:00
extras                                                                                            | 2.9 kB  00:00:00
(1/6): base/x86_64/group_gz                                                                       | 153 kB  00:00:02
(2/6): extras/x86_64/primary_db                                                                   | 253 kB  00:00:00
(3/6): epel/x86_64/group                                                                          | 399 kB  00:00:02
(4/6): epel/x86_64/updateinfo                                                                     | 1.0 MB  00:00:03
(5/6): base/x86_64/primary_db                                                                     | 6.1 MB  00:00:05
(6/6): epel/x86_64/primary_db                                                                     | 8.7 MB  00:00:06
repo id                                  repo name                                                                 status
base/x86_64                              CentOS-7 - Base                                                           10,072
epel/x86_64                              Extra Packages for Enterprise Linux 7 - x86_64                            13,791
extras/x86_64                            CentOS-7 - Extras                                                            526
repolist: 24,389
```
```
[root@master yum.repos.d]# yum install -y phpldapadmin
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * epel: d2lzkl7pfhq30w.cloudfront.net
Resolving Dependencies
--> Running transaction check
---> Package phpldapadmin.noarch 0:1.2.5-1.el7 will be installed
--> Processing Dependency: php >= 5.0.6 for package: phpldapadmin-1.2.5-1.el7.noarch
--> Processing Dependency: php-ldap for package: phpldapadmin-1.2.5-1.el7.noarch
--> Processing Dependency: webserver for package: phpldapadmin-1.2.5-1.el7.noarch
--> Running transaction check
---> Package lighttpd.x86_64 0:1.4.54-1.el7 will be installed
--> Processing Dependency: psmisc for package: lighttpd-1.4.54-1.el7.x86_64
--> Processing Dependency: libfam.so.0()(64bit) for package: lighttpd-1.4.54-1.el7.x86_64
---> Package php.x86_64 0:5.4.16-48.el7 will be installed
--> Processing Dependency: php-common(x86-64) = 5.4.16-48.el7 for package: php-5.4.16-48.el7.x86_64
--> Processing Dependency: php-cli(x86-64) = 5.4.16-48.el7 for package: php-5.4.16-48.el7.x86_64
--> Processing Dependency: httpd-mmn = 20120211x8664 for package: php-5.4.16-48.el7.x86_64
--> Processing Dependency: httpd for package: php-5.4.16-48.el7.x86_64
---> Package php-ldap.x86_64 0:5.4.16-48.el7 will be installed
--> Running transaction check
---> Package gamin.x86_64 0:0.1.10-16.el7 will be installed
---> Package httpd.x86_64 0:2.4.6-95.el7.centos will be installed
--> Processing Dependency: httpd-tools = 2.4.6-95.el7.centos for package: httpd-2.4.6-95.el7.centos.x86_64
--> Processing Dependency: /etc/mime.types for package: httpd-2.4.6-95.el7.centos.x86_64
--> Processing Dependency: libaprutil-1.so.0()(64bit) for package: httpd-2.4.6-95.el7.centos.x86_64
--> Processing Dependency: libapr-1.so.0()(64bit) for package: httpd-2.4.6-95.el7.centos.x86_64
---> Package php-cli.x86_64 0:5.4.16-48.el7 will be installed
---> Package php-common.x86_64 0:5.4.16-48.el7 will be installed
--> Processing Dependency: libzip.so.2()(64bit) for package: php-common-5.4.16-48.el7.x86_64
---> Package psmisc.x86_64 0:22.20-17.el7 will be installed
--> Running transaction check
---> Package apr.x86_64 0:1.4.8-7.el7 will be installed
---> Package apr-util.x86_64 0:1.5.2-6.el7 will be installed
---> Package httpd-tools.x86_64 0:2.4.6-95.el7.centos will be installed
---> Package libzip.x86_64 0:0.10.1-8.el7 will be installed
---> Package mailcap.noarch 0:2.1.41-2.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=========================================================================================================================
 Package                       Arch                    Version                               Repository             Size
=========================================================================================================================
Installing:
 phpldapadmin                  noarch                  1.2.5-1.el7                           epel                  797 k
Installing for dependencies:
 apr                           x86_64                  1.4.8-7.el7                           base                  104 k
 apr-util                      x86_64                  1.5.2-6.el7                           base                   92 k
 gamin                         x86_64                  0.1.10-16.el7                         base                  128 k
 httpd                         x86_64                  2.4.6-95.el7.centos                   base                  2.7 M
 httpd-tools                   x86_64                  2.4.6-95.el7.centos                   base                   93 k
 libzip                        x86_64                  0.10.1-8.el7                          base                   48 k
 lighttpd                      x86_64                  1.4.54-1.el7                          epel                  438 k
 mailcap                       noarch                  2.1.41-2.el7                          base                   31 k
 php                           x86_64                  5.4.16-48.el7                         base                  1.4 M
 php-cli                       x86_64                  5.4.16-48.el7                         base                  2.7 M
 php-common                    x86_64                  5.4.16-48.el7                         base                  565 k
 php-ldap                      x86_64                  5.4.16-48.el7                         base                   53 k
 psmisc                        x86_64                  22.20-17.el7                          base                  141 k

Transaction Summary
=========================================================================================================================
Install  1 Package (+13 Dependent packages)

Total download size: 9.2 M
Installed size: 32 M
Downloading packages:
(1/14): apr-1.4.8-7.el7.x86_64.rpm                                                                | 104 kB  00:00:01
(2/14): gamin-0.1.10-16.el7.x86_64.rpm                                                            | 128 kB  00:00:00
(3/14): apr-util-1.5.2-6.el7.x86_64.rpm                                                           |  92 kB  00:00:01
(4/14): httpd-tools-2.4.6-95.el7.centos.x86_64.rpm                                                |  93 kB  00:00:00
(5/14): libzip-0.10.1-8.el7.x86_64.rpm                                                            |  48 kB  00:00:00
(6/14): mailcap-2.1.41-2.el7.noarch.rpm                                                           |  31 kB  00:00:00
(7/14): php-5.4.16-48.el7.x86_64.rpm                                                              | 1.4 MB  00:00:00
(8/14): httpd-2.4.6-95.el7.centos.x86_64.rpm                                                      | 2.7 MB  00:00:02
warning: /var/cache/yum/x86_64/7/epel/packages/lighttpd-1.4.54-1.el7.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID 352c64e5: NOKEY
Public key for lighttpd-1.4.54-1.el7.x86_64.rpm is not installed
(9/14): lighttpd-1.4.54-1.el7.x86_64.rpm                                                          | 438 kB  00:00:01
(10/14): php-common-5.4.16-48.el7.x86_64.rpm                                                      | 565 kB  00:00:00
(11/14): php-ldap-5.4.16-48.el7.x86_64.rpm                                                        |  53 kB  00:00:00
(12/14): psmisc-22.20-17.el7.x86_64.rpm                                                           | 141 kB  00:00:00
(13/14): php-cli-5.4.16-48.el7.x86_64.rpm                                                         | 2.7 MB  00:00:01
(14/14): phpldapadmin-1.2.5-1.el7.noarch.rpm                                                      | 797 kB  00:00:02
-------------------------------------------------------------------------------------------------------------------------
Total                                                                                    1.4 MB/s | 9.2 MB  00:00:06
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
Importing GPG key 0x352C64E5:
 Userid     : "Fedora EPEL (7) <epel@fedoraproject.org>"
 Fingerprint: 91e9 7d7c 4a5e 96f1 7f3e 888f 6a2f aea2 352c 64e5
 Package    : epel-release-7-11.noarch (@extras)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : apr-1.4.8-7.el7.x86_64                                                                               1/14
  Installing : apr-util-1.5.2-6.el7.x86_64                                                                          2/14
  Installing : httpd-tools-2.4.6-95.el7.centos.x86_64                                                               3/14
  Installing : psmisc-22.20-17.el7.x86_64                                                                           4/14
  Installing : libzip-0.10.1-8.el7.x86_64                                                                           5/14
  Installing : php-common-5.4.16-48.el7.x86_64                                                                      6/14
  Installing : php-ldap-5.4.16-48.el7.x86_64                                                                        7/14
  Installing : php-cli-5.4.16-48.el7.x86_64                                                                         8/14
  Installing : mailcap-2.1.41-2.el7.noarch                                                                          9/14
  Installing : httpd-2.4.6-95.el7.centos.x86_64                                                                    10/14
  Installing : php-5.4.16-48.el7.x86_64                                                                            11/14
  Installing : gamin-0.1.10-16.el7.x86_64                                                                          12/14
  Installing : lighttpd-1.4.54-1.el7.x86_64                                                                        13/14
  Installing : phpldapadmin-1.2.5-1.el7.noarch                                                                     14/14
  Verifying  : php-ldap-5.4.16-48.el7.x86_64                                                                        1/14
  Verifying  : httpd-tools-2.4.6-95.el7.centos.x86_64                                                               2/14
  Verifying  : lighttpd-1.4.54-1.el7.x86_64                                                                         3/14
  Verifying  : gamin-0.1.10-16.el7.x86_64                                                                           4/14
  Verifying  : mailcap-2.1.41-2.el7.noarch                                                                          5/14
  Verifying  : apr-1.4.8-7.el7.x86_64                                                                               6/14
  Verifying  : php-cli-5.4.16-48.el7.x86_64                                                                         7/14
  Verifying  : apr-util-1.5.2-6.el7.x86_64                                                                          8/14
  Verifying  : phpldapadmin-1.2.5-1.el7.noarch                                                                      9/14
  Verifying  : httpd-2.4.6-95.el7.centos.x86_64                                                                    10/14
  Verifying  : php-common-5.4.16-48.el7.x86_64                                                                     11/14
  Verifying  : php-5.4.16-48.el7.x86_64                                                                            12/14
  Verifying  : libzip-0.10.1-8.el7.x86_64                                                                          13/14
  Verifying  : psmisc-22.20-17.el7.x86_64                                                                          14/14

Installed:
  phpldapadmin.noarch 0:1.2.5-1.el7

Dependency Installed:
  apr.x86_64 0:1.4.8-7.el7               apr-util.x86_64 0:1.5.2-6.el7                gamin.x86_64 0:0.1.10-16.el7
  httpd.x86_64 0:2.4.6-95.el7.centos     httpd-tools.x86_64 0:2.4.6-95.el7.centos     libzip.x86_64 0:0.10.1-8.el7
  lighttpd.x86_64 0:1.4.54-1.el7         mailcap.noarch 0:2.1.41-2.el7                php.x86_64 0:5.4.16-48.el7
  php-cli.x86_64 0:5.4.16-48.el7         php-common.x86_64 0:5.4.16-48.el7            php-ldap.x86_64 0:5.4.16-48.el7
  psmisc.x86_64 0:22.20-17.el7
```
