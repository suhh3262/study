<h2 class="red-title">💡 OpenLDAP 구성 매뉴얼</h2>

|        |       | 
|--------|-------|
| **작성자** | **강수현 (suhyeon.kang@ecorehpc.com)**|
| 작성일 | 2025-07-08 |
| OS     | CentOS 8.3.2011 |
| domain | iREMB-R.HPC |
| 서버 | iREMB-R-M.iREMB-R.HPC |

---

## 선행작업
- hostname 수정
  ```
  hostnamectl set-hostname iREMB-R-M.iREMB-R.HPC
  ```


- `/etc/hosts` 수정

### 본 테스트는 sssd를 사용하지 않고, nslcd 데몬만 사용하였다.

---

> #  1. 1번 서버 OpenLDAP 설치 및 구성 (`iREMB-R-M`)

> ### 1-1. LDAP 패키지 설치 및 실행
- 기존에 깔려있을 경우, RPM update 진행
```sh
yum install –y openldap openldap-clients openldap-servers
또는
rpm –Uvh openldap-2.4.46-18.el8.ppc64le.rpm
rpm –Uvh openldap-servers-2.4.46-18.el8.ppc64le.rpm
rpm –Uvh openldap-clients-2.4.46-18.el8.ppc64le.rpm

systemctl start slapd
systemctl enable slapd
```

<br>
 
> ### 1-2. LDAP 관리 패스워드 설정
- LDAP 관리자가 사용할 패스워드 설정 (설정 변경용 관리자 계정)
```sh
[root@iREMB-R-M ~]# slappasswd
New password:
Re-enter new password:
{SSHA}w3Zdxlp8BTQOSzHCjtWX+jg3PvV+JiXH
```

<BR>

> ### 1-3. 관리자 계정과 암호를 설정하는 ldif 파일 작성
- `ldaprootpw.ldif`
```sh
dn: olcDatabase={0}config,cn=config
changetype: modify
add: olcRootDN
olcRootDN: cn=admin,cn=config

dn: olcDatabase={0}config,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}w3Zdxlp8BTQOSzHCjtWX+jg3PvV+JiXH
```

```
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/slapd.d/ldaprootpw.ldif

```

<br>

> ### 1-4. 기본 스키마 삽입
```sh
[root@iREMB-R-M slapd.d]# cd /etc/openldap/schema/
[root@iREMB-R-M schema]# ll
total 312
-r--r--r--. 1 root root  2036 Jun  2  2021 collective.ldif
-r--r--r--. 1 root root  6190 Jun  2  2021 collective.schema
-r--r--r--. 1 root root  1845 Jun  2  2021 corba.ldif
(생략)

[root@iREMB-R-M schema]# ldapadd -Y EXTERNAL -H ldapi:/// -f cosine.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=cosine,cn=schema,cn=config"

[root@iREMB-R-M schema]# ldapadd -Y EXTERNAL -H ldapi:/// -f nis.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=nis,cn=schema,cn=config"

[root@iREMB-R-M schema]# ldapadd -Y EXTERNAL -H ldapi:/// -f inetorgperson.ldif
(생략)
```

<br>

> ### 1-5. 도메인 정보 변경
- `domain.ldif`

- OpenLDAP DB(사용자 데이터) 내의 관리자 패스워드 설정
```
[root@iREMB-R-M slapd.d]# slappasswd
New password:
Re-enter new password:
{SSHA}qZOJoOYRvzpK5zfZnP4H8GJlHgL28muf
```

<BR>

- 도메인 설정
```sh
[root@iREMB-R-M schema]# cd /etc/openldap/slapd.d/
[root@iREMB-R-M slapd.d]# vi domain.ldif

dn: olcDatabase={1}monitor,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0, cn=peercred, cn=external, cn=auth" read by dn.base="cn=Manager,dc=iREMB-R,dc=HPC" read by * none

dn: olcDatabase={2}mdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=iREMB-R,dc=HPC
-
replace: olcRootDN
olcRootDN: cn=Manager,dc=iREMB-R,dc=HPC
-
replace: olcRootPW
olcRootPW: {SSHA}qZOJoOYRvzpK5zfZnP4H8GJlHgL28muf
-
add: olcAccess
olcAccess:
 {0}to attrs=userPassword,shadowLastChange by dn="cn=Manager,dc=iREMB-R,dc=HPC" write by anonymous auth by self write by * none
olcAccess: {1}to dn.base="" by * read
olcAccess: {2}to * by dn="cn=Manager,dc=iREMB-R,dc=HPC" write by * read
```

- 적용
```sh
 ldapmodify -Y EXTERNAL -H ldapi:/// -f domain.ldif
```

<br>

> ### 1-6. 기본 도메인 설정
- `basedomain.ldif`
- BaseDN, 사용자 계정, 사용자 그룹 섹션에 사용할 그룹을 생성한다.

