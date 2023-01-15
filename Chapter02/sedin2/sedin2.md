## 2. 설치와 설정

### 2.1 MySQL 서버 설치
- 각 OS에 따른 설치 
    - Linux, macOS, Window
``` bash
# MySQL 설치 시 여러 기본 디렉토리들 중 삭제되선 안되는 디렉토리들
├── /usr/local/mysql
                 ├── bin
                 ├── data
                 ├── include
                 ├── lib
                 └── share
```
### 2.2 MySQL 서버의 시작과 종료
- linux에서 MySQL 서버 시작과 종료
``` bash
linux> systemctl start mysqld # MySQL 서버 시작
linux> systemctl stop mysqld  # MySQL 서버 종료
```

- 원격에서 MySQL 서버 종료
``` bash
mysql> SHUTDOWN; # MySQL 서버에 접속한 계정이 SHUTDOWN 권한을 가지고 있어야 가능
```

- Clean Shutdown
    - MySQL 서버 종료 직전 모든 커밋된 데이터를 데이터 파일에 적용하고 종료하는 것

``` bash
mysql> SET GLOBAL innodb_fast_shutdown=0; # default = 1;
mysql> SHUTDOWN;
```

- MySQL 서버 접속
    - 옵션의 유무에 따른 접속 방법 종류
    - host=localhost -> Unix domain socket 방식
    - host=127.0.0.1 -> TCP/IP 통신 방식
    - 별도 명시 X -> localhost가 기본값이 되며 Unix domain socket 방식 사용
### 2.3 MySQL 서버 업그레이드
- 인 플레이스 업그레이드 (In-Place-Upgrade)
    > MySQL 서버의 데이터 파일을 그대로 두고 업그레이드 하는 방법
- 논리적 업그레이드 (Logical Upgrade)
    > mysqldump 도구 등을 이용해 MySQL 서버의 데이터를 SQL 문장이나 텍스트 파일로 덤프한 후, 
    새로 업그레이드 된 버전의 MySQL 서버에서 덤프된 데이터를 적재하는 방법

- MySQL 8.0 업그레이드 시 고려 사항
    1. 사용자 인증 방식 변경
    2. MySQL 8.0과의 호환성 체크
    3. 외래키 이름의 길이
    4. 인덱스 힌트
    5. GROUP BY에 사용된 정렬 옵션
    6. 파티션을 위한 공용 테이블스페이스
### 2.4 MySQL 서버 설정
- MySQL 설정 파일은 단 하나의 설정 파일을 사용
- 설정 파일의 이름은 my.cnf
- 설정 파일의 위치는 여러 곳일 수 있지만 폴더 검색 우선순위에 따라 한 개의 파일만 참조
``` bash
shell> mysql --help # 해당 명령어로 확인 가능
```

- MySQL 시스템 변수
``` bash
mysql> SHOW GLOBAL VARIABLES; # 시스템 변수 확인
```
- MySQL 시스템 변수의 속성
    - Cmd-Line: CLI로 변경 가능한지 여부를 나타냄
    - Option file: my.cnf로 제어 가능한지 여부를 나타냄
    - System Var: 시스템 변수의 여부를 나타냄
    - Var Scope: 시스템 변수의 scope
    - Dynamic: 시스템 변수가 동적 변수인지 여부를 나타냄