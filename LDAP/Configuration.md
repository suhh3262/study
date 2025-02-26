## LDAP 구성 과정

### 1. 패키지 설치

|Packages|Description|
|------------|----------------|
|openldap|OpenLDAP 서버 및 클라이언트 애플리케이션을 실행하는데 필요한 라이브러리 포함|
|openldap-clients|LDAP 서버에서 디렉터리를 보고 수정하기 위한 명령줄 유틸리티가 포함된 패키지|
|openldap-servers|LDAP 서버를 구성하고 실행하는 서비스 및 유틸리티 모두 포함|
|compat-openldap|OpenLDAP 호환성 라이브러리를 포함하는 패키지|

<br>

#### 1.1 일반적으로 설치된 추가 LDAP 패키지 목록
|Packages|Description|
|------------|----------------|
|nss-pam-ldapd|사용자가 로컬 LDAP 쿼리를 수행할 수 있는 로컬 LDAP 이름 서비스인 nslcd 가 포함된 패키지|
|openldap-clients|`mod_authnz_ldap` 및 `mod_ldap` 모듈이 포함된 패키지. 전자는 LDAP 디렉터리에 대해 사용자의 자격증명을 인증할 수 있으며, 사용자 이름, 전체 DN, 그룹 멤버십, 임의의 속성 또는 전체 필터 문자열에 따라 액세스 제어 적용. 동일한 패키지에 포함된 `mod_ldap` 모듈은 구성 가능한 공유 메모리 캐시를 제공하여 많은 http요청에서 반복된 디렉터리 액세스를 방지하고 ssl/tls도 지원함.|

<br>

#### 1.2 설치 진행
```
# yum install compat-openldap openldap openldap-clients openldap-servers openldap-servers-sql openldap-devel
# yum install net-tools
```

##### 1.2.1 서비스 시작 및 확인
```
# systemctl start slapd
# systemctl enable slapd
```