```
[root@iREMB-R-M slapd.d]# slappasswd
New password:
Re-enter new password:
{SSHA}WDR12EG4NyI0Gtp1tabYKnVgTukBMsRX
```

```sh
[root@iREMB-R-M slapd.d]# vi basedomain.ldif
dn: dc=iREMB-R,dc=HPC
objectClass: top
objectClass: dcObject
objectClass: organization
o: irembr
dc: irembr

dn: cn=Manager,dc=iREMB-R,dc=HPC
objectClass: organizationalRole
objectClass: simpleSecurityObject
cn: Manager
description: Directory Manager
userPassword: {SSHA}WDR12EG4NyI0Gtp1tabYKnVgTukBMsRX

dn: ou=Users,dc=iREMB-R,dc=HPC
objectClass: organizationalUnit
ou: Users

dn: ou=Groups,dc=iREMB-R,dc=HPC
objectClass: organizationalUnit
ou: Groups
```

```
[root@iREMB-R-M slapd.d]#  ldapadd -x -D "cn=Manager,dc=iREMB-R,dc=HPC" -W -f basedomain.ldif
Enter LDAP Password:
adding new entry "dc=iREMB-R,dc=HPC"

adding new entry "cn=Manager,dc=iREMB-R,dc=HPC"

adding new entry "ou=Users,dc=iREMB-R,dc=HPC"

adding new entry "ou=Groups,dc=iREMB-R,dc=HPC"
```

<br>
<br>

> #  2. 2번 서버 OpenLDAP 설치 및 구성

> ### 2-1. LDAP 패키지 설치 및 실행
- 기존에 깔려있을 경우, RPM update 진행
```
yum install –y openldap openldap-clients openldap-servers
또는
rpm –Uvh openldap-2.4.46-18.el8.ppc64le.rpm
rpm –Uvh openldap-servers-2.4.46-18.el8.ppc64le.rpm
rpm –Uvh openldap-clients-2.4.46-18.el8.ppc64le.rpm

systemctl start slapd
systemctl enable slapd
```

<br>
 
> ### 2-2. LDAP 관리 패스워드 설정
- LDAP 관리자가 사용할 패스워드 설정 (설정 변경용 관리자 계정)
```
[root@iREMB-R-S ~]# slappasswd
New password:
Re-enter new password:
{SSHA}w3Zdxlp8BTQOSzHCjtWX+jg3PvV+JiXH
```

<BR>

> ### 2-3. 관리자 계정과 암호를 설정하는 ldif 파일 작성
- `ldaprootpw.ldif`

```
[root@iREMB-R-S ~]# vi /etc/openldap/slapd.d/ldaprootpw.ldif
dn: olcDatabase={0}config,cn=config
changetype: modify
add: olcRootDN
olcRootDN: cn=admin,cn=config

dn: olcDatabase={0}config,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}w3Zdxlp8BTQOSzHCjtWX+jg3PvV+JiXH
```

```
ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/slapd.d/ldaprootpw.ldif

```

<br>

> ### 2-4. 기본 스키마 삽입
```sh
[root@iREMB-R-S slapd.d]# cd /etc/openldap/schema/
[root@iREMB-R-S schema]# ll
total 312
-r--r--r--. 1 root root  2036 Jun  2  2021 collective.ldif
-r--r--r--. 1 root root  6190 Jun  2  2021 collective.schema
-r--r--r--. 1 root root  1845 Jun  2  2021 corba.ldif
(생략)

[root@iREMB-R-S schema]# ldapadd -Y EXTERNAL -H ldapi:/// -f cosine.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=cosine,cn=schema,cn=config"

[root@iREMB-R-S schema]# ldapadd -Y EXTERNAL -H ldapi:/// -f nis.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=nis,cn=schema,cn=config"

[root@iREMB-R-S schema]# ldapadd -Y EXTERNAL -H ldapi:/// -f inetorgperson.ldif
(생략)
```

<br>

> ### 2-5. 도메인 정보 변경
- `domain.ldif`

- OpenLDAP DB(사용자 데이터) 내의 관리자 패스워드 설정
```
[root@iREMB-R-S slapd.d]# slappasswd
New password:
Re-enter new password:
{SSHA}qZOJoOYRvzpK5zfZnP4H8GJlHgL28muf
```

<BR>

