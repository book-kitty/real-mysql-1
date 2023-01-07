## 사용자 및 권한

### `사용자 식별`

- 사용자의 계정, 접속 지점(host, ip)도 계정의 일부
    - 예시로 'test'@'127.0.0.1'
- 동일 계정에 대해서는 접속 지점의 범위가 좁은 것을 우선
    - 'test'@'192.168.0.10', 'test'@'%' 에서는 'test'@'192.168.0.10'를 우선

### `계정 및 비밀번호 관리`

- 계정은 SYSTEM_USER 권한 소유 여부에 따라 시스템, 일반 계정으로 구분
    - 시스템 계정은 데이터베이스 서버 관리자를 위함
    - 일반 계정은 응용 프로그램이나 개발자를 위함
- 내장된 계정이 존재, mysql.* 형태 <sup>[reserved-accounts](https://dev.mysql.com/doc/refman/8.0/en/reserved-accounts.html)</sup>
- 비밀번호 유효기간, 이력 관리, 글자 조합 강제, 금칙어 설정하는 컴포넌트를 사용할 수 있음 <sup>[validate-password](https://dev.mysql.com/doc/refman/8.0/en/validate-password.html)</sup>
- 계정 비밀번호 보안을 위해 계정의 비밀번호로 2개 값을 동시에 사용 가능 (이중 비밀번호) <sup>[set-password](https://dev.mysql.com/doc/refman/8.0/en/set-password.html)</sup>
    - 프라이머리와 세컨더리 비밀번호로 구성, 최근 설정한 비밀번호가 프라이머리임
    - 모든 응용프로그램이 프라이머리 비밀번호로 변경 후에 세컨더리 비밀번호를 삭제
        - 세컨더리 비밀번호 삭제를 무조건 해야하는 것은 아니지만 보안상 삭제하는 것을 권장

### `권한`

- 글로벌 권한과 객체 권한으로 분류 <sup>[privileges-provided](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html)<sup>
    - 글로벌 권한 : 데이터베이스나 테이블 이외의 객체엥 적용되는 권한
    - 객체 권한 : 데이터베이스나 테이블을 제어하는 데 필요한 권한
- GRANT 명령어를 통해 권한을 부여
    - 글로벌 권한을 부여할 경우 DB나 테이블에 부여가 불가하여 ON 절에 *.*만 가능
    - DB 권한은 특정 테이블을 명시할 수 없고 *만 가능
- 각 계정이나 권한, 역할 확인을 위해 SHOW GRANTS를 사용하거나 관련 테이블을 볼 수 있음 <sup>[grant-tables](https://dev.mysql.com/doc/refman/8.0/en/grant-tables.html)<sup>

### `역할`

- 역할과 계정은 서버 내부적으로는 같은 모습
- 역할은 생성 및 권한을 부여 받을 수 있지만 사용하기 위해서는 GRANT 명령어를 통해 사용자에게 부여해야 함
- 역할은 초기에는 비활성화 되어 있기 때문에 활성화를 해야만 사용자가 역할에 부여된 권한을 사용함
- 계정과 역할은 내부적으로 구분이 어렵기 때문에 'role_' 과 같은 prefix 를 붙여 생성하는 것을 권장

```shell
# 역할 활성화, 다시 로그인하면 비활성화로 초기화
mysql> SET ROLE 'role_emp_read';

# 역할을 자동으로 활성화하는 시스템 변수
mysql> SET GLOBAL activate_all_roles_on_login=ON;
```

- 역할 생성 시, 호스트 부분을 명시할 수 있음
    - 특정 계정에 할당되었을 경우 아무런 영향이 없음
    - 역할을 계정에 부여하지 않고 직접 로그인할 경우 호스트 부분 중요
- 계정과 역할은 내외부적으로 같은 객체이나 create user, create role 명령어는 구분되어 사용
    - 데이터베이스 관리적인 측면에서 분리하여 보안을 강화할 수 있음
    - 만약 하나의 명령어로만 관리가 되었다면 계정과 역할에 대해 전부 무분별한 생성이 가능