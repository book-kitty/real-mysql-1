# 2. 설치와 설정
> 가능하면 리눅스의 RPM이나 운영체제별 인스톨러를 이용하길 권장함

### 버전과 에디션 선택
- 가능하면 최신 버전을 설치하는 것을 권장함. 
- 기존 버전에서 새로운 메이저 버전으로 업그레이드하는 경우라면 최소 패치버전이 15~20번 이상 릴리스된 버전을 선택하는 것이 안정적이다.
- 초기 버전의 MySQL은 엔터프라이즈와 커뮤니티 버전이 나뉘어있긴 했지만 실제로 서버 기능의 차이는 없었고 기술 지원의 차이만 있었다.
  - MySQL 5.5버전 이상부터 엔터프라이즈와 커뮤니티 기능의 차이가 생기고 엔터프라이즈 코드는 더이상 공개되지 않음

### 엔터프라이즈에서만 제공하는 기능
1. Thread Pool
2. Enterprise Audit
3. Enterprise TDE(Master Key) 관리
4. Enterprise Authentication 
5. Enterprise FireWall
6. Enterprise Monitor
7. Enterprise BackUp
8. MySQL 기술지원

## MySQL 설치

### 사용자 인증 방식 선택
- Use Legacy Password Encryption
  - Native Authentication 방식
  - 사설 네트웨크에서만 접속하게 된다면 이 방식을 추천함
- Use Strong Password Encryption
  - Caching SHA-2 Authentication 방식
  - 인터넷을 경유하여 접속한다면 이 방식을 추천함

### MySQL 디렉터리 정보
- bin
  - MySQL 서버와 클라이언트 프로그램, 유틸리티를 위한 디렉터리
- data 
  - 로그 파일과 데이터 파일들이 저장되는 디렉터리
- include 
  - C/C++ 헤더 파일들이 저장된 디렉터리
- lib
  - 라이브러리 파일들이 저장된 디렉터리
- share
  - 다양한 지원 파일들이 저장돼 있으며, 에러 메세지가 샘플 설정 파일(my.cnf)이 존재하는 디렉터리

## MySQL 서버의 시작과 종료
```text
**linux**
linux> systemctl start mysqld // 시작 명령어
linux> systemctl status mysqld // MySQL 서버 상태확인 명령어  
linux> systemctl stop mysqld // 종료 명령어
```
- systemd를 이용하게 되면 mysql 설정파일의 `mysqld_safe`를 섹션을 무시하고 실행하게 된다. 
- 이런 방식 말고도 `mysqld_safe` 스크립트를 이용해서 MySQL 서버를 시작하고 종료할 수 있다.
- `mysqld_safe` 스크립트를 이용하면 `my.cnf`를 참조하여 서버를 실행 시킬 수 있게된다.

### 원격으로 MySQL 서버 종료하기
```
mysql> SHUTDOWN // MySQL 서버에 로그인한 상태에서 명령어를 입력하여 종료
```
- 원격으로 MySQL 서버를 종료시키고 싶을 땐 위와같은 명령어를 이용하면 된다.
- 주의할 점은 원격으로 MySQL 서버를 셧다운 하려면 SHUTDOWN 권한(Privileges)을 가지고 있어야 한다.

### 클린 셧다운
```text
mysql> SET GLOBAL innodb_fast_shutdown=0;
linux> systemctl stop mysqld.service

**원격 종료시**
mysql> SET GLOBAL innodb_fast_shutdown=0;
mysql> SHUTDOWN;
```
MySQL은 트랜잭션이 정상적으로 수행되어도 데이터 파일에 반영되지 않고 로그파일에만 반영되는 경우가 있을 수 있다.
이는 비정상적인 상황은 아니다. 하지만 서버가 재시작 될 때 트랜잭션 복구 과정을 거쳐야 하므로 시작 시간이 오래 걸릴 수 있게 된다. 

이럴 때 위 명령어를 실행하면 모든 커밋된 내용을 데이터 파일에 기록하고 종료한다. 서버 시작 시간을 조금이라도 단축해야 할 때나 데이터파일에 꼭 기록해야 하는 경우라면
`clean shutdown`을 활용하는 것도 좋을 것 같다고 느껴진다.

