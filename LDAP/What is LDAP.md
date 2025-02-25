# 1. What is LDAP? 🧐
- Lightweight Directory Access Protocol
- client-server 아키텍처를 사용하여 네트워크에서 액세스할 수 있는 중앙 정보 디렉토리를 만들 수 있는 안정적 수단 제공
- 클라이언트가 이 디렉터리 내에서 정보를 수정하려고 하면, 서버에서 사용자에게 변경을 수행할 수 있는 권한이 있는지 확인 후 요청된 대로 항목을 추가하거나 업데이트
- 통신의 보안을 유지하기 위해 TLS (Transport Layer Security) 암호화 프로토콜을 사용하여 공격자가 전송을 가로채지 않도록 할 수 있음
- 여러 데이터베이스 시스템을 지원하므로 관리자가 서비스하려는 정보 유형에 가장 적합한 솔류선 선택 가능

<br>

### 1.1 LDAP 용어
- `DN` : LDAP 디렉터리 내의 단일 단위. 각 항목은 DN(Distinguished Name)으로 식별
- `LDIF` : LDAP Data Interchange Format, LDAP 항목의 일반 텍스트 표현
    ```
    [id] dn: distinguished_name
    attribute_type: attribute_value…
    attribute_type: attribute_value…
    ...
    ```
    - 선택사항인 `ID`는 항목을 편집하는데 사용되는 애플리케이션에 의해 결정된 숫자
    - 각 항목에는 모두 해당 스키마 파일에 정의된 한 개수의 attribute_type 및 attribute_value 쌍을 필요에 따라 포함할 수 있음
    - 빈 줄은 항목의 끝을 나타냄

