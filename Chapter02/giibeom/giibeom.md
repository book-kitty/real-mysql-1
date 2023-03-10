# Chapter 2. 설치와 설정

### MySQL 서버 디렉터리 구조

> 아래 폴더들이 삭제된다면 MySQL 서버가 정상적으로 실행되지 않을 수 있음

- bin : MySQL 서버와 클라이언트 프로그램, 그리고 유틸리티를 위한 디렉터리
- include : C/C++ 헤더 파일들이 저장된 디렉터리
- lib : 라이브러리 파일들이 저장된 디렉터리
- share : 다양한 지원 파일들이 저장돼 있으며, 에러 메시지나 샘플 설정 파일(my.ini)이 있는 디렉터리

<br>

### MySQL 서버 실행/종료

- MySQL 서버 시작 및 종료될 때 MySQL 서버(InnoDB 스토리지 엔진)의 버퍼 풀 내용을 백업하고 복구하는 과정이 내부적으로 실행
- 실제 버퍼 풀의 내용을 백업하는 것이 아닌, 버퍼 풀에 적재돼 있던 데이터 파일의 데이터 페이지에 대한 메타 정보를 백업하기 때문에 용량이 크지 않고, 백업 자체는 매우 빠르게 완료

- MySQL 서버가 새로 시작될 때는 디스크에서 데이터 파일들을 모두 읽어서 적재해야 하므로 상당한 시간이 걸릴 수도 있다.
- 따라서 MySQL 서버의 시작 시간이 오래 걸린다면 MySQL 서버가 버퍼 풀의 내용을 복구하고 있는지 확인

<br>

#### MySQL 서버를 새로 시작할 때 쿼리 처리 성능이 평상시의 1/10 수준이다. 어떻게 대처할까

> 자세한건 4장 아키텍처 챕터에서 다시 다루기
>
- 일반적으로는 버퍼 풀에 쿼리들이 사용할 데이터가 이미 준비되어 있으므로, 데이터를 디스크에서 읽지 않아도 쿼리가 처리될 수 있기에 빠르다 (캐시)
    - [버퍼 풀] - [참고자료](https://www.ibm.com/docs/ko/db2/11.1?topic=databases-buffer-pools) <br>
      테이블 및 인덱스 데이터를 디스크에서 읽을 때 이를 캐시하기 위해 데이터베이스 관리자를 통해 할당한 주 기억장치 영역
- MySQL 5.5 버전
    - 서비스 오픈 전 강제 워밍업을 위해 주요 테이블과 인덱스에 대해 풀 스캔을 한번씩 실행하여, 디스크의 데이터가 버퍼 풀에 잘 적재될 수 있도록 사전 작업
- MySQL 5.6 버전 이후
    - MySQL 서버 셧다운 전 백업 시스템 변수(`innodb_buffer_pool_dump_now`)를 이용해 현재 InnoDB 버퍼 풀의 상태를 백업
    - MySQL 서버 재시작 후 복구 시스템 변수(`innodb_buffer_pool_load_now`)를 이용해 백업된 버퍼 풀의 상태를 복구

<br>

### MySQL 서버 연결

- localhost로 접속

  > `mysql -uroot -p --host=localhost --socket=/tmp/mysql.sock`
  >
    - MySQL 클라이언트 프로그램은 항상 소켓 파일을 통해 접속
    - Unix domain socket 방식을 이용하여 TCP/IP 통신 방식이 아닌 유닉스의 프로세스 간 통신(IPC) 방식의 일종
- 127.0.0.1로 접속

  > `mysql -uroot -p --host=127.0.0.1 --port=3306`
  >
    - 자기 서버를 가리키는 루프백 IP이기는 하지만 TCP/IP 통신 방식을 통해 접속
- host 없이 접속

  > `mysql -uroot -p`
  >
    - 기본 값으로 localhost가 되어 소켓 파일을 사용


<br>

### MySQL 서버 설정

- MySQL 서버가 시작될 때 단 하나의 설정 파일을 사용(참조)
    - 리눅스를 포함한 유닉스 계열 : `my.cnf`
    - 윈도우 계열 : `my.ini`
- 설정 파일 경로는 여러 곳 일 수 있음
    - `mysql --help` or `mysql --verbose --help` 명령어로 어느 경로에 있는 설정 파일을 참조하는지 확인 가능
    - ex) `/etc/my.cnf`, `/etc/mysql/my.cnf`, `/usr/etc/my.cnf`, `~/.my.cnf`

