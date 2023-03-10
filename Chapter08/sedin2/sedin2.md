## 8. 인덱스

### 8.1 디스크 읽기 방식
- **랜덤 I/O**, **순차 I/O**
- 데이터베이스의 성능 튜닝은 어떻게 **디스크 I/O**를 줄이느냐가 관건

    #### HDD와 SSD
    - HDD: 데이터 저장용 원판이 있고 데이터에 접근하기 위해 이 원판을 회전 시켜야 함
    - SSD: HDD의 원판 대신 플래시 메모리를 장착하고 있어서, 회전이 필요 없으므로 HDD보다 빠른 속도를 보임

    #### 랜덤 I/O와 순차 I/O
    - 랜덤 I/O는 데이터를 디스크에 저장하기 위해 헤드를 움직인다. 따라서 부하가 크다
    - 그룹 커밋, 바이너리 로그 버퍼, InnoDB 로그 버퍼등의 기능이 내장되어있음

---

### 8.2 인덱스란?
- 책의 목차 - 인덱스로 비유
- 책의 내용 - 데이터 파일로 비유
- DBMS 인덱스는 칼럼의 값을 주어진 순서로 미리 정렬하여 보관
- 인덱스는 SortedList, 데이터 파일은 ArrayList와 같은 자료구조를 사용
- **인덱스**는 데이터의 저장(INSERT, UPDATE, DELETE) 성능을 희생하고 **데이터 읽기(SELECT) 성능**을 높이는 기능

---

