## 5. 트랜잭션과 잠금

### 5.1 트랜잭션
- MySQL에서의 트랜잭션
    - 하나의 논리적인 작업 셋에 하나의 쿼리가 한 개 있던 두 개 있던 작업 셋 자체가 100% 적용되거나(COMMIT) 아무것도 적용되지 않아야(ROLLBACK) 함을 보장 해주는 것이다.

---

- 주의사항
    - 트랜잭션의 범위를 최소화 하기
    - 외부 네트워크와 통신하는 로직이 있다면 반드시 트랜잭션 내에서 제거

---

### 5.2 MySQL 엔진의 잠금
- 글로벌 락
    - MySQL에서 제공하는 잠금 가운데 가장 범위가 큼
    - MyISAM or MEMORY 테이블에 대해 mysqldump로 일관된 백업을 받아야 할 때 사용
    - 일반적으로 사용 X 

--- 

- 테이블 락
    - 명시적 또는 묵시적으로 획득 가능
    - 글로벌 락과 동일하게 일반적으로 사용하면 온라인 서비스에 큰 영향을 끼칠수 있음

---

- 네임드 락
    - 특정 문자열에 대해 잠금을 얻고 해제
    - 배치성 프로그램에서 사용하면 간단하게 데드락을 해결 가능

---

- 메타데이터 락
    - 데이터베이스 객체의 이름이나 구조를 변경하는 경우에 획득 하는 잠금
    - DDL은 단일 스레드로 동작함

---

### 5.3 InnoDB 스토리지 엔진 잠금
- InnoDB 스토리지 엔진의 잠금
    - 레코드 락
        - 레코드 자체만 잠그는 락
        - 인덱스의 레코드를 잠그는 것
    - 갭 락
        - 레코드와 바로 인접한 레코드 사이의 간격만을 잠그는 것
        - 레코드와 레코드 사이에 INSERT 되는 것을 제어
    - 넥스트 키 락
        - 레코드락 + 갭 락
        - STATEMENT 포맷 바이너리 로그를 사용하는 MySQL서버에선 REPEATABLE READ 격리수준을 사용 해야함
        - 바이너리 로그에 기록되는 쿼리가 레플리카 서버에서 실행될 때 소스 서버에서 만들어 낸 결과와 동일한 결과를 만들어내도록 보장하는 것이 주목적이다.
    - 자동 증가 락
        - 명시적으로 획득할 수 없음
        - AUTO_INCREMENT 값을 가져오는 순간만 락이 걸렸다 해제됨
        - 대부분의 경우 성능상 문제는 없다함

---

- 인덱스와 잠금
    - InnoDB의 잠금은 레코드를 잠그는 것이 아니라 인덱스를 잠그는 방식으로 처리
    - 인덱스를 잘 못걸면 검색한 인덱스의 레코드들을 모두 잠금
    - 이러한 이유 때문에 인덱스를 잘 알고 사용 해야함

- 레코드 수준의 잠금 확인 및 해제

---

### 5.4 MySQL 격리 수준
- ![image](https://user-images.githubusercontent.com/53131108/214332277-901b6113-3657-4240-a9d0-bc54b23272f4.png)

- READ UNCOMMITTED
- ![image](https://user-images.githubusercontent.com/53131108/214332366-c1e6061a-eaab-4905-a64e-346fb2b38e7d.png)

---

- READ COMMITTED
- ![image](https://user-images.githubusercontent.com/53131108/214332437-7163ec3c-bac5-435e-bc6b-7a9b475e7640.png)

---

- REPEATABLE READ
- ![image](https://user-images.githubusercontent.com/53131108/214332498-530452cc-1e10-4f6e-8bb3-7607192a4ddb.png)
- ![image](https://user-images.githubusercontent.com/53131108/214332518-6f3abeab-13e1-40bc-9571-052ec549fd29.png)

---

- SERIALIZABLE
- ![image](https://user-images.githubusercontent.com/53131108/214332561-d791f220-010d-4458-9172-f0f2ceca20e1.png)
- ![image](https://user-images.githubusercontent.com/53131108/214332582-8e9831b8-bab7-4229-9209-83849bc76d06.png)

---