<br>

### MySQL 변수

#### 시스템 변수

- MySQL 서버는 기동하면서 설정 파일의 내용을 읽어 메모리나 작동 방식을 초기화하고, 접속된 사용자를 제어하기 위한 값을 변수로 저장
    - MySQL 8.0부터는 모든 시스템 변수는 언더스코어(`_`)을 사용
    - 명령행 옵션으로만 사용 가능한 설정들은 하이푼(`-`)을 사용
- Var Scope : 시스템 변수의 적용 범위(글로벌 변수 or 세션 변수)를 나타냄
- Dynamic : 시스템 변수가 동적인지 정적인지 구분하는 변수

<br>

#### 글로벌 변수와 세션 변수

- 글로벌 변수 : MySQL 서버 인스턴스에서 전체적으로 영향을 미치는 시스템 변수
    - 주로 MySQL 서버 자체에 관련된 설정 (InnoDB 버퍼 풀 크기 or MyISAM의 키 캐시 크기 등)
- 세션 변수 : 클라이언트의 필요에 따라 개별 커넥션 단위로 다른 값으로 변경할 수 있는 변수
    - MySQL 클라이언트가 서버에 접속할 때 기본으로 부여하는 옵션의 기본 값을 제어하는데 사용
    - 기본 값은 글로벌 시스템 변수이고, 각 클라이언트가 가지는 값이 세션 시스템 변수이다

<br>

#### 정적 변수와 동적 변수

- MySQL 서버가 기동 중인 상태에서 변경 가능한지에 따라 정적인지 동적인지 나눠짐
    - 디스크에 저장돼 있는 설정 파일(my.cnf or my.ini)을 변경하는 경우
    - 이미 기동 중인 MySQL 서버의 메모리에 있는 MySQL 서버의 시스템 변수를 변경하는 경우

<br>

#### SET PERSIST

- SET PERSIST 명령을 이용하여 실행중인 MySQL 서버의 시스템 변수를 변경함과 동시에 자동으로 설정 파일로도 기록할 수 있음 (동적 변수)
    - 변경된 시스템 변수는 my.cnf 파일이 아닌 별도의 파일에 기록된다
- 동적 변수의 경우 MySQL 서버에서 SET GLOBAL 명령으로 변경하면 즉시 MySQL 서버에 반영
- 하지만 기본 설정 파일에는 자동으로 적용되지 않는 문제로 인해 MySQL 8.0부터는 SET PERSIST 명령이 도입
    - SET PERSIST로 시스템 변수를 변경하면 MySQL 서버는 변경된 값을 즉시 적용
    - 동시에 별도의 설정 파일(mysqld-auto.cnf)에 변경 내용을 추가로 기록
    - MySQL 서버가 재시작 될 때 기본 설정 파일(my.cnf)뿐만 아니라 자동 생성된 mysqld-auto.cnf 파일을 같이 참조해서 시스템 변수를 적용
- #### 주의 mysqld-auto.cnf 파일 내용을 직접 변경하다가 내용상 오류를 만드는 경우 MySQL 서버가 시작되지 못할 수도 있음
    - 따라서 특정 시스템 변수만 삭제하거나 mysqld-auto.cnf 파일의 모든 시스템 변수를 삭제하는 명령어를 사용하여 변수를 수정하는 것이 좋음
    - 특정 시스템 변수만 삭제 : `RESET PERSIST max_connections;`
    - mysqld-auto.cnf 파일의 모든 시스템 변수 삭제 : `RESET PERSIST;`