- 도메인 설정
```
[root@iREMB-R-S schema]# cd /etc/openldap/slapd.d/
[root@iREMB-R-S slapd.d]# vi domain.ldif

dn: olcDatabase={1}monitor,cn=config
changetype: modify
replace: olcAccess
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0, cn=peercred, cn=external, cn=auth" read by dn.base="cn=Manager,dc=iREMB-R,dc=HPC" read by * none

dn: olcDatabase={2}mdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=iREMB-R,dc=HPC
-
replace: olcRootDN
olcRootDN: cn=Manager,dc=iREMB-R,dc=HPC
-
replace: olcRootPW
olcRootPW: {SSHA}qZOJoOYRvzpK5zfZnP4H8GJlHgL28muf
-
add: olcAccess
olcAccess: {0}to attrs=userPassword,shadowLastChange by dn="cn=Manager,dc=iREMB-R,dc=HPC" write by anonymous auth by self write by * none
olcAccess: {1}to dn.base="" by * read
olcAccess: {2}to * by dn="cn=Manager,dc=iREMB-R,dc=HPC" write by * read
```

- 적용
```
 ldapmodify -Y EXTERNAL -H ldapi:/// -f domain.ldif
```

<br>

> ### 2-6. 기본 도메인 설정
- `basedomain.ldif`
- BaseDN, 사용자 계정, 사용자 그룹 섹션에 사용할 그룹을 생성한다.

```
[root@iREMB-R-S slapd.d]# slappasswd
New password:
Re-enter new password:
{SSHA}WDR12EG4NyI0Gtp1tabYKnVgTukBMsRX
```

```
[root@iREMB-R-S slapd.d]# vi basedomain.ldif
dn: dc=iREMB-R,dc=HPC
objectClass: top
objectClass: dcObject
objectClass: organization
o: irembr
dc: irembr 

dn: cn=Manager,dc=iREMB-R,dc=HPC
objectClass: organizationalRole
objectClass: simpleSecurityObject
cn: Manager
description: Directory Manager
userPassword: {SSHA}WDR12EG4NyI0Gtp1tabYKnVgTukBMsRX

dn: ou=Users,dc=iREMB-R,dc=HPC
objectClass: organizationalUnit
ou: Users

dn: ou=Groups,dc=iREMB-R,dc=HPC
objectClass: organizationalUnit
ou: Groups
```

```
[root@iREMB-R-S slapd.d]#  ldapadd -x -D "cn=Manager,dc=iREMB-R,dc=HPC" -W -f basedomain.ldif
Enter LDAP Password:
adding new entry "dc=iREMB-R,dc=HPC"

adding new entry "cn=Manager,dc=iREMB-R,dc=HPC"
adding new entry "ou=Users,dc=iREMB-R,dc=HPC"
adding new entry "ou=Groups,dc=iREMB-R,dc=HPC"
```

<BR><bR>

> #  3. OpenLDAP SSL 인증서 생성

> ### 3-1. 자체 서명 CA 생성 (서버 1에서만 진행)

### CA 비공개 키 생성 (암호X 버전)
- RSA 비밀키 생성
```sh
openssl genrsa -out /etc/pki/tls/private/ca.key 4096
```

<Br>

### CA 인증서 생성
```sh
openssl req -x509 -new -key /etc/pki/tls/private/ca.key -sha256 -days 2650 -out /etc/pki/tls/certs/ca.crt

KR
Daegu
Daegu
DGIST
hpc center
irembr-CA
```
- `-x509` : CSR이 아닌 **자체 서명된 x.509 인증서** 생성
- `-days 3650` : 유효기간 10년으로 설정

<br>

```
[root@iREMB-R-M ~]# cd /etc/pki/tls/certs/
[root@iREMB-R-M certs]# cp ca.crt /etc/openldap/certs
[root@iREMB-R-M certs]# cd /etc/pki/tls/private/
[root@iREMB-R-M private]# cp * /etc/openldap/certs
```

<Br>

> ### 3-2. 서버별 키와 CSR 생성 (서버 각각 진행)
```sh
# 서버 1
openssl req -new -nodes -out iREMB-R-M.csr -keyout iREMB-R-M.key

# 서버 2
openssl req -new -nodes -out iREMB-R-S.csr -keyout iREMB-R-S.key
```

<BR>

> ### 3-3. CA로 서버별 CSR 생성

### 서버 1
```
scp /etc/openldap/certs/* iREMB-R-S:/etc/openldap/certs
```

```sh
openssl x509 -req -in iREMB-R-M.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out iREMB-R-M.crt -days 3650 -sha256
scp ca.srl iREMB-R-S:/etc/openldap/certs/
```

<br>

### 서버 2
```
openssl x509 -req -in iREMB-R-S.csr -CA ca.crt -CAkey ca.key -out iREMB-R-S.crt -days 3650 –sha256
```
- 서버 2에서는 `-CAcreateserial` 과정을 생략한다.

<br>

> ### 3-4. /etc/openldap/certs 확인
- 인증 관련 파일들을 모두 `/etc/openldap/certs` 디렉터리 내에 넣어준다. (관리의 용이성)
    - key, crt, csr 파일
    - ca.key, ca.crt, ca.srl 파일

