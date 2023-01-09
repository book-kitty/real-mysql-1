## 설치와 설정

- 가능하다면 리눅스의 RPM, 운영체제별 인스톨러를 이용하여 설치

> 왜 mysql 버전은 5에서 8로 넘어갈까?
> > https://dba.stackexchange.com/questions/207506/what-happened-to-mysql-6-7

### `버전과 에디션`
- 엔터프라이즈 에디션과 커뮤니티 에디션으로 분리, 엔터프라이즈 에디션은 소스 코드 공개하지 않음
- 기존 버전에서 새로운 메이저 버전 (ex: 5.1, 5.5 .. 8.0 등에서 ) 15~20번 이상 패치된 버전을 사용하는 것이 안정적
    - 갓 출시된 메이저 버전은 치명적이거나 보완점이 필요한 버그가 있을 수 있음
- 엔터프라이즈 에디션에서만 지원하는 부가 기능
    - Thread Pool
    - Enterprise Audit
    - EnterPrise TDE
    - ..
- Percona <sup>[percona](#percona)</sup> 에서 출시하는 Percona Server 백업 및 모니터링 도구, 플러그인을 활용하면 커뮤니티 에디션을 보완할 수 있음

### `설정 파일 및 데이터 파일 준비`

- 초기에 트랜잭션 로그 파일이나 시스템 테이블이 준비되지 않았기 때문에 mysql 서버 시작 불가능
- 초기에 my.cnf 에는 기본적인 설정만 존재
- 'root'@'localhost'는 관리용으로 비밀번호를 지정해야 함
- 'root'@'localhost' 계정의 비밀번호는 서버 초기화 방법에 따라 달라짐 <sup>[data-directory-initialization](https://dev.mysql.com/doc/refman/8.0/en/data-directory-initialization.html)</sup>

```shell
# 임시 비밀번호를 생성, 에러 로그 파일 확인
linux> mysqld --defaults-file=/etc/my.cnf --initialize
```
```shell
# 비밀번호 없이 가능
linux> mysqld --defaults-file=/etc/my.cnf --initialize-insecure
```

### `시작과 종료`

```shell
# 시작|상태|종료
linux> systemctl start|status|stop mysqld
```

- mysqld_safe 스크립트를 이용하여 mysql 서버를 시작 및 종료 가능
    - mysqld_safe는 오류가 발생할 때 서버를 다시 시작하고 몇가지 안전 기능을 추가 <sup>[mysqld-safe](https://dev.mysql.com/doc/refman/8.0/en/mysqld-safe.html)</sup>

```shell
# 서버 종료될 때 모든 트랜잭션 커밋에 대해 파일에 반영하고 종료 옵션, 클린 셧다운
# mysql 기동할 때 트랜잭션 복구 과정을 불필요하게 함
# 참고 : https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_fast_shutdown
mysql> SET GLOBAL innodb_fast_shutdown=0;
# 원격으로 접속하여 서버를 종료
# 원격으로 종료할 경우 SHUTDOWN 에 대한 권한 필요 
mysql> SHUTDOWN;
```

### `서버 연결`

```shell
# host 명시하지 않을 경우 localhost
# host가 localhost인 경우 IPC 통신, 127.0.0.1로 하면 TCP/IP 통신
linux> mysql -uroot -p --host=127.0.0.1 --port=3306
```

- 원격에서 서버를 직접 로그인하지 않고 접속 가능 여부만 확인할 경우 telnet 사용

### `서버 업그레이드`

- 업그레이드에는 2가지 방식이 존재
    - 데이터 파일을 그대로 활용한 업그레이드 (인플레이스 업그레이드, 제약 사항 높음/시간 소요 적음)
    - 데이터를 SQL 문장이나 텍스트 파일로 덤프한 후, 업그레이드된 새로운 서버에 데이터 복구 (논리적 업그레이드, 제약 사항 적음/시간 소요 높음)
- 인플레이스 업그레이드 <sup>[upgrade-procedure-inplace](https://dev.mysql.com/doc/refman/5.7/en/upgrade-binary-package.html#upgrade-procedure-inplace)</sup>
    - 마이너 버전 간 업그레이드는 대부분 데이터 파일의 변경 없이 진행, 여러 버전을 건너뛰는 것도 허용
    - 메이저 버전 간 업그레이드는 대부분 데이터 파일의 변경이 필요

### `설정 파일 구성`

- 설정 파일로 유닉스 계열은 my.cnf, 윈도우는 my.ini
    - 지정된 여러 개의 디렉터리를 순차적으로 탐색하면서 처음 발견된 my.cnf 사용

```shell
# my.cnf 경로를 확인할 수 있음
linux> mysql --help
...
```

- VM을 이용한 mysql 서버를 중복으로 실행할 경우 경로가 충돌할 수 있음
    - 별도 디렉터리에 설정 파일을 준비
- my.cnf 에는 "[mysqld_safe]", "[mysql]"과 같은 설정 그룹으로 분류
    - my.cnf에 옵션을 설정할 수 있는 규칙이 존재 <sup>[option-files](https://dev.mysql.com/doc/refman/8.0/en/option-files.html)</sup>

### `시스템 변수`

- 기동하면서 설정 파일의 내용을 읽어 메모리, 작동 방식을 초기화 및 접속된 사용자를 제어하기 위해 값을 별도로 저장
- 시스템 변수는 글로벌, 세션 변수로 구분 / 정적, 동적 벼누로 구분 <sup>[server-system-variables](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html)</sup>

```shell
# 검색틍해 변수 확인
mysql> SHOW GLOBAL VARIABLES LIKE '%max%'
...

# 수정
mysql> SET GLOBAL max_connections=500;
```

- 동적 변수를 수정할 경우 재시작 없이 반영할 수 있음
- 서버를 재식할 경우 해당 설정은 다시 원복

```shell
# 변경된 값을 적용 및 mysqld-auth.cnf 에 변경 내용을 기록, 세션 변수에는 적용되지 않음
mysql> SET PERSIST max_connections=500;

# 현재 서버에는 적용하지 않고 오직 mysqld-auth.cnf 기록, 재시작을 하면 적용
# 정적 변수는 해당 명령어를 활용
mysql> SET PERSIST_ONLY max_connections=500;
```

- PERSIST, PERSIST_ONLY 명령을 통한 시스템 변수 내용을 삭제해야할 수 있음

```shell
# 특정 시스템 변수 삭제
mysql> RESET PERSIST max_connections;ß
mysql> RESET PERSIST IF EXISTS max_connections;

# mysqld-auth.cnf 파일의 모든 시스템 변수 삭제
mysql> RESET PERSIST;
```

---

<a name="percona">MySQL에서 근무하던 엔지니어들을 주축으로 2006년도에 설립된 MySQL관련 컨설팅 전문 회사</a>