```
# netstat -antup | grep -i 389
```
![Image](https://github.com/user-attachments/assets/f8d712ba-ac27-47b8-ae34-954b000dbf43)

<br>

##### 1.2.2 구성 전 사전 작업
```
[root@docker1 ~]# slappasswd
New password:
Re-enter new password:
{SSHA}9Q/qqUelIBspvFIVTZBIVFsnmaATeuLr
```

#### [1.3 OpenLDAP 서버 유틸리티](https://docs.redhat.com/ko/documentation/red_hat_enterprise_linux/7/html/system-level_authentication_guide/openldap#s3-ldap-packages-openldap-servers)
링크 참고

#### [1.4 OpenLDAP 클라이언트 유틸리티](https://docs.redhat.com/ko/documentation/red_hat_enterprise_linux/7/html/system-level_authentication_guide/openldap#s3-ldap-packages-openldap-clients)
링크 참고

<br>

---

### 2. OpenLDAP 서버 구성
기본적으로 OpenLDAP 구성은 `/etc/openldap/` 디렉터리에 저장됨

<br>

#### 2.1 OpenLDAP 구성 파일 및 디렉터리 목록
`/etc/openldap/ldap.conf` : 클라이언트의 애플리케이션 구성 파일. `ldapadd`, `ldapsearch`, `Evolution` 등 <br>
`/etc/openldap/slapd.d/` : 슬래핑 구성이 포함된 디렉터리
- 슬래핑 구성 : LDAP 서버 데몬인 slapd의 설정을 의미
    - `slapd`: LDAP 프로토콜을 통해 클라이언트의 요청을 처리하는 서버 프로그램
- OpenLDAP은 더 이상 `/etc/openldap/slapd.conf`에서 구성을 읽지 않음
  - `/etc/openldap/slapd.d/` 디렉터리에 있는 구성 데이터베이스 사용함
- 이전 설치의 기존 `slapd.conf` 파일이 있는 경우 새 형식으로 변환 가능
    ```
    ~]# slaptest -f /etc/openldap/slapd.conf -F /etc/openldap/slapd.d/
    ```

<br>

#### 2.2 구성
- LDAP 설정파일
  - `/etc/openldap/slapd.d/` 경로 안에 존재
  - 설정항목 중 `olcSuffix`, `olcRootDN`, `olcRootPW`의 변경 필요
  - 설정파일 : `/etc/openldap/slapd.d/cn=config/olcDatabase={2}hdb.ldif`
    - 직접 수정하는 것은 권장하지 않음
        ```
        [root@docker1 ~]# cat /etc/openldap/slapd.d/cn\=config/olcDatabase\=\{2\}hdb.ldif
        
        # AUTO-GENERATED FILE - DO NOT EDIT!! Use ldapmodify.
        # CRC32 7a3e4c0b
        dn: olcDatabase={2}hdb
        objectClass: olcDatabaseConfig
        objectClass: olcHdbConfig
        olcDatabase: {2}hdb
        olcDbDirectory: /var/lib/ldap
        olcSuffix: dc=my-domain,dc=com
        olcRootDN: cn=Manager,dc=my-domain,dc=com
        olcDbIndex: objectClass eq,pres
        olcDbIndex: ou,cn,mail,surname,givenname eq,pres,sub
        structuralObjectClass: olcHdbConfig
        entryUUID: a9601950-8798-103f-9c5f-6f178c6f951b
        creatorsName: cn=config
        createTimestamp: 20250225074835Z
        entryCSN: 20250225074835.419749Z#000000#000#000000
        modifiersName: cn=config
        modifyTimestamp: 20250225074835Z
        ```
<br>

##### 2.2.1 데이터베이스 설정 변경
```
# vim db.ldif
```

```
dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=ksh,dc=co,dc=kr

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootDN
olcRootDN: cn=ldapadm,dc=ksh,dc=co,dc=kr

dn: olcDatabase={2}hdb,cn=config
changetype: modify
replace: olcRootPW
olcRootPW: {SSHA}SafttnL5f0DkIdOxr9kN2Hx0hNCnlsft
```
- olcRootPW : 앞서 생성한 암호화된 패스워드 입력
- dc : 사용하는 도메인 (e.g., ksh.co.kr)
  
💡 위의 순서대로 작성
    - `additional info: <olcRootPW> can only be set when rootdn is under suffix` // 오류 발생 주의

<br>

##### 2.2.2 적용
```
[root@docker1 slapd.d]# ldapmodify -Y EXTERNAL  -H ldapi:/// -f db.ldif

SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "olcDatabase={2}hdb,cn=config"

modifying entry "olcDatabase={2}hdb,cn=config"

modifying entry "olcDatabase={2}hdb,cn=config"
```

##### 2.2.3 monitor 접속 계정 설정
- monitor.ldfi 파일 생성 후 원본 설정파일 (`/etc/openldap/slad.d/cn=config/olcDatabase={1}monitor.ldif`)에 업데이트

<br>

##### 2.2.3.1 파일 생성
아래와 같이 ldapadm 계정만 접속 가능하게 설정

```
# vim monitor.ldif
# dn: olcDatabase={1}monitor,cn=configchangetype: modify
replace: olcAccess
olcAccess: {0}to * by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external, cn=auth" read by dn.base="cn=ldapadm,dc=ksh,dc=co,dc=kr" read by * none
```
- `dn: olcDatabase={1}monitor,cn=config` : LDAP 모니터링 위한 특수 데이터베이스 (LDAP 서버 상태 확인용)
- `olcAccess: {0}to * by ...` : ACL의 우선순위를 0번으로 설정
- `by dn.base="gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth" read`
      - UID=0, GID=0인 사용자 (root 권한 보유)가 LDAP 서버 내부에서 인증할 경우 read 권한 부여
- `by dn.base="cn=ldapadm,dc=ksh,dc=co,dc=kr" read` : LDAP 관리자에게 읽기 권한 부여
- `by * none` : 그 외 모든 사용자는 접근 불가

<br>

##### 2.2.3.2 적용
```
[root@docker1 slapd.d]# ldapmodify -Y EXTERNAL  -H ldapi:/// -f monitor.ldif
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
modifying entry "olcDatabase={1}monitor,cn=config"
```

<br>

#### 2.2.4 ldap database 설정
1. `usr/share/openldap-servers/` 폴더 내의 sample 파일 업데이트 후 권한 설정

   ```
   [root@docker1 slapd.d]# cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
    [root@docker1 slapd.d]# chown ldap:ldap /var/lib/ldap/
    ```

2. ldap schema 적용
    ```
    [root@docker1 slapd.d]# sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif
    [root@docker1 slapd.d]# sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif
    [root@docker1 slapd.d]# sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/inetorgperson.ldif
    ```

3. 도메인 정보 변경
    ```
    # vim base.ldif
    dn: dc=ksh,dc=co,dc=kr
    dc: ksh
    objectClass: top
    objectClass: domain
    
    dn: cn=ldapadm, dc=ksh,dc=co,dc=kr
    objectClass: organizationalRole
    cn: ldapadm
    description: LDAP Manager
    
    dn: ou=People, dc=ksh,dc=co,dc=kr
    objectClass: organizationalUnit
    ou: People
    
    dn: ou=Group, dc=ksh,dc=co,dc=kr
    objectClass: organizationalUnit
    ou: Group
    ```

4.  업데이트
    ```
    ldapadd -x -W -D "cn=ldapadm,dc=ksh,dc=co,dc=kr" -f base.ldif

    Enter LDAP Password:
    adding new entry "dc=ksh,dc=co,dc=kr"

    adding new entry "cn=ldapadm, dc=ksh,dc=co,dc=kr"

    adding new entry "ou=People, dc=ksh,dc=co,dc=kr"

    adding new entry "ou=Group, dc=ksh,dc=co,dc=kr"
    ```

   #### 2.2.5 계정 등록해보기
   ```
   # vim suhyeon.ldif
   ```

  1. 위 파일에 엔트리 작성
      ```
      [root@docker1 slapd.d]# vim suhyeon.ldif
        dn: uid=suhyeon,ou=People,dc=ksh,dc=co,dc=kr
        objectClass: top
        objectClass: account
        objectClass: posixAccount
        objectClass: shadowAccount
        cn: suhyeon
        uid: suhyeon
        uidNumber: 8888
        gidNumber: 100
        homeDirectory: /home/suhyeon
        loginShell: /bin/bash
        gecos: suhyeon [Admin (at) Ksh]
        userPassword: {crypt}x
        shadowLastChange: 17058
        shadowMin: 0
        shadowMax: 99999
        shadowWarning: 7
       ```

  2. 생성
     ```
     [root@docker1 slapd.d]# ldapadd -x -D "cn=ldapadm,dc=ksh,dc=co,dc=kr" -W -f suhyeon.ldif
     Enter LDAP Password:
     adding new entry "uid=suhyeon,ou=People,dc=ksh,dc=co,dc=kr"
     ```
    
  3. 확인
     ```
     [root@docker1 slapd.d]# ldapsearch -x -b "ou=People,dc=ksh,dc=co,dc=kr" "(uid=suhyeon)"                                     # extended LDIF
     #
     # LDAPv3
     # base <ou=People,dc=ksh,dc=co,dc=kr> with scope subtree
     # filter: (uid=suhyeon)
     # requesting: ALL
     #
     
     # search result
     search: 2
     result: 0 Success
     
     # numResponses: 1
     ```
     
  4. 패스워드 변경
     ```
     [root@docker1 slapd.d]# ldappasswd -s 1111 -W -D "cn=ldapadm,dc=ksh,dc=co,dc=kr" -x                                         "uid=suhyeon,ou=People,dc=ksh,dc=co,dc=kr"
     Enter LDAP Password:
     ```

  5. 계정 확인
     ```
     [root@docker1 slapd.d]# ldapsearch -x -b "ou=People,dc=ksh,dc=co,dc=kr" "(uid=suhyeon)"                                     # extended LDIF
     #
     # LDAPv3
     # base <ou=People,dc=ksh,dc=co,dc=kr> with scope subtree
     # filter: (uid=suhyeon)
     # requesting: ALL
     #
     
     # suhyeon, People, ksh.co.kr
     dn: uid=suhyeon,ou=People,dc=ksh,dc=co,dc=kr
     objectClass: top
     objectClass: account
     objectClass: posixAccount
     objectClass: shadowAccount
     cn: suhyeon
     uid: suhyeon
     uidNumber: 8888
     gidNumber: 100
     homeDirectory: /home/suhyeon
     loginShell: /bin/bash
     gecos: suhyeon [Admin (at) Ksh]
     shadowLastChange: 17058
     shadowMin: 0
     shadowMax: 99999
     shadowWarning: 7
     userPassword:: e1NTSEF9L0JlM3c2MWtlRmM5NDBGM1JCWjIzc1o0ZFBYOFdYb2k=
     
     # search result
     search: 2
     result: 0 Success
     
     # numResponses: 2
     # numEntries: 1
     ```
  6. 추가적인 설정 사항
     - 방화벽
       ```
       # firewall-cmd --permanent --add-service=ldap
       # firewall-cmd --reload

     - ldap server log
        ```
        # vim /etc/rsyslog.conf
        ```

        ```
        ldap 로그 설정 항목 추가
        # local4.* /var/log/ldap.log
        # systemctl restart rsyslog
        ```