**✔️ 주의 !**
> MySQL 서버가 시작하거나 종료될 때 MySQL 서버의 버퍼 풀 내용을 백업하고 복구하는 과정이 내부적으로 실행된다.
> 실제 버퍼풀의 내용을 백업하는게 아니라 버퍼 풀에 적재대 있던 데이터 페이지에 대한 메타정보를 백업하기 떄문에 용량이 크지않아 빠르게 수행된다.
> 하지만 MySQL 서버가 새로 시작할 때는 디스크에서 파일들을 모두 읽어서 적재해야 하므로 상당한 시간이 걸릴 수도 있다.
> MySQL 서버의 시작 시간이 오래걸린다면 MySQL 서버가 버퍼 풀의 내용을 복구하고 있는지 확인해보는 것이 좋다.


### 서버 연결 
```text
linux > mysql -uroot -p --host=localhost --socket=/tmp/mysql.sock
linux > mysql -uroot -p --host=127.0.0.1 --port=3306 
```

- 첫번째 예제
  - 첫번째 방식은 `Unix Domain Socket`을 이용하는 방식이다.
  - TCP/IP 방식이 아닌 유닉스의 프로세스간 통신 `IPC:Inter Process Communication`방식을 통해 접속한다.
    - ICP는 local Machine에서 프로세스간에 협력을 위한 통신 방식이다. 때문에 원격 접속에선 localhost를 활용할 수 없다.
- 두번째 예제
  - 두번째 방식은 TCP/IP를 활용한 접근 방식이다. 127.0.0.1도 자기 PC를 가리키는 루프백 IP이지만 TCP/IP 방식을 이용하여 접근한다.
  - 두번째 방식으로 접근할 때는 host와 port를 명시해줘야 한다.

### 관리자 계정
- 처음 설치된 MySQL 서버에는 root라는 관리자 계정이 준비되어 있다. 
- --initialize-insecure 옵션으로 MySQL 서버가 초기화 되었다면 비밀번호 없이 로그인 할 수 있다.
- --initialize 옵션으로 초기화 되었으면 MySQL 서버의 로그파일에 기록돼 있는 비밀번호를 이용해서 로그인하면 된다.

## 시스템 변수의 특징
> MySQL 서버는 기동하면서 설정파일의 정보를  읽어 메모리나 작동 방식을 초기화하고, 접속된 사용자를 제어하기 위한 별도값을 설정해둔다. 
> MySQL에서는 이러한 값을 시스템 변수라고 한다. 시스템 변수 설정값이 서버와 클라이언트에 영향을 미치는지 판단하려면 글로벌 변수와 세션 변수를 구분해야 한다. 

```text
**시스템 변수 조회**
mysql> SHOW GLOBAL VARIABLES;
```

### 글로벌 변수
글로벌 변수의 범위의 시스템 변수는 하나의 MySQL 서버 인스턴스에서 전체적으로 영향을 미치는 시스템 변수를 의미하며, 주로 MySQL 서버 자체에 관련된 설정일 떄가 많다.

### 세션 변수
세션 변수는 MySQL 클라이언트가 MySQL 서버에 접속할 때 기본으로 부여하는 옵션의 기본 값을 제어하는데 사용된다.


### 정적 변수
정적 변수는 시스템에 가동 중일 떄 변경할 수 없는 변수를 정적 변수라고 한다.

### 동적 변수
동적 변수는 시스템이 가동 중인 상태에서도 변경이 가능하다. 동적 변수는 아래와 같은 명령어를 통해 변경할 수 있다.

```text
SET GLOBAL max_connections=500;
SET PERSIST max_connection=500; // 8.0 부터 사용가능
```
**SET**
- SET 명령어를 통해 변경한 시스템 변수 값은 my.cnf 파일에 반영되는 것이 아닌 실행 중인 인스턴스에서만 유효하다.
- SET 명령어를 통해 변경한 시스템 변수를 my.cnf 파일에 반영하지 않으면 예상치 못한 이슈를 겪게될 수 있다.

**SET PERSIST**
- SET 명령어의 문제점을 해결하기 위해 MySQL 8.0부터 등장한 명령어가 `SET PERSIST`이다.
- SET PERSIST 명령어로 변경한 변수의 값은 별도의 파일로 따로 저장해두기 때문에 서버가 재구동 되어도 변경한 변수의 값이 반영된다.
- `SET PERSIST`는 변경한 시스템 변수의 값을 `mysqld-auto.cnf` 파일에 기록해둔다. 
  - MySQL은 서버가 시작될 때 `mysql.cnf`의 값과 `mysqld-auto.cnf`의 값을 참조하여 실행한다.