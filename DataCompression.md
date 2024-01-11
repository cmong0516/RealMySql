# 데이터 압축
> - MySQL 서버에서 디스크에 저장된 데이터 파일의 크기는 일반적으로 쿼리의 처리 성능과 직결되지만 백업 및 복구 시간과도 밀접하게 연결된다.
- 데이터 크기가 크면 클수록 쿼리를 처리하기 위해 더 많은 데이터 페이지를 InnoDB 버퍼 풀로 읽어야 할수도 있고 새로운 페이지가 버퍼 풀로 적재되기 때문에 그만큼 더티 페이지가 더 자주 디스크로 기록되어야 한다.
- 데이터 파일이 크면 클수록 백업시간과 복구 시간이 오래 걸리고 그만큼의 저장 공간이 필요하기 때문에 비용 문제가 발생하기도 하여 데이터 압축이 필요하다.

## 6.1 페이지 압축
>- MySQL 서버가 디스크에 저장하는 시점에 데이터 페이지가 압축되어 저장되고 서버가 디스크에서 데이터 페이지를 읽어올때 압축이 해제된다.
- InnoDB 스토리지 엔진은 압축이 해제된 상태로만 데이터 페이지를 관리하기 때문에 서버의 내부 코드에서는 압축 여부와 관계없이 투명하게 작동한다.
- 16KB 데이터 페이지를 압축한 결과의 용량이 얼마나 될지 예측이 불가능하여 하나의 테이블은 동일한 크기의 페이지로 통일되어야 하는데
동일한 크기의 페이지로 통일하기 위해 펀치홀 이라는 기능을 사용한다.
1. 16KB 페이지를 압축(7KB 라고 가정)
2. MySQL 서버는 디스크에 압축된 결과 7KB 를 기록
(MySQL 서버는 압축 데이터 7KB 에 9KB 의 빈 데이터를 기록)
3. 디스크에 데이터를 기록하고 7KB 이후의 공간 9KB 에 대해 펀치 홀을 생성.
4. 파일 시스템은 7KB 만 남기고 나머지 디스크의 9KB 공간은 운영체제에 반납.
- 위 과정에서 9KB 만큼의 펀치 홀이 생성되었고 실제 디스크의 공간은 7KB 이지만 운영체제에서 16KB 를 읽으면 압축된 데이터 + 펀치홀 을 합쳐서 16KB 를 읽는다.
- MySQL 서버의 페이지 압축이 가진 문제는 펀치 홀 기능은 운영체제 뿐만 아니라 하드웨어 자체에서도 해당 기능을 지원해야만 하는데 파일 시스템 관련 명령어가 펀치 홀을 지원하지 못한다.
- 이런 이유 때문에 실제로 페이지 압축은 많이 사용되지 않는다.

## 6.2 테이블 압축
>- 테이블 압축은 운영체제나 하드웨어에 대한 제약 없이 사용할수 있기 때문에 더 활용도가 높고 디스크의 데이터 파일 크기를 줄일수 있다는 장점이 있다.
- 단점
>> - 버퍼 풀 공간 활용률 낮음
>>- 쿼리 처리 성능이 낮음
>>- 빈번한 데이터 변경시 압축률이 떨어짐

### 6.2.1 압축 테이블 생성
> - 테이블 압축을 사용하기 위한 전제 조건으로 압축을 사용하려는 테이블이 별도의 테이블 스페이스를 사용해야 하는데 이를 위해 innodb_file_per_table 시스템 변수가 활성화 된 상태에서 생성해야 하고 테이블을 생성할때 row_format=compressed 옵션을 명시해야 한다.
- key_block_size 옵션을 이용해 ㅇ바축된 페이지의 타킷 크기를 명시해야 한다.(2의 배수 값으로 설정할수 있다.)
1. set global innodb_file_per_table=on;
2. create table compressed_table(c1 in primary key)
row_format-compressed
key_block_size=8;

