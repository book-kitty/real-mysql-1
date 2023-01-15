## 3. 사용자 및 권한

### 3.1 사용자 식별
- 사용자 계정 + 사용자의 접속 지점(클라이언트가 실행된 호스트명, 도메인, IP주소)도 계정의 일부
``` bash
# 서로 다른 계정이다.
'user01'@'192.168.0.10'
'user01'@'%'
```
### 3.2 사용자 계정 관리
- 시스템 계정 (System Account)
    - SYSTEM_USER 권한이 할당된 계정
    - 계정 관리
    - 다른 세션 또는 그 세션에서 실행 중인 쿼리를 강제 종료
    - 스토어드 프로그램 생성 시 DEFINER를 타 사용자로 설정
- 일반 계정 (Regular Account)
    - SYSTEM_USER 권한이 할당되지 않은 계정

- 계정 생성
    - CREATE USER 명령어로 생성
    - 인증방식, 비밀번호 설정
    - 비밀번호 관련 옵션
    - 기본 역할(ROLE) 부여
    - SSL 옵션 여부
    - 계정 잠금 여부
- 권한 부여
    - GRANT 명령으로 권한 부여
### 3.3 비밀번호 관리
- 고수준 비밀번호 관리
``` bash
mysql> SET GLOBAL validate_password.dictionary_file='prohibitive_word.data' # 금칙어 목록이 저장된 파일
mysql> SET GLOBAL validate_password.policy='STRONG'; # 패스워드 정책을 STRONG으로 부여
```

- 이중 비밀번호 (Dual Password)
    > 많은 응용 프로그램 서버들이 공용으로 DB 서버를 사용하기 때문에, DB 계정의 경우 서비스가 실행 중인 상태에서 변경이 불가능 했다.
    이러한 문제를 해결하기 위해 계정의 비밀번호를 2개로 사용 할 수있는 기능을 추가했다.
### 3.4 권한 (Privilege)
- MySQL 8.0의 권한은 3종류
    - 글로벌 권한: DB나 테이블 이외의 객체에 적용되는 권한
    - 객체 권한: DB나 테이블을 제어하는 데 필요한 권한
    - 동적 권한: MySQL 서버가 시작되면서 동적으로 생성되는 권한
### 3.5 역할 (Role)
- MySQL 8.0에서 새로 생긴 기능
- 권한을 묶어서 사용
- 내부적으로 계정과 같은 객체로 관리 (account_locked 필드로 구별 가능)
- 같은 객체로 관리 되기 때문에 계정과 역할을 구분하기 위해 역할(ROLE)에는 prefix나 keyword를 붙여 관리하길 권장