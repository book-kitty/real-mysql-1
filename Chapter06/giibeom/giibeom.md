## Chapter 6. 데이터 압축

## 페이지 압축

> MySQL 서버가 디스크에 저장하는 시점에 데이터 페이지가 압축되어 저장 <br>
반대로 MySQL 서버가 디스크에서 데이터 페이지를 읽어올 때 압축이 해제되어 조회
>
- 버퍼 풀에 데이터 페이지가 한번 적재되면 InnoDB 스토리지 엔진은 압축이 해제된 상태로만 데이터 페이지를 관리
- InnoDB I/O 레이어에서 압축/해제가 이루어진다
- 페이지 압축 기능에 필요한 펀치 홀 기능은 운영체제뿐만 아니라 하드웨어 자체에서도 해당 기능을 지원해야 사용 가능하다는 단점이 있다

<br>

## 테이블 압축

- 운영체제나 하드웨어에 대한 제약 없이 사용할 수 있어 활용도가 높다
- 아래와 같은 단점이 있다
  - 버퍼 풀 공간 활용률이 낮음
  - 쿼리 처리 성능이 낮음
  - 빈번한 데이터 변경 시 압축률 떨어짐
- 압축을 사용하려면 별도의 테이블 스페이스를 사용해야 한다
- `KEY_BLOCK_SIZE` 옵션을 통해 압축된 페이지가 저장될 페이지의 크기를 지정한다

<br>

### KEY_BLOCK_SIZE

- 압축된 결과를 어느 크기로 디스크에 저장할 지 정하는 시스템 변수
- 압축 적용 전 임시 테이블을 생성해서 샘플 데이터를 저장해보면서 압축 실패율을 측정해보면서 판단하는 것이 좋다
  - 압축 실패 : 압축된 결과가 변수에 설정된 값보다 크면 다시 데이터 페이지를 스플릿하는 것(사이즈 맞을 때까지 반복)
- 압축 실패율은 3~5% 미만으로 유지할 수 있게 값을 선택하는 것이 좋다
- 그렇다면 실패율이 높으면 압축을 사용하면 안될까?
  - INSERT만 되는 로그 테이블의 경우는 한번 INSERT 되면 다시는 변경되지 않음
  - 따라서 한번 정도는 압축 실패 시 페이지 스플릿 후 재압축한다고 하더라도 전체적으로 데이터 파일의 크기가 큰 폭으로 줄어든다면 큰 손해는 아닐 듯 함
  - 반대로 실패율이 높지 않더라도 테이블의 데이터가 매우 빈번하게 조회, 변경된다면 압축을 하지 않는 것이 더 효율적일 것이다.
- 이처럼 압축 알고리즘은 많은 CPU 자원을 소모하므로 Trade-off를 잘 고려하자

<br>

### 버퍼 풀

- InnoDB 스토리지 엔진은 압축된 테이블의 데이터 페이지를 버퍼 풀에 적재할 때 2개 버전의 상태를 관리한다
  - 압축된 상태인 데이터 페이지
  - 압축이 해제된 상태인 데이터 페이지
- 따라서 InnoDB 스토리지 엔진에서는 버퍼풀의 공간을 이중으로 사용함으로써 메모리를 낭비하는 효과를 가진다
- 또한 데이터를 읽거나 변경할 때에는 압축된 페이지를 해제해야됨
  - CPU를 많이 소모하므로, 이를 보완하기 위해 Unzip_LUR 리스트를 별도로 관리하여 적절한 처리를 진행함
