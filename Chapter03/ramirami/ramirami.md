# 사용자 및 권한
> MySQL의 사용자 계정은 단순히 사용자 ID 뿐만 아니라 사용자가 어느 IP에서 접속하였는지 확인한다.
> 또한 MySQL8.0 버전부터는 권한을 묶어서 관리하는 역할(Role, 롤)의 개념이 도입되었기 때문에 
> 각 사용자의 권한으로 미리 준비된 권한(Role) 세트를 부여하는 것도 가능하다.


### 사용자 식별
```text
1. 'svd_id'@'192.168.0.10' (계정의 비밀번호는 123)
2. 'svd_id'@'%' (계정의 비밀번호는 abc)
```
MySQL 계정으로 `1번`과`2번`이 등록되어있다고 가정했을 때 MySQL은 범위가 가장 작은 계정부터 선택한다.
위 계정 중 범위가 가장 좁은 것은 `'svd_id'@'192.168.0.10'` 계정이기 떄문에 IP가 192.168.0.10인 PC에서
"scv_id"라는 아이디와 "abc"라는 비밀번호로 접속했을 때 비밀번호가 일치하지 않는 이유로 접속이 거부될 수 있다.
이러한 이유로 계정을 생성할 떄는 항상 주의해야 한다. 중첩된 계정을 생성할 경우 원치않게 로그인이 불가능하게 되는 일이 발생할 수 있다.


### 시스템 계정과 일반 계정
**시스템 계정**
- 데이터베이스 관리자를 위한 계정
- 시스템 계정은 시스템 계정과 일반 계정을 관리할 수 있다.
- 데이터베이스 서버 관리와 관련된 중요한 작업은 시스템 계정으로만 수행할 수 있다.
  - 계정 관리(계정 생성, 삭제와 권한 부여 및 제거)
  - 다른 세션 또는 그 세션에서 실행 중인 쿼리를 강제 종료
  - 스토어드 프로그램 생성 시 DEFINER를 타 사용자로 설정

**일반 계정**
- 응용 프로그램이나 개발자를 위한 계정
- 계정 마다 부여된 권한이 다를 수 있다.

### 역할
- MySQL 8.0 부터는 권한을 묶어서 역할(Role)을 사용할 수 있다. 
- 실제 MySQL 서버 내부적으로는 계정과 권한을 따로 구분하지 않는다.

**역할 정의 방식**
```text
mysql> CREATE ROLE role_emp_read, role_emp_write
```
위의 CREATE ROLE 명령어는 빈 껍데기만 있는 역할을 정의한 것이다. GRANT 명령어를 이용하여 역할에 대해 실질적인 권한을 부여해야 한다.

**역할에 권한 부여**
```text
mysql> GRANT SELECT ON employees.* TO role_emp_read; 
mysql> GRANT INSERT, UPDATE, DELETE ON employees.* TO role_emp_write;
```
GRANT 명령어를 이용하여 역할에 권한을 부여 했다. 이제 역할을 사용하려면 계정에 부여해서 사용해야 하므로 계정을 생성해야 한다.


**계정 생성**
```text
mysql> CREATE USER reader@'127.0.0.1' IDENTIFIED BY `qwerty`;
mysql> CREATE USER writer@'127.0.0.1' IDENTIFIED BY `qwerty`;
```
CREATE USER 명령어를 이용하여 계정을 생성했다. 생성된 계정은 아직 아무런 권한을 부여받지 않았기 때문에 아무런 쿼리도 실행할 수 없다.

**계정에 역할 부여**
```text
mysql> GRANT role_emp_read To reader@'127.0.0.1'
mysql> GRANT role_emp_read, role_emp_write To writer@'127.0.0.1'
```
계정에 역할을 부여했다. 그러나 쿼리를 실행하려고 권한이 없다는 에러를 만나게 될 것이다. 
역할의 권한을 사용하려면 사용할 역할을 활성화 시켜야 한다. 활성화 시킨 역할은 로그아웃하면 다시 비활성화 상태로 변경된다.
이는 MySQL 서버의 역할이 자동으로 활성화 되지않게 설정되어 있어서 발생한 문제인데 자동으로 역할 활성화 여부를 `activate_all_roles_on_login` 시스템 변수로 설정할 수 있다.


```text
**역할 활성화**
mysql> SET ROLE 'role_emp_read';

**자동 활성화 옵션으로 변경**
mysql>SET GLOBAL activate_all_roles_on_login=ON;
```

### 계정과 역할이 분리된 이유
역할과 계정은 내부적으로 동일한 객체로 취급받지만 이를 분리한 이유는 데이터 베이스 관리 직무를 분리할 수 있게 해서
보안을 강화하는 용도로 사용하기 위함이다. `CREATE USER` 명령어에 대한 권한은 없지만 `CREATE ROLE` 명령어만 실행 가능한 사용자는 역할을 생성할 수 있게 된다.