<br>

> ### 3-5. 인증서 및 키 권한 설정 (서버 각각 진행)
```
cd /etc/openldap/certs
chown -R ldap:ldap *
```

<br>

> ### 3-6. CA 인증서를 시스템 신뢰 저장소에 등록 (서버 각각 진행)
```
cp ca.crt /etc/pki/ca-trust/source/anchors/
update-ca-trust extract
```

<br>

> ### 3-8. 인증서 적용 (서버 각각 진행)
```
[root@iREMB-R-M certs]# cd /etc/openldap/slapd.d/

[root@iREMB-R-M slapd.d]# vi ssl.ldif
dn: cn=config
changetype: modify
add: olcTLSCACertificateFile
olcTLSCACertificateFile: /etc/openldap/certs/ca.crt
-
replace: olcTLSCertificateFile
olcTLSCertificateFile: /etc/openldap/certs/iREMB-R-M.crt //해당 서버로 이름 변경
-
replace: olcTLSCertificateKeyFile
olcTLSCertificateKeyFile: /etc/openldap/certs/iREMB-R-M.key
```

```
ldapmodify -Y EXTERNAL -H ldapi:/// -f ssl.ldif
```

<br>

> ### 3-9. ldaps 프로토콜을 통한 접속 허용 설정
```
[root@iREMB-R-M slapd.d]# vi /etc/sysconfig/slapd
[root@iREMB-R-M slapd.d]# cat /etc/sysconfig/slapd
SLAPD_URLS="ldapi:/// ldap:/// ldaps:///"
[root@iREMB-R-M slapd.d]# systemctl restart slapd
```

<br>

> ### 3-10. ldaps 테스트
```
ldapsearch -x -H ldaps://iREMB-R-M.iREMB-R.HPC -D "cn=Manager,dc=iREMB-R,dc=HPC" -W -b "dc=iREMB-R,dc=HPC"
```

### 추가 디버깅이 필요한 경우 아래 명령어 사용
```
ldapsearch -x -H ldaps://iREMB-R-M.iREMB-R.HPC -d 1 -D "cn=Manager,dc=iREMB-R,dc=HPC" -W -b "dc=iREMB-R,dc=HPC"
```

<br>

#  4. LDAP 동기화 모듈 로드 (1번 서버)

### 4-1. `mod_syncprov.ldif` 작성
```
[root@iREMB-R-M ~]# cat mod_syncprov.ldif 
dn: cn=module,cn=config
objectClass: olcModuleList
cn: module
olcModulePath: /usr/lib64/openldap
olcModuleLoad: syncprov.la
```

```
ldapadd -Y EXTERNAL -H ldapi:/// -f mod_syncprov.ldif 
```

<br>

### 4-2. `syncprov.ldif` 작성
```
[root@iREMB-R-M ~]# cat syncprov.ldif 
dn: olcOverlay=syncprov,olcDatabase={2}mdb,cn=config
objectClass: olcOverlayConfig
objectClass: olcSyncProvConfig
olcOverlay: syncprov
olcSpSessionLog: 100
```

```
ldapadd -Y EXTERNAL -H ldapi:/// -f syncprov.ldif 
```

<br>

### 4-3. LDAP Config 파일 작성 (수정)
- config 파일을 수정하여 이중화 설정 값을 넣는다.

```
[root@iREMB-R-M ~]# vi master01.ldif 
==================================================================
dn: cn=config
changetype: modify
replace: olcServerID
olcServerID: 10   // iREMB-R-S와 다른 유니크한 값을 지정한다.

dn: olcDatabase={2}mdb,cn=config
changetype: modify
add: olcSyncRepl
olcSyncRepl: rid=001 	// server2와 다른 고유의 값 지정
  provider=ldaps://iREMB-R-S.iREMB-R.HPC/    // 서버2 인증서 생성 시 CN에 넣은 값
  bindmethod=simple
  binddn="cn=Manager,dc=iREMB-R,dc=HPC"  //서버2의 관리자 dn을 넣는다.
  credentials=iREMB-R-S-ldap-password  // 암호화되지 않은 패스워드 원본을 입력한다.
  searchbase="dc=iREMB-R,dc=HPC"
  scope=sub
  schemachecking=on
  type=refreshAndPersist
  retry="30 5 300 3"
  interval=05:00:00:00

add: olcMirrorMode
olcMirrorMode: TRUE

dn: olcOverlay=syncprov,olcDatabase={2}mdb,cn=config
changetype: add
objectClass: olcOverlayConfig
objectClass: olcSyncProvConfig
olcOverlay: syncprov
```

```
ldapmodify -Y EXTERNAL -H ldapi:/// -f master01.ldif
```

