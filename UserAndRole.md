
# 3.1 사용자 식별

>'mong'@'127.0.0.1' 

MySQL 의 사용자는 사용자의 접속지점 도 계정의 일부이다.
따라서 계정을 언급할 때는 아이디와 호스트를 함께 명시해야 한다.

>'mong'@'%'

MySQL 서버가 기동중인 로컬 호스트에서 mong 이라는 아이디로 접속할 때만 사용될수 있는 계정이다.만약 사용자 계정에 다음과 같이 등록되어 있다면 다른 컴퓨터에서는 접속이 불가능하다.
만약 모든 외부 컴퓨터에서 접속이 가능하게 하려면 호스트 부분을 % 로 대체한다.

> 'mong'@'127.0.0.1'  ,  'mong'@'%'

동일한 아이디가 있을경우 MySQL 서버가 해당 사용자의 인증을 위해 어떤 계정을 선택할까 ?

> MySQL 은 범위가 가장 작은것을 우선적으로 선택한다. 위의 두 아이디중 범위가 작은것은 127.0.0.1 로컬 이기 때문에 이 아이를 먼저 선택한다.

# 3.2 사용자 계정 관리

## 3.2.1 시스템 계정과 일반 계정

> SYSTEM_USER 권한을 가지고 있느냐 없느냐에 따라 시스템 계정과 일반계정 으로 구분한다.

### 시스템 계정
> - 계정관리
- 다른 세션 또는 그 세션에서 실행중인 쿼리를 강제 종료
- 스토어드 프로그램 생성시 DEFINER 를 타 사용자로 설정

### MySQL 내장 계정
> MySQL 서버에는 내장된 계정들이 있는데 내부적으로 각기 다른 목적으로 사용되므로 삭제하지 않도록 주의한다.
1. 'mysql.sys'@'localhost' : MySQL 8.0 부터 내장된 sys 스키마의 객체들의 DEFINER 로 사용되는 계정
2. 'mysql.session'@'localhost' : MySQL 플로그인이 서버로 접근할때 사용되는 계정
3. 'mysql.infoschema'@'localhost' : information_schema 에 정의된 뷰의 DEFINER 로 사용되는 계정
위 세 계정은 처음부터 잠겨있는 상태이므로 의도적으로 풀지 않는한 보안걱정은 하지 않아도 된다.

## 3.2.2 계정 생성
>5.7 버전까지는 grant 명령으로 권한의 부여와 동시에 계정 생성이 가능했지만 8.0 부터 계정 생성은 create user , 권한 부여는 grant 로 구분해서 실행하도록 변경되었다.
계정을 생성할때 다음 옵션들을 설정할수 있다.
- 계정의 인증방식과 비밀번호
- 비밀번호 관련 옵션
- 기본 역할
- SSL 옵션
- 계정 잠금 여부

> create user 'user'@'%'
	identified with 'mysql_native_password by 'password'
    require none
    password expire interval 30 day
    account unlock
    password history defualt
    password reuse interval defualt
    password require current defualt;
    
### 1. idntified with

>사용자의 인증 방식과 비밀번호를 설정한다.
identified with 뒤에는 반드시 인증방식을 명시해야 하는데 기본 인증방식을 사용하려면 'password'

- Native Pluggable Authentication : 비밀번호에 대한 해시 값을 저장해두고 클라이언트가 보낸 해시값과 일치하는지 판별

- Caching SHA-2 Pluggable Authentication : 암호화 해시값 생성을 위해 SHA-2 알고리즘을 사용한다. 해시값을 구하는데 많은 시간이 소모되어 해시 결과값을 메모리에 캐시해서 사용한다. 이 인증방식을 사용하려면 SSL/TLS/RSA키페어 를 반드시 사용해야 하는데 이를 위해 클라이언트에 접속할때 SSL 옵션을 활성화해야한다.

- PAM Pluggable Authentication : 유닉스나 리눅스 패스워드 또는 LDAP 같은 외부 인증을 사용할수 있게 해주는 인증방식으로 엔터프라이즈 에디션에서만 사용가능

- LDAP Pluggable Authentication : LDAP 를 이용한 외부인증을 사용할수 있게 해주는 인증방식으로 엔터프라이즈 에디션에서만 사용가능

> Native Authentication 이 기본 인증방식으로 사용됐었지만 Caching SHA-2 방식으로 변경되었다.
이방식은 SSL/TLS/RSA키페어 가 필요로 하기 때문에 Native Authentication 을 사용하고자 한다면 MySQL 설정을 변경하자

> set global default_authentication_plugin="mysql_native_password"


### 2. REQUIRE
>MySQL 서버에 접속할때 암호화된 SSL/TLS 채널을 사용할지 여부를 설정한다.
만약 설정하지 않을경우 비암호화 채널로 연결한다.

