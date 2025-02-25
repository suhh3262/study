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
[root@docker1 openldap]# yum install compat-openldap openldap openldap-clients openldap-servers
```

#### [1.3 OpenLDAP 서버 유틸리티](https://docs.redhat.com/ko/documentation/red_hat_enterprise_linux/7/html/system-level_authentication_guide/openldap#s3-ldap-packages-openldap-servers)
링크 참고

#### [1.4 OpenLDAP 클라이언트 유틸리티](https://docs.redhat.com/ko/documentation/red_hat_enterprise_linux/7/html/system-level_authentication_guide/openldap#s3-ldap-packages-openldap-clients)
링크 참고

<br>

---

### 2. OpenLDAP 서버 구성
- 기본적으로 OpenLDAP 구성은 `/etc/openldap/` 디렉터리에 저장됨

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

#### 2.2 글로벌 구성 변경
LDAP 서버의 글로벌 구성 옵션은 `/etc/openldap/slapd.d/cn=config.ldif` 파일에 저장됨

`olcAllows` : 활성화할 기능 지정 가능
```
olcAllows: feature…
```