```
[root@iREMB-R-M ~]# vi /etc/openldap/slapd.d/cn\=config/olcDatabase\=\{2\}mdb.ldif
olcMirrorMode: TRUE

  - 마지막줄에 위 한줄 추가
```

<br>

#  5. LDAP 동기화 모듈 로드 (2번 서버)

### 5-1. `mod_syncprov.ldif` 작성
```
[root@iREMB-R-S ~]# vi mod_syncprov.ldif 
=========================================================
dn: cn=module,cn=config
objectClass: olcModuleList
cn: module
olcModulePath: /usr/lib64/openldap
olcModuleLoad: syncprov.la
```

```
ldapadd -Y EXTERNAL -H ldapi:/// -f mod_syncprov.ldif 
```

<br>

### 5-2. `syncprov.ldif` 작성
```
[root@iREMB-R-S ~]# vi syncprov.ldif 
======================================================
dn: olcOverlay=syncprov,olcDatabase={2}mdb,cn=config
objectClass: olcOverlayConfig
objectClass: olcSyncProvConfig
olcOverlay: syncprov
olcSpSessionLog: 100
```

```
ldapadd -Y EXTERNAL -H ldapi:/// -f syncprov.ldif 
```

<br>

### 5-3. LDAP Config 파일 작성 (수정)
- config 파일을 수정하여 이중화 설정 값을 넣는다.

```
[root@iREMB-R-S ~]# vi master02.ldif
===================================================================
dn: cn=config
changetype: modify
replace: olcServerID
olcServerID: 20   // iREMB-R-M와 다른 유니크한 값을 지정한다.

dn: olcDatabase={2}mdb,cn=config
changetype: modify
add: olcSyncRepl
olcSyncRepl: rid=002  // iREMB-R-M과 다른 유니크한 값 지정
  provider=ldaps://iREMB-R-M.iREMB-R.HPC/    //위의 CN에서 등록한 값 입력
  bindmethod=simple
  binddn="cn=Manager,dc=iREMB-R,dc=HPC"  //iREMB-R-M의 관리자 dn을 넣는다.
  credentials=iREMB-R-M-ldap-passwor  // 암호화하지 말고 그냥 패스워드 넣기
  searchbase="dc=iREMB-R,dc=HPC"
  scope=sub
  schemachecking=on
  type=refreshAndPersist
  retry="30 5 300 3"
  interval=05:00:00:00

add: olcMirrorMode
olcMirrorMode: TRUE

dn: olcOverlay=syncprov,olcDatabase={2}mdb,cn=config
changetype: add
objectClass: olcOverlayConfig
objectClass: olcSyncProvConfig
olcOverlay: syncprov

```

```
ldapmodify -Y EXTERNAL -H ldapi:/// -f master01.ldif
```

```
[root@iREMB-R-S ~]# vi /etc/openldap/slapd.d/cn\=config/olcDatabase\=\{2\}mdb.ldif
olcMirrorMode: TRUE

  - 마지막줄에 위 한줄 추가
```

<br>

### 이중화 성공여부 확인
- 두 서버에서 출력되는 `contextCSN`의 값이 같으면 이중화가 성공한 것임

```
<서버1>
ldapsearch –x –H ldaps://iREMB-R-M.iREMB-R.HPC –D "cn=Manager,dc=iREMB-R,dc=HPC" -W –b "dc=iREMB-R,dc=HPC" contextCSN

<서버2>
ldapsearch –x –H ldaps://iREMB-R-S.iREMB-R.HPC –D "cn=admin02,dc=iREMB-R,dc=HPC" -W –b "dc=iREMB-R,dc=HPC" contextCSN
```

<Br>

#  6. OpenLDAP 클라이언트 설정
- OpenLDAP 서버에도 로그인 기능을 부여할 것이라면, 서버에도 똑같이 클라이언트 설정을 해주면 된다.
- 이중화되어 있는 두 서버에 각각 설정해주어야 한다.

<br>

> ### 6-1. 필수 패키지 설치
```sh
yum install -y nss-pam-ldapd openldap-clients oddjob-mkhomedir authconfig oddjob
```

<Br>

> ### 6-2. `/etc/openldap/ldap.conf` 수정 
- 전역 LDAP 클라이언트 설정

- 먼저 LDAP 서버에서 클라이언트로 ca.crt를 전송해주어야 한다.

```
[root@iREMB-R-M certs]# vi /etc/openldap/ldap.conf

TLS_CACERT /etc/openldap/certs/ca.crt
SASL_NOCANON    on
URI ldaps://iREMB-R-M.iREMB-R.HPC ldaps://iremb-u-s.iREMB-R.HPC
BASE dc=iREMB-R,dc=HPC
```

<br>

> ### 6-3. `/etc/nslcd.conf` 수정
- nslcd 데몬 설정