### 3. PASSWORD EXPIRE
> 비밀번호의 유효기간을 설정하는 옵션이며 별도로 설정하지 않을경우 default_password_lifetime 시스템 변수에 저장된 값으로 설정된다.
개발자나 데이터 베이스 관리자의 비밀번호는 유효기간을 설정하는것이 보안상 안전하지만 응용 프로그램 접속용 계정에 유효기간을 설정하는 것은 위험할수 있으니 주의하자.

- password expire : 계정 생성과 동시에 비밀번호의 만료 처리
- password expire naver : 계정 비밀번호의 만료기간 없음
- password expire default : default_password_lifetime시스템 변수에 저장된 기간으로 설정
- password expire interval n day : 비밀번호의 유효기간을 오늘부터 n 일자로 설정

### 4. PASSWORD HISTORY
> 한번 사용했던 비밀번호를 재사용하지 못하게 설정하는 옵션
password history default : 한번 사용한 비밀번호 재사용 불가
password history n : 비밀번호 이력을 최근 n 개 까지만 저장하여 n개만 재사용 불가

### 5. PASSWORD REUSE INTERVAL
> 한번 사용했던 비밀번호의 재사용 금지 기간을 설정하는 옵션으로 설정하지 않으면 password_reuse_interval 시스템 변수의 기간으로 설정된다.
password reuse interval default : password_reuse_interval 변수사용
password reuse interval n day : n일후 비밀번호 재사용 가능

### 6. PASSWORD REQUIRE
> 비밀번호가 만료되어 새로운 비밀번호로 변경할때 현재 비밀번호를 필요로 할지 결정하는 옵션으로 설정하지 않으면 password_require_current 시스템 변수값으로 설정된다.
password require current : 비밀번호를 변경할때 현재 비밀번호 입력
password require optional : 비밀번호를 변경할때 현재 비밀번호 입력하지 않음
password require default : 시스템 변수값으로 설정

### 7. ACCOUNT LOCK/UNLOCK
> 계정 생성시 ,정보 변경시 계정을 사용하지 못하게 잠글지 여부를 결정
ACCOUNT LOCK : 계정을 사용하지 못하게 잠금
ACCOUNT UNLOCK : 잠긴 계정을 다시 사용가능 상태로 만듬

# 비밀번호 관리

## 3.3.1 고수준 비밀번호
> MySQL 서버의 비밀번호는 유효기간이나 이력관리를 통한 관리뿐만 아니라 비밀번호를 쉽게 유추할수 있는 단어들이 사용되지 않게 글자의 조합을 강제하거나 금칙어를 설정할수 있다.
비밀번호 유효성 체크 규칙을 적용하려면 validate_password 컴포넌트를 이용하면 된다.

### validate_password 컴포넌트 설치
> install component 'file://conponent_validate_password';

### 설치된 컴포넌트 확인
> select * from mysql.component;

### 비밀번호 정책

> LOW : 비밀번호 길이만 검증
MIDIUM : 비밀번호의 길이를 검증하여 숫자,대소문자 , 특수문자 배합을 검증
STRONG : MIDIUM + 금칙어 포함여부 검증

#### 금칙어 파일 등록
> set global validate_password.dictionary_file='{fileName}';
set global validate_password.policy='STRONG';

### 3.3.2 이중 비밀번호
> 일반적으로 많은 응용프로그램 서버들이 공용으로 데이터베이스를 사용하기 때문에 서버의 계정 정보는 응용 프로그램 서버로부터 공용으로 사용되는 경우가 많다.
이런 구현 특징 때문에 데이터베이스 서버의 계정 정보는 쉽게 변경하기 어려웠는데 데이터베이스 계정의 비밀번호는 서비스가 실행중인 상태로 변경이 불가능했다.
이를 해결하기 위해 8버전 이후 계정의 비밀번호로 2개의 값을 동시에 사용하는 기능을 추가했다.
이중 비밀번호는 Primary , Secondary 로 구분된다.
이중 비밀번호를 사용하려면 기존 비밀번호 변경 구문에 retain current password 옵션을 추가하면 된다.

> alter user 'root@'localhost' identified by 'old_password';
alter user 'root@'localhost' identified by 'new_password' retain current password;

> 첫 명령어가 실행되면 root 계정의 Primary 는 old_password 가 되고 세컨더리는 비어있다.
두번째 명령어가 실행되면 old_password 는 Secondary 가 되고 new_password 가 Primary 가 되며 두개중 어느것을 사용해도 로그인 된다.

>alter user 'root'@'localhost' discard old password;
MySQL 서버에 접속하는 모든 응용 프로그램이 재시작 완료되면 위의 명령어로 Secondary 비밀번호는 보안을 위해 삭제하는것이 좋다.