### 8.3 B-Tree 인덱스
- B-Tree의 "B"는 이진 트리가 아닌 **Balanced**를 의미
- 대부분의 인덱스는 거의 B-Tree를 사용

    #### 구조 및 특성
    - 루트 노드(Root node): 최상위에 존재하는 하나의 노드
    - 리프 노드(Leaf node): 가장 하위에 있는 노드들
    - 브랜치 노드(Branch node): 루트 노드와 리프 노드가 아닌 중간의 노드들
    - INSERT된 순서로 저장되는 것은 아님 -> 데이터가 삭제되면 해당 공간을 재활용 하기 때문에

    #### B-Tree 인덱스 키 추가 및 삭제
    - 테이블의 레코드를 저장 또는 변경하는 경우 인덱스 키 추가나 삭제 작업이 발생함
        ##### 인덱스 키 추가
        - B-Tree에 저장될 때는 저장될 키 값을 이용해 B-Tree상의 적절한 위치를 검색
        - 저장될 위치가 결정되면 레코드의 키 값과 대상 레코드의 주소 정보를 B-Tree의 리프 노드에 저장
        - 리프 노드가 꽉 차서 저장할 수 없을 때는 리프 노드가 분리(Split)돼야 하는데, 이는 상위 브랜치 노드까지 처리 범위가 넓어짐
        - 브랜치 노드는 자식 노드의 주소를 가지고 있는데, 리프 노드가 분리되면 새로 분리된 리프 노드의 주소 값을 해당 리프 노드의 상위 브랜치 노드의 자식 노드 주소 값을 변경 해줘야 하기 때문
        - 이러한 이유 때문에 상대적으로 쓰기 작업에 비용이 많이 드는 것으로 알려짐
        ##### 인덱스 키 삭제
        - 해당 키 값이 저장된 B-Tree의 리프 노드를 찾아서 그냥 삭제 마크만 하면 작업이 완료
        ##### 인덱스 키 변경
        - B-Tree의 키 값이 변경되는 경우 삭제 한 후(삭제 마크), 다시 새로운 키 값을 추가하는 형태로 처리
        - InnoDB 스토리지 엔진을 사용하는 테이블에 대해서는 이 작업이 모두 체인지 버퍼를 통해 지연 처리 가능함

    #### B-Tree 인덱스 사용에 영향을 미치는 요소
    - 인덱스를 구성하는 칼럼의 크기, 레코드 건수, 유니크한 인덱스 키 값의 개수 등

    #### B-Tree 인덱스를 통한 데이터 읽기
    - 인덱스를 통한 읽기 방식
        ##### 인덱스 레인지 스캔
        - 검색해야 할 인덱스의 범위가 결정됐을 때 사용하는 방식
        - 인덱스 레인지 스캔 탐색 순서
        1. 인덱스에서 조건을 만족하는 값이 저장된 위치를 찾음(인덱스 탐색)
        2. 1번에서 탐색된 위치부터 필요한 만큼 인덱스를 차례대로 쭉 읽음(인덱스 스캔)
        3. 2번에서 읽어 들인 인덱스 키와 레코드 주소를 이용해 레코드가 저장된 페이지를 가져오고, 최종 레코드를 읽어옴
        4. 쿼리가 필요로 하는 데이터에 따라 3번 과정이 필요하지 않을 수도 있음(커버링 인덱스)
        
        - 커버링 인덱스: 랜덤 읽기가 상당히 줄어들고 성능은 빨라짐

        ##### 인덱스 풀 스캔
        - 인덱스의 처음부터 끝까지 모두 읽는 방식을 인덱스 풀 스캔이라 함
        - 예를들어 인덱스가 (A, B, C) 칼럼 순으로 만들어져 있지만, 쿼리의 조건절이 B 또는 C 칼럼으로 검색하는 경우에 인덱스 풀 스캔을 함
        - 테이블 풀 스캔보다는 효율적임
        - 인덱스에 포함된 컬럼만으로 쿼리를 처리할 수 있는 경우 테이블의 레코드를 읽을 필요가 없음. 
        이는 곳 테이블 풀 스캔보다 더 적은 디스크 I/O로 쿼리를 처리할 수 있음

        ##### 루스 인덱스 스캔
        - 느슨하게 또는 듬성듬성하게 인덱스를 읽는 것을 의미
        - 주로 GROUP BY, MAX(), MIN() 함수에 대해 최적화를 하는 경우 사용됨

        ##### 인덱스 스킵 스캔
        - 예를들어 인덱스가 (A, B) 칼럼 순으로 만들어져 있을때 쿼리의 조건절이 B 컬럼으로 검색하는 경우 인덱스 풀 스캔을 함.
        - 하지만 인덱스 스킵 스캔 기능을 활성화 하면 해당 인덱스를 사용할 수 있음
        - 선행 칼럼의 유니크한 값의 개수가 소량일 때만 적용 가능한 최적화

    #### 다중 칼럼(Multi-column) 인덱스
    - 2개 이상의 칼럼을 포함하는 인덱스
    - 인덱스를 구성하는 칼럼의 순서에 따라 정렬되므로 인덱스 설계시 신중해야 함

    #### B-Tree 인덱스의 정렬 및 스캔 방향
    - 옵티마이저가 실시간으로 만들어내는 실행 계획에 따라 결정

    #### B-Tree 인덱스의 가용성과 효율성
    - EQUAL
    - IN
    - BETWEEN
    - LIKE '??%'

---

### 8.4 R-Tree 인덱스
- 

    #### 구조 및 특성
    -

    #### R-Tree 인덱스의 용도
    -

---

### 8.5 전문 검색 인덱스
- 

    #### 인덱스 알고리즘
    -

    #### 전문 검색 인덱스의 가용성
    -

---

### 8.6 함수 기반 인덱스
- 

    #### 가상 컬럼을 이용한 인덱스
    -

    #### 함수를 이용한 인덱스
    -

---

### 8.7 멀티 밸류 인덱스
- 

---

### 8.8 클러스터링 인덱스
- 

    #### 클러스터링 인덱스
    -

    #### 세컨더리 인덱스에 미치는 영향
    -

    #### 클러스터링 인덱스의 장점과 단점
    -

    #### 클러스터링 테이블 사용 시 주의사항
    -

---

### 8.9 유니크 인덱스
- 

    #### 유니크 인덱스와 일반 세컨더리 인덱스의 비교
    -

    #### 유니크 인덱스 사용 시 주의사항
    -

---

### 8.10 외래키
- 

    #### 자식 테이블의 변경이 대기하는 경우
    -

    #### 부모 테이블의 변경 작업이 대기하는 경우
    -

---