- `nslcd`는 사용자, 그룹 및 기타 NSS(이름 지정 조회)를 수행하거나, 사용자 인증, 권한부여 또는 암호 수정(PAM)을 수행하려는 로컬 프로세스에 대한 LDAP 쿼리를 수행하는 데몬이다.

```
uid root
gid root
uri ldaps://iREMB-R-M.iREMB-R.HPC ldaps://iremb-u-s.iREMB-R.HPC

#base dc=iREMB-R,dc=HPC
base passwd ou=Users,dc=iREMB-R,dc=HPC
base group  ou=Groups,dc=iREMB-R,dc=HPC

binddn cn=Manager,dc=iREMB-R,dc=HPC
bindpw Ecore123!@#

tls_reqcert demand
tls_cacertfile /etc/openldap/certs/ca.crt
```

<br>

> ### 6-4. `sssd` 완전 비활성화
```sh
systemctl stop sssd
systemctl disable sssd
systemctl mask sssd
```

<Br>

> ### 6-5. `nslcd.conf` 권한 설정 및 시작
```
[root@iREMB-R-M certs]# chown root:root /etc/nslcd.conf
[root@iREMB-R-M certs]# chmod 600 /etc/nslcd.conf

[root@iREMB-R-M certs]# chown root:root ca.crt
[root@iREMB-R-M certs]# chmod 644 /etc/openldap/certs/ca.crt

[root@iREMB-R-M certs]# systemctl start nslcd
[root@iREMB-R-M certs]# systemctl enable nslcd
```

<br>

> ### 6-6. `/etc/pam.d/password-auth` 수정
- "x-window (KDE, Gnome) 로그인", "ssh 원격 접속" 시 설정하는 파일

- ldap server가 인증 역할만 하는 것이 아닌 로그인 역할까지 같이 하게 된다면, 여기서부터 클라이언트에 설정하는 값들을 서버에도 동일하게 적용해주어야 한다.

```
auth        required      pam_env.so
auth        required      pam_faildelay.so delay=2000000
auth        sufficient    pam_unix.so nullok try_first_pass
auth        requisite     pam_succeed_if.so uid >= 1000 quiet_success
auth        sufficient    pam_ldap.so use_first_pass
auth        required      pam_deny.so

account     required      pam_unix.so
account     sufficient    pam_localuser.so
account     sufficient    pam_succeed_if.so uid < 1000 quiet
account     [default=bad success=ok user_unknown=ignore] pam_ldap.so
account     required      pam_permit.so

password    requisite     pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type=
password    sufficient    pam_unix.so sha512 shadow nullok try_first_pass use_authtok
password    sufficient    pam_ldap.so use_authtok
password    required      pam_deny.so

session     optional      pam_keyinit.so revoke
session     required      pam_limits.so
-session     optional      pam_systemd.so
session     optional      pam_oddjob_mkhomedir.so umask=0077
session     [success=1 default=ignore] pam_succeed_if.so service in crond quiet use_uid
session     required      pam_unix.so
session     optional      pam_ldap.so
```

- auth: 사용자 신원 확인, 비밀번호 검증, 보안 정책 목적

- account: 사용자 계정 상태 점검, 접근 권한 조건 분기 처리 목적

- pam_succeed_if.so: 조건(예: uid >= 1000) 만족하는 사용자만 인증을 허용해 시스템 계정 등 특정 사용자 접근을 제한하기 위함

- pam_succeed_if.so: UID가 1000 미만(시스템 계정 등)인 경우 별도로 허용해서 시스템 계정 관리 목적에 맞게 분기 처리.


> ### 6-7. `/etc/pam.d/system-auth` 수정
```

auth        required      pam_env.so
auth        required      pam_faildelay.so delay=2000000
auth        sufficient    pam_unix.so nullok try_first_pass
auth        requisite     pam_succeed_if.so uid >= 1000 quiet_success
auth        sufficient    pam_ldap.so use_first_pass
auth        required      pam_deny.so

account     required      pam_unix.so 
account     sufficient    pam_localuser.so
account     sufficient    pam_succeed_if.so uid < 1000 quiet
account     [default=bad success=ok user_unknown=ignore] pam_ldap.so
account     required      pam_permit.so

password    requisite     pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type=
password    sufficient    pam_unix.so sha512 shadow nullok try_first_pass use_authtok
password    sufficient    pam_ldap.so use_authtok
password    required      pam_deny.so

session     optional      pam_keyinit.so revoke
session     required      pam_limits.so
-session     optional      pam_systemd.so
session     optional      pam_oddjob_mkhomedir.so umask=0077
session     [success=1 default=ignore] pam_succeed_if.so service in crond quiet use_uid
session     required      pam_unix.so
session     optional      pam_ldap.so
```

<br>