# 권한
>권한은 글로벌 권한과 객체단위의 권한으로 구분된다.
객체권한 : grant 명령어로 권한을 부여할때 반드시 특정 객체를 명시해야 한다.
글로벌 : grant 명령어로 권한을 부여할때 특정 객체를 명시하지 말아야 한다.

## 글로벌 권한
>grant super on \*.\* to 'user'@'localhost';

> 글로벌 권한은 특정 DB나 테이블에 부여될수 없기 때문에 글로벌 권한을 부여할때 grant 명령어의 on 절에 항상 \*.\* 를 사용하게 된다.
\*.\* 는 모든 DB의 모브젝트를 포함해서 MySQL 서버 전체를 의미.


## DB 권한
>grant event on \*.\* to 'user'@'localhost';

> DB 권한은 특정 DB에 대해서만 권한을 보여하거나 서버에 존재하는 모든 DB 에 대해 권한을 부여할수 있기 때문에 위 예처럼 on 절에 \*.\* 이나 employees.\* 등을 사용할수 있다.
DB 권한은 테이블까지 명시할수는 없다.
DB 권한은 서버의 모든 DB에 적용할수 있기 때문에 \*.\* 를 사용할수 있다.

## 테이블 권한
>grant select,insert,update,delete on \*.\* to 'user'@'localhost';

>테이블 권한은 서버의 모든 DB에 대해 권한을 부여할수도 있고 특정 DB의 오브젝트 , 특정 DB의 특정 테이블에 대해서도 권한부여가 가능하다.
여러가지 레벨이나 범위로 권한을 설정하는것이 가능하지만 하나의 테이블에 대해 권한 설정을 하면 다른 모든 테이블에서 권한 체크를 하기때문에 성능에 문제가 생길수 있어서 잘 사용하지 않는다.

# 역할
>8버전부터 권한들을 묶어서 역할을 사용할수 있게 되었다.

## 예시

### 역할 생성
> create role role_emp_read,role_emp_write;

### 역할에 권한 부여
> grant select on employees.* to role_emp_read;
grant insert,update,delete on employees.* to role_emp_write;

### 계정 추가
>create user reader@'127.0.0.1' identified by '{password}';
create user writer@'127.0.0.1' identified by '{password}';
이때 password 레벨에(Row , Medium , Strong) 맞는 password 를 설정해주어야 한다.

### 계정에 권한 부여
>grant role_emp_read to reader@'127.0.0.1';
grant role_emp_read, role_emp_write to writer@'127.0.0.1';

### 생성한 계정으로 로그인
> mysql -u reader -p -h 127.0.0.1

### 로그인 계정 권한 확인
> show grants;

### select 해보기
> select * from employees.employees limit 10;


### 현재 역할 조회
> 무언가 이상하다 분명 역할을 만들고 역할에 권한을 부여하고 계정을 생성하여 역할을 주었는데 왜 조회가 안되는것일까 ?
select current_role();
명령어를 통해 현재 역할을 조회해보면 NONE 이 나온다.
reader 계정이 role_emp_read 역할을 수행하게 하려면 set role 명령으로 역할을 활성화 해주어야 한다. 역할을 활성화 하면 사용할수 있지만 로그아웃후 다시 로그인 하면 비활성 상태로 초기화된다.

### 역할 활성화
> set role 'role_emp_read';
명령어를 입력후 select current_role(); 명령어를 입력하면 권한이 제대로 표시된다.

### activate_all_roles_on_login
> 로그아웃 하면 활성화한 역할이 다시 비활성으로 초기화 된다고 했다.
이것은 MySQL 서버가 자동으로 활성화 되지 않게 설정되어 있기 때문이다.
이 설정은 activate_all_roles_on_login 설정을 ON 으로 바꾸어주면 로그인과 동시에 자동으로 활성화되게 변경된다.
다시 루트계정으로 접속하여 명령어 실행
set global activate_all_roles_on_login=on;

### 계정과 권한의 구분
>mysql DB 의 user 테이블에는 방금 생성한 사용자 계정과 권한이 들어있는데 account_locked 칼럼값이 다를뿐 구분하는 별도의 칼럼이 존재하지 않는다.
하나의 계정에 다른 계정의 권한을 포함하고 싶으면 계정을 병합하면 된다. 따라서 계정과 권한을 구분할 필요자체가 없다.
계정 생성시 이름과 호스트를 명시해서 생성했었는데 만약 명시하지 않는다면 자동으로 %(모든호스트) 가 추가되어 생성된다.
MySQL 내부에서 계정과 권한을 구분하지 않아 권한은 role_ 을 붙여서 권한임을 알수있게 한것이다.