#### 실행순서
>1. 16KB의 데이터 페이지를 압축
1.1 압축된 결과가 8KB 이하이면 그대로 디스크에 저장
1.2 압축된 결과가 8KB를 초과하면 원본 페이지를 스플릿해서 2개의 페이지에 8KB씩 저장
2. 나뉜 페이지 각각에 대해 1 단계를 반복
- 테이블 압축 방식에서 가장 중요한 것은 원본 데이터 페이지의 압축 결과가 목표 크기보다 작거나 같을 때까지 반복해서 페이지를 스플릿 하는 것이다.
- 그래서 목표 크기 (key_block_size) 의 크기가 잘못 설정되면 서버의 처리 성능이 급격히 떨어질수 있다.

### 6.2.2 KEY_BLOCK_SIZE 결정
>- 테이블 압축에서 가장 중요한 부분은 압축된 결과가 어느 정도일지 예측하여 key_block_size 를 결정하는 것이다.
- 우선 4KB , 8KB 로 테이블을 생성해서 샘플 데이터를 저장해 보고 적절한지 테스트 해보는것이 좋다.
- 이때 샘플 데이터는 많으면 많을수록 정확한 테스트가 가능한데 최소 테이블의 데이터 페이지가 10개 정도 되는 테스트 데이터를 insert 해보는것이 좋다.
1. use employees;
(RealMySQL 책을 따라 하고 있고 예제파일을 실행했다면 employees_comp4k 테이블을 드롭했다가 실행해야 결과를 확인할수 있다.)
2. create table employees_comp4k(
emp_no int not null,
birth_date date not null,
first_name varchar(14) not null,
last_name varchar(16) not null,
gender enum('M','F') not null,
hire_date date not null,
primary key(emp_no),
key ix_firstname (first_name),
key ix_hiredate (hire_date)
)row_format=compressed key_block_size=4;
3. set global innodb_cmp_per_index_enabled=on;
4. insert into employees_comp4k select * from employees;
5. select table_name, index_name, compress_ops, compress_ops_ok,
(compress_ops-compress_ops_ok)/compress_ops * 100 as compression_failure_pct
from information_schema.innodb_cmp_per_index;
![](https://velog.velcdn.com/images/cmong0516/post/1e450cb5-9c7b-49c7-96e7-1c5086821140/image.png)
- 18363번 압축을 진행했고 13479번 성공했다.
- 18363 - 13479 = 5176 번 압축했는데 압축 결과가 4KB 를 초과해서 데이터 페이지를 스플릿해서 다시 압축을 실행했다는 의미이다.
- 압축 실패율이 27.6722 % 이며 나머지 두개의 인덱스도 압축 실패율이 상대적으로 높게 나와있는데 일반적으로 압축 실패율은 3~5% 미만으로 유지할수 있게 key_block_size 를 설정하는것이 좋다.

#### key_block_size = 8
>- key_block_size 를 8로 설정하고 동일하게 테스트를 진행해보자.
![](https://velog.velcdn.com/images/cmong0516/post/c37a1575-5c5d-4535-b21d-829438f9aedd/image.png)
- 8KB로 설정했는데도 primary 키의 압축 실패율이 높게 나타나는데 압축을 적용하면 압축 실패율이 높아서 InnoDB버퍼 풀에서 디스크로 기록되기 전에 압축하는 과정에서 깨 오랜 시간이 걸릴것이라고 예측할수 있다.
- 압축 실패율은 낮으면서 압축 효율이 상대적으로 높은 8KB 를 선택하는것이 4KB 를 선택하는 것보다 효율적이다.
- 압축 실패율이 높게 나온다고 해서 압축을 사용하지 마랑야 하는것은 아니다.

### 6.2.3 압축된 페이지의 버퍼 풀 적재 및 사용
>- InnoDB 스토리지 엔지는 압축된 테이브르이 데이터 페이지를 버퍼 풀에 적재하면 압축된 상태와 압축이 해제된 상태 2개의 버전을 관리하는데 디스크에서 읽은 상태 그대로의 데이터 페이지 목록을 관리하는LRU 리스트와 압축된 페이지들의 압축 해제 버전인 Unzip_LRU 리스트를 별도로 관리한다.
- InnoDB 스토리지 엔진은 압축된 테이블에 대해 버퍼 풀의 공간을 이중으로 사용하여 메모리를 낭비하는 효과를 가지게 되고 압축된 페이지에서 데이터를 읽거나 변경하기 위해서는 압축을 해제해야 하는데 이 작업은 CPU를 많이 소모하게 된다.
- 위 문제점을 해결하기 위해 Unzip_LRU 리스트를 별도로 관리하고 있다가 MySQL 서버로 유입되는 요청 패턴에 따라 적절히 다음과 같은 처리를 수행한다.
1. InnoDB버퍼 풀의 공간이 필요한 경우 LRU 리스트에서 원본 데이터 페이지는 유지하고 Unzip_LRU 리스트에서 압축 해제된 버전은 제거하여 버퍼 풀의 공간을 확보한다.
2. 압축된 데이터 페이지가 자주 사용되는 경우 Unzip_LRU 리스트에 압축 해제된 페이지를 계속 유지하면서 압축 및 압축 해제 작업을 최소화 한다.
3. 압축된 데이터 페이지가 사용되지 않아서 LRU 리스트에서 제거되는 경우 Unzip_LRU 리스트에서도 함께 제거된다.

## 6.2.4 테이블 압축 관련 설정
>- innodb_cmp_per_index_enabled = on : 테이블 압축이 사용된 테이블의 모든 인덱스별로 압축 성공 및 압축 실행 횟수를 수집하도록 설정하며 information_schema.INNODB_CMP_PER_INDEX 테이블에 기록된다.
- innodb_cmp_per_index_enabled = off : 비활성화 할경우 테이블 단위의 압축 성공 및 압축 실행 횟수만 수입되고 테이블 단위로 수집된 정보는 information_schema.INNODB_CMP 테이블에 기록된다.
- innodb_compression_level : InnoDB의 테이블 압축은 zlib 압축 알고리즘만 지원하는데 이때 innodb_compression_level 시스템 변수를 이용해 0~9 까지 압축률을 선택할수 있는데 압축률이 작을수록 속도는 빨라지지만 저장공간은 커지며 압축률 속도는 CPU 자원 소모량과 동일한 의미이다.
- innodb_compression_failure_threshold_pct , innodb_compression_pad_pct_max : 테이블 단위로 압축 실패율이 innodb_compression_failure_threshold_pct 값보다 커지면 압축을 실행하기 전 원본 데이터 페이지의 끝에 의도적으로 일정 크기의 빈 공간을 추가하여 그 공간의 압축률을 높여 압축 결과가 key_block_size 보다 작아지게 만드는 효과를 내며 이때 추가하는 빈 공간을 패딩 이라고 하고 추가할수 있는 패딩의 최대 크기가 innodb_compression_pad_pct_max 이다.
- innodb_log_compressed_pages : MySQL 서버가 비정상 종료 되었다가 다시 시작될 경우 압축 알고리즘의 버전 차이가 있더라도 복구 과정이 실패하지 않도록 InnoDB스토리지 엔진은 압축된 데이터 페이지를 그대로 리두 로그에 기록하는데 압축 알고리즘을 업데이트 할때 도움이 되지만 데이터 페이지를 통째로 리두 로그에 저장하는 것은 리두 로그의 증가량에 상당한 영향을 줄수도 있다.
압축을 적용한 리두 로그 용량이 매우 빠르게 증가한다거나 버퍼 풀로부터 더티 페이지가 한번에 많이 기록되는 팬턴으로 바뀌었다면 innodb_log_compression_pages 시스템 변수를 비활성화 하고 모니터링 해보는것이 좋다.