> ### 6-7. `/etc/nsswitch.conf` 수정
- /etc/nsswitch.conf 파일은 데이터베이스의 검색 순서를 정의한다. 이 파일에는 노드에서 네임 확인을 위해 네임 서비스에 연결하는 기본 순서가 포함되어 있다. 각 항목에 대해 사용할 네임 서비스와 조회 순서가 이 파일에 저장된다.

- LDAP 연동 시 nsswitch.conf 파일 내용 중 `passwd`, `group`, `shadow`, `protocols`, `netgroup`, `automount` 항목에 `ldap`을 추가해야 nslcd가 사용자 계정 인증 시 참조한다. (인증 방식에 따라 요구하는 파일은 상이하다.)

```
automount: files ldap
groups: files ldap systemd
netgroup: files ldap
passwd: files ldap systemd
protocols: files ldap
shadow: files ldap
(생략..)
```

<br>

> ### 6-8. `/etc/sysconfig/authconfig` 생성 또는 수정
```
USELDAP=yes
USELDAPAUTH=yes
USEMKHOMEDIR=yes
USESSSD=no
```

<br>

#  7. 홈 디렉터리 자동 생성 설정
> ### 7-1. `oddjob` 데몬 설치 및 활성화
```sh
yum install oddjob oddjob-mkhomedir
systemctl enable --now oddjobd 
```
- `oddjob`은 PAM 요청을 받아 홈 디렉터리를 생성하는 역할을 한다.

<BR>

> ### 7-2. nslcd 활성화, oddjobd 재시작
```sh
systemctl restart nslcd
systemctl enable —now nslcd
systemctl restart oddjobd
systemctl enable —now oddjobd
```

<Br>

> ### 7-3. 백업 파일 생성
- 실수로 `sssd` 데몬을 활성화시킨다면 `/etc/pam.d/` 아래 수정해놓았던 파일들이 초기화된다. 이 상황을 방지하기 위하여 백업 파일을 생성한다.

```
cd /etc/pam.d/
mkdir backup
cp system-auth password-auth backup
cp /etc/nsswitch.conf backup

cd backup
mv password-auth password-auth.bak
mv system-auth system-auth.bak
mv nsswitch.conf nsswitch.conf.bak
```

<Br>

#  8. OpenLDAP과 local간 동기화 스크립트 작성
> #### 스크립트 실행 전 각 서버에 맞게 내용 변경을 해주어야 한다.

- 스크립트의 길이가 너무 긴 관계로 github 참고

