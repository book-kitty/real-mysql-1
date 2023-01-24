## 5. 트랜잭션과 잠금

### 5.1 트랜잭션
- MySQL에서의 트랜잭션
- 주의사항

---

### 5.2 MySQL 엔진의 잠금
- 글로벌 락
- 테이블 락
- 네임드 락
- 메타데이터 락

---

### 5.3 InnoDB 스토리지 엔진 잠금
- InnoDB 스토리지 엔진의 잠금
- 인덱스와 잠금
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