- run.sh (https://github.com/suhyeonkang-ecore/Study/blob/main/LDAP/run.sh)

- 이중화한 서버에는 LDAP DB간 복제만 가능하고, local은 복제가 안 되기 때문에 똑같이 스크립트를 실행해주어야 한다.

<br>

#  9.  NFS + LDAP + autofs 자동 홈디렉토리 마운트
- LDAP 사용자 홈디렉터리를 NFS 서버에서 공유하고 클라이언트가 자동으로 마운트해서 사용하게 할 것이다.
  
- 앞으로 계정 생성할 때 home 디렉터리 위치를 ‘/home/username’ 으로 지정

- 서버1만 설정 해놓고 서버 2에서 ldapsearch로 계정을 찾으려고 하면, 둘의 스키마가 달라졌기 때문에 다른 db를 소유하게 된다. 여기서 둘의 설정을 같이 해주어야 동기화가 완벽하게 된다.

```
ll /etc/openldap/slapd.d/cn\=config/cn\=schema
```

<br>

> ### 9-1. NFS 서버 설정
#### NFS 서버 IP : 192.168.50.93 (LDAP SERVER-1)

#### 1. NFS 서버 패키지 설치
```
yum install –y nfs-utils
systemctl enable --now nfs-server
```

#### 2. 홈 디렉토리 공유 준비
```
mkdir –p /home // 이미 있다면 생략
chmod 755 /home
```

#### 3. `/etc/exports` 설정
```
echo " /home *(rw,sync,no_root_squash,no_subtree_check)" >> /etc/exports
```


#### 4. `exports`로 적용
```
exportfs -rav
```

#### 5. 서비스 시작
```
systemctl enable --now nfs-server rpcbind
```

<Br>

> ### 9-2. LDAP 서버에 autofs 스키마 및 마운트 정보 등록
- LDAP 서버에서 수행 (기존 LDAP에 autofs 스키마 및 정보 추가)

####  1. `autofs.ldif` 적용
- objectClass는 모든 LDAP 엔트리에 필수로 기본 포함되는 속성이기 때문에, 별도로 MUST 조건으로 선언하면 안 된다.
- OpenLDAP는 objectClass가 MUST 속성에 포함되면 스키마 충돌로 등록을 거부한다.

```
vi autofs.ldif
===================================================================
dn: cn=autofs,cn=schema,cn=config
objectClass: olcSchemaConfig
cn: autofs
olcAttributeTypes: ( 1.3.6.1.1.1.1.25 NAME 'automountInformation' DESC 'Information used by the autofs automounter' EQUALITY caseExactIA5Match SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 SINGLE-VALUE )
olcObjectClasses: ( 1.3.6.1.1.1.1.13 NAME 'automount' DESC 'An entry in an automounter map' SUP top STRUCTURAL MUST ( cn $ automountInformation ) MAY description )
olcObjectClasses: ( 1.3.6.1.4.1.2312.4.2.2 NAME 'automountMap' DESC 'A group of related automount objects' SUP top STRUCTURAL MUST ou )
```

```
ldapadd -Y EXTERNAL -H ldapi:/// -f autofs.ldif
```

<br>

#### 2. `autofs.schema` 적용
- 스키마가 정의되어 있는지 확인

```
[root@iremb-s slapd.d]# ldapsearch -Y EXTERNAL -H ldapi:/// -b cn=schema,cn=config | grep -i automountInformation

SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
olcAttributeTypes: {0}( 1.3.6.1.1.1.1.25 NAME 'automountInformation' DESC 'Inf automounter map' SUP top STRUCTURAL MUST ( cn $ automountInformation ) MAY description )
```

<br>

- 스키마가 정의되어 있지 않다면 아래 파일을 적용시켜야 한다.

#### `autofs.schema.ldif`
```
vi autofs.schema.ldif
===================================================================
dn: cn=autofs,cn=schema,cn=config
objectClass: olcSchemaConfig
cn: autofs
olcAttributeTypes: {0}( 1.3.6.1.1.1.1.25 NAME 'automountInformation' DESC 'Information used by the autofs automounter' EQUALITY caseExactIA5Match SYNTAX 1.3.6.1.4.1.1466.115.121.1.26 SINGLE-VALUE )
olcObjectClasses: {0}( 1.3.6.1.1.1.1.13 NAME 'automount' DESC 'An entry in an automounter map' SUP top STRUCTURAL MUST ( cn $ automountInformation $ objectclass ) MAY description )
olcObjectClasses: {1}( 1.3.6.1.4.1.2312.4.2.2 NAME 'automountMap' DESC 'An group of related automount objects' SUP top STRUCTURAL MUST ou )
```

```
ldapadd -D cn=config -H ldapi:/// -W -f autofs.schema.ldif
```

<br>

#### 3. `auto.master.ldif` 적용
```
vi auto.master.ldif
===================================================================
dn: ou=auto.master,dc=iREMB-R,dc=HPC
objectClass: top
objectClass: automountMap
ou: auto.master

dn: cn=/home,ou=auto.master,dc=iREMB-R,dc=HPC
objectClass: top
objectClass: automount
cn: /home
automountInformation: ldaps://iremb-s.iREMB-R.HPC/ou=auto.home,dc=iREMB-R,dc=HPC
```

```
ldapadd -x -D "cn=Manager,dc=iREMB-R,dc=HPC" -W -f auto.master.ldif
```

<br>

#### 4. `auto.home.ldif` 적용
```
vi auto.home.ldif
=======================================================
dn: nisMapName=auto.home,dc=iREMB-R,dc=HPC
objectClass: top
objectClass: nisMap
nisMapName: auto.home

dn: cn=*,nisMapName=auto.home,dc=iREMB-R,dc=HPC
objectClass: nisObject
cn: *
nisMapEntry: -rw,sync nfssvr:/nethome/&
nisMapName: auto.home
```

```
ldapadd -x -D "cn=Manager,dc=iREMB-R,dc=HPC" -W -f auto.home.ldif
```

<br>

#  10. 모든 노드 (관리노드 + 계산노드) 공통 설정 – LDAP 클라이언트
- LDAP 서버를 클라이언트 겸용으로 사용할 것이라면 서버에서도 실행해주면 됨

> ### 10-1. autofs 설치
```sh
yum install -y autofs
```

<br>

> ### 10-2. `/etc/nsswitch.conf` 수정
```
automount: ldap files
```

<br>

> ### 10-3. autofs 설정
```
vi /etc/sysconfig/autofs
===================================================================
USE_MISC_DEVICE="yes"
LDAP_URI="ldaps://iremb-s.iREMB-R.HPC ldaps://iremb-s-m.iREMB-R.HPC"
MAP_OBJECT_CLASS="automountMap"
ENTRY_OBJECT_CLASS="automount"
MAP_ATTRIBUTE="ou"
ENTRY_ATTRIBUTE="cn"
VALUE_ATTRIBUTE="automountInformation"
SEARCH_BASE="dc=iREMB-R,dc=HPC"
```

- autofs 활성화
```
systemctl enable --now autofs
```

<br>
